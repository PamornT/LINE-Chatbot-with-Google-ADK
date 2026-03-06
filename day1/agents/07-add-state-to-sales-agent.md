# Adding State to Sales Agent

## 🎯 Objective
เพิ่ม state management ให้ Sales Agent จดจำ conversation history, customer preferences และ interaction context

---

## 💾 What is Agent State?

**State** คือข้อมูลที่ agent เก็บไว้เพื่อ:
- **Remember**: Customer preferences, previous purchases, interactions
- **Context**: Current conversation flow, decision stage
- **History**: Interaction timeline, purchase attempts
- **Personalization**: Tailor responses based on history

### State Types

```
Short-term (Session State)
├── Current conversation context
├── Current customer message
└── Active transaction

Long-term (Persistent State)
├── Customer profile
├── Purchase history
├── Preferences
└── Credit score
```

---

## 🏗️ State Management Architecture

### Basic State Structure

```javascript
class StatefulSalesAgent extends BasicAgent {
  constructor() {
    super('StatefulSalesAgent');
    this.sessionState = new Map(); // Session state (in-memory)
    this.customerProfiles = new Database(); // Persistent customer data
  }

  async respond(message, context = {}) {
    const customerId = context.customerId;

    // 1. Load current session state
    const session = this.getOrCreateSession(customerId);

    // 2. Load customer profile
    const profile = await this.customerProfiles.getProfile(customerId);

    // 3. Track message in state
    session.addMessage({
      role: 'user',
      content: message,
      timestamp: new Date()
    });

    // 4. Generate response with state context
    const response = await this.generateResponse(message, session, profile);

    // 5. Track response in state
    session.addMessage({
      role: 'agent',
      content: response,
      timestamp: new Date()
    });

    // 6. Update state
    this.updateState(customerId, session, profile);

    return response;
  }

  getOrCreateSession(customerId) {
    if (!this.sessionState.has(customerId)) {
      this.sessionState.set(customerId, {
        customerId,
        messages: [],
        currentVehicleOfInterest: null,
        conversationStage: 'initial',
        createdAt: new Date(),
        updatedAt: new Date()
      });
    }

    return this.sessionState.get(customerId);
  }
}
```

---

## 🧠 Conversation State Management

### State-Aware Response Generation

```javascript
class ConversationStateManager {
  constructor() {
    this.stages = {
      'initial': {
        name: 'Initial Contact',
        objectives: ['greet', 'understand_needs'],
        nextStages: ['exploration']
      },
      'exploration': {
        name: 'Vehicle Exploration',
        objectives: ['show_options', 'gather_preferences'],
        nextStages: ['consideration', 'initial']
      },
      'consideration': {
        name: 'Serious Consideration',
        objectives: ['detailed_info', 'pricing', 'test_drive_offer'],
        nextStages: ['purchase_intent', 'exploration']
      },
      'purchase_intent': {
        name: 'Purchase Intent',
        objectives: ['confirm_details', 'financing', 'schedule_visit'],
        nextStages: ['closing', 'financing_discussion']
      },
      'financing_discussion': {
        name: 'Financing Discussion',
        objectives: ['present_options', 'negotiate', 'approval'],
        nextStages: ['closing', 'consideration']
      },
      'closing': {
        name: 'Closing',
        objectives: ['finalize_deal', 'paperwork', 'delivery_schedule'],
        nextStages: ['post_sale']
      },
      'post_sale': {
        name: 'Post-Sale Support',
        objectives: ['warranty_info', 'service_reminder', 'referral'],
        nextStages: []
      }
    };
  }

  determineNextStage(currentStage, userIntent, conversationContext) {
    const currentStageDef = this.stages[currentStage];

    if (!currentStageDef) return 'initial';

    // Check if objective completed
    if (conversationContext.objectiveCompleted) {
      const nextOptions = currentStageDef.nextStages;
      return this.selectNextStage(userIntent, nextOptions);
    }

    return currentStage; // Stay in current stage
  }

  selectNextStage(intent, options) {
    // Logic to select next stage based on intent
    if (intent === 'purchase_intent' && options.includes('purchase_intent')) {
      return 'purchase_intent';
    }

    if (intent === 'financing' && options.includes('financing_discussion')) {
      return 'financing_discussion';
    }

    return options[0] || 'initial';
  }

  getStageContext(stage) {
    return this.stages[stage];
  }
}
```

### State-Driven Responses

```javascript
class StatefulSalesAgent extends BasicAgent {
  async generateResponse(message, session, profile) {
    const stateManager = new ConversationStateManager();
    const currentStage = session.conversationStage;
    const stageContext = stateManager.getStageContext(currentStage);

    // Analyze user intent
    const intent = await this.analyzeIntent(message);

    // Update conversation stage
    session.conversationStage = stateManager.determineNextStage(
      currentStage,
      intent.type,
      { objectiveCompleted: intent.objectiveMatched }
    );

    // Generate stage-appropriate response
    const prompt = `You are a car sales agent in the "${stageContext.name}" stage of the sales process.
