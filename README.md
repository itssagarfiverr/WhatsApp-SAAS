# WhatsApp SaaS Platform
### Powered by Baileys + Node.js + Socket.io

A production-ready, multi-user WhatsApp gateway platform with REST API, webhook support, and real-time QR code authentication.

---

## 📦 Quick Start

### 1. Install Dependencies
```bash
cd whatsapp-saas
npm install
```

### 2. Create Folders
```bash
mkdir -p database/sessions logs public
```

### 3. Run in Development
```bash
npm run dev
# OR
node app.js
```

### 4. Open Browser
```
http://localhost:3000
```

---

## 🚀 Production Deployment (Ubuntu VPS)

### Install Node.js 20+
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### Install PM2 globally
```bash
npm install -g pm2
```

### Configure environment
Edit `ecosystem.config.js` and change `JWT_SECRET` to a secure random string.

### Start with PM2
```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup  # Follow the printed command to auto-start on reboot
```

### Monitor
```bash
pm2 logs whatsapp-saas    # View logs
pm2 status                # Check status
pm2 restart whatsapp-saas # Restart
pm2 stop whatsapp-saas    # Stop
```

---

## 🔌 REST API Reference

All API requests require `api_key` (in body, query param, or `x-api-key` header).

### Check Status
```http
GET /api/status?api_key=YOUR_KEY
```

### Send Message
```http
POST /api/send-message
Content-Type: application/json

{
  "api_key": "YOUR_KEY",
  "number": "919876543210",
  "message": "Hello from WA Cloud!"
}
```

### Send Bulk Messages
```http
POST /api/send-bulk
Content-Type: application/json

{
  "api_key": "YOUR_KEY",
  "numbers": ["919876543210", "918765432109"],
  "message": "Broadcast message",
  "delay": 1500
}
```

### Send Media
```http
POST /api/send-media
Content-Type: multipart/form-data

api_key=YOUR_KEY
number=919876543210
caption=Check this out!
media=<file>
```

### Send Bulk Media
```http
POST /api/send-bulk-media
Content-Type: multipart/form-data

api_key=YOUR_KEY
numbers=["919876543210","918765432109"]
caption=Hello!
media=<file>
```

### Message History
```http
GET /api/messages?api_key=YOUR_KEY&limit=100
```

---

## 🔗 Webhook

Set your webhook URL in the dashboard. On every incoming message, WA Cloud POSTs:

```json
{
  "user_id": "user_abc123",
  "from": "919876543210",
  "message": "Hello!",
  "type": "conversation",
  "timestamp": "2024-01-01T00:00:00.000Z",
  "raw_jid": "919876543210@s.whatsapp.net"
}
```

---

## 📁 Project Structure

```
whatsapp-saas/
├── app.js                    # Entry point
├── ecosystem.config.js       # PM2 config
├── package.json
├── routes/
│   ├── auth.js               # Register, login, profile
│   ├── api.js                # Messaging endpoints
│   └── session.js            # QR, connect, disconnect
├── services/
│   └── baileys.js            # Core WhatsApp logic
├── middleware/
│   └── auth.js               # JWT + API key auth
├── utils/
│   └── database.js           # JSON file DB
├── public/
│   └── index.html            # Frontend SPA
├── database/
│   ├── users.json            # User accounts
│   ├── messages.json         # Message log
│   └── sessions/             # WhatsApp session files
│       └── {user_id}/        # Per-user Baileys auth state
└── logs/                     # PM2 logs
```

---

## 🔐 Security Notes

1. **Change JWT_SECRET** in `ecosystem.config.js` before deploying
2. Use **nginx** as reverse proxy with SSL in production
3. API keys are **per-user** and can be regenerated anytime
4. Passwords are hashed with **bcrypt** (12 rounds)

### Nginx Config (optional)
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 🧪 Testing the API (curl)

```bash
# Register
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"123456","name":"Test User"}'

# Send message (after connecting WhatsApp)
curl -X POST http://localhost:3000/api/send-message \
  -H "Content-Type: application/json" \
  -d '{"api_key":"YOUR_KEY","number":"919876543210","message":"Hello!"}'
```

---

## 📊 Capacity

- Supports **50+ concurrent users** (limited by VPS RAM)
- Message log: last **10,000 messages** per server
- Bulk limit: **500 numbers** per request
- Auto-reconnect with **exponential backoff**
- PM2 **auto-restart** on crash

---

Made with ❤️ using [@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys) by [@itssagarfiverr](https://github.com/itssagarfiverr)
