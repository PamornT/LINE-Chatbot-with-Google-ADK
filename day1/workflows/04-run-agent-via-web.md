# Running Agents via Web Interface

## 🎯 Objective
สร้าง web interface สำหรับทดสอบและรัน agents แบบ interactive

---

## 🌐 Web Interface Basics

### Why Web Interface?

Web interface ช่วยให้:
- **ทดสอบ** agents ได้ง่ายขึ้น
- **แสดง** results อย่าง user-friendly
- **เก็บ** conversation history
- **เตรียม** สำหรับ production deployment

---

## 🛠️ Setting Up Express Server

### 1. Install Dependencies

```bash
npm install express cors body-parser @google/generative-ai
npm install --save-dev nodemon
```

### 2. Create Server File

```javascript
// server.js
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const { GoogleGenerativeAI } = require('@google/generative-ai');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(bodyParser.json());
app.use(express.static('public')); // For HTML/CSS/JS

// Initialize Gemini
const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);

// Store conversations (in-memory for now)
const conversations = {};

// Generate unique conversation ID
function generateConversationId() {
  return 'conv_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
}

// Initialize conversation
app.post('/api/conversations/new', (req, res) => {
  const conversationId = generateConversationId();
  conversations[conversationId] = {
    messages: [],
    createdAt: new Date(),
    updatedAt: new Date()
  };

  res.json({
    success: true,
    conversationId,
    message: 'Conversation created'
  });
});

// Send message to agent
app.post('/api/conversations/:id/message', async (req, res) => {
  const { id } = req.params;
  const { message, agentRole = 'assistant' } = req.body;

  if (!conversations[id]) {
    return res.status(404).json({
      success: false,
      error: 'Conversation not found'
    });
  }

  try {
    // Build conversation history
    const model = genAI.getGenerativeModel({ model: 'gemini-pro' });

    // Create system prompt based on agent role
    const systemPrompt = getSystemPrompt(agentRole);

    // Get recent messages for context (limit to last 10)
    const recentMessages = conversations[id].messages.slice(-10);
    const history = recentMessages.map(msg => ({
      role: msg.role,
      parts: [{ text: msg.content }]
    }));

    // Start chat session with history
    const chat = model.startChat({ history });

    // Send user message
    const result = await chat.sendMessage(message);
    const agentResponse = await result.response;
    const responseText = agentResponse.text();

    // Store messages in conversation
    conversations[id].messages.push(
      { role: 'user', content: message, timestamp: new Date() },
      { role: 'model', content: responseText, timestamp: new Date() }
    );
    conversations[id].updatedAt = new Date();

    res.json({
      success: true,
      userMessage: message,
      agentResponse: responseText,
      conversationId: id,
      messageCount: conversations[id].messages.length
    });

  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

// Get conversation history
app.get('/api/conversations/:id', (req, res) => {
  const { id } = req.params;

  if (!conversations[id]) {
    return res.status(404).json({
      success: false,
      error: 'Conversation not found'
    });
  }

  res.json({
    success: true,
    conversationId: id,
    messages: conversations[id].messages,
    createdAt: conversations[id].createdAt,
    updatedAt: conversations[id].updatedAt
  });
});

// Get system prompt based on agent role
function getSystemPrompt(role) {
  const prompts = {
    'sales': 'You are a helpful sales representative. Help customers with product inquiries, pricing, and recommendations.',
    'finance': 'You are a finance analyst. Help with financial analysis, reporting, and insights.',
    'operations': 'You are an operations manager. Help with logistics, inventory, and process coordination.',
    'assistant': 'You are a helpful AI assistant. Answer questions and provide information clearly.'
  };

  return prompts[role] || prompts['assistant'];
}

// Health check
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date() });
});

// Start server
app.listen(PORT, () => {
  console.log(`✅ Server running at http://localhost:${PORT}`);
  console.log('API endpoints:');
  console.log('  POST   /api/conversations/new');
  console.log('  POST   /api/conversations/:id/message');
  console.log('  GET    /api/conversations/:id');
  console.log('  GET    /api/health');
});
```

---

## 🎨 Create HTML Interface

### Create Public Folder Structure

```bash
mkdir -p public
touch public/index.html
touch public/style.css
touch public/app.js
```

### HTML Interface

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Agent Web Interface</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <header>
      <h1>🤖 AI Agent Web Interface</h1>
      <p>Interactive Testing Environment for Google ADK Agents</p>
    </header>

    <div class="controls">
      <select id="agentRole" class="role-select">
        <option value="assistant">Assistant</option>
        <option value="sales">Sales Agent</option>
        <option value="finance">Finance Agent</option>
        <option value="operations">Operations Agent</option>
      </select>
      <button id="newConversation" class="btn btn-primary">New Conversation</button>
      <span id="convId" class="conversation-id"></span>
    </div>

    <div class="chat-container">
      <div id="chatMessages" class="messages"></div>
    </div>

    <div class="input-area">
      <input
        type="text"
        id="userInput"
        placeholder="Type your message here..."
        class="input-field"
      >
      <button id="sendButton" class="btn btn-send">Send</button>
    </div>

    <footer>
      <p>Connected to: <span id="status" class="status-indicator"></span></p>
    </footer>
  </div>

  <script src="app.js"></script>
</body>
</html>
```

