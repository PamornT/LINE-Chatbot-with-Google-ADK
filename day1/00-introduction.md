# Day 1: Introduction to Multi-Agent Systems

## 🎯 Welcome to Day 1!

ยินดีต้อนรับสู่ **Workshop 2 วัน: Multi-Agent Systems ด้วย Google ADK และ LINE Chatbot Integration**

วันนี้เราจะเริ่มต้นจากการเรียนรู้พื้นฐานของ AI Agents และพัฒนาเป็นระบบ Multi-Agent ที่ทำงานร่วมกันได้อย่างมีประสิทธิภาพ

---

## 📋 Agenda วันนี้

| เวลา | หัวข้อ | วัตถุประสงค์ |
|------|--------|---------------|
| 09:00 - 09:30 | Course Introduction & Setup | ทำความรู้จัก course และตรวจสอบ environment |
| 09:30 - 11:00 | Agent Fundamentals | เข้าใจหลักการทำงานของ AI agents |
| 11:00 - 12:30 | Building First Agent & Tools | สร้าง agent แรกและเพิ่ม capabilities |
| 13:30 - 15:00 | Workflow Patterns | เรียนรู้ sequential, parallel, และ loop workflows |
| 15:00 - 17:00 | Business Agents Development | สร้าง Sales, Finance, และ Operations agents |
| 17:00 - 17:30 | Day 1 Wrap-up | สรุปและ preview วันที่ 2 |

---

## 🎯 Learning Objectives

หลังจากเรียนจบวันนี้ คุณจะสามารถ:

1. **เข้าใจหลักการทำงานของ AI Agents**
   - Agent thinking patterns และ reasoning
   - Role-based agent design
   - Tool integration และ capabilities

2. **พัฒนา Agent พื้นฐาน**
   - สร้าง agent ด้วย Google ADK
   - เพิ่ม tools และ functions
   - จัดการ agent state และ memory

3. **ออกแบบ Workflow Patterns**
   - Sequential workflows สำหรับ step-by-step processes
   - Parallel workflows สำหรับ concurrent tasks
   - Loop workflows สำหรับ iterative operations

4. **สร้าง Business Agents**
   - Sales agent สำหรับ customer interactions
   - Finance agent สำหรับ data analysis
   - Operations agent สำหรับ logistics management

---

## 🏗️ Course Architecture Overview

```
User Request
    ↓
Agent Router/Orchestrator
    ├── Sales Agent (leads, pricing, customer service)
    ├── Finance Agent (analysis, reporting, forecasting)
    └── Operations Agent (inventory, logistics, coordination)
         ↓
    LINE Chatbot Integration (Day 2)
```

วันนี้เราจะ focus ที่การสร้างและทดสอบ agents แต่ละตัว
พรุ่งนี้จะเรียนรู้การรวม agents เข้าด้วยกันและ deploy จริง

---

## 🛠️ Technical Prerequisites

ก่อนเริ่ม workshop ตรวจสอบว่าคุณมี:

- ✅ **GitHub Codespace** พร้อมใช้งาน
- ✅ **Gemini API Key** จาก Google AI Studio
- ✅ **LINE Developer Account** (สำหรับพรุ่งนี้)
- ✅ **Basic JavaScript/Node.js** ความรู้

หากยังไม่มี สามารถดูได้ที่ [00-setup-environment.md](../00-setup-environment.md)

---

## 💡 What is an AI Agent?

### คำนิยาม
**AI Agent** คือโปรแกรมที่สามารถ:
- **รับรู้** สิ่งแวดล้อมและ user inputs
- **ตัดสินใจ** ตาม logic และ data
- **ดำเนินการ** tasks โดยอัตโนมัติ
- **เรียนรู้** และปรับปรุงจากประสบการณ์

### ใน Context ของ Business
- **Sales Agent**: จัดการ leads, ตอบคำถามผลิตภัณฑ์, จอง demo
- **Finance Agent**: วิเคราะห์ข้อมูลการเงิน, สร้าง reports, ทำ forecasting
- **Operations Agent**: จัดการ inventory, coordinate logistics, handle orders

---

## 🔧 Google ADK Overview

**Google Agent Development Kit (ADK)** คือ framework สำหรับสร้าง AI agents ที่:

