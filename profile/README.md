## System Architecture


<img width="1536" height="1024" alt="neuro AI architecture" src="https://github.com/user-attachments/assets/f68046be-33c7-4a22-ad9f-cc9aaf34af47" />





## Repositories

| Repo | Role |
|------|------|
| **cryptocee-api** | REST API + WebSocket hub, auth, business logic, DB, calls AI service |
| **cryptocee-app** | End-user UI; only talks to BACKEND |
| **cryptocee-admin** | Admin UI; only talks to BACKEND (admin routes) |
| **cryptocee-ai-predict** | Python FastAPI service: runs ML models, returns forecasts |

---

## 1. cryptocee-api

- Exposes **HTTP APIs** (and optionally **WebSocket**) to browsers and the admin app.
- **Authenticates** users (e.g. JWT) and enforces **roles** (user vs admin).
- Owns **business rules**: subscriptions, market data aggregation, saving predictions, schedulers.
- Reads/writes **PostgreSQL** (or MySQL) and uses **Redis** for cache, sessions, or queues.
- **Calls AI-PREDICT** over an internal HTTP URL when a forecast is needed; maps the JSON response into your DB models before responding to clients.

**Why it sits in the middle:** browsers and admin apps never need AI service URLs or model secrets only BACKEND does.

---

## 2. cryptocee-app

- Renders the **customer** product (markets, charts, predictions, portfolio, etc.).
- Sends **REST** requests to BACKEND (base URL + `/api/...`).
- Uses **WebSocket** only to BACKEND for live ticks, notifications, or streaming updates if you expose them there.
- Does **not** call AI-PREDICT directly.

---

## 3. cryptocee-admin

- Same pattern as FRONTEND, but for **admins**: users, analytics, logs, monitoring.
- Calls BACKEND **admin** endpoints (still REST; same origin pattern: only BACKEND).
- Uses the same auth mechanism in principle (e.g. admin JWT or role claim), stricter routes on BACKEND side.

---

## 4. cryptocee-ai-predict

- Exposes a small **FastAPI** surface (e.g. `/predict`, `/predicts`) that accepts **symbol, interval, horizon**, etc.
- Loads **ML/DL** models, runs inference, returns **numbers + confidence** (and any extra fields you define).
- Expects callers to be **trusted** (BACKEND on a private network), not the public internet.

---

## 5. How they work together

### Who talks to whom

```text
  FRONTEND ──────► BACKEND ◄────── DASHBOARD
                     │
                     ├──► PostgreSQL / MySQL
                     ├──► Redis
                     └──► AI-PREDICT   (server-to-server only)
```

