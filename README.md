# backendfile
# AYARSHI — Service Booking Platform

A full-stack web application with real-time chat (Socket.io), Razorpay & Stripe payments, and JWT authentication.

---

## Project Structure

```
ayarshi/
├── backend/          # Node.js + Express + Socket.io API
│   ├── src/
│   │   ├── app.js              # Entry point
│   │   ├── models/             # Mongoose models (User, Booking, Message, Payment)
│   │   ├── routes/             # REST API routes
│   │   ├── services/           # Payment service
│   │   ├── socket/             # Socket.io handler
│   │   └── middleware/         # Auth, error handler
│   ├── Dockerfile
│   └── package.json
│
├── frontend/         # React app
│   ├── src/
│   │   ├── App.jsx             # Router
│   │   ├── pages/              # Login, Register, Dashboard, Chat, Bookings, Payment
│   │   ├── components/         # Sidebar, ChatWindow
│   │   ├── hooks/              # useAuth, useChat
│   │   ├── api/                # Axios instance
│   │   └── styles/             # global.css (design system)
│   ├── Dockerfile
│   └── package.json
│
└── docker-compose.yml
```

---

## Quick Start (Local Development)

### Prerequisites
- Node.js 18+
- MongoDB (local or Atlas)
- npm or yarn

### 1. Backend

```bash
cd backend
cp .env.example .env
# Edit .env with your credentials

npm install
npm run dev       # starts on http://localhost:5000
```

### 2. Frontend

```bash
cd frontend
cp .env.example .env
# Edit .env:
#   REACT_APP_API_URL=http://localhost:5000

npm install
npm start         # starts on http://localhost:3000
```

---

## Docker Deployment (Recommended)

### 1. Create a root `.env` file

```bash
cp .env.example .env
```

Fill in:
```env
JWT_SECRET=your_very_long_secret_key

RAZORPAY_KEY_ID=rzp_live_xxxx
RAZORPAY_KEY_SECRET=xxxx
RAZORPAY_WEBHOOK_SECRET=xxxx

STRIPE_SECRET_KEY=sk_live_xxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxx

REACT_APP_API_URL=https://api.yourdomain.com
REACT_APP_STRIPE_PUBLIC_KEY=pk_live_xxxx
FRONTEND_URL=https://yourdomain.com
```

### 2. Build and run

```bash
docker-compose up --build -d
```

Services:
| Service   | URL                     |
|-----------|-------------------------|
| Frontend  | http://localhost:3000   |
| Backend   | http://localhost:5000   |
| MongoDB   | mongodb://localhost:27017|

---

## Deploy to Cloud

### Option A: Railway (Easiest)

1. Push to GitHub
2. Go to [railway.app](https://railway.app) → New Project → Deploy from GitHub
3. Add MongoDB plugin
4. Set environment variables in the Railway dashboard
5. Deploy backend and frontend as separate services

### Option B: Render

1. **Backend**: New Web Service → select `backend/` → Build: `npm install`, Start: `node src/app.js`
2. **Frontend**: New Static Site → select `frontend/` → Build: `npm run build`, Publish: `build/`
3. Add a MongoDB Atlas connection string

### Option C: VPS (Ubuntu)

```bash
# Install Node, MongoDB
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs mongodb

# Clone your project
git clone <your-repo> /var/www/ayarshi
cd /var/www/ayarshi

# Backend
cd backend && npm install --production
cp .env.example .env  # fill credentials
node src/app.js &

# Frontend
cd ../frontend && npm install && npm run build
# Serve build/ with nginx (see below)
```

Nginx config for frontend + backend proxy:
```nginx
server {
  listen 80;
  server_name yourdomain.com;
  root /var/www/ayarshi/frontend/build;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;
  }

  location /api/ {
    proxy_pass http://localhost:5000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
  }

  location /socket.io/ {
    proxy_pass http://localhost:5000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```

---

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/auth/register` | Register user |
| POST | `/api/v1/auth/login` | Login |
| GET | `/api/v1/auth/me` | Get current user |
| GET | `/api/v1/bookings` | List bookings |
| POST | `/api/v1/bookings` | Create booking |
| PATCH | `/api/v1/bookings/:id/status` | Update status |
| GET | `/api/v1/chat/conversations` | List conversations |
| GET | `/api/v1/chat/:bookingId` | Get messages |
| POST | `/api/v1/payments/create-order` | Razorpay order |
| POST | `/api/v1/payments/verify` | Razorpay verify |
| POST | `/api/v1/payments/create-payment-intent` | Stripe intent |
| POST | `/api/v1/payments/confirm-stripe` | Stripe confirm |

---

## Payment Test Cards

**Razorpay:**
- Success: `4111 1111 1111 1111` | CVV: any | Date: any future
- Fail: `4111 1111 1111 1112`

**Stripe:**
- Success: `4242 4242 4242 4242` | CVV: any | Date: any future

---

## Socket Events

**Client → Server:**
- `join_chat` `{ bookingId }`
- `send_message` `{ bookingId, messageText }`
- `read_message` `{ messageId, bookingId }`
- `typing` / `stop_typing` `{ bookingId }`

**Server → Client:**
- `connection_established`
- `chat_history` `{ messages }`
- `new_message`
- `message_read`
- `user_typing` / `user_stop_typing`

