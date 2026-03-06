# Parallel Workflow Concepts

## 🎯 Objective
เข้าใจและ implement parallel workflows สำหรับ tasks ที่สามารถทำพร้อมกันได้ เพื่อเพิ่ม performance

---

## ⚡ What is Parallel Workflow?

**Parallel Workflow** คือการทำหลาย tasks พร้อมๆ กัน ไม่ต้องรอ tasks ก่อนหน้าเสร็จ

```
Task 1 ─┐
        ├─→ Combine Results
Task 2 ─┤
        │
Task 3 ─┘
```

### เมื่อไหร่ใช้ Parallel?
- **Independent Tasks**: Tasks ไม่มี dependency ต่อกัน
- **Speed Critical**: ต้องการผลลัพธ์เร็วที่สุด
- **Multiple Sources**: ดึงข้อมูลจากหลายแหล่ง
- **Concurrent Operations**: APIs, database queries

---

## 🏗️ Parallel Workflow in ADK

### Basic Parallel Structure

```javascript
class ParallelWorkflowAgent extends BasicAgent {
  constructor(tasks) {
    super('ParallelWorkflowAgent');
    this.tasks = tasks;
  }

  async executeInParallel(input) {
    console.log(`🚀 Executing ${this.tasks.length} tasks in parallel...`);

    try {
      // Execute all tasks simultaneously
      const results = await Promise.all(
        this.tasks.map((task, index) =>
          this.executeTask(task, input, index)
        )
      );

      return this.synthesizeResults(results);

    } catch (error) {
      console.error('❌ Parallel execution failed:', error);
      throw error;
    }
  }

  async executeTask(taskFunction, input, taskIndex) {
    const startTime = Date.now();

    try {
      console.log(`📋 Task ${taskIndex + 1} started`);
      const result = await taskFunction(input);
      const duration = Date.now() - startTime;

      console.log(`✅ Task ${taskIndex + 1} completed in ${duration}ms`);

      return {
        taskIndex,
        success: true,
        result,
        duration
      };

    } catch (error) {
      const duration = Date.now() - startTime;

      console.error(`❌ Task ${taskIndex + 1} failed in ${duration}ms:`, error);

      return {
        taskIndex,
        success: false,
        error: error.message,
        duration
      };
    }
  }

  synthesizeResults(results) {
    const successful = results.filter(r => r.success);
    const failed = results.filter(r => !r.success);
    const totalDuration = Math.max(...results.map(r => r.duration));

    return {
      totalTasks: results.length,
      successfulTasks: successful.length,
      failedTasks: failed.length,
      totalDuration,
      results: successful.map(r => r.result),
      errors: failed.map(r => r.error)
    };
  }
}
```

---

## 🎯 Practical Examples for Automotive Business

### Vehicle Information Collection Workflow

```javascript
// Parallel tasks for customer inquiry
const vehicleInformationTasks = [
  // Task 1: Get vehicle specifications
  async (input) => {
    console.log('1️⃣ Fetching vehicle specs...');
    const specs = await getVehicleSpecifications(input.vehicleModel);
    return {
      type: 'specifications',
      data: specs,
      includes: ['engine', 'transmission', 'dimensions', 'weight']
    };
  },

  // Task 2: Get pricing and financing options
  async (input) => {
    console.log('2️⃣ Calculating financing options...');
    const pricing = await calculateFinancing(
      input.vehicleModel,
      input.downPayment || 0,
      input.interestRate || 'current'
    );
    return {
      type: 'financing',
      data: pricing,
      includes: ['cash_price', 'monthly_payment', 'total_cost']
    };
  },

  // Task 3: Get availability in nearby dealerships
  async (input) => {
    console.log('3️⃣ Checking dealership inventory...');
    const availability = await checkDealershipInventory(
      input.vehicleModel,
      input.customerLocation
    );
    return {
      type: 'availability',
      data: availability,
      includes: ['dealership', 'distance', 'inventory', 'color_options']
    };
  },

  // Task 4: Get customer reviews and ratings
  async (input) => {
    console.log('4️⃣ Gathering customer reviews...');
    const reviews = await getVehicleReviews(input.vehicleModel);
    return {
      type: 'reviews',
      data: reviews,
      includes: ['rating', 'reliability', 'user_feedback']
    };
  },

  // Task 5: Get environmental/fuel efficiency info
  async (input) => {
    console.log('5️⃣ Getting fuel economy and emissions...');
    const efficiency = await getEcoRating(input.vehicleModel);
    return {
      type: 'eco_rating',
      data: efficiency,
      includes: ['mpg', 'co2_emissions', 'eco_score']
    };
  }
];

// Execute all tasks in parallel
const autoAgent = new ParallelWorkflowAgent(vehicleInformationTasks);
const result = await autoAgent.executeInParallel({
  vehicleModel: 'Toyota Camry Hybrid 2025',
  downPayment: 200000,
  customerLocation: 'Bangkok',
  interestRate: 3.5
});

console.log('📊 Comprehensive Vehicle Information Ready:', result);
```

