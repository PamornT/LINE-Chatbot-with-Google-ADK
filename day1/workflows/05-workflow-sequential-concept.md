# Sequential Workflow Concepts

## 🎯 Objective
เข้าใจและ implement sequential workflows ใน Google ADK สำหรับ tasks ที่ต้องทำตามลำดับ

---

## 🔄 What is Sequential Workflow?

**Sequential Workflow** คือการทำงานที่แต่ละ step ต้องทำเสร็จก่อน จึงจะไป step ถัดไป

```
Step 1 → Step 2 → Step 3 → Complete
```

### เมื่อไหร่ใช้ Sequential?
- **Dependencies**: Step ถัดไปต้องใช้ผลจาก step ก่อน
- **Order Matters**: ลำดับสำคัญต่อผลลัพธ์
- **Validation**: แต่ละ step ต้อง validate ก่อนไปต่อ

---

## 🏗️ Sequential Workflow in ADK

### Basic Sequential Structure

```javascript
class SequentialWorkflowAgent extends BasicAgent {
  constructor(steps) {
    super('SequentialWorkflowAgent');
    this.steps = steps; // Array of step functions
    this.currentStep = 0;
    this.results = [];
  }

  async executeWorkflow(input) {
    console.log('🚀 Starting sequential workflow...');

    for (let i = 0; i < this.steps.length; i++) {
      console.log(`📋 Executing step ${i + 1}/${this.steps.length}`);

      try {
        const stepResult = await this.steps[i](input, this.results);
        this.results.push(stepResult);

        // Check if workflow should continue
        if (stepResult.stopWorkflow) {
          console.log('🛑 Workflow stopped by step decision');
          break;
        }

      } catch (error) {
        console.error(`❌ Step ${i + 1} failed:`, error);
        throw error;
      }
    }

    return this.synthesizeResults();
  }

  synthesizeResults() {
    const finalPrompt = `Workflow completed with ${this.results.length} steps.
Results: ${JSON.stringify(this.results, null, 2)}

Please provide a comprehensive summary of the workflow execution.`;

    return this.think(finalPrompt);
  }
}
```

### Example: Customer Onboarding Workflow

```javascript
// Define workflow steps
const onboardingSteps = [
  // Step 1: Validate customer information
  async (input, previousResults) => {
    console.log('1️⃣ Validating customer info...');
    const validation = await validateCustomerData(input.customerData);
    return {
      step: 'validation',
      valid: validation.isValid,
      issues: validation.issues,
      stopWorkflow: !validation.isValid
    };
  },

  // Step 2: Create customer account
  async (input, previousResults) => {
    console.log('2️⃣ Creating customer account...');
    const validation = previousResults[0];
    if (!validation.valid) {
      throw new Error('Cannot create account: validation failed');
    }

    const account = await createCustomerAccount(input.customerData);
    return {
      step: 'account_creation',
      accountId: account.id,
      email: account.email
    };
  },

  // Step 3: Send welcome email
  async (input, previousResults) => {
    console.log('3️⃣ Sending welcome email...');
    const account = previousResults[1];
    const emailResult = await sendWelcomeEmail(account.email, account.accountId);
    return {
      step: 'welcome_email',
      emailSent: emailResult.success,
      emailId: emailResult.id
    };
  },

  // Step 4: Setup initial preferences
  async (input, previousResults) => {
    console.log('4️⃣ Setting up preferences...');
    const account = previousResults[1];
    const preferences = await setupDefaultPreferences(account.accountId);
    return {
      step: 'preferences_setup',
      preferences: preferences.settings
    };
  }
];

// Create workflow agent
const onboardingAgent = new SequentialWorkflowAgent(onboardingSteps);

// Execute workflow
const result = await onboardingAgent.executeWorkflow({
  customerData: {
    name: 'John Doe',
    email: 'john@example.com',
    company: 'ABC Corp'
  }
});

console.log('✅ Onboarding completed:', result);
```

---

## 🎯 Practical Examples

### Sales Process Workflow