Your objectives for this stage: ${stageContext.objectives.join(', ')}

Customer Message: "${message}"
Customer Profile: ${JSON.stringify(profile)}
Conversation History: ${session.messages.slice(-5).map(m => `${m.role}: ${m.content}`).join('\n')}
Vehicle of Interest: ${session.currentVehicleOfInterest || 'None yet'}

Generate an appropriate response that:
1. Moves toward the stage objectives
2. References previous conversation context
3. Maintains conversation flow
4. Personalizes based on customer profile`;

    return await this.think(prompt);
  }

  async analyzeIntent(message) {
    const prompt = `Analyze this customer message for sales stage progression:

Message: "${message}"

Determine:
1. Primary intent (explore_vehicles, pricing_inquiry, test_drive, financing, purchase_ready, support)
2. Whether an objective from current stage is being addressed
3. Key topics or vehicle interests mentioned

Return JSON: {
  "type": "intent_type",
  "objectiveMatched": true/false,
  "topics": ["topic1", "topic2"]
}`;

    const analysis = await this.think(prompt);
    return JSON.parse(analysis);
  }
}
```

---

## 💾 Persistent State Storage

### Customer Profile Database

```javascript
class CustomerProfileStore {
  constructor(connectionString) {
    this.db = new Database(connectionString);
  }

  async getProfile(customerId) {
    const profile = await this.db.query(
      `SELECT * FROM customer_profiles WHERE id = $1`,
      [customerId]
    );

    return profile ? this.hydrate(profile[0]) : this.createNewProfile(customerId);
  }

  createNewProfile(customerId) {
    return {
      customerId,
      name: null,
      email: null,
      phone: null,
      monthlyIncome: null,
      creditScore: null,
      previousPurchases: [],
      preferences: {
        vehicleType: null,
        priceRange: null,
        fuelType: null,
        features: []
      },
      interactionHistory: [],
      createdAt: new Date(),
      updatedAt: new Date()
    };
  }

  async updateProfile(profile) {
    profile.updatedAt = new Date();

    await this.db.query(
      `INSERT INTO customer_profiles 
       (id, name, email, phone, monthly_income, credit_score, preferences, interaction_history)
       VALUES ($1, $2, $3, $4, $5, $6, $7, $8)
       ON CONFLICT (id) DO UPDATE SET
       name = $2, email = $3, phone = $4, monthly_income = $5,
       credit_score = $6, preferences = $7, interaction_history = $8,
       updated_at = NOW()`,
      [
        profile.customerId,
        profile.name,
        profile.email,
        profile.phone,
        profile.monthlyIncome,
        profile.creditScore,
        JSON.stringify(profile.preferences),
        JSON.stringify(profile.interactionHistory)
      ]
    );
  }

  async trackInteraction(customerId, message, response, stage) {
    const profile = await this.getProfile(customerId);

    profile.interactionHistory.push({
      timestamp: new Date(),
      stage,
      message,
      response,
      success: response.includes('✓') || response.includes('สำเร็จ')
    });

    await this.updateProfile(profile);
  }

  hydrate(dbRow) {
    return {
      customerId: dbRow.id,
      name: dbRow.name,
      email: dbRow.email,
      phone: dbRow.phone,
      monthlyIncome: dbRow.monthly_income,
      creditScore: dbRow.credit_score,
      preferences: JSON.parse(dbRow.preferences || '{}'),
      interactionHistory: JSON.parse(dbRow.interaction_history || '[]'),
      createdAt: dbRow.created_at,
      updatedAt: dbRow.updated_at
    };
  }
}
```

---

## 🎯 Vehicle Interest Tracking

### Tracking Customer Preferences

```javascript
class VehicleInterestTracker {
  constructor() {
    this.interestMap = new Map();
  }

  trackVehicleInterest(customerId, vehicleId, interestLevel) {
    if (!this.interestMap.has(customerId)) {
      this.interestMap.set(customerId, {});
    }

    const interests = this.interestMap.get(customerId);

    interests[vehicleId] = {
      vehicleId,
      interestLevel, // 'low', 'medium', 'high'
      firstViewedAt: interests[vehicleId]?.firstViewedAt || new Date(),
      lastViewedAt: new Date(),
      viewCount: (interests[vehicleId]?.viewCount || 0) + 1,
      comparison: interests[vehicleId]?.comparison || []
    };
  }

  getTopInterest(customerId) {
    const interests = this.interestMap.get(customerId) || {};
    return Object.values(interests)
      .sort((a, b) => (b.viewCount + (b.interestLevel === 'high' ? 10 : 0)) -
                       (a.viewCount + (a.interestLevel === 'high' ? 10 : 0)))
      [0];
  }

  recommendNextVehicle(customerId, currentVehicleId) {
    const interests = this.interestMap.get(customerId) || {};
    const currentVehicle = interests[currentVehicleId];

    if (!currentVehicle) return null;

    // Find similar vehicles with high interest
    return Object.values(interests)
      .filter(v => v.vehicleId !== currentVehicleId && v.interestLevel !== 'low')
      .sort((a, b) => b.viewCount - a.viewCount)[0];
  }
}
```

