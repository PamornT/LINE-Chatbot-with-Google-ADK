# Building Sales Agent

## 🎯 Objective
สร้าง Sales Agent สำหรับธุรกิจรถยนต์ที่สามารถจัดการ sales inquiries, ให้ข้อมูลรุ่นรถ ฟีเจอร์ และช่วยลูกค้าเลือกและจองรถยนต์ได้อย่างมีประสิทธิภาพ


---

## 🏗️ Sales Agent Architecture

### Core Capabilities

Sales Agent ควรมี:
- **Vehicle Knowledge**: รู้จักรุ่นรถยนต์ เครื่องยนต์, สี, และ options
- **Customer Interaction**: ตอบคำถามเกี่ยวกับการซื้อ, ออกรถ, และผ่อนชำระ
- **Lead Qualification**: ประเมินความพร้อมด้านงบและความต้องการ
- **Sales Process**: ช่วยลูกค้าเลือกแพ็กเกจ, ทดลองขับ, และปิดการขาย
- **CRM Integration**: บันทึกข้อมูลลูกค้าและประวัติการติดต่อ


### Agent Structure

```javascript
class SalesAgent extends BasicAgent {
  constructor() {
    super('SalesAgent');
    this.productCatalog = new ProductCatalog();
    this.crm = new CRMConnector();
    this.salesPlaybook = new SalesPlaybook();
  }

  async respond(message, context = {}) {
    // Analyze customer intent
    const intent = await this.analyzeIntent(message);

    // Get customer data if available
    const customerData = await this.getCustomerData(context.customerId);

    // Generate sales response
    const response = await this.generateSalesResponse(message, intent, customerData);

    // Log interaction
    await this.logInteraction(context.customerId, message, response, intent);

    return response;
  }

  async analyzeIntent(message) {
    const prompt = `Analyze this customer message and determine their intent:

Message: "${message}"

Possible intents:
- product_inquiry: Asking about products/features
- pricing_question: Asking about prices/costs
- purchase_intent: Ready to buy or serious interest
- objection: Expressing concerns or doubts
- support_request: Needs help with existing purchase
- general_chat: Casual conversation

Return JSON: {"intent": "intent_type", "confidence": 0-1, "keyTopics": ["topic1", "topic2"]}`;

    const analysis = await this.think(prompt);
    return JSON.parse(analysis);
  }
}
```

---

## 📊 Product Knowledge Integration

### Product Catalog Tool

```javascript
class ProductCatalog {
  constructor() {
    // ตัวอย่างข้อมูลรถยนต์ (สามารถเชื่อมกับฐานข้อมูลจริงได้)
    this.products = {
      'ToyotaCamry': {
        name: 'Toyota Camry Hybrid 2025',
        price: 1_259_000,
        features: ['2.5L Hybrid', 'LED Headlights', 'Leather Seats', 'Toyota Safety Sense'],
        category: 'sedan'
      },
      'HondaCivicTypeR': {
        name: 'Honda Civic Type R 2024',
        price: 2_399_000,
        features: ['2.0L Turbo', 'Recaro Seats', 'Adaptive Damper System', 'Sport Exhaust'],
        category: 'hatchback'
      }
    };
  }

  searchProducts(query) {
    const results = Object.values(this.products).filter(product =>
      product.name.toLowerCase().includes(query.toLowerCase()) ||
      product.features.some(feature =>
        feature.toLowerCase().includes(query.toLowerCase())
      )
    );
    return results;
  }

  getProduct(productId) {
    return this.products[productId] || null;
  }

  getProductsByCategory(category) {
    return Object.values(this.products).filter(p => p.category === category);
  }
}
```

### Product Information Response

```javascript
class SalesAgent extends BasicAgent {
  async handleProductInquiry(productQuery, customerData) {
    // Search for products
    const products = this.productCatalog.searchProducts(productQuery);

    if (products.length === 0) {
      return "ขออภัยครับ ไม่พบสินค้าที่คุณกำลังมองหา คุณต้องการให้ช่วยแนะนำสินค้าอื่นหรือไม่?";
    }

    if (products.length === 1) {
      const product = products[0];
      return this.formatProductDetails(product, customerData);
    }

    // Multiple products - show options
    const options = products.map(p => `${p.name} - ฿${p.price.toLocaleString()}`).join('\n');
    return `พบสินค้าที่ตรงกับคำค้นหาของคุณ ${products.length} รายการ:\n\n${options}\n\nคุณสนใจสินค้ารายการไหนเป็นพิเศษครับ?`;
  }

  formatProductDetails(product, customerData) {
    const features = product.features.join(' • ');

    let response = `� รุ่น: ${product.name}\n`;
    response += `💰 ราคาจำหน่าย: ฿${product.price.toLocaleString()}\n`;
    response += `🔧 ฟีเจอร์เด่น: ${features}\n\n`;

    // Personalized recommendations based on customer data
    if (customerData && customerData.previousPurchases) {
      response += `💡 จากประวัติการติดต่อของคุณ เราคิดว่า ${product.name} จะตอบโจทย์เรื่อง ${customerData.preferences || 'ความสะดวกสบาย'}\n`;
    }

    response += `สนใจนัดทดลองขับหรือสอบถามรายละเอียดเพิ่มเติมครับ?`;

    return response;
  }
}
```

