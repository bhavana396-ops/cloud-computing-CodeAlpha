# ☁️ Cloud Computing Internship Projects

> A collection of three production-grade cloud-native applications developed during my Cloud Computing Internship. Each project demonstrates a distinct aspect of modern cloud architecture — scalability, data integrity, and AI integration.

---

## 📁 Projects Overview

| # | Project | Tech Stack | Key Cloud Concept |
|---|---------|-----------|-------------------|
| 1 | [Cloud-Based Bus Pass System](#1-cloud-based-bus-pass-system) | Node.js, MongoDB Atlas, Redis, AWS | Auto-scaling, QR Ticketing |
| 2 | [Smart Deduplication System](#2-smart-deduplication-system) | Node.js, React, MongoDB, NLP | Data Integrity, Fuzzy Matching |
| 3 | [NexusBot AI Chatbot](#3-nexusbot-ai-chatbot) | Node.js, Anthropic Claude API, Docker | AI Cloud Services, NLP |

---

## 1. Cloud-Based Bus Pass System

### Overview
An online bus ticket booking platform built for the cloud. Eliminates ticket loss and theft using cryptographically signed QR codes, prevents pricing manipulation through server-side fare calculation, and handles traffic spikes via AWS Auto Scaling.

### Folder: `cloud-bus-pass-system/`

### Tech Stack
- **Backend:** Node.js + Express.js
- **Database:** MongoDB Atlas (cloud-hosted, connection pooling)
- **Cache & Locking:** Redis (AWS ElastiCache)
- **Auth:** JWT (access + refresh tokens) + bcrypt
- **QR Security:** HMAC-SHA256 signed digital passes
- **Payments:** Razorpay integration
- **Hosting:** AWS EC2 Auto Scaling Group + ALB + CloudFront CDN
- **CI/CD:** GitHub Actions → AWS ECR → EC2 rolling deploy
- **Containers:** Docker + Docker Compose

### Key Features
- 🔐 **Tamper-proof tickets** — HMAC-SHA256 integrity hash on every ticket; conductor scan rejects fakes instantly
- 💰 **Server-side pricing** — fares computed from distance × bus type multiplier on the backend; client cannot override
- ⚡ **Redis distributed lock** (`SETNX`) — prevents double-booking under concurrent traffic
- ☁️ **Auto Scaling** — EC2 ASG scales 1→10 instances based on CPU; zero downtime during peak hours
- 🌍 **CloudFront CDN** — static assets served globally with <50ms latency
- ♻️ **Graceful shutdown** — SIGTERM handler drains connections before EC2 replacement

### Architecture
```
User → CloudFront CDN → ALB → EC2 Auto Scaling Group (1-10)
                                      ↓
                              Redis ElastiCache (locks + cache)
                                      ↓
                              MongoDB Atlas (multi-region)
```

### Quick Start
```bash
cd cloud-bus-pass-system
cp .env.example .env        # fill in MONGO_URI, JWT_SECRET etc.
npm install
node database/seed.js       # seed sample routes & admin user
npm run dev                 # http://localhost:5000

# OR with Docker:
docker-compose up --build
```

**Demo credentials (after seeding):**
- Admin: `admin@buspass.cloud` / `Admin@1234`
- User: `demo@buspass.cloud` / `Demo@1234`

### API Endpoints (summary)
| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/auth/register` | Register user |
| POST | `/api/auth/login` | Login, get JWT |
| GET | `/api/bookings/search` | Search buses |
| POST | `/api/bookings/initiate` | Lock seats + create pending ticket |
| POST | `/api/bookings/confirm` | Confirm after payment, get QR |
| GET | `/api/tickets` | My ticket history |
| GET | `/api/admin/stats` | Admin dashboard metrics |

---

## 2. Smart Deduplication System

### Overview
A cloud-hosted data deduplication engine that detects duplicate records using a 4-layer pipeline — exact fingerprint matching, fuzzy string similarity, semantic scoring, and false-positive filtering. Designed for data cleaning pipelines, CRM systems, and any scenario where data integrity is critical.

### Folder: `smart-dedup-system/`

### Tech Stack
- **Backend:** Node.js + Express.js
- **Frontend:** React + Vite + Tailwind CSS
- **Database:** MongoDB (with strategic compound indexes)
- **NLP/Fuzzy:** `natural` (Jaro-Winkler, Levenshtein), `fuse.js`, Jaccard coefficient
- **Hashing:** SHA-256 fingerprint via `crypto-js`
- **Scheduling:** `node-cron` (cleanup jobs)
- **Logging:** Winston audit trail for every dedup decision

### Key Features
- 🧬 **4-layer dedup pipeline:**
  - **L1** — SHA-256 fingerprint exact match (instant)
  - **L2** — Levenshtein distance on email/phone
  - **L3** — Jaro-Winkler on names + Jaccard on all fields
  - **L4** — False-positive guard rules
- 📊 **Classification output:** `DUPLICATE` / `NEAR_DUPLICATE` / `UNIQUE` / `FALSE_POSITIVE`
- 🗂️ **Full audit log** — every check stored in `DedupLog` with confidence score and algorithm breakdown
- ⚡ **Optimized MongoDB indexes** — compound index on `email+phone` for fast candidate pool fetching
- 📈 **Analytics dashboard** — React frontend showing dedup stats, recent activity, and confidence distribution

### Dedup Algorithm
```
SHA-256(normalise(email) | stripNonAlpha(phone) | normalise(name))
         ↓ no exact match
Composite Score = fuzzyScore × 0.7 + semanticScore × 0.3
         ↓ score ≥ threshold
DUPLICATE / NEAR_DUPLICATE / UNIQUE
```

### Quick Start
```bash
# Backend
cd smart-dedup-system/backend
cp .env.example .env
npm install
npm run dev           # http://localhost:4000

# Frontend
cd ../frontend
npm install
npm run dev           # http://localhost:5173
```

### API Endpoints (summary)
| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/records` | Insert record with dedup check |
| GET | `/api/records` | List all records (paginated) |
| DELETE | `/api/records/:id` | Delete a record |
| GET | `/api/analytics/summary` | Dedup stats |
| GET | `/api/analytics/logs` | Audit log |

---

## 3. NexusBot AI Chatbot

### Overview
A cloud-deployed conversational AI chatbot powered by Anthropic's Claude API. Built as a lightweight, embeddable Node.js service with persistent multi-turn conversation history, intent classification, and a ready-to-embed web widget. Demonstrates integration of cloud AI services into a production backend.

### Folder: `nexusbot-ai-chatbot/` (root: `ai-chatbot-cloud/`)

### Tech Stack
- **Backend:** Node.js + Express.js
- **AI:** Anthropic Claude API (`@anthropic-ai/sdk`)
- **Frontend:** Vanilla JS embedded widget
- **Auth:** Rate limiting per session (express-rate-limit)
- **Security:** Helmet, CORS, input sanitisation
- **Containers:** Docker
- **Deployment:** Any cloud (AWS/GCP/Render/Railway)

### Key Features
- 🤖 **Claude AI backbone** — uses Anthropic's Claude for natural language understanding and response generation
- 💬 **Multi-turn memory** — full conversation history passed per request; context-aware replies
- 🧠 **Intent classification** — local pattern matching for common intents before hitting the API (saves costs)
- 🌐 **Embeddable widget** — drop a `<script>` tag into any HTML page to add the chatbot
- 🛡️ **Rate limiting** — 30 messages/15 min per IP to prevent abuse
- ☁️ **Cloud-ready** — stateless design, environment-variable config, Docker container, health check endpoint

### Quick Start
```bash
cd ai-chatbot-cloud
cp .env.example .env
# Set ANTHROPIC_API_KEY=your_key_here in .env
npm install
npm run dev           # http://localhost:3000

# Docker:
docker build -t nexusbot .
docker run -p 3000:3000 --env-file .env nexusbot
```

### Embed the Widget
```html
<!-- Add to any HTML page -->
<script>
  window.NexusBot = { apiUrl: 'https://your-deployed-url.com' };
</script>
<script src="https://your-deployed-url.com/widget.js"></script>
```

### API Endpoints (summary)
| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/chat` | Send message, get AI reply |
| GET | `/api/chat/:sessionId/history` | Get conversation history |
| DELETE | `/api/chat/:sessionId` | Clear session |
| GET | `/health` | Health check (for ALB/uptime monitor) |

---

## 🚀 Common Setup Instructions

### Prerequisites
- Node.js v18+
- MongoDB (local) or MongoDB Atlas (cloud)
- Redis (for Bus Pass project)
- Anthropic API key (for NexusBot)

### Run All Projects Locally
```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/cloud-computing-internship.git
cd cloud-computing-internship

# Project 1 — Bus Pass
cd cloud-bus-pass-system && npm install && node database/seed.js && npm run dev

# Project 2 — Smart Dedup (new terminal)
cd smart-dedup-system/backend && npm install && npm run dev

# Project 3 — NexusBot (new terminal)
cd ai-chatbot-cloud && npm install && npm run dev
```

---

## ☁️ Cloud Deployment Summary

| Project | Hosting | Key Services |
|---------|---------|-------------|
| Bus Pass | AWS EC2 ASG + CloudFront | ECR, ALB, ElastiCache, Atlas |
| Smart Dedup | AWS EC2 / Render | MongoDB Atlas |
| NexusBot | AWS EC2 / Railway / Render | Anthropic API, Docker |

All three projects are containerised with Docker and include GitHub Actions CI/CD workflows for automated deployment on push to `main`.

---

## 🧪 Running Tests

```bash
# Bus Pass
cd cloud-bus-pass-system && npm test

# Smart Dedup
cd smart-dedup-system/backend && npm test

# NexusBot
cd ai-chatbot-cloud && npm test
```

---

## 📄 License

MIT — free to use for educational purposes.

---

*Cloud Computing Internship · 2024–25*