---

## 🔄 Session Timeout & Cleanup

### Session Management

```javascript
class SessionManager {
  constructor(sessionTimeout = 30 * 60 * 1000) { // 30 minutes default
    this.sessions = new Map();
    this.sessionTimeout = sessionTimeout;

    // Cleanup old sessions periodically
    setInterval(() => this.cleanupExpiredSessions(), 60 * 1000);
  }

  createSession(customerId) {
    const session = {
      customerId,
      messages: [],
      conversationStage: 'initial',
      currentVehicleOfInterest: null,
      createdAt: new Date(),
      lastActivityAt: new Date(),
      isActive: true
    };

    this.sessions.set(customerId, session);
    return session;
  }

  getSession(customerId) {
    const session = this.sessions.get(customerId);

    if (session) {
      session.lastActivityAt = new Date();
    }

    return session;
  }

  cleanupExpiredSessions() {
    const now = Date.now();

    for (const [customerId, session] of this.sessions) {
      const inactiveTime = now - session.lastActivityAt.getTime();

      if (inactiveTime > this.sessionTimeout) {
        session.isActive = false;
        this.sessions.delete(customerId);

        console.log(`🗑️ Session expired for customer ${customerId}`);
      }
    }
  }

  endSession(customerId) {
    const session = this.sessions.get(customerId);

    if (session) {
      session.isActive = false;
      this.sessions.delete(customerId);
    }
  }

  getActiveSessionCount() {
    return this.sessions.size;
  }
}
```

---

## 🎮 Hands-on Exercises

### Exercise 1: Stateful Sales Progression

**Task:** สร้าง agent ที่จำขั้นตอนการขายและปรับ response

```javascript
// Track conversation stage
// Move through: awareness → interest → consideration → purchase
// Personalize next steps based on stage
```

### Exercise 2: Customer Preference Learning

**Task:** Track customer preferences และ make recommendations

```javascript
// Remember vehicle interests
// Track viewed vehicles
// Recommend similar options
// Remember previous interactions
```

### Exercise 3: State Persistence

**Task:** Save customer state ใน database

```javascript
// Save conversation state
// Load previous context
// Resume interrupted conversations
// Track interaction history
```

---

## 📊 State Analytics

### Conversation Flow Analysis

```javascript
class ConversationAnalytics {
  analyzeConversationFlow(sessions) {
    const stageTransitions = {};

    sessions.forEach(session => {
      const messages = session.messages || [];
      let previousStage = 'initial';

      messages.forEach(msg => {
        if (msg.stage) {
          const key = `${previousStage} → ${msg.stage}`;
          stageTransitions[key] = (stageTransitions[key] || 0) + 1;
          previousStage = msg.stage;
        }
      });
    });

    return stageTransitions;
  }

  getAverageConversationLength(sessions) {
    if (sessions.length === 0) return 0;
    const totalMessages = sessions.reduce((sum, s) => sum + (s.messages?.length || 0), 0);
    return totalMessages / sessions.length;
  }

  getConversionRate(sessions) {
    const converted = sessions.filter(s => s.conversationStage === 'post_sale').length;
    return (converted / sessions.length) * 100;
  }
}
```

---

## 🚀 Best Practices

### State Management Guidelines
1. **Minimal State**: เก็บเฉพาะข้อมูลที่จำเป็น
2. **Regular Cleanup**: ลบ old sessions periodically
3. **Secure Storage**: Encrypt sensitive data
4. **Backup**: Backup customer profiles regularly
5. **Privacy**: Follow data protection regulations

### Performance Optimization
1. **Lazy Load**: Load profile data on demand
2. **Cache Frequently Used**: Cache top customers
3. **Index Queries**: Optimize database queries
4. **Batch Updates**: Combine updates
5. **Monitor Size**: Keep session sizes small

---

## 🎯 Key Takeaways

1. **State is Essential**: Makes agents intelligent and personalized
2. **Multi-tier Storage**: Session state + persistent storage
3. **Stage Progression**: Guide customers through sales funnel
4. **Personalization**: Use state for relevant recommendations
5. **Cleanup is Important**: Manage memory and database efficiently

---

## 🚀 Next Steps

ใน lesson ถัดไป เราจะเรียนรู้ **Loop Workflows** สำหรับ iterative processes

**พร้อมทำให้ agent ของคุณมี long-term memory แล้วหรือยัง?** 🧠💾🤖

---

*Code Examples: [exercises/day1/agents/stateful-sales/](exercises/day1/agents/stateful-sales/)*