### Automotive Sales Parallel Analysis

```javascript
const salesAnalysisTasks = [
  // Analyze lead quality
  async (leadData) => {
    return await analyzeLead(leadData);
  },

  // Check customer credit score
  async (leadData) => {
    return await checkCreditScore(leadData.customerId);
  },

  // Get competitive pricing for trade-in
  async (leadData) => {
    return await evaluateTradeIn(leadData.currentVehicle);
  },

  // Retrieve warranty options
  async (leadData) => {
    return await getWarrantyOptions(leadData.vehicleModel);
  },

  // Check insurance rate quotes
  async (leadData) => {
    return await getInsuranceQuotes(leadData.vehicleModel, leadData.customerAge);
  }
];
```

---

## 🔄 Advanced Parallel Patterns

### Parallel with Fallback

```javascript
class ResilientParallelAgent extends ParallelWorkflowAgent {
  async executeWithFallback(input) {
    console.log('🔄 Executing with fallback strategy...');

    const results = await Promise.allSettled(
      this.tasks.map(task => task(input))
    );

    return results.map((result, index) => {
      if (result.status === 'fulfilled') {
        return { taskIndex: index, success: true, result: result.value };
      } else {
        console.warn(`⚠️ Task ${index} failed, using fallback...`);
        return {
          taskIndex: index,
          success: false,
          fallback: `Default response for task ${index}`,
          error: result.reason
        };
      }
    });
  }
}
```

### Parallel with Race Condition

```javascript
class FastestResultAgent extends BasicAgent {
  async executeRaceCondition(input, tasks) {
    console.log('🏁 Racing to get first successful result...');

    try {
      const result = await Promise.race(
        tasks.map(task => task(input))
      );

      console.log('🏆 First result received');
      return result;

    } catch (error) {
      console.error('❌ All tasks failed in race condition');
      throw error;
    }
  }
}
```

### Batched Parallel Execution

```javascript
class BatchedParallelAgent extends ParallelWorkflowAgent {
  async executeBatched(input, batchSize = 3) {
    console.log(`📦 Executing ${this.tasks.length} tasks in batches of ${batchSize}...`);

    const results = [];
    for (let i = 0; i < this.tasks.length; i += batchSize) {
      const batch = this.tasks.slice(i, i + batchSize);
      const batchResults = await Promise.all(
        batch.map((task, index) =>
          this.executeTask(task, input, i + index)
        )
      );

      results.push(...batchResults);
      console.log(`✅ Batch ${Math.floor(i / batchSize) + 1} complete`);
    }

    return this.synthesizeResults(results);
  }
}
```

---

## 📊 Performance Comparison

### Sequential vs Parallel

```javascript
class PerformanceComparison {
  async compareExecutionMethods(tasks, input) {
    // Sequential execution
    const seqStart = Date.now();
    let seqResult = null;
    for (const task of tasks) {
      seqResult = await task(input);
    }
    const seqDuration = Date.now() - seqStart;

    // Parallel execution
    const parStart = Date.now();
    const parResult = await Promise.all(
      tasks.map(task => task(input))
    );
    const parDuration = Date.now() - parStart;

    const improvement = ((seqDuration - parDuration) / seqDuration * 100).toFixed(2);

    console.log('⏱️ Performance Comparison:');
    console.log(`  Sequential: ${seqDuration}ms`);
    console.log(`  Parallel:   ${parDuration}ms`);
    console.log(`  Speed-up:   ${improvement}%`);

    return {
      sequential: seqDuration,
      parallel: parDuration,
      improvement: `${improvement}%`
    };
  }
}
```

