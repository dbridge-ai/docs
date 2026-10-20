# Deployment Guide

## 1. Overview

DBridge uses a unified frontend-backend architecture, ultimately packaged as a single Docker image to simplify deployment. It also provides cross-platform build and startup scripts, supporting Linux, macOS, and Windows (Git Bash) environments.

**Core Capabilities**:
- Single Docker image deployment
- Docker Compose orchestration
- Cross-platform build scripts
- Multi-environment configuration

---

## 2. Prerequisites

| Component | Minimum Version | Description |
|-----------|----------------|-------------|
| Docker | 20.10+ | Container runtime |
| Docker Compose | 2.0+ | Container orchestration (optional) |
| Go | 1.26+ | Backend compilation (required for source deployment) |
| Node.js | 20+ | Frontend compilation (required for source deployment) |

---

## 3. Docker Deployment

### 3.1 Quick Start

```bash
docker run -d \
  --name dbridge \
  -p 8082:8082 \
  -v dbridge-data:/app/data \
  -v dbridge-logs:/app/logs \
  -v dbridge-config:/app/configs \
  dbridge/dbridge-pro:latest
```

### 3.2 Port Description

| Port | Description |
|------|-------------|
| 8082 | Web service port |

### 3.3 Volume Description

| Volume | Description |
|--------|-------------|
| `/app/data` | Database files (SQLite) |
| `/app/logs` | Log files |
| `/app/configs` | Configuration files |

### 3.4 Custom Configuration

Mount a custom `config.yaml` into the container:

```bash
docker run -d \
  --name dbridge \
  -p 8082:8082 \
  -v ./config.yaml:/app/configs/config.yaml \
  -v dbridge-data:/app/data \
  -v dbridge-logs:/app/logs \
  dbridge/dbridge-pro:latest
```

---

## 4. Docker Compose Deployment

### 4.1 Creating compose.yaml

```yaml
services:
  dbridge:
    image: dbridge/dbridge-pro:latest
    container_name: dbridge
    ports:
      - "8082:8082"
    volumes:
      - dbridge-data:/app/data
      - dbridge-logs:/app/logs
      - ./configs/config.yaml:/app/configs/config.yaml
    environment:
      - TZ=Asia/Shanghai
    restart: unless-stopped

volumes:
  dbridge-data:
  dbridge-logs:
```

### 4.2 Starting Services

```bash
docker compose up -d
```

### 4.3 Viewing Logs

```bash
docker compose logs -f dbridge
```

### 4.4 Stopping Services

```bash
docker compose down
```

### 4.5 Updating Version

```bash
docker compose pull
docker compose up -d
```

---

## 5. Source Deployment

### 5.1 Cloning the Code

```bash
git clone https://github.com/your-org/dbridge-pro.git
cd dbridge-pro
```

### 5.2 Building

The project provides cross-platform build scripts:

**Linux / macOS**:

```bash
./scripts/build.sh
```

**Windows (Git Bash)**:

```bash
./scripts/build.sh
```

The build script automatically completes the following steps:
1. Compile frontend (`web/` directory)
2. Compile backend (`cmd/server/`)
3. Output to `target/` directory

### 5.3 Starting

**Linux / macOS**:

```bash
./scripts/start.sh
```

**Windows (Git Bash)**:

```bash
./scripts/start.sh
```

### 5.4 Stopping

```bash
./scripts/stop.sh
```

---

## 6. Configuration Reference

The configuration file is located at `configs/config.yaml`. Key settings:

### 6.1 Server Configuration

```yaml
server:
  env: development        # development | production
  port: 8082              # Service port
  read_timeout: 30s       # Read timeout
  write_timeout: 300s     # Write timeout (AI conversations need longer timeout)
```

### 6.2 Database Configuration

```yaml
database:
  type: sqlite            # sqlite | mysql | postgresql
  sqlite:
    path: ./data/dbridge.db
  mysql:
    host: localhost
    port: 3306
    database: dbridge
    username: root
    password: ""
  postgresql:
    host: localhost
    port: 5432
    database: dbridge
    username: postgres
    password: ""
    ssl_mode: disable
```

### 6.3 AI Configuration

```yaml
ai:
  provider: openai        # openai | azure | ollama, etc.
  model: gpt-4
  api_key: ""
  base_url: ""
  temperature: 0.1
  max_tokens: 2000
  timeout: 30s
  daily_quota: 1000
  rate_limit: 20
```

### 6.4 Authentication Configuration

```yaml
subscription:
  enabled: true
  auth_mode: "usercenter"  # usercenter | password
  base_url: "http://localhost:8080"
  app_id: "your_app_id"
  product_id: "your_product_id"
```

### 6.5 Cache Configuration

```yaml
cache:
  backend: local           # local | redis
  default_ttl: 300
  redis:
    addr: localhost:6379
    password: ""
    db: 0
```

### 6.6 Log Configuration

```yaml
log:
  level: debug             # debug | info | warn | error
  output: stdout           # stdout | file
  file_path: ./logs/dbridge.log
  format: console          # console | json
```

---

## 7. Production Recommendations

### 7.1 Database

- Use MySQL or PostgreSQL for production instead of SQLite
- Configure database connection pool parameters (`max_open_conns`, `max_idle_conns`)
- Back up the database regularly

### 7.2 Cache

- Use Redis as the cache backend for multi-instance deployments
- Local cache is sufficient for single-instance deployments

### 7.3 Security

- Change JWT Secret to a strong random password
- Enable HTTPS (via reverse proxy)
- Configure CORS allowed domains
- Enable audit log auto-cleanup

### 7.4 Reverse Proxy

Nginx is recommended as a reverse proxy:

```nginx
server {
    listen 443 ssl;
    server_name dbridge.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # SSE support (AI conversation streaming)
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 300s;
    }
}
```

### 7.5 Resource Requirements

| Scale | CPU | Memory | Disk |
|-------|-----|--------|------|
| Small (< 10 data sources) | 2 cores | 4 GB | 20 GB |
| Medium (10-50 data sources) | 4 cores | 8 GB | 50 GB |
| Large (> 50 data sources) | 8 cores | 16 GB | 100 GB |

---

## 8. FAQ

### 8.1 Port Already in Use

Modify `server.port` in `config.yaml`, or use Docker port mapping:

```bash
docker run -d -p 9090:8082 dbridge/dbridge-pro:latest
```

### 8.2 Database Connection Failed

Check if database configuration is correct and confirm the database service is running and network-accessible.

### 8.3 AI Features Not Available

Check if `api_key` and `base_url` in the AI configuration are correct, and confirm the AI service endpoint is accessible.

### 8.4 Where Are the Logs

- Docker deployment: `docker logs dbridge`
- Source deployment: Check the `log.file_path` configured in `config.yaml`

### 8.5 How to Back Up

Back up the following data:
- Database files (`data/` directory or external database)
- Configuration files (`configs/config.yaml`)
- Storage files (if using local storage)