---

## 💰 Pricing & Negotiation Tools

### Dynamic Pricing Strategy

```javascript
class PricingEngine {
  calculatePrice(product, customerSegment, quantity = 1) {
    let basePrice = product.price;

    // Volume discounts
    if (quantity >= 10) basePrice *= 0.9; // 10% discount
    else if (quantity >= 5) basePrice *= 0.95; // 5% discount

    // Customer segment pricing
    switch (customerSegment) {
      case 'premium':
        basePrice *= 0.95; // 5% premium discount
        break;
      case 'enterprise':
        basePrice *= 0.85; // 15% enterprise discount
        break;
    }

    return {
      originalPrice: product.price * quantity,
      finalPrice: basePrice * quantity,
      discount: (product.price - basePrice) * quantity,
      discountPercent: ((product.price - basePrice) / product.price) * 100
    };
  }

  generatePricingResponse(product, pricing, customerData) {
    let response = `💰 ราคา ${product.name}:\n`;

    if (pricing.discount > 0) {
      response += `~~฿${pricing.originalPrice.toLocaleString()}~~ `;
      response += `฿${pricing.finalPrice.toLocaleString()}\n`;
      response += `🎉 ประหยัด ฿${pricing.discount.toLocaleString()} (${pricing.discountPercent.toFixed(1)}%)\n`;
    } else {
      response += `฿${pricing.finalPrice.toLocaleString()}\n`;
    }

    // Add payment options
    response += `\n💳 ช่องทางการชำระเงิน:\n`;
    response += `• บัตรเครดิต/เดบิต\n`;
    response += `• QR Payment\n`;
    response += `• Bank Transfer\n`;

    return response;
  }
}
```

---

## 🎯 Lead Qualification System

### Qualification Criteria

```javascript
class LeadQualifier {
  async qualifyLead(message, customerData, interactionHistory) {
    const scores = {
      budget: 0,
      authority: 0,
      need: 0,
      timeline: 0,
      total: 0
    };

    // Analyze budget signals (e.g., "งบ", "ผ่อน", "ดาวน์")
    if (message.toLowerCase().includes('ราคา') ||
        message.toLowerCase().includes('งบ') ||
        message.toLowerCase().includes('ผ่อน') ||
        message.toLowerCase().includes('ดาวน์')) {
      scores.budget = 25;
    }

    // Check authority (decision maker)
    if (customerData && customerData.jobTitle) {
      const title = customerData.jobTitle.toLowerCase();
      if (title.includes('manager') || title.includes('director') ||
          title.includes('owner') || title.includes('ceo')) {
        scores.authority = 25;
      }
    }

    // Assess need/urgency
    if (message.toLowerCase().includes('urgent') ||
        message.toLowerCase().includes('ด่วน') ||
        interactionHistory.length > 3) {
      scores.need = 25;
    }

    // Timeline assessment
    if (message.toLowerCase().includes('สัปดาห์นี้') ||
        message.toLowerCase().includes('เดือนนี้') ||
        message.toLowerCase().includes('now')) {
      scores.timeline = 25;
    }

    scores.total = scores.budget + scores.authority + scores.need + scores.timeline;

    return {
      score: scores.total,
      grade: scores.total >= 75 ? 'A' : scores.total >= 50 ? 'B' : 'C',
      criteria: scores,
      qualified: scores.total >= 50
    };
  }
}
```

---

## 💬 Objection Handling

### Common Objections & Responses

```javascript
class ObjectionHandler {
  constructor() {
    this.objectionResponses = {
      'expensive': {
        acknowledge: 'ฉันเข้าใจครับว่าคุณกังวลเรื่องราคา',
        respond: 'แต่เมื่อพิจารณาถึงคุณภาพและความคุ้มค่าที่จะได้รับ',
        solution: 'เรามีโปรโมชั่นและผ่อนชำระได้สบายๆ'
      },
      'competitor': {
        acknowledge: 'ลูกค้าหลายท่านก็มีข้อกังวลคล้ายกัน',
        respond: 'แต่มันมีจุดเด่นที่แตกต่างอย่างชัด',
        solution: 'อนุญาตให้ผมอธิบายความแตกต่างให้ฟังครับ'
      },
      'timing': {
        acknowledge: 'ฉันเข้าใจว่าการตัดสินใจซื้อเป็นเรื่องสำคัญ',
        respond: 'แต่โอกาสที่ดีมักไม่รอใคร',
        solution: 'เราสามารถเริ่มต้นด้วย pilot project เล็กๆ ก่อนได้'
      }
    };
  }

  async handleObjection(objectionType, context) {
    const response = this.objectionResponses[objectionType];

    if (!response) {
      // Use AI to generate response for unknown objections
      const prompt = `Customer raised objection: "${context.message}"