---

## 🧪 Error Handling in Parallel Tasks

### Handling Partial Failures

```javascript
class RobustParallelAgent extends ParallelWorkflowAgent {
  async executeWithPartialFailure(input, failureThreshold = 0.5) {
    const results = await Promise.allSettled(
      this.tasks.map((task, index) => this.executeTask(task, input, index))
    );

    const failures = results.filter(r => r.status === 'rejected').length;
    const failureRate = failures / this.tasks.length;

    if (failureRate > failureThreshold) {
      throw new Error(
        `Too many task failures (${failures}/${this.tasks.length}). ` +
        `Threshold: ${failureThreshold * 100}%`
      );
    }

    return results
      .filter(r => r.status === 'fulfilled')
      .map(r => r.value);
  }
}
```

---

## 💾 Resource Management

### Connection Pooling for Parallel Tasks

```javascript
class PooledParallelAgent extends ParallelWorkflowAgent {
  constructor(tasks, poolSize = 5) {
    super(tasks);
    this.poolSize = poolSize;
    this.connectionPool = [];
  }

  async executeWithPool(input) {
    console.log(`🔌 Using connection pool of size ${this.poolSize}...`);

    const taskQueue = [...this.tasks];
    const results = [];
    const active = [];

    while (taskQueue.length > 0 || active.length > 0) {
      // Fill up to pool size
      while (active.length < this.poolSize && taskQueue.length > 0) {
        const task = taskQueue.shift();
        const promise = this.executeTask(task, input, results.length)
          .then(result => {
            results.push(result);
            return result;
          });

        active.push(promise);
      }

      // Wait for at least one to complete
      if (active.length > 0) {
        await Promise.race(active);
        // Remove completed promises
        active.splice(
          active.findIndex(p => p.resolved),
          1
        );
      }
    }

    return this.synthesizeResults(results);
  }
}
```

---

## 🎮 Hands-on Exercises

### Exercise 1: Multi-Source Vehicle Information

**Task:** สร้าง agent ที่ดึงข้อมูลรถจาก 5 แหล่ง parallel

```javascript
// Fetch: specs, pricing, reviews, availability, eco-rating
// Combine results into comprehensive vehicle profile
// Present to customer
```

### Exercise 2: Parallel Dealership Check

**Task:** Check inventory ใน dealerships หลายแห่ง พร้อมกัน

```javascript
// Get availability from multiple dealerships
// Check closest locations
// Offer best deals
```

### Exercise 3: Financing Quick Calculator

**Task:** Calculate financing options parallel

```javascript
// Different interest rates
// Various down payments
// Multiple loan terms
// Present all options simultaneously
```

---

## 🚀 Best Practices

### Design for Parallelization
1. **Identify Independent Tasks**: แยก tasks ที่ไม่มี dependency
2. **Resource Limits**: Set connection pool sizes
3. **Error Isolation**: Handle failures individually
4. **Timeout Management**: Set reasonable timeouts

### Performance Optimization
1. **Monitor Pool Size**: ใช้ pool size ที่เหมาะสม
2. **Batch Heavy Tasks**: Group resource-intensive tasks
3. **Cache Results**: Reuse data ระหว่าง tasks
4. **Load Balancing**: Distribute load evenly

### Monitoring & Observability
1. **Track Task Duration**: Monitor execution times
2. **Log Failures**: Log detailed error info
3. **Measure Throughput**: Track tasks per second
4. **Alert on Bottlenecks**: Identify slowdowns

---

## 🎯 Key Takeaways

1. **Parallel Execution** เร่ง response time สำนักงาน significantly
2. **Task Independence** เป็น prerequisite สำหรับ parallelization
3. **Resource Management** สำคัญใน production environments
4. **Error Handling** ต่างกัน sequential workflows
5. **Performance Monitoring** ช่วย optimize strategy

---

## 🚀 Next Steps

ใน lesson ถัดไป เราจะสร้าง **Loop Workflows** สำหรับ iterative processes และการซ้ำงาน

**พร้อมทำระบบ parallel ที่ปลอดภัยแล้วหรือยัง?** ⚡🚀🤖

---

*Code Examples: [exercises/day1/workflows/parallel/](exercises/day1/workflows/parallel/)*
