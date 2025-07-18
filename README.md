<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# README.md

```markdown
# project-yum: Chat Backend Service

A horizontally scalable, real-time chat backend built with Node.js, Express, Socket.IO, and PostgreSQL (via Drizzle ORM or Prisma), with Redis for pub/sub and clustering support.

---

## Table of Contents

1. [Features](#features)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Getting Started](#getting-started)
5. [Environment Variables](#environment-variables)
6. [Project Structure](#project-structure)
7. [Development](#development)
8. [Testing](#testing)
9. [Deployment](#deployment)
10. [API & Events](#api--events)
11. [Best Practices](#best-practices)
12. [License](#license)

---

## Features

- 🏗️ **Modular, layered architecture** for HTTP and WebSocket concerns
- ⚡ **Real-time messaging** via Socket.IO with Redis adapter for horizontal scaling
- 🔒 **Security** with JWT auth, CORS, Helmet, rate limiting, and input sanitization
- 📦 **Type-safe ORM** using Drizzle or Prisma for PostgreSQL
- 🌐 **Stateless** Node.js service with clustering support (PM2 or native cluster)
- 🚀 **Production-ready**: logging, monitoring hooks, and Docker support

---

## Architecture
```

project-root/
├── src/
│ ├── config/ \# Env loading, DB \& Redis clients, logger
│ ├── http/ \# Express app, middleware \& REST routes
│ ├── sockets/ \# Socket.IO init \& event handlers
│ ├── services/ \# Business logic
│ ├── repositories/ \# Data access (ORM)
│ ├── middlewares/ \# Auth, error handling, rate limiting
│ ├── utils/ \# Helpers (tokens, validators, constants)
│ └── index.ts \# Cluster bootstrap \& entry point
├── tests/ \# Unit \& integration tests
├── .env.example
├── package.json
├── tsconfig.json
└── README.md

```

---

## Prerequisites

- Node.js v16+
- PostgreSQL database
- Redis server
- Docker (optional)
- Git

---

## Getting Started

1. **Clone the repo**
```

git clone https://github.com/your-org/chat-backend.git
cd chat-backend

```

2. **Install dependencies**
```

npm install

```

3. **Set up environment**
- Copy `.env.example` to `.env` and fill in your credentials
- Required variables:
```

PORT=4000
DATABASE_URL=postgres://USER:PASSWORD@HOST:PORT/DB_NAME
REDIS_URL=redis://HOST:PORT
JWT_SECRET=your_jwt_secret

```

4. **Generate ORM client & run migrations**
- **Prisma**:
```

npx prisma migrate deploy
npx prisma generate

```
- **Drizzle**:
```

npx drizzle-kit migrate:deploy

```

5. **Start in development**
```

npm run dev

```
- Uses `ts-node-dev` or `nodemon` to watch for changes.

---

## Environment Variables

| Key             | Description                          |
| --------------- | ------------------------------------ |
| `PORT`          | Server port (default: 4000)          |
| `DATABASE_URL`  | PostgreSQL connection string         |
| `REDIS_URL`     | Redis connection string              |
| `JWT_SECRET`    | Secret for signing JWTs              |

---

## Project Structure

- **src/config**: Load `.env`, validate schema, initialize DB & Redis clients, configure logger
- **src/http**: Express setup, middleware (Helmet, CORS, compression, rate limit), REST routes (e.g., health check, message history)
- **src/sockets**: Socket.IO server init, Redis adapter, event handlers for `connection`, `joinRoom`, `leaveRoom`, and `sendMessage`
- **src/services**: Encapsulate business logic (e.g., `ChatService.createMessage()`)
- **src/repositories**: Abstract ORM queries for messages, rooms, and users
- **src/middlewares**: JWT auth, error handling, input validation, rate limiting
- **src/utils**: Token helpers, logging wrapper, constants, and validators
- **src/index.ts**: Cluster setup to fork worker processes

---

## Development

- **Lint & format**
```

npm run lint
npm run format

```
- **Run tests**
```

npm test

```
- **Type-check**
```

npm run type-check

```

---

## Testing

- Unit tests for services and repositories (with Jest)
- Integration tests for HTTP routes and Socket.IO events using an in-memory PostgreSQL instance (`pg-mem`) and mocked Redis

---

## Deployment

1. **Build**
```

npm run build

```
2. **Docker**
```

docker build -t chat-backend .
docker run -d \
-e PORT=4000 \
-e DATABASE_URL=$DATABASE_URL \
     -e REDIS_URL=$REDIS_URL \
-e JWT_SECRET=\$JWT_SECRET \
-p 4000:4000 \
chat-backend

```
3. **Kubernetes**
- Use a Deployment with Horizontal Pod Autoscaler
- Ensure Redis and PostgreSQL are reachable via services
- Configure readiness and liveness probes on `/health`

---

## API & Events

### REST Endpoints

| Method | Path         | Description              |
| ------ | ------------ | ------------------------ |
| GET    | `/health`    | Liveness/readiness probe |
| GET    | `/api/messages?roomId=&limit=` | Fetch paginated message history |

### Socket.IO Events

| Event         | Direction      | Payload                                          |
| ------------- | -------------- | ------------------------------------------------ |
| `joinRoom`    | Client → Server | `{ roomId: string }`                             |
| `leaveRoom`   | Client → Server | `{ roomId: string }`                             |
| `sendMessage` | Client → Server | `{ roomId, userId, content }`                    |
| `receiveMessage` | Server → Client | `{ roomId, userId, content, timestamp }`         |

---

## Best Practices

- **Horizontal Scaling**: Use Redis adapter and sticky sessions.
- **Statelessness**: Store no in-memory session state; use JWTs/Redis.
- **Security**: Enforce HTTPS/WSS, sanitize inputs, validate permissions.
- **Monitoring**: Integrate logs with ELK/Prometheus; expose metrics.
- **CI/CD**: Automate linting, tests, and vulnerability scans.

---

## License

This project is licensed under the MIT License.
```
