# Building Finance Agent for Automotive Sales

## 🎯 Objective
สร้าง Finance Agent ที่สามารถจัดการ financing options, calculate payments, handle financing inquiries สำหรับรถยนต์

---

## 🏦 Finance Agent Architecture

### Core Capabilities

Finance Agent ควรมี:
- **Financing Calculations**: Calculate monthly payments, total interest, loan terms
- **Loan Products**: Different financing options available
- **Credit Assessment**: Evaluate customer creditworthiness
- **Insurance Integration**: Calculate insurance estimates
- **Trade-in Evaluation**: Assess trade-in vehicle values
- **Compliance**: Interest rates, fees, terms & conditions

### Agent Structure

```javascript
class FinanceAgent extends BasicAgent {
  constructor() {
    super('FinanceAgent');
    this.financingEngine = new FinancingCalculator();
    this.creditScorer = new CreditAssessment();
    this.insuranceCalculator = new InsuranceCalculator();
    this.tradeInEvaluator = new TradeInEvaluator();
  }

  async respond(message, context = {}) {
    const intent = await this.analyzeFinancialIntent(message);
    const customerData = await this.getFinancialProfile(context.customerId);
    const response = await this.generateFinancialAdvice(message, intent, customerData);

    await this.logTransaction(context.customerId, message, response, intent);

    return response;
  }

  async analyzeFinancialIntent(message) {
    const prompt = `Analyze customer's financial inquiry:

Message: "${message}"

Possible intents:
- monthly_payment: Customer asks about monthly payment
- loan_amount: Asking about loan amount to borrow
- interest_rate: Questions about interest rates
- term_length: Asking about loan terms (12-84 months)
- trade_in: Trade-in vehicle valuation
- insurance_quote: Insurance cost estimate
- affordability: Can they afford this car?
- financing_options: What financing plans available?

Return JSON: {"intent": "intent_type", "confidence": 0-1}`;

    const analysis = await this.think(prompt);
    return JSON.parse(analysis);
  }
}
```

---

## 💰 Financing Calculator Engine

### Core Calculation Logic

```javascript
class FinancingCalculator {
  constructor() {
    this.loanProducts = {
      'standard_36': {
        name: 'Standard 36-month',
        term_months: 36,
        interest_rate_range: [2.5, 4.5],
        down_payment_percent: 20
      },
      'standard_60': {
        name: 'Standard 60-month',
        term_months: 60,
        interest_rate_range: [3.0, 5.0],
        down_payment_percent: 15
      },
      'extended_84': {
        name: 'Extended 84-month',
        term_months: 84,
        interest_rate_range: [4.0, 6.5],
        down_payment_percent: 10
      },
      'zero_interest': {
        name: 'Zero Interest Promotion',
        term_months: 48,
        interest_rate_range: [0, 0],
        down_payment_percent: 30
      }
    };
  }

  calculateMonthlyPayment(vehiclePrice, downPayment, monthlyRate, termMonths) {
    const principal = vehiclePrice - downPayment;

    if (monthlyRate === 0) {
      return principal / termMonths;
    }

    const monthlyPayment =
      (principal * monthlyRate * Math.pow(1 + monthlyRate, termMonths)) /
      (Math.pow(1 + monthlyRate, termMonths) - 1);

    return monthlyPayment;
  }

  generateFinancingOptions(vehiclePrice, customerProfile) {
    const options = [];
    const customerRate = this.determineInterestRate(customerProfile);

    for (const [productId, product] of Object.entries(this.loanProducts)) {
      const downPayment = vehiclePrice * (product.down_payment_percent / 100);
      const monthlyRate = (customerRate || product.interest_rate_range[0]) / 100 / 12;

      const monthlyPayment = this.calculateMonthlyPayment(
        vehiclePrice,
        downPayment,
        monthlyRate,
        product.term_months
      );

      const totalPayment = monthlyPayment * product.term_months;
      const totalInterest = totalPayment - (vehiclePrice - downPayment);

      options.push({
        productId,
        productName: product.name,
        downPayment: Math.round(downPayment),
        monthlyPayment: Math.round(monthlyPayment),
        interestRate: customerRate || product.interest_rate_range[0],
        termMonths: product.term_months,
        totalPayment: Math.round(totalPayment),
        totalInterest: Math.round(totalInterest),
        affairability: this.calculateAffordability(monthlyPayment, customerProfile)
      });
    }

    return options.sort((a, b) => a.monthlyPayment - b.monthlyPayment);
  }

  determineInterestRate(customerProfile) {
    if (!customerProfile) return null;

    const creditScore = customerProfile.creditScore || 600;

    if (creditScore >= 750) return 2.5;
    if (creditScore >= 700) return 3.5;
    if (creditScore >= 650) return 4.5;
    if (creditScore >= 600) return 5.5;
    return 6.5;
  }

  calculateAffordability(monthlyPayment, customerProfile) {
    if (!customerProfile || !customerProfile.monthlyIncome) {
      return 'Unable to calculate';
    }

    const affordabilityRatio = monthlyPayment / (customerProfile.monthlyIncome * 0.2);

    if (affordabilityRatio <= 1) return 'Affordable';
    if (affordabilityRatio <= 1.2) return 'Possible with tight budget';
    return 'May be challenging';
  }
}
```

