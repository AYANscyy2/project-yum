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
│   ├── config/           # Env loading, DB & Redis clients, logger
│   ├── http/             # Express app, middleware & REST routes
│   ├── sockets/          # Socket.IO init & event handlers
│   ├── services/         # Business logic
│   ├── repositories/     # Data access (ORM)
│   ├── middlewares/      # Auth, error handling, rate limiting
│   ├── utils/            # Helpers (tokens, validators, constants)
│   └── index.ts          # Cluster bootstrap & entry point
├── tests/                # Unit & integration tests
├── .env.example
├── package.json
├── tsconfig.json
└── README.md
```

---

## Prerequisites

- **Node.js** v16+
- **PostgreSQL** database
- **Redis** server
- **Docker** (optional)
- **Git**

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-org/chat-backend.git
cd chat-backend
```

### 2. Install dependencies

```bash
npm install
```

### 3. Set up environment

Copy `.env.example` to `.env` and fill in your credentials:

```bash
cp .env.example .env
```

Required variables:

```env
PORT=4000
DATABASE_URL=postgres://USER:PASSWORD@HOST:PORT/DB_NAME
REDIS_URL=redis://HOST:PORT
JWT_SECRET=your_jwt_secret
```

### 4. Generate ORM client & run migrations

#### For Prisma:

```bash
npx prisma migrate deploy
npx prisma generate
```

#### For Drizzle:

```bash
npx drizzle-kit migrate:deploy
```

### 5. Start in development

```bash
npm run dev
```

> Uses `ts-node-dev` or `nodemon` to watch for changes.

---

## Environment Variables

| Variable       | Description                  | Default    |
| -------------- | ---------------------------- | ---------- |
| `PORT`         | Server port                  | `4000`     |
| `DATABASE_URL` | PostgreSQL connection string | _required_ |
| `REDIS_URL`    | Redis connection string      | _required_ |
| `JWT_SECRET`   | Secret for signing JWTs      | _required_ |

---

## Project Structure

| Directory           | Purpose                                                                        |
| ------------------- | ------------------------------------------------------------------------------ |
| `src/config/`       | Load `.env`, validate schema, initialize DB & Redis clients, configure logger  |
| `src/http/`         | Express setup, middleware (Helmet, CORS, compression, rate limit), REST routes |
| `src/sockets/`      | Socket.IO server init, Redis adapter, event handlers for connection management |
| `src/services/`     | Encapsulate business logic (e.g., `ChatService.createMessage()`)               |
| `src/repositories/` | Abstract ORM queries for messages, rooms, and users                            |
| `src/middlewares/`  | JWT auth, error handling, input validation, rate limiting                      |
| `src/utils/`        | Token helpers, logging wrapper, constants, and validators                      |
| `src/index.ts`      | Cluster setup to fork worker processes                                         |

---

## Development

### Code Quality

```bash
# Lint code
npm run lint

# Format code
npm run format

# Type checking
npm run type-check
```

### Testing

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

---

## Testing

The project includes comprehensive testing:

- **Unit tests** for services and repositories using Jest
- **Integration tests** for HTTP routes and Socket.IO events
- **In-memory PostgreSQL** instance (`pg-mem`) for isolated testing
- **Mocked Redis** for pub/sub testing

---

## Deployment

### 1. Build the application

```bash
npm run build
```

### 2. Docker Deployment

```bash
# Build Docker image
docker build -t chat-backend .

# Run container
docker run -d \
  -e PORT=4000 \
  -e DATABASE_URL=$DATABASE_URL \
  -e REDIS_URL=$REDIS_URL \
  -e JWT_SECRET=$JWT_SECRET \
  -p 4000:4000 \
  chat-backend
```

### 3. Kubernetes Deployment

- Use a **Deployment** with Horizontal Pod Autoscaler
- Ensure Redis and PostgreSQL are reachable via services
- Configure readiness and liveness probes on `/health`

Example deployment configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chat-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: chat-backend
  template:
    metadata:
      labels:
        app: chat-backend
    spec:
      containers:
        - name: chat-backend
          image: chat-backend:latest
          ports:
            - containerPort: 4000
          env:
            - name: PORT
              value: "4000"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: chat-secrets
                  key: database-url
          livenessProbe:
            httpGet:
              path: /health
              port: 4000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 4000
            initialDelaySeconds: 5
            periodSeconds: 5
```

---

## API & Events

### REST Endpoints

| Method | Path            | Description                     | Parameters        |
| ------ | --------------- | ------------------------------- | ----------------- |
| `GET`  | `/health`       | Liveness/readiness probe        | -                 |
| `GET`  | `/api/messages` | Fetch paginated message history | `roomId`, `limit` |

### Socket.IO Events

#### Client → Server

| Event         | Payload                       | Description              |
| ------------- | ----------------------------- | ------------------------ |
| `joinRoom`    | `{ roomId: string }`          | Join a chat room         |
| `leaveRoom`   | `{ roomId: string }`          | Leave a chat room        |
| `sendMessage` | `{ roomId, userId, content }` | Send a message to a room |

#### Server → Client

| Event            | Payload                                  | Description           |
| ---------------- | ---------------------------------------- | --------------------- |
| `receiveMessage` | `{ roomId, userId, content, timestamp }` | Receive a new message |
| `userJoined`     | `{ roomId, userId }`                     | User joined the room  |
| `userLeft`       | `{ roomId, userId }`                     | User left the room    |

---

## Best Practices

### 🔧 Scaling

- **Horizontal Scaling**: Use Redis adapter and sticky sessions
- **Load Balancing**: Implement proper session affinity for Socket.IO
- **Database**: Use connection pooling and read replicas

### 🔒 Security

- **Transport Security**: Enforce HTTPS/WSS in production
- **Input Validation**: Sanitize all inputs and validate permissions
- **Authentication**: Use JWT tokens with proper expiration
- **Rate Limiting**: Implement per-user and per-IP rate limiting

### 📊 Monitoring

- **Logging**: Integrate with ELK stack or similar
- **Metrics**: Expose Prometheus metrics
- **Health Checks**: Implement comprehensive health endpoints
- **Error Tracking**: Use Sentry or similar service

### 🚀 CI/CD

- **Automated Testing**: Run tests on every commit
- **Code Quality**: Enforce linting and formatting
- **Security Scanning**: Regular vulnerability assessments
- **Deployment**: Use blue-green or rolling deployments

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Support

For questions or issues:

- 📧 Email: [your-email@example.com](mailto:your-email@example.com)
- 🐛 Issues: [GitHub Issues](https://github.com/your-org/chat-backend/issues)
- 📖 Documentation: [Wiki](https://github.com/your-org/chat-backend/wiki)
