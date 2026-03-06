# Adding Tools to Your Agent

## 🎯 Objective
เรียนรู้การเพิ่ม tools และ capabilities ให้ agent ใน Google ADK เพื่อเพิ่มฟังก์ชันการต่างๆ

---

## 🛠️ What are Agent Tools?

**Tools** คือ Python functions ที่ agent สามารถเรียกใช้เพื่อ:
- **ดึงข้อมูล** จาก external sources
- **ดำเนินการ** operations ต่างๆ
- **คำนวณ** หรือ process ข้อมูล
- **ติดต่อ** กับ APIs หรือ databases

### Tools vs Plain AI Knowledge

| Plain AI | With Tools |
|----------|-----------|
| General knowledge | Specific to business |
| Can't update in real-time | Real-time data access |
| Generic responses | Personalized responses |
| No external actions | Integration with systems |

---

## 🔧 Tool Structure in Google ADK

### Requirements for Tools

ใน Google ADK Python tools ต้อง:
1. เป็น Python function ทั่วไป
2. มี **type hints** สำหรับ parameters และ return
3. มี **docstring** อธิบาย purpose และ parameters
4. Return ค่าที่ JSON-serializable (dict, str, int, bool, list)

### Simple Tool Example

```python
# tools.py - เก็บ tool functions ไว้
from typing import Dict, Optional

def get_vehicle_specs(model_name: str) -> Dict[str, any]:
    """Get specifications for a vehicle model.
    
    Args:
        model_name: Name of the vehicle (e.g., 'Toyota Camry')
    
    Returns:
        Dictionary with vehicle specifications
    """
    specs_db = {
        'Toyota Camry Hybrid': {
            'engine': '2.5L Hybrid',
            'transmission': 'Automatic CVT',
            'mpg': '54 city / 50 highway',
            'seating': 5,
            'cargo': '15.1 cu ft'
        },
        'Honda Civic Type R': {
            'engine': '2.0L Turbocharged',
            'transmission': '6-speed Manual',
            'hp': 306,
            'seating': 5,
            'cargo': '12.3 cu ft'
        }
    }
    
    if model_name in specs_db:
        return specs_db[model_name]
    else:
        return {'error': f'Model {model_name} not found'}

def calculate_monthly_payment(
    price: float,
    down_payment: float,
    loan_term_months: int,
    interest_rate: float
) -> Dict[str, float]:
    """Calculate monthly car payment.
    
    Args:
        price: Vehicle price in THB
        down_payment: Down payment amount
        loan_term_months: Loan duration (36, 60, 84)
        interest_rate: Annual interest rate (e.g., 3.5)
    
    Returns:
        Dictionary with payment calculations
    """
    monthly_rate = interest_rate / 100 / 12
    principal = price - down_payment
    
    if monthly_rate == 0:
        monthly_payment = principal / loan_term_months
    else:
        monthly_payment = principal * monthly_rate * (1 + monthly_rate) ** loan_term_months / \
                         ((1 + monthly_rate) ** loan_term_months - 1)
    
    total_paid = monthly_payment * loan_term_months
    total_interest = total_paid - principal
    
    return {
        'monthly_payment': round(monthly_payment, 2),
        'total_amount': round(total_paid, 2),
        'total_interest': round(total_interest, 2),
        'loan_term_months': loan_term_months
    }

def search_inventory(
    model: str,
    color: Optional[str] = None,
    max_price: Optional[float] = None
) -> Dict:
    """Search vehicle inventory.
    
    Args:
        model: Vehicle model to search
        color: Optional color filter
        max_price: Optional max price filter (THB)
    
    Returns:
        List of matching vehicles
    """
    inventory = [
        {'model': 'Toyota Camry', 'color': 'white', 'price': 1299000, 'mileage': 0},
        {'model': 'Toyota Camry', 'color': 'black', 'price': 1299000, 'mileage': 500},
        {'model': 'Honda Civic', 'color': 'red', 'price': 2499000, 'mileage': 0},
    ]
    
    results = inventory
    
    # Filter by model
    results = [v for v in results if v['model'].lower() == model.lower()]
    
    # Filter by color if provided
    if color:
        results = [v for v in results if v['color'].lower() == color.lower()]
    
    # Filter by price if provided
    if max_price:
        results = [v for v in results if v['price'] <= max_price]
    
    return {'count': len(results), 'vehicles': results}
```

---

## 🤖 Agent with Multiple Tools

### Creating Agent with Tools

