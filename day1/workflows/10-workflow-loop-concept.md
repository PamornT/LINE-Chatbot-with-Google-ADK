# Loop Workflow Concepts

## 🎯 Objective
เข้าใจและ implement loop workflows สำหรับ iterative tasks ที่ต้องทำซ้ำจนกว่าจะบรรลุเป้าหมาย

---

## 🔁 What is Loop Workflow?

**Loop Workflow** คือการทำงาน repetitive จนกว่าจะ satisfy condition หรือ complete goal

```
┌─────────────────┐
│   Initialize    │
└────────┬────────┘
         │
         ↓
    ┌─────────┐
    │ Execute │
    │ Step    │
    └────┬────┘
         │
         ↓
    ┌──────────┐    No
    │ Complete?├──────┐
    └────┬─────┘      │
         │ Yes        │
         │            │ Continue
         ↓            │
  ┌─────────────┐     │
  │   Return    │     │
  │  Results   │◄─────┘
  └─────────────┘
```

### เมื่อไหร่ใช้ Loop?
- **Retry Logic**: ลองใหม่เมื่อ fail
- **Refinement**: ปรับปรุงผลลัพธ์ step-by-step
- **Iteration**: Process ที่ต้องทำหลายรอบ
- **Convergence**: จนกว่าจะ converge ไป solution

---

## 🏗️ Loop Workflow in ADK

### Basic Loop Structure

```javascript
class LoopWorkflowAgent extends BasicAgent {
  constructor(config = {}) {
    super('LoopWorkflowAgent');
    this.maxIterations = config.maxIterations || 10;
    this.exitCondition = config.exitCondition || (() => false);
    this.refinementFunction = config.refinementFunction;
  }

  async executeLoop(initialInput) {
    console.log('🔁 Starting loop workflow...');

    let iteration = 0;
    let currentState = initialInput;
    const history = [];

    while (iteration < this.maxIterations) {
      console.log(`📍 Iteration ${iteration + 1}/${this.maxIterations}`);

      try {
        // Execute step
        const stepResult = await this.executeStep(currentState);
        history.push(stepResult);

        // Check exit condition
        if (this.exitCondition(stepResult, currentState)) {
          console.log('✅ Exit condition met');
          break;
        }

        // Refine for next iteration
        currentState = this.refinementFunction
          ? await this.refinementFunction(stepResult, currentState)
          : stepResult;

        iteration++;

      } catch (error) {
        console.error(`❌ Iteration ${iteration + 1} error:`, error);

        // Decide whether to retry
        if (this.shouldRetry(error, iteration)) {
          iteration++;
          continue;
        } else {
          throw error;
        }
      }
    }

    if (iteration >= this.maxIterations) {
      console.warn('⚠️ Max iterations reached');
    }

    return this.synthesizeResults(history, currentState);
  }

  async executeStep(state) {
    // Override in subclass
    return state;
  }

  shouldRetry(error, iteration) {
    // Override in subclass
    return iteration < this.maxIterations - 1;
  }

  synthesizeResults(history, finalState) {
    return {
      iterations: history.length,
      finalState,
      history
    };
  }
}
```

---

## 🎯 Practical Examples for Automotive

### Vehicle Recommendation Loop