### Formatting Financing Response

```javascript
class FinanceAgent extends BasicAgent {
  formatFinancingOptions(options, vehicleModel, customerProfile) {
    let response = `🏦 ${vehicleModel} - Financing Options\n`;
    response += `========================================\n\n`;

    options.forEach((option, index) => {
      response += `📋 Option ${index + 1}: ${option.productName}\n`;
      response += `   💵 Down Payment: ฿${option.downPayment.toLocaleString()}\n`;
      response += `   📅 Monthly Payment: ฿${option.monthlyPayment.toLocaleString()}\n`;
      response += `   📊 Interest Rate: ${option.interestRate}%\n`;
      response += `   ⏳ Loan Term: ${option.termMonths} months\n`;
      response += `   💰 Total Cost: ฿${option.totalPayment.toLocaleString()}\n`;
      response += `   💸 Interest Paid: ฿${option.totalInterest.toLocaleString()}\n`;
      response += `   ✅ Affordability: ${option.affairability}\n\n`;
    });

    return response;
  }

  async handleMonthlyPaymentQuery(vehiclePrice, message, customerProfile) {
    // Extract down payment if mentioned
    const downPaymentMatch = message.match(/(\d+)\s*(?:บาท|บ|ะหยัด)?/);
    const downPayment = downPaymentMatch ? parseInt(downPaymentMatch[1]) : vehiclePrice * 0.2;

    const options = this.financingEngine.generateFinancingOptions(vehiclePrice, customerProfile);
    const defaultOption = options[0]; // Most affordable

    let response = `สำหรับ ${vehiclePrice.toLocaleString()} บาท\n`;
    response += `ด้วยเงินดาวน์ ${downPayment.toLocaleString()} บาท\n\n`;
    response += `ค่าผ่อนต่อเดือนประมาณ: ฿${defaultOption.monthlyPayment.toLocaleString()}\n`;
    response += `(${defaultOption.termMonths} เดือน อัตรา ${defaultOption.interestRate}%)\n\n`;
    response += `เรามีตัวเลือกอื่นๆ อีก ${options.length - 1} แบบครับ ต้องการดูรายละเอียดทั้งหมดหรือไม่?`;

    return response;
  }
}
```

---

## 💳 Credit Assessment & Risk Scoring

### Credit Scoring System

```javascript
class CreditAssessment {
  assessCreditWorthiness(customerData) {
    const scores = {
      credit_history: 0,
      debt_to_income: 0,
      employment_stability: 0,
      collateral: 0,
      down_payment: 0,
      total: 0
    };

    // Credit history score (0-25)
    if (customerData.creditScore >= 750) scores.credit_history = 25;
    else if (customerData.creditScore >= 700) scores.credit_history = 20;
    else if (customerData.creditScore >= 650) scores.credit_history = 15;
    else if (customerData.creditScore >= 600) scores.credit_history = 10;
    else scores.credit_history = 0;

    // Debt-to-income ratio (0-25)
    const dtiRatio = customerData.totalDebt / customerData.monthlyIncome;
    if (dtiRatio <= 0.3) scores.debt_to_income = 25;
    else if (dtiRatio <= 0.5) scores.debt_to_income = 20;
    else if (dtiRatio <= 0.7) scores.debt_to_income = 10;
    else scores.debt_to_income = 0;

    // Employment stability (0-20)
    const yearsEmployed = customerData.yearsAtCurrentJob || 0;
    if (yearsEmployed >= 5) scores.employment_stability = 20;
    else if (yearsEmployed >= 3) scores.employment_stability = 15;
    else if (yearsEmployed >= 1) scores.employment_stability = 10;
    else scores.employment_stability = 5;

    // Down payment size (0-15)
    const downPaymentPercent = customerData.downPaymentPercent || 0;
    if (downPaymentPercent >= 30) scores.down_payment = 15;
    else if (downPaymentPercent >= 20) scores.down_payment = 12;
    else if (downPaymentPercent >= 10) scores.down_payment = 8;
    else scores.down_payment = 5;

    // Trade-in value as collateral (0-15)
    if (customerData.hasTradeIn) {
      scores.collateral = Math.min(15, customerData.tradeInValue / 50000);
    }

    scores.total = Object.values(scores).reduce((a, b) => a + b) - scores.total;

    return {
      overall_score: scores.total,
      grade: this.scoreToGrade(scores.total),
      max_loan_amount: this.calculateMaxLoan(scores.total, customerData),
      likely_approval: scores.total >= 60,
      breakdown: scores
    };
  }

  scoreToGrade(score) {
    if (score >= 90) return 'Excellent';
    if (score >= 75) return 'Good';
    if (score >= 60) return 'Fair';
    if (score >= 45) return 'Poor';
    return 'Very Poor';
  }

  calculateMaxLoan(score, customerData) {
    const baseAmount = (customerData.monthlyIncome * 12) * 3; // 3x annual income

    if (score >= 90) return baseAmount * 1.2;
    if (score >= 75) return baseAmount * 1.0;
    if (score >= 60) return baseAmount * 0.8;
    if (score >= 45) return baseAmount * 0.6;
    return baseAmount * 0.4;
  }
}
```