```python
# agent.py
from google.adk.agents.llm_agent import Agent
from tools import (
    get_vehicle_specs,
    calculate_monthly_payment,
    search_inventory
)

# สร้าง sales agent ด้วย multiple tools
sales_agent = Agent(
    model='gemini-1.5-flash',
    name='AutomotiveSalesAgent',
    description='Sales assistant with vehicle lookup and financing tools',
    instruction="""You are an expert automotive sales consultant for a Thai dealership.

You have access to comprehensive tools to help customers:
- Look up vehicle specifications
- Calculate financing options
- Search our inventory

Always use the tools to provide accurate information.
Present information clearly in Thai currency (฿).
Help customers find the perfect vehicle based on their needs and budget.""",
    tools=[
        get_vehicle_specs,
        calculate_monthly_payment,
        search_inventory
    ]
)
```

### How Agent Decides to Use Tools

Google ADK agent automatically:
1. **Analyzes** customer message
2. **Determines** if any tool matches the request
3. **Calls** appropriate tool with parameters
4. **Formats** tool result into response
5. **Responds** to customer

ตัวอย่าง execution flow:

```
Customer: "Toyota Camry ราคาเท่าไหร่ครับ และผ่อนเดือนละเท่าไหร่"
              ↓
Agent analyzes request
              ↓
Decides to use: search_inventory() + calculate_monthly_payment()
              ↓
Calls: search_inventory(model='Toyota Camry')
               → Returns: price = ฿1,299,000
              ↓
Calls: calculate_monthly_payment(
    price=1299000,
    down_payment=300000,
    loan_term_months=60,
    interest_rate=3.5
)
               → Returns: monthly_payment = ฿20,500
              ↓
Agent formats response:
"Toyota Camry อยู่ในสต็อก ราคา ฿1,299,000 ฿
สำหรับการผ่อนช่วงเวลา 60 เดือน ด้วย ฿300,000 ดาวน์
จะเป็นประมาณ ฿20,500 ต่อเดือน"
```

---

## 🎯 Advanced Tool Patterns

### Tool with State/Side Effects

```python
def book_test_drive(
    customer_name: str,
    vehicle_model: str,
    preferred_date: str,
    preferred_time: str
) -> Dict:
    """Book a test drive appointment.
    
    Args:
        customer_name: Customer's full name
        vehicle_model: Which vehicle to test
        preferred_date: Date in YYYY-MM-DD format
        preferred_time: Time in HH:MM format
    
    Returns:
        Booking confirmation
    """
    import uuid
    
    booking = {
        'confirmation_id': str(uuid.uuid4())[:8],
        'customer': customer_name,
        'vehicle': vehicle_model,
        'date': preferred_date,
        'time': preferred_time,
        'status': 'confirmed',
        'location': 'Bangkok Showroom, Rama IV Rd.'
    }
    
    # In real scenario, save to database
    # database.save_booking(booking)
    
    return booking

def get_financing_options(
    vehicle_price: float,
    customer_credit_score: int
) -> Dict:
    """Get available financing options based on credit score.
    
    Args:
        vehicle_price: Price of vehicle in THB
        customer_credit_score: Credit score (0-850 US style or equivalent)
    
    Returns:
        Available financing products
    """
    if customer_credit_score >= 750:
        interest_rate = 3.5
        max_term = 84
    elif customer_credit_score >= 650:
        interest_rate = 5.5
        max_term = 60
    else:
        interest_rate = 7.5
        max_term = 48
    
    return {
        'interest_rate': interest_rate,
        'max_loan_term': max_term,
        'max_loan_amount': vehicle_price * 0.9,  # Can finance 90%
        'products': [
            {'term': 36, 'rate': interest_rate},
            {'term': 60, 'rate': interest_rate + 0.5},
            {'term': max_term, 'rate': interest_rate + 1.0}
        ]
    }
```

---

## 🎮 Hands-on Exercise: Chat with Agent

### Running Your Agent

```bash
# ไปที่ project directory
cd sales_agent

# รัน web interface
adk web --port 8000
```

### Test Scenarios

Try these conversations:

**Scenario 1: Spec Lookup**
```
You: "specifications of Honda Civic Type R"
Expected: Agent ใช้ get_vehicle_specs() เพื่อตอบ
```

**Scenario 2: Financing**
```
You: "ผ่อนรถ 1 ล้านบาท 60 เดือน อัตราดอกเบี้ย 4% เดือนละเท่าไหร่"
Expected: Agent ใช้ calculate_monthly_payment() เพื่อคำนวณ
```