Generate a helpful response that:
1. Acknowledges their concern
2. Provides counter-point
3. Offers solution or next step`;

      return await this.think(prompt);
    }

    return `${response.acknowledge} ${response.respond} ${response.solution}`;
  }
}
```

---

## 🔄 Sales Funnel Management

### Funnel Stages

```javascript
class SalesFunnelManager {
  constructor() {
    this.stages = {
      'awareness': { name: 'Awareness', actions: ['educate', 'inform'] },
      'interest': { name: 'Interest', actions: ['demonstrate', 'engage'] },
      'consideration': { name: 'Consideration', actions: ['compare', 'evaluate'] },
      'purchase': { name: 'Purchase', actions: ['close', 'convert'] },
      'retention': { name: 'Retention', actions: ['support', 'upsell'] }
    };
  }

  determineStage(customerData, interactionHistory) {
    const interactions = interactionHistory.length;
    const hasPurchased = customerData.purchases > 0;

    if (hasPurchased) return 'retention';
    if (interactions > 10) return 'purchase';
    if (interactions > 5) return 'consideration';
    if (interactions > 2) return 'interest';
    return 'awareness';
  }

  getNextAction(stage, customerIntent) {
    const stageActions = this.stages[stage].actions;

    // Match intent with appropriate action
    if (customerIntent === 'product_inquiry' && stage === 'awareness') {
      return 'provide_product_info';
    }

    if (customerIntent === 'pricing_question' && stage === 'consideration') {
      return 'offer_discount';
    }

    // Default action for stage
    return stageActions[0];
  }
}
```

---

## 🎮 Hands-on Exercise

### Exercise 1: Automotive Demo Agent

**Task:** สร้าง agent ที่สามารถอธิบายรถยนต์แต่ละรุ่นพร้อม demo ข้อมูล

```javascript
// Implement demo flow for cars
// Respond to questions about engine, fuel economy, and options
// Offer test drive scheduling
```

### Exercise 2: Financing & Negotiation Agent

**Task:** สร้าง agent ที่ช่วยคำนวณผ่อนชำระและเจรจาต่อรองเงื่อนไข

```javascript
// Implement financing calculator
// Handle downpayment and interest rate questions
// Provide negotiation responses and trade-in offers
```

### Exercise 3: Lead Nurturing Agent for Car Buyers

**Task:** สร้าง agent ที่ nurture ลูกค้าที่สนใจซื้อรถผ่าน email หรือ chat sequences

```javascript
// Create nurturing campaigns for different segments (first-time buyer, family, etc.)
// Personalize messages based on preferences (e.g., fuel efficiency, power)
// Track engagement and schedule follow-ups
```

---

## 📊 Performance Metrics

### Sales Agent KPIs

```javascript
class SalesMetricsTracker {
  constructor() {
    this.metrics = {
      conversations: 0,
      qualifiedLeads: 0,
      conversions: 0,
      avgResponseTime: 0,
      customerSatisfaction: 0
    };
  }

  trackInteraction(interaction) {
    this.metrics.conversations++;

    if (interaction.intent === 'purchase_intent') {
      this.metrics.qualifiedLeads++;
    }

    if (interaction.conversion) {
      this.metrics.conversions++;
    }
  }

  getConversionRate() {
    return this.metrics.conversions / this.metrics.conversations;
  }

  getLeadQualityRate() {
    return this.metrics.qualifiedLeads / this.metrics.conversations;
  }
}
```

---

## 🚀 Best Practices

### Sales Agent Guidelines
1. **Listen First**: วิเคราะห์ความต้องการก่อนเสนอขาย
2. **Qualify Early**: แยก leads ที่มีโอกาสสูง
3. **Personalize**: ปรับ response ตาม customer profile
4. **Follow Up**: ติดตามและ nurture relationships
5. **Know When to Escalate**: ส่งต่อ sales team เมื่อเหมาะสม

### Technical Best Practices
1. **CRM Integration**: Sync กับ CRM systems
2. **Analytics**: Track และ analyze performance
3. **A/B Testing**: Test different approaches
4. **Continuous Learning**: Update จาก successful interactions

---

## 🎯 Key Takeaways

1. **Sales Agents** ต้องการ domain knowledge และ sales skills
2. **Lead Qualification** สำคัญในการ prioritize efforts
3. **Objection Handling** เป็น core sales skill
4. **Personalization** เพิ่ม conversion rates
5. **Metrics Tracking** ช่วย optimize performance

---

## 🚀 Next Steps

ใน lesson ถัดไป เราจะเพิ่ม **State Management** ให้ sales agent จดจำ context ระหว่าง conversations

**พร้อมทำให้ sales agent ของคุณฉลาดขึ้นแล้วหรือยัง?** 💼🤖

---

*Code Examples: [exercises/day1/agents/sales-agent/](exercises/day1/agents/sales-agent/)*