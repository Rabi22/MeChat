# 💬 ChatApp — Microservices Real-Time Chat

> **OTP-authenticated · Real-time · Microservice architecture · Deployed on AWS**

[![Live Demo](https://img.shields.io/badge/🚀_Live_Demo-AWS_EC2-orange?style=for-the-badge)](http://43.204.149.4:3000/login)
[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Next.js](https://img.shields.io/badge/Next.js-15-black?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org)
[![Socket.io](https://img.shields.io/badge/Socket.io-4.x-010101?style=for-the-badge&logo=socket.io)](https://socket.io)
[![MongoDB](https://img.shields.io/badge/MongoDB-8.x-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://mongodb.com)
[![Redis](https://img.shields.io/badge/Redis-5.x-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io)
[![RabbitMQ](https://img.shields.io/badge/RabbitMQ-0.10-FF6600?style=for-the-badge&logo=rabbitmq&logoColor=white)](https://rabbitmq.com)

---

## ✨ What is this?

A production-grade real-time chat application built on a **microservices architecture** — three independent backend services communicating via **RabbitMQ** message queues, a **Redis**-backed OTP system, **Socket.io** for live messaging, and **Cloudinary** for image delivery. Auth is passwordless — email OTP only, no passwords stored.

```
📡 User hits AWS EC2 → Next.js frontend → REST APIs → Services talk via RabbitMQ
                                        ↘ Socket.io → Real-time messages
```

---

## 🌐 Live Deployment

> Deployed on **AWS EC2** (Mumbai region)

| Surface | URL |
|---|---|
| **Frontend** | [http://43.204.149.4:3000/login](http://43.204.149.4:3000/login) |
| **User Service** | `http://43.204.149.4:5000` |
| **Chat Service** | `http://43.204.149.4:5002` |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                   AWS EC2 Instance                  │
│                                                     │
│  ┌─────────────┐    ┌────────────┐   ┌───────────┐  │
│  │  Next.js 15 │    │    User    │   │   Chat    │  │
│  │  Frontend   │◄──►│  Service  │   │  Service  │  │
│  │  :3000      │    │  :5000    │   │  :5002    │  │
│  └─────────────┘    └─────┬─────┘   └─────┬─────┘  │
│                           │               │         │
│                     ┌─────▼───────────────▼─────┐   │
│                     │        RabbitMQ            │   │
│                     │   (send-otp queue)         │   │
│                     └───────────┬───────────────┘   │
│                                 │                   │
│                           ┌─────▼─────┐             │
│                           │   Mail    │             │
│                           │  Service  │             │
│                           │ (nodemailer)            │
│                           └───────────┘             │
│                                                     │
│         Redis (OTP + Rate Limiting)                 │
│         MongoDB (Users + Chats + Messages)          │
│         Cloudinary (Image Storage + CDN)            │
└─────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
chatapp/
│
├── backend/
│   │
│   ├── user/                          # 🧑 User Service (Port 5000)
│   │   └── src/
│   │       ├── config/
│   │       │   ├── db.js              # MongoDB connection
│   │       │   ├── generateToken.js   # JWT token generation
│   │       │   ├── rabbitmq.js        # RabbitMQ producer
│   │       │   └── TryCatch.js        # Async error wrapper
│   │       ├── controllers/
│   │       │   └── user.js            # Login, verify OTP, profile, update
│   │       ├── middleware/
│   │       │   └── isAuth.js          # JWT auth middleware
│   │       ├── model/
│   │       │   └── User.js            # User schema
│   │       ├── routes/
│   │       │   └── user.js            # User routes
│   │       └── index.js               # Entry point + Redis client
│   │
│   ├── chat/                          # 💬 Chat Service (Port 5002)
│   │   └── src/
│   │       ├── config/
│   │       │   ├── cloudinary.js      # Cloudinary config
│   │       │   ├── db.js              # MongoDB connection
│   │       │   ├── socket.js          # Socket.io server setup
│   │       │   └── TryCatch.js        # Async error wrapper
│   │       ├── controllers/
│   │       │   └── chat.js            # Create chat, send/get messages
│   │       ├── middlewares/
│   │       │   ├── isAuth.js          # JWT auth middleware
│   │       │   └── multer.js          # Cloudinary image upload
│   │       ├── models/
│   │       │   ├── Chat.js            # Chat schema
│   │       │   └── Messages.js        # Message schema (text + image)
│   │       ├── routes/
│   │       │   └── chat.js            # Chat + message routes
│   │       └── index.js               # Entry point
│   │
│   └── mail/                          # 📧 Mail Service (RabbitMQ Consumer)
│       └── src/
│           ├── consumer.js            # RabbitMQ consumer + nodemailer
│           └── index.js               # Entry point
│
└── frontend/                          # ⚛️ Next.js 15 App (Port 3000)
    └── src/
        ├── app/
        │   ├── chat/page.jsx          # Main chat UI
        │   ├── login/page.jsx         # Email input page
        │   ├── verify/page.jsx        # OTP verification page
        │   ├── profile/page.jsx       # Profile edit page
        │   ├── layout.jsx             # Root layout + providers
        │   └── globals.css            # Tailwind base
        ├── components/
        │   ├── ChatHeader.jsx         # User header + typing indicator
        │   ├── ChatMessages.jsx       # Message list + seen receipts
        │   ├── ChatSidebar.jsx        # Conversation list + user search
        │   ├── MessageInput.jsx       # Text + image input
        │   ├── VerifyOtp.jsx          # OTP input (6-digit boxes)
        │   └── Loading.jsx            # Full-screen spinner
        └── context/
            ├── AppContext.jsx         # Global state (user, chats, auth)
            └── SocketContext.jsx      # Socket.io connection manager
```

---

## ⚡ Features

### Auth Flow
- **Passwordless OTP login** via email (6-digit code)
- JWT tokens (15-day expiry)
- **Rate limiting** on OTP requests via Redis (60s cooldown)
- OTP expiry enforced at 5 minutes in Redis

### Real-Time Chat
- **Live messaging** via Socket.io WebSockets
- **Typing indicators** with auto-clear on 2s idle
- **Online/offline presence** broadcast to all connected clients
- **Message seen receipts** — single ✓ (sent) → double ✓ (seen) with timestamp
- **Unseen message count** badges per conversation

### Media
- **Image sharing** in chats — Cloudinary upload with auto-resize (800×600 max)
- Image preview before sending with caption support
- 5MB file size limit, images-only enforced

### UX
- Conversations sorted by latest message (real-time reorder)
- User search when starting new chats
- Profile name editing
- Fully responsive — sidebar collapses on mobile

---

## 🔌 API Reference

### User Service — `PORT 5000`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/v1/login` | ❌ | Send OTP to email |
| `POST` | `/api/v1/verify` | ❌ | Verify OTP, get JWT |
| `GET` | `/api/v1/me` | ✅ | Get logged-in user |
| `GET` | `/api/v1/user/all` | ✅ | Get all users |
| `GET` | `/api/v1/user/:id` | ❌ | Get user by ID |
| `POST` | `/api/v1/update/user` | ✅ | Update display name |

### Chat Service — `PORT 5002`

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/v1/chat/new` | ✅ | Create or get existing chat |
| `GET` | `/api/v1/chat/all` | ✅ | Get all user's chats |
| `POST` | `/api/v1/message` | ✅ | Send message (text/image) |
| `GET` | `/api/v1/message/:chatId` | ✅ | Get messages + mark seen |

### Socket.io Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `newMessage` | Server → Client | Message object |
| `messagesSeen` | Server → Client | `{ chatId, seenBy, messageIds }` |
| `getOnlineUser` | Server → Client | Array of user IDs |
| `userTyping` | Client → Server | `{ chatId, userId }` |
| `userStoppedTyping` | Client → Server | `{ chatId, userId }` |
| `joinChat` | Client → Server | `chatId` |
| `leaveChat` | Client → Server | `chatId` |

---

## 🛠️ Tech Stack

| Layer | Tech |
|---|---|
| **Frontend** | Next.js 15, React 19, Tailwind CSS 4, Socket.io-client |
| **User Service** | Express 5, Mongoose, JWT, Redis, amqplib |
| **Chat Service** | Express 5, Mongoose, Socket.io 4, Multer, Cloudinary |
| **Mail Service** | Express 5, amqplib, Nodemailer |
| **Database** | MongoDB Atlas (shared `Chatappmicroserviceapp` DB) |
| **Cache / OTP** | Redis (OTP storage + rate limiting) |
| **Queue** | RabbitMQ (`send-otp` queue, durable) |
| **Media** | Cloudinary (upload + CDN + transformation) |
| **Deployment** | AWS EC2 (Ubuntu) |

---

## 🚀 Local Setup

### Prerequisites

- Node.js 18+
- MongoDB URI
- Redis URL
- RabbitMQ credentials
- Cloudinary account
- Gmail app password (for nodemailer)

### 1. Clone

```bash
git clone https://github.com/Rabi22/chatapp.git
cd chatapp
```

### 2. Environment Variables

**`backend/user/.env`**
```env
PORT=5000
MONGO_URI=your_mongodb_uri
JWT_SECRET=your_jwt_secret
REDIS_URL=your_redis_url
Rabbitmq_Host=your_rabbitmq_host
Rabbitmq_Username=your_username
Rabbitmq_Password=your_password
```

**`backend/chat/.env`**
```env
PORT=5002
MONGO_URI=your_mongodb_uri
JWT_SECRET=your_jwt_secret
USER_SERVICE=http://localhost:5000
Cloud_Name=your_cloudinary_cloud_name
Api_Key=your_cloudinary_api_key
Api_Secret=your_cloudinary_api_secret
```

**`backend/mail/.env`**
```env
PORT=5001
Rabbitmq_Host=your_rabbitmq_host
Rabbitmq_Username=your_username
Rabbitmq_Password=your_password
USER=your_gmail@gmail.com
PASSWORD=your_gmail_app_password
```

**`frontend/.env.local`**
```env
NEXT_PUBLIC_USER_SERVICE=http://localhost:5000
NEXT_PUBLIC_CHAT_SERVICE=http://localhost:5002
```
> Note: Update the hardcoded service URLs in `frontend/src/context/AppContext.jsx` to use env vars.

### 3. Install & Run

```bash
# Terminal 1 — User Service
cd backend/user && npm install && npm run dev

# Terminal 2 — Chat Service
cd backend/chat && npm install && npm run dev

# Terminal 3 — Mail Service
cd backend/mail && npm install && npm run dev

# Terminal 4 — Frontend
cd frontend && npm install && npm run dev
```

App runs at `http://localhost:3000`

---

## 🗺️ Data Flow

```
User enters email
    │
    ▼
User Service → publishes {to, subject, body} to RabbitMQ "send-otp" queue
    │
    ▼
Mail Service (consumer) → picks message → sends OTP via nodemailer/Gmail
    │
    ▼
OTP stored in Redis with 5min TTL | rate limit key stored with 60s TTL
    │
    ▼
User enters OTP → User Service validates against Redis → deletes key → issues JWT
    │
    ▼
JWT stored in browser cookie (15 days)
    │
    ▼
All subsequent requests → Bearer token → isAuth middleware → req.user populated
```

---

## 🔐 Security Notes

- OTP rate-limited per email (1 request per 60 seconds)
- OTPs expire after 5 minutes (Redis TTL)
- JWT tokens signed with `JWT_SECRET` — keep this secret and rotate it
- Cloudinary credentials scoped to `chat-images` folder
- CORS currently set to `*` — restrict to your domain in production
- Message-level authorization: users can only read/send in chats they belong to

---

## 📸 Screenshots

> App live at: **[http://43.204.149.4:3000/login](http://43.204.149.4:3000/login)**

| Screen | Description |
|--------|-------------|
| `/login` | Email input — triggers OTP |
| `/verify` | 6-box OTP entry with paste support |
| `/chat` | Main chat — sidebar + messages + typing |
| `/profile` | Edit display name |

---

## 👨‍💻 Author

**Rabi** — Final Year B.Tech (Mechanical Engineering), GCEA Kalahandi  
Self-taught Full Stack & AI Application Developer

[![GitHub](https://img.shields.io/badge/GitHub-Rabi22-181717?style=flat-square&logo=github)](https://github.com/Rabi22)

---

<div align="center">

Built with Node.js · Next.js · Socket.io · MongoDB · Redis · RabbitMQ · AWS EC2

</div>