```javascript
const salesWorkflowSteps = [
  // 1. Qualify lead
  async (input) => {
    const qualification = await qualifyLead(input.leadData);
    return {
      step: 'qualification',
      qualified: qualification.score > 70,
      score: qualification.score,
      stopWorkflow: qualification.score < 30
    };
  },

  // 2. Research company
  async (input, results) => {
    if (!results[0].qualified) return { step: 'skipped', reason: 'not qualified' };

    const research = await researchCompany(input.leadData.company);
    return {
      step: 'research',
      companyInfo: research,
      insights: research.keyInsights
    };
  },

  // 3. Prepare proposal
  async (input, results) => {
    const research = results[1];
    const proposal = await generateProposal(input.leadData, research.companyInfo);
    return {
      step: 'proposal',
      proposalId: proposal.id,
      value: proposal.value
    };
  },

  // 4. Schedule demo
  async (input, results) => {
    const proposal = results[2];
    const demo = await scheduleDemo(input.leadData, proposal.proposalId);
    return {
      step: 'demo_scheduled',
      demoId: demo.id,
      dateTime: demo.scheduledTime
    };
  }
];
```

### Content Creation Workflow

```javascript
const contentWorkflowSteps = [
  // 1. Research topic
  async (input) => {
    const research = await researchTopic(input.topic);
    return {
      step: 'research',
      sources: research.sources,
      keyPoints: research.keyPoints
    };
  },

  // 2. Generate outline
  async (input, results) => {
    const research = results[0];
    const outline = await generateOutline(input.topic, research.keyPoints);
    return {
      step: 'outline',
      sections: outline.sections,
      wordCount: outline.estimatedWords
    };
  },

  // 3. Write content
  async (input, results) => {
    const outline = results[1];
    const content = await writeContent(input.topic, outline.sections);
    return {
      step: 'writing',
      content: content.text,
      wordCount: content.wordCount
    };
  },

  // 4. Edit and proofread
  async (input, results) => {
    const content = results[2];
    const edited = await editContent(content.content);
    return {
      step: 'editing',
      finalContent: edited.text,
      edits: edited.changes
    };
  }
];
```

---

## 🔄 Error Handling & Recovery

### Handling Step Failures

```javascript
class ResilientSequentialAgent extends SequentialWorkflowAgent {
  async executeWorkflow(input) {
    const maxRetries = 3;

    for (let stepIndex = 0; stepIndex < this.steps.length; stepIndex++) {
      let attempt = 0;
      let success = false;

      while (attempt < maxRetries && !success) {
        try {
          console.log(`📋 Step ${stepIndex + 1}, attempt ${attempt + 1}`);

          const result = await this.steps[stepIndex](input, this.results);
          this.results.push(result);
          success = true;

          if (result.stopWorkflow) break;

        } catch (error) {
          attempt++;
          console.error(`❌ Step ${stepIndex + 1} attempt ${attempt} failed:`, error);

          if (attempt >= maxRetries) {
            // Try recovery strategy
            const recovery = await this.attemptRecovery(stepIndex, error);
            if (!recovery.success) {
              throw new Error(`Step ${stepIndex + 1} failed after ${maxRetries} attempts`);
            }
            this.results.push(recovery.result);
            success = true;
          }
        }
      }

      if (!success) break;
    }

    return this.synthesizeResults();
  }

  async attemptRecovery(stepIndex, error) {
    const recoveryPrompt = `Step ${stepIndex + 1} failed with error: ${error.message}

Previous results: ${JSON.stringify(this.results)}

Suggest a recovery strategy or alternative approach.`;

    const recoveryPlan = await this.think(recoveryPrompt);

    // Implement recovery logic based on AI suggestion
    return {
      success: true,
      result: {
        step: `recovery_${stepIndex}`,
        originalError: error.message,
        recoveryAction: recoveryPlan
      }
    };
  }
}
```

---

## 📊 Monitoring & Logging

### Workflow Metrics

