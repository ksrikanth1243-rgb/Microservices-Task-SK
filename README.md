# Microservices Containerization — Node.js + Docker + Docker Compose

## Overview

This project containerizes a microservices-based application with four independent Node.js services, orchestrated using Docker Compose and connected through a shared bridge network (`microservices-net`).

| Service          | Port | Endpoint              |
|------------------|------|-----------------------|
| user-service     | 3000 | `http://localhost:3000/users`    |
| product-service  | 3001 | `http://localhost:3001/products` |
| order-service    | 3002 | `http://localhost:3002/orders`   |
| gateway-service  | 3003 | `http://localhost:3003/api/...`  |

---

## Project Structure

```
Microservices-Task/
├── submission/
│   ├── user-service/
│   │   └── Dockerfile
│   ├── product-service/
│   │   └── Dockerfile
│   ├── order-service/
│   │   └── Dockerfile
│   ├── gateway-service/
│   │   └── Dockerfile
│   ├── docker-compose.yml
│   └── README.md
└── Microservices/
    ├── user-service/
    │   └── app.js
    ├── product-service/
    │   └── app.js
    ├── order-service/
    │   └── app.js
    └── gateway-service/
        └── app.js
```

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (includes Docker Compose)
- Git

---

## Setup & Running

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/Microservices-Task.git
cd Microservices-Task
```

### 2. Build and start all services

From the **root of the repository** (where `docker-compose.yml` is located):

```bash
docker compose up --build
```

To run in detached (background) mode:

```bash
docker compose up --build -d
```

### 3. Verify all containers are running

```bash
docker compose ps
```

You should see all four services in `running` state.

### 4. Stop all services

```bash
docker compose down
```

---

## Testing Each Service

### User Service — Port 3000

```bash
curl http://localhost:3000/users
```
Or open in browser: http://localhost:3000/users

**Expected response:** JSON array of users.

---

### Product Service — Port 3001

```bash
curl http://localhost:3001/products
```
Or open in browser: http://localhost:3001/products

**Expected response:** JSON array of products.

---

### Order Service — Port 3002

```bash
curl http://localhost:3002/orders
```
Or open in browser: http://localhost:3002/orders

**Expected response:** JSON array of orders.

---

### Gateway Service — Port 3003

The gateway proxies requests to the other services. Test all routes via the gateway:

```bash
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders
```

**Expected response:** Same JSON data as direct service calls, routed through the gateway.

---

## Dockerfile Design

Each service uses a consistent, optimised Dockerfile pattern:

```dockerfile
FROM node:18-alpine          # Lightweight official Node.js image
WORKDIR /app                 # Isolated working directory
COPY package*.json ./        # Copy dependency manifest first (cache layer)
RUN npm install --omit=dev   # Install only production dependencies
COPY . .                     # Copy source code
EXPOSE <port>                # Declare service port
CMD ["node", "app.js"]       # Start the service
```

Key decisions:
- **`node:18-alpine`** — LTS release, minimal attack surface, ~50 MB vs ~900 MB for full Debian image.
- **Layer caching** — `package*.json` is copied before source code so `npm install` is only re-run when dependencies change, not on every source edit.
- **`--omit=dev`** — Excludes devDependencies from the production image, keeping it lean.

---

## Docker Compose Design

The `docker-compose.yml`:
- Defines all four services with their build context pointing to the correct source directory.
- Maps each container port to the same host port for direct accessibility.
- Places all services on a shared **`microservices-net` bridge network**, enabling DNS-based inter-container communication (e.g., the gateway can reach `http://user-service:3000` by container name).
- Uses `depends_on` on `gateway-service` to ensure backend services start before the gateway.
- Sets `restart: unless-stopped` for automatic recovery from crashes.

---

## Troubleshooting

### Port already in use

**Error:** `Bind for 0.0.0.0:3000 failed: port is already allocated`

**Fix:** Find and stop the process using the port:
```bash
# On Linux/Mac
lsof -i :3000 | grep LISTEN
kill -9 <PID>

# On Windows (PowerShell)
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

---

### Container exits immediately

**Check logs:**
```bash
docker compose logs user-service
docker compose logs gateway-service
```

Common cause: `app.js` entry point not found. Verify the source folder contains `app.js` and the `Dockerfile` `COPY . .` instruction includes it.

---

### Gateway cannot reach backend services

**Error:** `ECONNREFUSED` or `ETIMEDOUT` in gateway logs

**Check:** All services must be on the same network. Confirm with:
```bash
docker network inspect microservices-net
```
Ensure all four containers are listed under `Containers`. The gateway should reference other services by their **container name** (e.g., `http://user-service:3000`), not `localhost`.

---

### Images not rebuilding after code changes

Run with the `--build` flag to force a rebuild:
```bash
docker compose up --build
```

Or rebuild a single service:
```bash
docker compose build user-service
docker compose up -d user-service
```

---

### Clean slate restart

Remove all containers, networks, and cached images for this project:
```bash
docker compose down --volumes --rmi all
docker compose up --build
```

---

## Useful Commands Reference

| Task                          | Command                                      |
|-------------------------------|----------------------------------------------|
| Start all services            | `docker compose up --build`                  |
| Start in background           | `docker compose up -d`                       |
| Stop all services             | `docker compose down`                        |
| View running containers       | `docker compose ps`                          |
| Follow logs (all services)    | `docker compose logs -f`                     |
| Follow logs (one service)     | `docker compose logs -f gateway-service`     |
| Rebuild a single service      | `docker compose build user-service`          |
| Open shell in container       | `docker exec -it user-service sh`            |
| Inspect the shared network    | `docker network inspect microservices-net`   |
| Remove everything             | `docker compose down --volumes --rmi all`    |

---

## Screenshots

> Screenshots showing all services running are included in the repository under `/screenshots/`.

Key screenshots to include:
1. `docker compose up --build` output showing all four images being built.
2. `docker compose ps` showing all four containers in `running` state.
3. Browser/curl output for `http://localhost:3000/users`.
4. Browser/curl output for `http://localhost:3003/api/users` (gateway routing).
5. `docker network inspect microservices-net` showing all containers connected.