---

## 🚗 Trade-in Evaluation

### Trade-in Assessment System

```javascript
class TradeInEvaluator {
  async evaluateTradeIn(tradeInVehicle) {
    const marketValue = await this.getMarketValue(tradeInVehicle.make, tradeInVehicle.model, tradeInVehicle.year);

    const condition = tradeInVehicle.condition || 'average'; // excellent, good, average, fair, poor
    const conditionMultiplier = {
      'excellent': 1.0,
      'good': 0.9,
      'average': 0.75,
      'fair': 0.6,
      'poor': 0.45
    }[condition];

    const mileage = tradeInVehicle.mileage || 100000;
    const mileageDeduction = Math.max(0, (mileage - 120000) / 10000 * 100);

    let adjustedValue = marketValue * conditionMultiplier;
    adjustedValue -= (adjustedValue * mileageDeduction / 100);

    return {
      vehicle: `${tradeInVehicle.year} ${tradeInVehicle.make} ${tradeInVehicle.model}`,
      marketValue: Math.round(marketValue),
      mileage: mileage,
      condition: condition,
      adjustedValue: Math.round(adjustedValue),
      tradeInCredit: Math.round(adjustedValue * 0.95) // 95% of adjusted value
    };
  }

  async getMarketValue(make, model, year) {
    // Call real market value API or use database
    // This is simplified example
    const values = {
      'Toyota_Camry_2020': 850000,
      'Honda_Civic_2019': 750000,
      'Toyota_Vios_2021': 550000
    };

    const key = `${make}_${model}_${year}`;
    return values[key] || 500000;
  }

  formatTradeInResponse(tradeIn) {
    let response = `🚗 Trade-In Evaluation\n`;
    response += `─────────────────────\n`;
    response += `Vehicle: ${tradeIn.vehicle}\n`;
    response += `Mileage: ${tradeIn.mileage.toLocaleString()} km\n`;
    response += `Condition: ${tradeIn.condition}\n\n`;
    response += `Market Value: ฿${tradeIn.marketValue.toLocaleString()}\n`;
    response += `Adjusted Value: ฿${tradeIn.adjustedValue.toLocaleString()}\n`;
    response += `Trade-In Credit: ฿${tradeIn.tradeInCredit.toLocaleString()}\n`;

    return response;
  }
}
```

---

## 🎮 Hands-on Exercises

### Exercise 1: Financing Calculator Bot

**Task:** สร้าง agent ที่ calculate monthly payments ได้ง่าย

```javascript
// Parse price, down payment from customer message
// Calculate all available options
// Present recommendation based on income
```

### Exercise 2: Trade-in Valuation Agent

**Task:** สร้าง agent ที่ evaluate trade-in vehicles

```javascript
// Get trade-in vehicle info
// Calculate adjusted value
// Create financing offer including trade-in credit
```

### Exercise 3: Credit Pre-approval Agent

**Task:** สร้าง agent ที่ give quick pre-approval

```javascript
// Quick credit assessment
// Provide max loan amount
// Suggest suitable financing options
```

---

## 📊 Financial Metrics & Reporting

### Finance Agent KPIs

```javascript
class FinanceMetrics {
  constructor() {
    this.metrics = {
      total_inquiries: 0,
      avg_vehicle_price: 0,
      avg_loan_amount: 0,
      approval_rate: 0,
      avg_interest_rate: 0
    };
  }

  trackFinancialInquiry(inquiry) {
    this.metrics.total_inquiries++;
    this.metrics.avg_vehicle_price = (
      (this.metrics.avg_vehicle_price * (this.metrics.total_inquiries - 1) + inquiry.vehiclePrice) /
      this.metrics.total_inquiries
    );
  }
}
```

---

## 🚀 Best Practices

### Financial Agent Guidelines
1. **Accuracy**: Double-check calculations
2. **Transparency**: Disclose all terms and fees
3. **Compliance**: Follow lending regulations
4. **Privacy**: Protect customer financial data
5. **Personalization**: Tailor to customer profile

### Risk Management
1. **Credit Verification**: Verify customer data
2. **Documentation**: Keep detailed records
3. **Fraud Detection**: Monitor for suspicious patterns
4. **Compliance Audit**: Regular compliance checks

---

## 🎯 Key Takeaways

1. **Financing is Complex**: Multiple variables affect offers
2. **Personalization Matters**: Different rates for different customers
3. **Transparency Builds Trust**: Clear disclosure of all costs
4. **Automation Saves Time**: Quick calculations and approvals
5. **Integration is Essential**: Connect with CRM, insurance, valuation systems

---

## 🚀 Next Steps

ใน lesson ถัดไป เราจะ integrate state management เพื่อจำ context ระหว่าง conversations

**พร้อมทำให้ customers ได้ดีล ที่ดีที่สุดแล้วหรือยัง?** 💰🤖

---

*Code Examples: [exercises/day1/agents/finance-agent/](exercises/day1/agents/finance-agent/)*