```javascript
class MonitoredSequentialAgent extends SequentialWorkflowAgent {
  constructor(steps) {
    super(steps);
    this.metrics = {
      startTime: null,
      endTime: null,
      stepDurations: [],
      totalSteps: steps.length,
      completedSteps: 0,
      failedSteps: 0
    };
  }

  async executeWorkflow(input) {
    this.metrics.startTime = Date.now();
    console.log('🚀 Starting monitored workflow...');

    try {
      const result = await super.executeWorkflow(input);
      this.metrics.endTime = Date.now();
      this.logMetrics();
      return result;
    } catch (error) {
      this.metrics.endTime = Date.now();
      this.metrics.failedSteps++;
      this.logMetrics();
      throw error;
    }
  }

  async executeStep(stepFunction, stepIndex, input) {
    const stepStart = Date.now();

    try {
      const result = await stepFunction(input, this.results);
      const duration = Date.now() - stepStart;

      this.metrics.stepDurations.push(duration);
      this.metrics.completedSteps++;

      console.log(`✅ Step ${stepIndex + 1} completed in ${duration}ms`);

      return result;
    } catch (error) {
      const duration = Date.now() - stepStart;
      this.metrics.stepDurations.push(duration);
      this.metrics.failedSteps++;

      console.log(`❌ Step ${stepIndex + 1} failed in ${duration}ms`);
      throw error;
    }
  }

  logMetrics() {
    const totalDuration = this.metrics.endTime - this.metrics.startTime;
    const avgStepDuration = this.metrics.stepDurations.reduce((a, b) => a + b, 0) / this.metrics.stepDurations.length;

    console.log('📊 Workflow Metrics:');
    console.log(`   Total duration: ${totalDuration}ms`);
    console.log(`   Steps completed: ${this.metrics.completedSteps}/${this.metrics.totalSteps}`);
    console.log(`   Steps failed: ${this.metrics.failedSteps}`);
    console.log(`   Average step duration: ${avgStepDuration.toFixed(2)}ms`);
  }
}
```

---

## 🎮 Hands-on Exercises

### Exercise 1: Order Processing Workflow

**Task:** สร้าง workflow สำหรับ process order

**Steps:**
1. Validate order
2. Check inventory
3. Process payment
4. Update shipping
5. Send confirmation

### Exercise 2: Interview Process Workflow

**Task:** สร้าง workflow สำหรับ hiring process

**Steps:**
1. Resume screening
2. Phone interview
3. Technical assessment
4. Final interview
5. Offer letter

### Exercise 3: Content Publishing Workflow

**Task:** สร้าง workflow สำหรับ publish article

**Steps:**
1. Topic research
2. Content writing
3. Editing
4. SEO optimization
5. Publishing

---

## 🚀 Best Practices

### Design Principles
1. **Keep Steps Focused**: แต่ละ step ทำหน้าที่เดียว
2. **Handle Dependencies**: จัดการ data flow ระหว่าง steps
3. **Add Validation**: Validate inputs/outputs ของแต่ละ step
4. **Plan for Failure**: มี recovery strategies

### Performance Optimization
1. **Parallelize When Possible**: ใช้ parallel สำหรับ independent steps
2. **Cache Results**: Cache expensive operations
3. **Async Operations**: ใช้ async/await อย่างถูกต้อง
4. **Resource Management**: จัดการ memory และ connections

### Monitoring & Debugging
1. **Log Step Progress**: Track execution progress
2. **Measure Performance**: Monitor step durations
3. **Error Context**: Log detailed error information
4. **Retry Logic**: Implement intelligent retries

---

## 🎯 Key Takeaways

1. **Sequential Workflows** เหมาะสำหรับ dependent processes
2. **Step Isolation** ช่วย debugging และ maintenance
3. **Error Recovery** สำคัญสำหรับ production workflows
4. **Monitoring** ช่วย optimize performance
5. **Flexibility** สามารถ adapt ตาม business needs

---

## 🚀 Next Steps

ใน lesson ถัดไป เราจะเรียนรู้ **Parallel Workflows** สำหรับ tasks ที่ทำงานพร้อมกันได้

**พร้อมทำให้ workflow ของคุณเร็วขึ้นแล้วหรือยัง?** ⚡🔄🤖

---

*Code Examples: [exercises/day1/workflows/sequential/](exercises/day1/workflows/sequential/)*