```javascript
class VehicleRecommendationLoop extends LoopWorkflowAgent {
  constructor(customerProfile, availableVehicles) {
    super({
      maxIterations: 5,
      exitCondition: (result) => result.recommendations.length >= 3 || result.userSatisfied,
      refinementFunction: (result, currentState) => ({
        ...currentState,
        usedFilters: [...currentState.usedFilters, ...result.appliedFilters],
        previousRecommendations: [...currentState.previousRecommendations, ...result.recommendations]
      })
    });

    this.customerProfile = customerProfile;
    this.availableVehicles = availableVehicles;
  }

  async executeStep(state) {
    console.log('🔄 Refining vehicle recommendations...');

    // Get customer preferences
    const preferences = this.customerProfile.preferences || {};

    // Filter vehicles based on applied filters
    let filtered = this.availableVehicles;

    if (state.priceRange) {
      filtered = filtered.filter(v =>
        v.price >= state.priceRange.min && v.price <= state.priceRange.max
      );
    }

    if (state.fuelType) {
      filtered = filtered.filter(v => v.fuelType === state.fuelType);
    }

    if (state.vehicleType) {
      filtered = filtered.filter(v => v.type === state.vehicleType);
    }

    // Score remaining vehicles
    const scored = filtered.map(vehicle => ({
      ...vehicle,
      matchScore: this.calculateMatchScore(vehicle, preferences),
      matchReasons: this.getMatchReasons(vehicle, preferences)
    }));

    // Top recommendations
    const topRecommendations = scored
      .filter(v => !state.previousRecommendations.some(prev => prev.id === v.id))
      .sort((a, b) => b.matchScore - a.matchScore)
      .slice(0, 3);

    return {
      recommendations: topRecommendations,
      appliedFilters: state.refinedFilters || [],
      refinementRound: (state.refinementRound || 0) + 1,
      totalVehiclesConsidered: filtered.length
    };
  }

  calculateMatchScore(vehicle, preferences) {
    let score = 0;

    if (preferences.fuelType && vehicle.fuelType === preferences.fuelType) score += 25;
    if (preferences.budget && vehicle.price <= preferences.budget) score += 20;
    if (preferences.features) {
      const matchingFeatures = preferences.features.filter(f =>
        vehicle.features.includes(f)
      ).length;
      score += (matchingFeatures / preferences.features.length) * 30;
    }
    if (vehicle.rating >= 4.5) score += 15;

    return score;
  }

  getMatchReasons(vehicle, preferences) {
    const reasons = [];

    if (preferences.fuelType && vehicle.fuelType === preferences.fuelType) {
      reasons.push(`Matches your preferred ${vehicle.fuelType} fuel type`);
    }
    if (vehicle.rating >= 4.5) {
      reasons.push(`High customer rating: ${vehicle.rating}/5`);
    }

    return reasons;
  }
}
```

### Price Negotiation Loop

```javascript
class NegotiationLoop extends LoopWorkflowAgent {
  constructor(initialPrice, targetPrice, negotiationStrategy) {
    super({
      maxIterations: 8,
      exitCondition: (result) => 
        Math.abs(result.currentOffer - result.targetPrice) < 50000 || result.dealClosed,
      refinementFunction: (result, state) => ({
        ...state,
        currentOffer: result.currentOffer,
        negotiationRound: (state.negotiationRound || 0) + 1,
        previousOffers: [...(state.previousOffers || []), result.currentOffer]
      })
    });

    this.initialPrice = initialPrice;
    this.targetPrice = targetPrice;
    this.strategy = negotiationStrategy;
  }

  async executeStep(state) {
    const round = state.negotiationRound || 1;

    // Calculate negotiation position
    const priceGap = this.initialPrice - this.targetPrice;
    const progressPercent = Math.min(1, (round - 1) / 5); // Closer each round

    let counterOffer;

    if (this.strategy === 'aggressive') {
      counterOffer = this.initialPrice - (priceGap * progressPercent * 0.8);
    } else if (this.strategy === 'moderate') {
      counterOffer = this.initialPrice - (priceGap * progressPercent * 0.5);
    } else {
      counterOffer = this.initialPrice - (priceGap * progressPercent * 0.3);
    }

    // Prepare negotiation message
    const message = this.generateNegotiationMessage(
      counterOffer,
      this.targetPrice,
      round
    );

    return {
      round,
      currentOffer: Math.round(counterOffer),
      message,
      dealClosed: Math.abs(counterOffer - this.targetPrice) < 50000
    };
  }

  generateNegotiationMessage(offer, target, round) {
    if (Math.abs(offer - target) < 50000) {
      return `We're very close! How about ฿${Math.round(offer).toLocaleString()}?`;
    }

    if (round > 5) {
      return `This is our best offer: ฿${Math.round(offer).toLocaleString()}. Shall we finalize?`;
    }

    return `Counter offer: ฿${Math.round(offer).toLocaleString()}`;
  }
}
```

### Test Drive Scheduling Loop

```javascript
class TestDriveSchedulingLoop extends LoopWorkflowAgent {
  constructor(availableSlots, customerPreferences) {
    super({
      maxIterations: 4,
      exitCondition: (result) => result.confirmed || result.decline,
    });

    this.availableSlots = availableSlots;
    this.customerPreferences = customerPreferences;
  }

  async executeStep(state) {
    const offering = state.offeringIndex || 0;

    if (offering >= this.availableSlots.length) {
      return {
        confirmed: false,
        decline: true,
        message: 'No more available slots'
      };
    }

    const slot = this.availableSlots[offering];

    return {
      proposal: {
        date: slot.date,
        time: slot.time,
        location: slot.location,
        duration: '30 minutes'
      },
      offeringIndex: offering + 1,
      message: this.formatProposal(slot)
    };
  }

