# Agent Thinking: Understanding AI Agent Reasoning

## 🎯 Objective
เข้าใจหลักการคิดและตัดสินใจของ AI agents รวมถึง reasoning patterns ที่ใช้ใน Google ADK

---

## 🤔 What is Agent Thinking?

**Agent Thinking** คือกระบวนการที่ AI agent ใช้ในการ:
- **รับรู้** inputs และ context
- **วิเคราะห์** ข้อมูลและสถานการณ์
- **ตัดสินใจ** ว่าจะดำเนินการอย่างไร
- **เรียนรู้** จาก feedback และปรับปรุง

### Thinking vs Traditional Programming

| Traditional Code | AI Agent Thinking |
|------------------|-------------------|
| Rule-based logic | Contextual reasoning |
| Fixed algorithms | Adaptive behavior |
| Predictable output | Dynamic responses |
| Manual updates | Self-learning |

---

## 🧠 Core Thinking Patterns

### 1. **Perception → Reasoning → Action (PRA) Cycle**

```
User Input / Environment Data
        ↓
   Perception Layer
   (Understand context)
        ↓
   Reasoning Layer
   (Analyze & decide)
        ↓
   Action Layer
   (Execute tasks)
        ↓
   Feedback Loop
   (Learn & improve)
```

### 2. **Context-Aware Reasoning**

Agent จะพิจารณา:
- **Current Context**: สถานการณ์ปัจจุบัน
- **Historical Data**: ประวัติการสนทนา
- **User Profile**: ข้อมูลผู้ใช้
- **Business Rules**: นโยบายองค์กร
- **External Data**: ข้อมูลจาก APIs

---

## 💭 Reasoning Techniques in Google ADK

### 1. **Chain-of-Thought (CoT) Reasoning**

Agent จะ:
1. **Break down** complex problems เป็น steps ย่อย
2. **Explain** logic ในแต่ละ step
3. **Build up** solution อย่างเป็นระบบ

**Example:**
```
User: "ช่วยหาเส้นทางจากบ้านไปสนามบิน"

Agent Thinking:
1. ต้องการข้อมูล: ตำแหน่งปัจจุบัน, สนามบินปลายทาง
2. ตรวจสอบโหมดการเดินทาง: รถยนต์, รถไฟ, แท็กซี่
3. พิจารณาเวลาและค่าใช้จ่าย
4. ให้ตัวเลือกพร้อมเหตุผล
```

### 2. **Tool-Augmented Reasoning**

Agent ใช้ tools เพื่อ:
- **Gather Information**: เรียก APIs, search data
- **Perform Calculations**: คำนวณราคา, ระยะทาง
- **Execute Actions**: ส่ง email, update database

### 3. **Memory-Enhanced Reasoning**

Agent ใช้ memory เพื่อ:
- **Remember Context**: ข้อมูลจาก conversation ก่อนหน้า
- **Learn Patterns**: ปรับปรุงจากประสบการณ์
- **Personalize Responses**: ปรับตามผู้ใช้

---

## 🏗️ Agent Architecture in ADK

### Basic Agent Structure

```javascript
const agent = {
  // Core Components
  name: "SalesAgent",
  role: "Handle sales inquiries",

  // Thinking Components
  perception: {
    inputParser: "understand user messages",
    contextAnalyzer: "extract relevant information"
  },

  reasoning: {
    decisionEngine: "gemini-pro model",
    toolSelector: "choose appropriate tools",
    logicValidator: "ensure sound reasoning"
  },

  action: {
    toolExecutor: "run selected tools",
    responseGenerator: "create user-friendly output"
  },

  // Learning Components
  memory: {
    shortTerm: "conversation context",
    longTerm: "learned patterns"
  },

  feedback: {
    successEvaluator: "measure task completion",
    improvementTracker: "identify areas for growth"
  }
};
```

---

## 🎯 Practical Examples

### Example 1: Sales Agent Thinking

**Scenario:** User asks "ราคา iPhone 15 Pro"

**Agent Thinking Process:**
```
1. PERCEPTION: User asking about product pricing
2. CONTEXT: This is a sales inquiry, need pricing info
3. REASONING:
   - Check current pricing from database
   - Consider promotions or discounts
   - Calculate total with tax/shipping
4. ACTION: Provide pricing with options
5. LEARNING: Remember this interaction for future
```

