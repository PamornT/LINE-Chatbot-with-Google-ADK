# LINE Chatbot with Google ADK

## 🚀 Course: Multi-Agent Systems with LINE Integration

A comprehensive 2-day workshop teaching how to build intelligent multi-agent systems using Google Agent Development Kit (ADK) and integrate them with LINE Chatbot platform.

## 📋 Course Overview

**Duration:** 2 Days (16 hours total)
**Level:** Intermediate to Advanced
**Prerequisites:** Basic JavaScript/Node.js, REST APIs
**Outcome:** Complete multi-agent LINE chatbot system

### 🎯 Learning Objectives
- Master Google ADK for agent development
- Design and implement multi-agent architectures
- Build production-ready LINE chatbot integrations
- Apply agent patterns to real business scenarios

## 📚 Course Structure

### Day 1: Agent Fundamentals & Individual Agents
```
day1/
├── 00-introduction.md
├── fundamentals/
│   ├── 01-agent-thinking.md
│   ├── 02-first-agent.md
│   └── 03-first-tool.md
├── workflows/
│   ├── 04-run-agent-via-web.md
│   ├── 05-workflow-sequential-concept.md
│   ├── 08-workflow-parallel-concept.md
│   └── 10-workflow-loop-concept.md
└── agents/
    ├── 06-build-sales-agent.md
    ├── 07-add-state-to-sales-agent.md
    ├── 09-build-finance-agent.md
    └── 11-build-operations-agent.md
```
**Focus:** Single agent development, workflow patterns, business agent implementation

### Day 2: Memory Systems & Multi-Agent Orchestration
```
day2/
├── memory/
│   ├── 12-session-memory.md
│   └── 13-persistent-memory.md
├── multiagent/
│   ├── 14-custom-agent-concept.md
│   ├── 15-build-manager-agent.md
│   ├── 16-multi-agent-concept.md
│   └── 17-build-company-orchestrator.md
└── deployment/
    ├── 18-run-agent-via-api-server.md
    └── 19-connect-line-webhook.md
```
**Focus:** Memory management, multi-agent coordination, production deployment

### 📖 Additional Materials
- `00-setup-environment.md` - Prerequisites and setup guide
- `agenda.md` - Detailed course schedule
- `day1-wrap-up.md` & `day2-wrap-up.md` - Daily summaries
- `99-final-challenge.md` - Capstone project
- `resources/references.md` - Additional learning materials
- `troubleshooting.md` - Common issues and solutions

## 🛠️ Technology Stack

- **Google ADK** - Agent development framework
- **LINE Messaging API** - Chatbot platform
- **Node.js** - Runtime environment
- **Express.js** - Web framework
- **Database** - Memory persistence (PostgreSQL/Redis optional)

## 🚀 Quick Start

### Prerequisites
1. Complete [setup guide](00-setup-environment.md)
2. Install dependencies:
   ```bash
   npm install
   ```

### Basic Agent Example
```javascript
const { Agent } = require('@google-cloud/adk');

const agent = new Agent({
  name: 'SalesAgent',
  description: 'Handles sales inquiries and lead generation'
});

// Add tools and capabilities
agent.addTool(salesTool);
agent.addMemory(sessionMemory);

await agent.initialize();
```

### LINE Integration
```javascript
const line = require('@line/bot-sdk');

const config = {
  channelAccessToken: process.env.LINE_CHANNEL_ACCESS_TOKEN,
  channelSecret: process.env.LINE_CHANNEL_SECRET
};

const client = new line.Client(config);
```

## 📁 Project Structure

```
LINE-Chatbot-with-Google-ADK/
├── 00-setup-environment.md      # Prerequisites
├── agenda.md                     # Course schedule
├── README.md                     # This file
├── troubleshooting.md            # Common issues
├── day1/                         # Day 1 materials
├── day2/                         # Day 2 materials
├── 99-final-challenge.md         # Capstone project
├── resources/                    # Additional resources
│   └── references.md
├── code-examples/                # Code samples
├── assets/                       # Slides and diagrams
│   └── slides/
└── day1-wrap-up.md              # Day 1 summary
```

## 🎓 Course Outcomes

By the end of this workshop, participants will be able to:

- ✅ Design multi-agent system architectures
- ✅ Implement agents with Google ADK
- ✅ Manage agent memory and state
- ✅ Deploy agents to production environments
- ✅ Integrate with LINE chatbot platform
- ✅ Build complete end-to-end chatbot solutions

## 👥 Target Audience

- **Developers** interested in AI agent development
- **Solution Architects** designing conversational systems
- **Product Managers** planning AI-powered applications
- **Entrepreneurs** building chatbot businesses

## 📞 Support & Resources

- **Documentation:** [Google ADK Docs](https://cloud.google.com/agent-development-kit)
- **LINE Developers:** [Messaging API](https://developers.line.biz/en/docs/messaging-api/)
- **Community:** [GitHub Issues](https://github.com/your-repo/issues)
- **Support:** [Contact Information]

## 📄 License

This course material is provided for educational purposes.

## 🤝 Contributing

Contributions to improve the course materials are welcome! Please see our [contributing guidelines](CONTRIBUTING.md).

---

**Ready to build intelligent multi-agent systems? Let's get started!** 🚀