  formatProposal(slot) {
    return `มีช่วงเวลาว่างดังนี้ นะครับ
📅 วันที่: ${slot.date} 
⏰ เวลา: ${slot.time}
📍 สถานที่: ${slot.location}

สนใจตัวเลือกนี้หรือมีเวลาอื่นที่เหมาะสมกว่าหรือครับ?`;
  }
}
```

---

## 🛡️ Error Recovery & Backoff

### Retry with Exponential Backoff

```javascript
class RobustLoopAgent extends LoopWorkflowAgent {
  async executeLoopWithBackoff(initialInput) {
    let iteration = 0;
    let currentState = initialInput;
    let backoffDelay = 100; // Start with 100ms

    while (iteration < this.maxIterations) {
      try {
        const result = await this.executeStep(currentState);

        if (this.exitCondition(result)) break;

        currentState = this.refinementFunction(result, currentState);
        iteration++;
        backoffDelay = 100; // Reset backoff

      } catch (error) {
        console.warn(`⚠️ Attempt ${iteration + 1} failed, retrying in ${backoffDelay}ms...`);

        await new Promise(resolve => setTimeout(resolve, backoffDelay));

        backoffDelay = Math.min(backoffDelay * 2, 10000); // Cap at 10s
        iteration++;
      }
    }

    return currentState;
  }
}
```

---

## 📊 Loop Metrics & Monitoring

### Tracking Loop Performance

```javascript
class LoopMetrics {
  constructor() {
    this.metrics = {
      totalLoops: 0,
      successfulLoops: 0,
      failedLoops: 0,
      avgIterations: 0,
      avgDuration: 0
    };
  }

  trackLoop(result) {
    this.metrics.totalLoops++;

    if (result.success) {
      this.metrics.successfulLoops++;
    } else {
      this.metrics.failedLoops++;
    }

    if (this.metrics.avgIterations === 0) {
      this.metrics.avgIterations = result.iterations;
    } else {
      this.metrics.avgIterations = (
        (this.metrics.avgIterations * (this.metrics.totalLoops - 1) + result.iterations) /
        this.metrics.totalLoops
      );
    }

    this.metrics.avgDuration = (
      (this.metrics.avgDuration * (this.metrics.totalLoops - 1) + result.duration) /
      this.metrics.totalLoops
    );
  }

  getSuccessRate() {
    return (this.metrics.successfulLoops / this.metrics.totalLoops * 100).toFixed(2);
  }
}
```

---

## 🎮 Hands-on Exercises

### Exercise 1: Recommendation Refinement Loop

**Task:** สร้าง agent ที่ refine recommendations loop

```javascript
// Start with broad filters
// Get feedback on recommendations
// Narrow down based on feedback
// Repeat until satisfied
```

### Exercise 2: Quote Refinement Loop

**Task:** สร้าง loop ที่ refine financing quotes

```javascript
// Generate initial quote
// Adjust based on customer feedback
// Recalculate with new parameters
// Repeat until customer approves
```

### Exercise 3: Appointment Scheduling Loop

**Task:** สร้าง loop ที่ schedule appointments

```javascript
// Offer time slots
// Get customer preference
// If declined, offer alternatives
// Repeat until booked
```

---

## 🚀 Best Practices

### Loop Design Guidelines
1. **Clear Exit Condition**: Define when loop should stop
2. **Max Iterations**: Always set to prevent infinite loops
3. **Progressive Refinement**: Improve each iteration
4. **Logging**: Track progress and issues
5. **Timeout**: Set timeout per iteration

### Performance Optimization
1. **Efficient Filtering**: Optimize data filtering
2. **Batch Operations**: Do multiple things if possible
3. **Cache Results**: Reuse calculations
4. **Early Exit**: Exit as soon as condition met
5. **Resource Cleanup**: Free resources after loop

---

## 🎯 Key Takeaways

1. **Loops Enable Iteration**: Support complex refinement processes
2. **Exit Conditions Matter**: Define clear stopping points
3. **Error Handling is Critical**: Implement retry logic
4. **Monitoring Helps Optimize**: Track performance metrics
5. **Progressive Refinement**: Improve results with each iteration

---

## 🚀 Next Steps

ใน lesson ถัดไป เราจะสร้าง **Operations Agent** สำหรับ logistics และ operations management

**พร้อมใช้ loops สำหรับ complex workflows แล้วหรือยัง?** 🔁🎯🤖

---

*Code Examples: [exercises/day1/workflows/loops/](exercises/day1/workflows/loops/)*