- **Integrate กับ Gemini AI**: ใช้ Gemini models สำหรับ reasoning และ generation
- **Tool Integration**: เชื่อมต่อกับ APIs, databases, และ external services
- **Workflow Management**: จัดการ complex multi-step processes
- **Memory Systems**: จัดการ context และ conversation history
- **Multi-Agent Coordination**: จัดการ agents ที่ทำงานร่วมกัน

### Basic ADK Structure
```javascript
const { Agent } = require('@google-cloud/adk');

const agent = new Agent({
  name: 'SalesAgent',
  model: 'gemini-pro',
  apiKey: process.env.GEMINI_API_KEY,
  tools: [/* tools array */],
  memory: {/* memory config */}
});
```

---

## 🚀 Hands-on Approach

Workshop นี้เน้น **practical learning**:

- **Code-Along**: Instructor จะอธิบายและเขียน code พร้อมกัน
- **Exercises**: แยกกลุ่มเพื่อ implement features
- **Testing**: ทดสอบ agents ใน real-time
- **Iteration**: ปรับปรุงและ optimize agents

### Development Workflow
1. **Design** → วางแผน agent capabilities
2. **Implement** → เขียน code และ integrate tools
3. **Test** → ทดสอบใน Codespace environment
4. **Deploy** → Run via web interface หรือ API
5. **Iterate** → ปรับปรุงจาก feedback

---

## 📁 Project Structure

วันนี้เราจะทำงานใน folder `day1/`:

```
day1/
├── 00-introduction.md          ← คุณกำลังอ่านไฟล์นี้
├── fundamentals/               ← พื้นฐาน agents
├── workflows/                  ← Workflow patterns
└── agents/                     ← Business agents
```

แต่ละ folder มี exercises และ code examples

---

## 🎮 Getting Started

### 1. Open Your Codespace
```bash
# ใน GitHub repository
# คลิก "Code" → "Codespaces" → "Create codespace"
```

### 2. Set Environment Variables
```bash
# ใน terminal ของ Codespace
export GEMINI_API_KEY="your-api-key-here"
```

### 3. Test Setup
```bash
node --version  # Should show v18+
npm --version   # Should show 9+
echo $GEMINI_API_KEY  # Should show your key
```

### 4. Clone/Navigate to Project
```bash
cd /workspaces/LINE-Chatbot-with-Google-ADK
ls -la  # Check project structure
```

---

## 🤝 Collaboration Guidelines

### ใน Workshop
- **Ask Questions**: ไม่มีคำถามที่ dumb
- **Share Ideas**: เรียนรู้จากกันและกัน
- **Help Others**: ถ้าคุณทำได้แล้ว ช่วยเพื่อน
- **Take Breaks**: พักสายตาและเดินไปเดินมา

### Code Quality
- **Comment Code**: อธิบาย logic ที่ซับซ้อน
- **Use Meaningful Names**: ตัวแปรและฟังก์ชัน
- **Test Frequently**: ตรวจสอบก่อน commit
- **Follow Patterns**: ใช้ conventions ที่ instructor แนะนำ

---

## 📞 Support & Help

### หากมีปัญหา
1. **Check Documentation**: [troubleshooting.md](../troubleshooting.md)
2. **Ask Instructor**: 举手หรือ chat ใน group
3. **Search Issues**: ดู common problems ใน repo
4. **Pair Programming**: ขอความช่วยเหลือจากเพื่อน

### Emergency Contacts
- **Technical Issues**: Instructor Lead
- **API Problems**: Google AI Studio Support
- **Codespace Issues**: GitHub Support

---

## 🎯 Success Criteria

วันนี้ถือว่าสำเร็จเมื่อคุณ:

- ✅ เข้าใจ agent concepts และ ADK framework
- ✅ สร้าง agent แรกที่ทำงานได้
- ✅ Implement workflow pattern อย่างน้อย 1 แบบ
- ✅ พัฒนา business agent อย่างน้อย 1 ประเภท
- ✅ ทดสอบ agents ผ่าน web interface

---

## 🚀 What's Next?

หลังจาก introduction เสร็จ เราจะเริ่มด้วย **Agent Thinking Concepts**

**พร้อมเริ่มต้นการเดินทางสู่โลกของ AI Agents แล้วหรือยัง?** 🤖✨

---

*Last updated: March 6, 2026*