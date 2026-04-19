# Docker

## What is Docker?

**Docker** is a platform for packaging and running applications in **containers** — isolated environments that include the application and all its dependencies.

Key properties:

- **Portable** — the same image runs identically on any machine with Docker installed
- **Isolated** — each container has its own filesystem, network, and processes
- **Lightweight** — containers share the host OS kernel, no full OS per instance

> ### 💡 **Containers vs VMs**
>
> A VM emulates an entire machine (OS included) — ~200 MB overhead just to boot.
> A container runs as a process on the host kernel — ~10 MB, starts in seconds.
> Docker automates what you would otherwise configure manually on a VM: users, ports, services, networking, logs.

| Skill | Score |
|-------|:-----:|
| Explain the difference between containers and VMs | 5 |
| Explain what Docker solves over manual VM setup | 5 |
| Understand the container/image distinction | 5 |

---

## Dockerfile

A **Dockerfile** defines how to build an image — the immutable snapshot your containers run from.

### Multi-stage build (Go app)

```dockerfile
# Build stage — full Go toolchain
FROM golang:1.21-alpine AS builder

RUN apk add --no-cache ca-certificates git
WORKDIR /app

# Copy dependency files first → better layer caching
COPY go.mod go.sum ./
RUN go mod download

COPY *.go ./
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Production stage — minimal Alpine image
FROM alpine:latest

RUN apk --no-cache add ca-certificates

# Non-root user for security
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

WORKDIR /app
COPY --from=builder /app/main .
COPY static/ ./static/
COPY *.html ./

RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8000/ || exit 1

CMD ["./main"]
```

**Why multi-stage?** The builder stage needs the full Go toolchain (~300 MB). The production image only needs the compiled binary — final image is ~15 MB.

**Why non-root user?** Containers run as root by default. Creating a dedicated user reduces the attack surface if the app is compromised.

**Why copy `go.mod` before source?** Docker caches each layer. Dependencies only re-download when `go.mod` changes, not on every code edit.

| Skill | Score |
|-------|:-----:|
| Write a Dockerfile for a Go application | 5 |
| Use multi-stage builds | 5 |
| Understand layer caching | 4 |
| Run containers as non-root | 5 |
| Use `HEALTHCHECK` in a Dockerfile | 4 |

---

## Docker Compose

**Docker Compose** defines and runs multi-container applications via a YAML file.

### Base structure

```yaml
services:
  app:
    build:
      context: ..
      dockerfile: Dockerfile
    container_name: infra-lab-app
    ports:
      - "${APP_PORT:-8000}:8000"
    environment:
      - PORT=8000
      - LOG_LEVEL=${LOG_LEVEL:-info}
    volumes:
      - ../data.json:/app/data.json:ro
    restart: unless-stopped
    networks:
      - app-network
      - monitoring-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

networks:
  app-network:
    driver: bridge
  monitoring-network:
    driver: bridge

volumes:
  postgres_data:
  prometheus_data:
```

| Skill | Score |
|-------|:-----:|
| Write a Docker Compose file from scratch | 5 |
| Use environment variables with defaults (`${VAR:-default}`) | 5 |
| Mount volumes (named and bind mounts) | 4 |
| Configure restart policies | 5 |
| Define custom networks | 5 |

---

## Multi-environment Setup

A **base + override** strategy avoids duplicating configuration across environments.

```
docker-compose.yml          ← shared base (services, networks, volumes)
docker-compose.dev.yml      ← dev overrides (verbose logs, large limits)
docker-compose.prod.yml     ← prod overrides (hardened, minimal)
```

**Run dev:**
```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

**Run prod:**
```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Production overrides example

```yaml
services:
  app:
    environment:
      - LOG_LEVEL=error
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 256M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
```

| Skill | Score |
|-------|:-----:|
| Use Compose file overrides for multi-environment | 5 |
| Set resource limits (CPU/RAM) | 5 |
| Configure log rotation | 4 |
| Apply security hardening (read-only, no-new-privileges, tmpfs) | 4 |

