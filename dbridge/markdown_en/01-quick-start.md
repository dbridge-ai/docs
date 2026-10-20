# Quick Start

This chapter covers how to install, start, and use DBridge Pro for the first time.

## Prerequisites

### Building from Source

| Dependency | Version Requirement | Notes |
|:-----------|:-------------------|:------|
| Go | 1.26+ | Backend compilation with CGO support (required by SQLite driver) |
| Node.js | 18+ | Frontend compilation |
| C Compiler | gcc / clang / zig | CGO compilation for SQLite driver |

### Using Pre-built Binaries

Simply download the binary for your platform and run — no dependencies required.

| Platform | Binary | CGO Compiler |
|:---------|:-------|:-------------|
| Linux (amd64/arm64) | `dbridge-pro` | gcc / cc / clang |
| macOS (amd64/arm64) | `dbridge-pro` | clang (Xcode CLT) |
| Windows (amd64/arm64) | `dbridge-pro.exe` | gcc or zig cc |

## One-Command Build & Start

```bash
# 1. Clone the project
git clone https://github.com/dbridge-ai/dbridge.git
cd dbridge

# 2. Build full stack (backend + frontend)
./build.sh

# 3. Start the service
./start.sh

# 4. Open browser to access
#    http://localhost:8082
```

On first startup, the system will automatically:
1. Create the SQLite database file (`./data/dbridge.db`)
2. Initialize the table structure
3. Create the default administrator account
4. Load built-in AI skills, agents, MCP tools, and other system data

## Default Account

| Auth Mode | Account | Password |
|:----------|:--------|:---------|
| password (local account) | admin | 123456 |
| usercenter (OAuth) | Log in via OAuth provider | — |

> Change the default password immediately after your first login.

## Startup Scripts Reference

DBridge Pro provides a set of cross-platform scripts (Linux / macOS / Windows Git Bash):

```bash
./build.sh   [release] [backend|frontend|all|force]
./start.sh   [prod|dev] [--port N] [--foreground] [--no-build]
./stop.sh    [--force] [--port N]
./restart.sh [prod|dev] [--port N] [--build|--no-build]
./status.sh  [--port N] [--env]
```

### build.sh — Build

```bash
./build.sh              # Default debug mode, build full stack
./build.sh release      # Release mode (smaller binary, no debug info)
./build.sh release backend   # Build backend only
./build.sh release frontend  # Build frontend only
./build.sh force        # Force reinstall frontend dependencies
```

### start.sh — Start

```bash
./start.sh                      # Default prod mode, run in background
./start.sh dev                  # Dev mode (warns on port conflicts instead of failing)
./start.sh --port 9090          # Specify port
./start.sh --foreground         # Run in foreground (logs to terminal)
./start.sh --no-build           # Skip auto-build
```

Port priority: `--port` > `PORT` environment variable > `server.port` in `configs/config.yaml` > default `8082`.

### stop.sh — Stop

```bash
./stop.sh              # Graceful stop (SIGTERM → wait → SIGKILL)
./stop.sh --force      # Force stop (including residual processes)
```

### restart.sh — Restart

```bash
./restart.sh                    # Restart (no rebuild)
./restart.sh --build            # Rebuild backend then restart
./restart.sh dev --port 9090    # Dev mode restart with custom port
```

### status.sh — Status

```bash
./status.sh             # Check running status
./status.sh --env       # Also output toolchain version info
```

Status checks include:
- Process status (PID, CPU, memory, uptime)
- Port listening status
- Health check endpoints (`/health/live`, `/health/ready`)
- Binary info (size, platform match)
- SQLite database info (file size, table count)
- Log file info

## Docker Deployment

### Docker Compose (Recommended)

```bash
docker-compose up -d
```

Docker Compose includes 4 services:

| Service | Port | Description |
|:--------|:-----|:------------|
| app | 8082 | DBridge Pro application |
| postgres | 5432 | PostgreSQL 14 metadata storage |
| redis | 6379 | Redis 7 cache |
| nats | 4222 | NATS 2.10 message queue |

### Standalone Docker Image

```bash
docker build -t dbridge-pro .
docker run -d -p 8082:8082 -v ./data:/app/data dbridge-pro
```

## Frontend Development Mode

For frontend development with hot-reload:

```bash
cd web
npm install
npm run dev
# Access http://localhost:5172
```

The dev server automatically proxies `/api` requests to the backend at `http://localhost:8082`.

## Configuration

The configuration file is located at `configs/config.yaml`. Key settings:

```yaml
# Server port
server:
  port: 8082

# Metadata storage
database:
  type: sqlite              # sqlite | mysql | postgresql
  sqlite:
    path: ./data/dbridge.db

# JWT secret (must be changed in production)
jwt:
  secret: change-me-in-production

# AI configuration
ai:
  provider: openai
  model: gpt-4o
  api_key: "your-api-key"
  base_url: ""              # Custom API endpoint (OpenAI-compatible protocol)

# Authentication mode
subscription:
  auth_mode: password       # password | usercenter
```

## First-Time Usage Flow

1. **Start the service** — Run `./start.sh`
2. **Open browser** — Visit `http://localhost:8082`
3. **Log in** — Use default account admin / 123456
4. **Add a data source** — Go to "Data Source Management", click "New", and fill in the database connection details
5. **Test connection** — Click "Test Connection" to verify connectivity
6. **Start querying** — Go to "SQL Editor", select a data source, write and execute SQL

## FAQ

### Startup Failed: Port Already in Use

```bash
# Check port usage
lsof -i :8082

# Start with a different port
./start.sh --port 9090
```

### Build Failed: C Compiler Not Found

The SQLite driver requires CGO, which needs a C compiler:

```bash
# Linux
sudo apt install build-essential

# macOS
xcode-select --install

# Windows (use zig as C compiler)
# Download zig and add to PATH
```

### Slow Frontend Dependency Installation

```bash
npm install --registry=https://registry.npmmirror.com --noproxy="*"
```

### Slow Go Module Downloads

```bash
export GOPROXY=https://goproxy.cn,direct
```

### Scripts Won't Execute on Windows

Make sure to use Git Bash or MSYS2 to run `.sh` scripts — do not run them in CMD or PowerShell.
