# Microservices Containerization — Node.js + Docker + Docker Compose

## Overview

This project containerizes a microservices-based application with four independent Node.js services, orchestrated using Docker Compose and connected through a shared bridge network (`microservices-net`).

| Service          | Port | Direct Endpoint                          |
|------------------|------|------------------------------------------|
| user-service     | 3000 | http://localhost:3000/users              |
| product-service  | 3001 | http://localhost:3001/products           |
| order-service    | 3002 | http://localhost:3002/orders             |
| gateway-service  | 3003 | http://localhost:3003/api/users etc.     |

---

## Repository Layout (after setup)

```
Microservices-Task-SK/
├── docker-compose.yml               ← place here (repo root)
├── Microservices/
│   ├── user-service/
│   │   ├── app.js
│   │   ├── package.json
│   │   └── Dockerfile               ← copy here
│   ├── product-service/
│   │   ├── app.js
│   │   ├── package.json
│   │   └── Dockerfile               ← copy here
│   ├── order-service/
│   │   ├── app.js
│   │   ├── package.json
│   │   └── Dockerfile               ← copy here
│   └── gateway-service/
│       ├── app.js
│       ├── package.json
│       └── Dockerfile               ← copy here
└── submission/                      ← assessment deliverable folder
    ├── user-service/Dockerfile
    ├── product-service/Dockerfile
    ├── order-service/Dockerfile
    ├── gateway-service/Dockerfile
    ├── docker-compose.yml
    └── README.md
```

> **Important:** Each `Dockerfile` must be copied into its corresponding `Microservices/<service>/` folder. The `docker-compose.yml` build context points to those folders, so Docker expects to find the `Dockerfile` there alongside `app.js` and `package.json`.

---

## Setup Instructions

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/Microservices-Task.git
cd Microservices-Task-SK
```

### 2. Place the Dockerfiles into the source folders

```bash
cp submission/user-service/Dockerfile     Microservices/user-service/
cp submission/product-service/Dockerfile  Microservices/product-service/
cp submission/order-service/Dockerfile    Microservices/order-service/
cp submission/gateway-service/Dockerfile  Microservices/gateway-service/
```

### 3. Place docker-compose.yml at the repo root

```bash
cp submission/docker-compose.yml .
```

### 4. Build and start all services

```bash
docker compose up --build
```

To run in detached (background) mode:

```bash
docker compose up --build -d
```

### 5. Verify all containers are running

```bash
docker compose ps
```

All four services should show `running`.

### 6. Stop all services

```bash
docker compose down
```

---

## Testing Each Service

### User Service — Port 3000

```bash
curl http://localhost:3000/users
```

**Expected:** JSON array of users.

---

### Product Service — Port 3001

```bash
curl http://localhost:3001/products
```

**Expected:** JSON array of products.

---

### Order Service — Port 3002

```bash
curl http://localhost:3002/orders
```

**Expected:** JSON array of orders.

---

### Gateway Service — Port 3003

```bash
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders
```

**Expected:** Same JSON data routed through the gateway.

---

## Dockerfile Design

Each service uses the same optimised pattern:

```dockerfile
FROM node:18-alpine          # Lightweight LTS image (~50 MB vs ~900 MB full Debian)
WORKDIR /app                 # Isolated working directory
COPY package*.json ./        # Copy manifest first — npm install layer is cached
RUN npm install --omit=dev   # Production dependencies only
COPY . .                     # Copy application source
EXPOSE <port>                # Declare the service port
CMD ["node", "app.js"]       # Start the service
```

---

## Docker Compose Design

- **Shared network** (`microservices-net`, bridge driver) — all containers resolve each other by service name (e.g. `http://user-service:3000`).
- **`depends_on`** on gateway-service — ensures backend services start before the gateway.
- **`restart: unless-stopped`** — containers auto-recover from crashes.
- No `version` field — compatible with Docker Compose V2 (current default in Docker Desktop).

---

## Troubleshooting

### `open Dockerfile: no such file or directory`

The Dockerfile is not in the service's source folder. Run the copy commands from Step 2 above, then retry `docker compose up --build`.

---

### Port already in use

```bash
# Find the process using the port (e.g. 3000)
lsof -i :3000
kill -9 <PID>
```

---

### Gateway returns connection refused

The gateway resolves backends by container name over `microservices-net`. Confirm all containers are on the network:

```bash
docker network inspect microservices-net
```

All four containers should appear under `Containers`.

---

### Force a clean rebuild

```bash
docker compose down
docker compose up --build
```

---

## Useful Commands

| Task                        | Command                                   |
|-----------------------------|-------------------------------------------|
| Build & start               | `docker compose up --build`               |
| Start in background         | `docker compose up -d`                    |
| Stop                        | `docker compose down`                     |
| View container status       | `docker compose ps`                       |
| Follow all logs             | `docker compose logs -f`                  |
| Logs for one service        | `docker compose logs -f gateway-service`  |
| Open shell in container     | `docker exec -it user-service sh`         |
| Inspect shared network      | `docker network inspect microservices-net`|