---

## Health Checks and Dependencies

```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s   # Grace period before first check

  prometheus:
    depends_on:
      app:
        condition: service_healthy   # Wait for app to pass health check
```

`depends_on` with `condition: service_healthy` ensures ordered startup — Prometheus only starts once the app is ready, not just running.

| Skill | Score |
|-------|:-----:|
| Implement health checks in Compose | 5 |
| Use `depends_on` with health conditions | 5 |

---

## Network Isolation

Splitting services across named networks controls which containers can communicate.

```yaml
services:
  app:
    networks: [app-network, monitoring-network]   # Bridge between both
  postgres:
    networks: [app-network]                       # Not reachable from Prometheus
  prometheus:
    networks: [monitoring-network]                # Only scrapes app metrics

networks:
  app-network:
    driver: bridge
    name: infra-lab-app
  monitoring-network:
    driver: bridge
    name: infra-lab-monitoring
```

Services on the same network resolve each other **by container name** — no IP addresses needed.

| Skill | Score |
|-------|:-----:|
| Understand Docker bridge networking | 5 |
| Isolate services with multiple networks | 5 |
| Use container name DNS resolution | 4 |

---

## Volumes and Data Persistence

```yaml
services:
  postgres:
    volumes:
      - postgres_data:/var/lib/postgresql/data   # Named volume — persists across restarts

  app:
    volumes:
      - ../data.json:/app/data.json:ro            # Bind mount — host file, read-only

volumes:
  postgres_data:
    name: postgres_data
  prometheus_data:
    name: infra-lab-prometheus-data
```

**Named volumes** are managed by Docker — data survives `docker compose down`.  
**Bind mounts** map a host path directly into the container — useful for config files or hot-reload.  
`docker compose down -v` removes volumes (destructive — use with care).

| Skill | Score |
|-------|:-----:|
| Use named volumes for persistence | 5 |
| Use bind mounts for config injection | 5 |
| Understand volume lifecycle | 4 |

---

## Core Commands

```bash
# Build and start
docker compose up -d                    # Start in background
docker compose -f base.yml -f prod.yml up -d  # With overrides

# Inspect
docker compose ps                       # Running containers
docker compose logs -f app              # Follow logs for a service
docker stats                            # Live CPU/RAM usage

# Lifecycle
docker compose down                     # Stop and remove containers
docker compose down -v                  # Also remove volumes
docker compose restart app              # Restart one service

# Build
docker build -t infra-lab:latest .    # Build image manually
docker images                           # List images
docker rmi <image>                      # Remove image
```

| Skill | Score |
|-------|:-----:|
| Use `docker compose up/down` | 5 |
| Read logs with `docker compose logs` | 5 |
| Monitor resource usage with `docker stats` | 5 |
| Build images manually | 4 |

---

## VM vs Docker — Key Takeaways

Docker automates what requires manual work on a VM:

| Concern | VM (manual) | Docker |
|---------|-------------|--------|
| Service management | systemd unit files | `restart: unless-stopped` |
| Dependency isolation | Manual installs, version conflicts | Exact versions in Dockerfile |
| Port conflicts | `fuser -k`, iptables | Container-level isolation |
| Logs | journalctl + rsyslog | `docker compose logs` |
| Resource limits | Manual cgroups | `deploy.resources.limits` |
| Health monitoring | Custom scripts | Native `healthcheck` |
| Rollback | Manual, risky | `docker compose down/up` with tags |
| Reproducibility | ~80% | 100% |

**Measured on infra-lab project:**
- VM deployment: 1 min 37 sec
- Docker deployment: 13–16 sec (with health checks)
- Memory: ~207 MB (VM) vs ~61 MB (3 containers)

> VM experience is still valuable — it explains *why* Docker abstractions exist and builds debugging intuition.

---

**Keywords:** `image`, `container`, `Dockerfile`, `multi-stage build`, `layer caching`, `docker-compose`, `named volume`, `bind mount`, `bridge network`, `healthcheck`, `depends_on`, `restart policy`