**Code Implementation:**
```javascript
const salesAgent = new Agent({
  name: 'SalesAgent',
  tools: [
    {
      name: 'getProductPrice',
      description: 'ดึงราคาสินค้าจาก database',
      execute: async (productName) => {
        // Call pricing API
        return await pricingAPI.getPrice(productName);
      }
    },
    {
      name: 'calculateTotal',
      description: 'คำนวณราคารวม',
      execute: async (price, tax, shipping) => {
        return price + tax + shipping;
      }
    }
  ]
});
```

### Example 2: Problem-Solving Agent

**Scenario:** User reports "สินค้าไม่ตรงตามที่สั่ง"

**Agent Thinking:**
```
1. UNDERSTAND: Customer complaint about wrong item
2. INVESTIGATE: Check order details, inventory
3. DECIDE: Offer replacement, refund, or compensation
4. ACT: Process resolution, update records
5. FOLLOW-UP: Ensure customer satisfaction
```

---

## 🔄 Decision Making Patterns

### 1. **Rule-Based Decisions**
```javascript
if (orderTotal > 1000) {
  applyDiscount(10);
} else if (customerTier === 'premium') {
  applyDiscount(5);
}
```

### 2. **AI-Powered Decisions**
```javascript
const decision = await agent.reason(
  "Should we offer discount to this customer?",
  {
    customerHistory: customerData,
    currentOrder: orderDetails,
    businessRules: companyPolicy
  }
);
```

### 3. **Hybrid Approach**
รวม rule-based กับ AI reasoning สำหรับ optimal results

---

## 🧪 Testing Agent Thinking

### Unit Testing Thinking Components

```javascript
// Test perception
test('should understand pricing questions', () => {
  const input = "ราคา iPhone เท่าไหร่";
  const perception = agent.perceive(input);
  expect(perception.intent).toBe('pricing_inquiry');
});

// Test reasoning
test('should choose correct tool', () => {
  const context = { intent: 'pricing_inquiry' };
  const decision = agent.reason(context);
  expect(decision.tool).toBe('getProductPrice');
});

// Test action
test('should execute tool correctly', async () => {
  const result = await agent.execute('getProductPrice', 'iPhone');
  expect(result).toHaveProperty('price');
});
```

---

## 🚀 Best Practices

### 1. **Clear Thinking Steps**
- แบ่ง complex decisions เป็น steps ย่อย
- Document reasoning logic
- Test each thinking component

### 2. **Context Management**
- เก็บ relevant context เท่านั้น
- Clear old context เมื่อไม่จำเป็น
- Use memory efficiently

### 3. **Error Handling**
- Handle unclear inputs gracefully
- Provide fallback responses
- Learn from failed interactions

### 4. **Performance Optimization**
- Cache frequent decisions
- Use async processing สำหรับ long tasks
- Monitor thinking time

---

## 🎮 Hands-on Exercise

### Exercise 1: Analyze Agent Thinking

**Task:** วเคราะห์ thinking process ของ agent ใน scenarios ต่างๆ

**Steps:**
1. อ่าน scenario descriptions
2. Map out PERCEPTION → REASONING → ACTION cycle
3. Identify required tools และ data
4. Design response strategy

**Scenarios:**
- Customer asking for product recommendations
- User requesting order status update
- Client needing technical support

### Exercise 2: Implement Simple Reasoning

**Task:** สร้าง agent ที่ใช้ basic reasoning patterns

**Code Template:**
```javascript
const { Agent } = require('@google-cloud/adk');

const reasoningAgent = new Agent({
  name: 'ReasoningAgent',
  apiKey: process.env.GEMINI_API_KEY,

  // เพิ่ม reasoning logic ที่นี่
  async reason(input) {
    // 1. Parse input
    // 2. Analyze context
    // 3. Make decision
    // 4. Return action plan
  }
});

// Test the agent
reasoningAgent.reason("ช่วยหาเส้นทางไปสนามบิน");
```

---

## 📚 Key Takeaways

1. **Agent Thinking** คือ core ของ intelligent behavior
2. **PRA Cycle** (Perception-Reasoning-Action) เป็น foundation
3. **Context Awareness** ทำให้ responses แม่นยำขึ้น
4. **Tool Integration** ขยาย capabilities ของ agents
5. **Testing** thinking components สำคัญสำหรับ reliability

---

## 🚀 Next Steps

ใน lesson ถัดไป เราจะ **สร้าง agent แรก** ด้วย Google ADK และทดสอบ thinking patterns ใน practice

**พร้อมสร้าง agent ที่คิดเป็นแล้วหรือยัง?** 🤖🧠

---

*References:*
- [Google ADK Reasoning Guide](https://cloud.google.com/adk/docs/reasoning)
- [Chain-of-Thought Prompting](https://arxiv.org/abs/2201.11903)