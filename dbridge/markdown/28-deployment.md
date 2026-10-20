# 部署指南

## 1. 功能概述

DBridge 采用前后端合一的架构，最终打包为单一 Docker 镜像，简化部署流程。同时提供跨平台的构建和启动脚本，支持 Linux、macOS 和 Windows（Git Bash）环境。

**核心能力**：
- Docker 单镜像部署
- Docker Compose 编排
- 跨平台构建脚本
- 多环境配置

---

## 2. 环境要求

| 组件 | 最低版本 | 说明 |
|------|---------|------|
| Docker | 20.10+ | 容器运行时 |
| Docker Compose | 2.0+ | 容器编排（可选） |
| Go | 1.26+ | 后端编译（源码部署时需要） |
| Node.js | 20+ | 前端编译（源码部署时需要） |

---

## 3. Docker 部署

### 3.1 快速启动

```bash
docker run -d \
  --name dbridge \
  -p 8082:8082 \
  -v dbridge-data:/app/data \
  -v dbridge-logs:/app/logs \
  -v dbridge-config:/app/configs \
  dbridge/dbridge-pro:latest
```

### 3.2 端口说明

| 端口 | 说明 |
|------|------|
| 8082 | Web 服务端口 |

### 3.3 数据卷说明

| 卷 | 说明 |
|----|------|
| `/app/data` | 数据库文件（SQLite） |
| `/app/logs` | 日志文件 |
| `/app/configs` | 配置文件 |

### 3.4 自定义配置

将自定义的 `config.yaml` 挂载到容器中：

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

## 4. Docker Compose 部署

### 4.1 创建 compose.yaml

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

### 4.2 启动服务

```bash
docker compose up -d
```

### 4.3 查看日志

```bash
docker compose logs -f dbridge
```

### 4.4 停止服务

```bash
docker compose down
```

### 4.5 更新版本

```bash
docker compose pull
docker compose up -d
```

---

## 5. 源码部署

### 5.1 克隆代码

```bash
git clone https://github.com/your-org/dbridge-pro.git
cd dbridge-pro
```

### 5.2 构建

项目提供跨平台构建脚本：

**Linux / macOS**：

```bash
./scripts/build.sh
```

**Windows（Git Bash）**：

```bash
./scripts/build.sh
```

构建脚本会自动完成以下步骤：
1. 编译前端（`web/` 目录）
2. 编译后端（`cmd/server/`）
3. 输出到 `target/` 目录

### 5.3 启动

**Linux / macOS**：

```bash
./scripts/start.sh
```

**Windows（Git Bash）**：

```bash
./scripts/start.sh
```

### 5.4 停止

```bash
./scripts/stop.sh
```

---

## 6. 配置参考

配置文件位于 `configs/config.yaml`，主要配置项：

### 6.1 服务器配置

```yaml
server:
  env: development        # development | production
  port: 8082              # 服务端口
  read_timeout: 30s       # 读超时
  write_timeout: 300s     # 写超时（AI 对话需要较长超时）
```

### 6.2 数据库配置

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

### 6.3 AI 配置

```yaml
ai:
  provider: openai        # openai | azure | ollama 等
  model: gpt-4
  api_key: ""
  base_url: ""
  temperature: 0.1
  max_tokens: 2000
  timeout: 30s
  daily_quota: 1000
  rate_limit: 20
```

### 6.4 认证配置

```yaml
subscription:
  enabled: true
  auth_mode: "usercenter"  # usercenter | password
  base_url: "http://localhost:8080"
  app_id: "your_app_id"
  product_id: "your_product_id"
```

### 6.5 缓存配置

```yaml
cache:
  backend: local           # local | redis
  default_ttl: 300
  redis:
    addr: localhost:6379
    password: ""
    db: 0
```

### 6.6 日志配置

```yaml
log:
  level: debug             # debug | info | warn | error
  output: stdout           # stdout | file
  file_path: ./logs/dbridge.log
  format: console          # console | json
```

---

## 7. 生产环境建议

### 7.1 数据库

- 生产环境建议使用 MySQL 或 PostgreSQL，而非 SQLite
- 配置数据库连接池参数（`max_open_conns`、`max_idle_conns`）
- 定期备份数据库

### 7.2 缓存

- 多实例部署时使用 Redis 作为缓存后端
- 单机部署可以使用 local 缓存

### 7.3 安全

- 修改 JWT Secret 为随机强密码
- 启用 HTTPS（通过反向代理）
- 配置 CORS 允许的域名
- 启用审计日志自动清理

### 7.4 反向代理

推荐使用 Nginx 作为反向代理：

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

        # SSE 支持（AI 对话流式输出）
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 300s;
    }
}
```

### 7.5 资源要求

| 规模 | CPU | 内存 | 磁盘 |
|------|-----|------|------|
| 小型（< 10 数据源） | 2 核 | 4 GB | 20 GB |
| 中型（10-50 数据源） | 4 核 | 8 GB | 50 GB |
| 大型（> 50 数据源） | 8 核 | 16 GB | 100 GB |

---

## 8. 常见问题

### 8.1 端口被占用

修改 `config.yaml` 中的 `server.port`，或使用 Docker 端口映射：

```bash
docker run -d -p 9090:8082 dbridge/dbridge-pro:latest
```

### 8.2 数据库连接失败

检查数据库配置是否正确，确认数据库服务已启动且网络可达。

### 8.3 AI 功能不可用

检查 AI 配置中的 `api_key` 和 `base_url` 是否正确，确认可以访问 AI 服务端点。

### 8.4 日志在哪里

- Docker 部署：`docker logs dbridge`
- 源码部署：查看 `config.yaml` 中配置的 `log.file_path`

### 8.5 如何备份

备份以下数据：
- 数据库文件（`data/` 目录或外部数据库）
- 配置文件（`configs/config.yaml`）
- 存储文件（如果使用了本地存储）