### CSS Styling

```css
/* public/style.css */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  width: 100%;
  max-width: 800px;
  height: 90vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
  text-align: center;
  border-bottom: 2px solid rgba(0, 0, 0, 0.1);
}

header h1 {
  font-size: 24px;
  margin-bottom: 8px;
}

header p {
  font-size: 14px;
  opacity: 0.9;
}

.controls {
  padding: 15px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
  display: flex;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap;
}

.role-select {
  padding: 8px 12px;
  border: 1px solid #dee2e6;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
}

.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-primary {
  background: #667eea;
  color: white;
}

.btn-primary:hover {
  background: #5568d3;
  transform: translateY(-2px);
}

.conversation-id {
  font-size: 12px;
  color: #666;
  font-family: monospace;
  padding: 0 10px;
}

.chat-container {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  background: white;
}

.messages {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.message {
  display: flex;
  gap: 10px;
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message.user {
  justify-content: flex-end;
}

.message-content {
  max-width: 70%;
  padding: 12px 16px;
  border-radius: 8px;
  word-wrap: break-word;
  font-size: 14px;
}

.message.user .message-content {
  background: #667eea;
  color: white;
  border-radius: 8px 2px 8px 8px;
}

.message.agent .message-content {
  background: #e9ecef;
  color: #333;
  border-radius: 2px 8px 8px 8px;
}

.message-time {
  font-size: 12px;
  color: #aaa;
  align-self: flex-end;
  margin-top: 4px;
}

.input-area {
  padding: 15px 20px;
  background: #f8f9fa;
  border-top: 1px solid #e9ecef;
  display: flex;
  gap: 10px;
}

.input-field {
  flex: 1;
  padding: 10px 16px;
  border: 1px solid #dee2e6;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.3s ease;
}

.input-field:focus {
  outline: none;
  border-color: #667eea;
}

.btn-send {
  background: #667eea;
  color: white;
  padding: 10px 24px;
}

.btn-send:hover {
  background: #5568d3;
}

.btn-send:active {
  transform: scale(0.98);
}

footer {
  padding: 10px 20px;
  background: #f8f9fa;
  border-top: 1px solid #e9ecef;
  text-align: center;
  font-size: 12px;
  color: #666;
}

.status-indicator {
  display: inline-block;
  width: 8px;
  height: 8px;
  background: #28a745;
  border-radius: 50%;
  margin-left: 4px;
}

.loading {
  display: inline-block;
  width: 8px;
  height: 8px;
  background: #ffc107;
  border-radius: 50%;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
```

### JavaScript Frontend

```javascript
// public/app.js
class AgentWebUI {
  constructor() {
    this.conversationId = null;
    this.agentRole = 'assistant';
    this.setupEventListeners();
    this.checkHealth();
  }

  setupEventListeners() {
    document.getElementById('newConversation').addEventListener('click', () => this.newConversation());
    document.getElementById('sendButton').addEventListener('click', () => this.sendMessage());
    document.getElementById('userInput').addEventListener('keypress', (e) => {
      if (e.key === 'Enter') this.sendMessage();
    });
    document.getElementById('agentRole').addEventListener('change', (e) => {
      this.agentRole = e.target.value;
    });
  }