**Scenario 3: Inventory Search**
```
You: "มี Honda Civic สีแดงอยู่มั้ย"
Expected: Agent ใช้ search_inventory() เพื่อค้นหา
```

**Scenario 4: Booking**
```
You: "สมัครทดสอบรถ Toyota Camry วันพรุ่งนี้ตอนเช้า"
Expected: Agent ใช้ book_test_drive() เพื่อจองเวลา
```

---

## 💡 Best Practices for Tools

1. **Clear Names & Descriptions**: Tool names ต้องชัดเจน ว่าทำอะไร
2. **Proper Type Hints**: ทุก parameter และ return ต้องมี type
3. **Useful Docstrings**: Docstring อธิบายให้ AI เข้าใจ
4. **Error Handling**: Return meaningful error messages
5. **Real Returns**: ส่วนใหญ่ return dict เป็น standard

---

## 🎯 Key Takeaways

1. ✅ Tools เป็น Python functions ที่ agent เรียกใช้
2. ✅ Type hints บ้านเป็นสิ่งจำเป็น
3. ✅ Docstrings ช่วยให้ agent เข้าใจ
4. ✅ Agent decide automatically เมื่อใช้ tool
5. ✅ ใช้ tools เพื่อให้ agent มีความจำเพาะและแม่นยำ

---

## 🚀 Next Steps

**Lesson 04**: รัน agent ผ่าน web interface และ API

**Code Examples**: [exercises/day1/sales_agent/tools.py](exercises/day1/sales_agent/tools.py)
    expect(result.error).to.exist;
  });
});

describe('Search Tool', () => {
  it('should return search results', async () => {
    const result = await searchTool.execute({ query: 'weather' });
    expect(result.query).to.equal('weather');
    expect(result.results).to.be.an('array');
  });
});
```

### Integration Tests

```javascript
describe('Agent with Tools', () => {
  it('should use calculator for math questions', async () => {
    const agent = new SmartToolAgent([calculatorTool]);
    const response = await agent.respond('What is 15 * 7?');

    expect(response).to.include('105'); // Should contain calculation result
  });

  it('should fallback to AI for non-tool questions', async () => {
    const agent = new SmartToolAgent([calculatorTool]);
    const response = await agent.respond('What is the meaning of life?');

    expect(response).to.be.a('string');
    expect(response.length).to.be.greaterThan(10);
  });
});
```

---

## 🌐 Real-World Tool Examples

### E-commerce Tools

```javascript
const productSearchTool = {
  name: 'searchProducts',
  description: 'ค้นหาสินค้าในร้าน',
  execute: async ({ query, category, maxPrice }) => {
    // Call e-commerce API
    const products = await api.searchProducts({
      q: query,
      category,
      price_max: maxPrice
    });
    return products;
  }
};

const orderStatusTool = {
  name: 'checkOrderStatus',
  description: 'ตรวจสอบสถานะคำสั่งซื้อ',
  execute: async ({ orderId }) => {
    const order = await api.getOrder(orderId);
    return {
      orderId,
      status: order.status,
      tracking: order.tracking_number,
      estimatedDelivery: order.delivery_date
    };
  }
};
```

### Customer Service Tools

```javascript
const faqTool = {
  name: 'searchFAQ',
  description: 'ค้นหาคำตอบจาก FAQ',
  execute: async ({ question }) => {
    const faqs = await database.query('faqs', {
      question: { $regex: question, $options: 'i' }
    });
    return faqs.map(faq => ({
      question: faq.question,
      answer: faq.answer,
      category: faq.category
    }));
  }
};

const ticketTool = {
  name: 'createSupportTicket',
  description: 'สร้าง ticket สนับสนุน',
  execute: async ({ customerId, issue, priority }) => {
    const ticket = await api.createTicket({
      customer_id: customerId,
      description: issue,
      priority: priority || 'medium',
      status: 'open'
    });
    return {
      ticketId: ticket.id,
      status: ticket.status,
      estimatedResponse: '2 hours'
    };
  }
};
```

---

## 🔄 Tool Chaining

### Sequential Tool Execution

```javascript
class ChainToolAgent extends SmartToolAgent {
  async respond(message) {
    const toolCalls = await this.decideToolChain(message);

    if (toolCalls && toolCalls.length > 0) {
      const results = [];

      for (const toolCall of toolCalls) {
        const result = await this.executeTool(toolCall.toolName, toolCall.parameters);
        results.push({
          tool: toolCall.toolName,
          result,
          reasoning: toolCall.reasoning
        });
      }

      // Combine results
      return this.synthesizeResults(results, message);
    }

    return await super.respond(message);
  }

