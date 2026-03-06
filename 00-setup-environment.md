# Setup Environment & Prerequisites

## Overview
ก่อนเริ่ม workshop 2 วันนี้ ผู้เข้าร่วมต้องเตรียม accounts และ API keys ที่จำเป็นสำหรับการพัฒนา Multi-Agent system ด้วย Google ADK และ LINE Chatbot

**หมายเหตุสำคัญ:** Workshop นี้ใช้ **GitHub Codespace** เป็น development environment ดังนั้นไม่จำเป็นต้องติดตั้ง software ใดๆ บนเครื่องของคุณ Codespace จะมีทุกอย่างพร้อมใช้งาน

## Development Environment (GitHub Codespace)

### การใช้งาน GitHub Codespace
1. **เปิด Repository ใน GitHub**
   - เข้าไปที่ repository ของ workshop
   - คลิกปุ่ม **"Code"** (สีเขียว)
   - เลือกแท็บ **"Codespaces"**
   - คลิก **"Create codespace on main"**

2. **รอการสร้าง Environment**
   - Codespace จะสร้าง container พร้อม Node.js, Python, Git และ VS Code
   - ใช้เวลาประมาณ 2-3 นาที

3. **เริ่มทำงาน**
   - VS Code interface จะเปิดขึ้นมาใน browser
   - คุณสามารถเขียน code, run terminal, และ test ได้ทันที

### คุณสมบัติของ Codespace สำหรับ Workshop
- ✅ **Python 3.10+** และ pip พร้อมใช้งาน (สำหรับ Google ADK)
- ✅ Node.js 18+ และ npm พร้อมใช้งาน (สำหรับ LINE webhook)
- ✅ Git พร้อมใช้งาน
- ✅ VS Code extensions ที่จำเป็น
- ✅ Terminal สำหรับ run commands
- ✅ Port forwarding สำหรับ web applications

## Required Accounts & API Keys

### 1. Google AI Studio Account & Gemini API Key
**สำคัญที่สุด:** คุณต้องมี Gemini API Key สำหรับการใช้งาน Google ADK

#### ขั้นตอนการสร้าง API Key:
1. **เข้าสู่ Google AI Studio**
   - ไปที่ [Google AI Studio](https://aistudio.google.com/)
   - Login ด้วย Google Account

2. **สร้าง API Key**
   - คลิก **"Get started"** หรือ **"Create API key"**
   - คลิก **"Create API key in new project"** หรือเลือก existing project
   - คัดลอก API key ที่สร้างขึ้นมา

3. **เก็บ API Key ไว้ให้ปลอดภัย**
   - เก็บไว้ใน password manager หรือ secure note
   - **ห้าม** share หรือ commit API key ไปกับ code

#### การใช้งาน API Key ใน Codespace:
```bash
# ใน terminal ของ Codespace
export GEMINI_API_KEY="your-api-key-here"
echo $GEMINI_API_KEY
```

### 2. LINE Developers Account
สำหรับการ integrate กับ LINE Chatbot ในวันสุดท้าย

#### ขั้นตอนการสมัคร:
1. **เข้าสู่ LINE Developers Console**
   - ไปที่ [LINE Developers](https://developers.line.biz/)
   - Login ด้วย LINE Account หรือ Google Account

2. **สร้าง Provider**
   - คลิก **"Create a new provider"**
   - ใส่ชื่อ Provider (เช่น "Workshop Bot")

3. **สร้าง Messaging API Channel**
   - เลือก **"Messaging API"**
   - ใส่ข้อมูล Channel:
     - Channel name: "Workshop Chatbot"
     - Channel description: "Multi-Agent Chatbot Workshop"
     - Category: เลือกที่เหมาะสม

4. **เก็บ Credentials**
   - **Channel Access Token:** คลิก "Issue" เพื่อสร้าง
   - **Channel Secret:** จะแสดงให้เห็น
   - เก็บไว้ให้ปลอดภัยเช่นเดียวกับ API key

### 3. GitHub Account (ถ้ายังไม่มี)
- สำหรับการใช้งาน Codespace
- สมัครได้ที่ [github.com](https://github.com)

## Environment Setup Checklist

ก่อนเริ่ม workshop วันที่ 1:

- [ ] มี GitHub account และสามารถเข้าถึง repository
- [ ] สร้าง Gemini API Key จาก Google AI Studio
- [ ] ทดสอบการสร้าง Codespace ได้
- [ ] ติดตั้ง Google ADK: `pip install google-adk`
- [ ] สมัคร LINE Developers account และสร้าง Messaging API Channel (สำหรับวันที่ 2)
- [ ] เก็บ API keys และ tokens ไว้ให้พร้อมใช้งาน

## Testing Setup ใน Codespace

หลังจากสร้าง Codespace แล้ว ทดสอบสิ่งเหล่านี้:

```bash
# ตรวจสอบ Python
python3 --version          # ต้อง 3.10+
pip --version

# ติดตั้ง Google ADK
pip install google-adk

# ตรวจสอบ ADK
adk --version

# ตรวจสอบ Git
git --version

# ทดสอบ Gemini API (หลังจาก set environment variable)
export GOOGLE_API_KEY="your-api-key-here"

# สร้าง test agent
adk create test_agent

# รัน test agent
cd test_agent
adk run test_agent
```

หากมีปัญหา ติดต่อ instructors ก่อนเริ่ม workshop

## 🔐 Security Notes

- **API Keys:** อย่า commit หรือ share API keys ใน code
- **Environment Variables:** ใช้ `.env` files และเพิ่ม `.env` ใน `.gitignore`
- **Codespace:** จะถูกลบหลังจาก inactive 30 วัน แต่ข้อมูลสำคัญควร backup

## 📞 Support

หากมีปัญหาในการ setup:
- ตรวจสอบ [Troubleshooting Guide](troubleshooting.md)
- ติดต่อ instructors ทาง email หรือ chat group
- ดู [GitHub Codespace Documentation](https://docs.github.com/en/codespaces)

## Additional Resources
- [Google AI Studio Documentation](https://ai.google.dev/docs)
- [LINE Messaging API Docs](https://developers.line.biz/en/docs/messaging-api/)
- [GitHub Codespace Guide](https://docs.github.com/en/codespaces/getting-started)