  async decideToolChain(message) {
    const prompt = `Analyze this request: "${message}"

Available tools: ${this.tools.map(t => `${t.name}: ${t.description}`).join(', ')}

Sometimes multiple tools are needed. Decide if a sequence of tools would be helpful.

Respond with JSON array of tool calls:
[
  {
    "toolName": "tool1",
    "parameters": {...},
    "reasoning": "why this tool first"
  },
  {
    "toolName": "tool2",
    "parameters": {...},
    "reasoning": "why this tool next"
  }
]

Or empty array if no tools needed.`;

    const response = await this.think(prompt);
    return JSON.parse(response);
  }

  synthesizeResults(results, originalMessage) {
    const summary = results.map(r =>
      `${r.tool}: ${JSON.stringify(r.result)}`
    ).join('\n');

    const prompt = `Original request: "${originalMessage}"

Tool execution results:
${summary}

Synthesize a comprehensive response for the user.`;

    return this.think(prompt);
  }
}
```

---

## 🎮 Hands-on Exercises

### Exercise 1: Weather Agent

**Task:** สร้าง agent ที่ให้ข้อมูลสภาพอากาศ

```javascript
// Implement weather tool
// Integrate with agent
// Test with different cities
```

### Exercise 2: Personal Assistant

**Task:** สร้าง agent ที่มี tools สำหรับ:
- Calendar management
- Reminder setting
- Task tracking

### Exercise 3: Business Intelligence

**Task:** สร้าง agent ที่มี tools สำหรับ:
- Sales data analysis
- Customer insights
- Performance metrics

---

## 🚀 Best Practices

### Tool Design
1. **Clear Descriptions**: อธิบาย function ของ tool ชัดเจน
2. **Parameter Validation**: ตรวจสอบ input parameters
3. **Error Handling**: จัดการ errors อย่าง graceful
4. **Caching**: Cache results สำหรับ performance

### Agent Integration
1. **Smart Selection**: เลือก tool ที่เหมาะสม
2. **Fallback Logic**: มี fallback เมื่อ tool ล้มเหลว
3. **Response Formatting**: จัดรูปแบบ output ให้ user-friendly
4. **Logging**: Log tool usage สำหรับ debugging

---

## 📊 Monitoring Tool Usage

```javascript
class MonitoredToolAgent extends SmartToolAgent {
  constructor(tools) {
    super(tools);
    this.usageStats = new Map();
  }

  async executeTool(toolName, parameters) {
    const startTime = Date.now();

    try {
      const result = await super.executeTool(toolName, parameters);
      const duration = Date.now() - startTime;

      // Record successful usage
      this.recordUsage(toolName, true, duration);

      return result;
    } catch (error) {
      // Record failed usage
      this.recordUsage(toolName, false, Date.now() - startTime);
      throw error;
    }
  }

  recordUsage(toolName, success, duration) {
    const stats = this.usageStats.get(toolName) || {
      calls: 0,
      successes: 0,
      failures: 0,
      totalDuration: 0,
      avgDuration: 0
    };

    stats.calls++;
    if (success) stats.successes++;
    else stats.failures++;

    stats.totalDuration += duration;
    stats.avgDuration = stats.totalDuration / stats.calls;

    this.usageStats.set(toolName, stats);
  }

  getUsageStats() {
    return Object.fromEntries(this.usageStats);
  }
}
```

---

## 🎯 Key Takeaways

1. **Tools Extend Capabilities**: เพิ่ม functionality เกิน AI built-in
2. **Smart Tool Selection**: ใช้ AI เพื่อเลือก tool ที่เหมาะสม
3. **Error Handling**: จัดการ failures อย่าง robust
4. **Tool Chaining**: รวม multiple tools สำหรับ complex tasks
5. **Monitoring**: Track tool performance และ usage

---

## 🚀 Next Steps

ใน lesson ถัดไป เราจะเรียนรู้ **การ run agent ผ่าน web interface** และสร้าง user experience ที่ดี

**พร้อมให้ agent ของคุณมี superpowers แล้วหรือยัง?** ⚡🔧🤖

---

*Code Examples: [exercises/day1/tools/](exercises/day1/tools/)*