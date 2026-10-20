# 快速开始

本章介绍如何安装、启动并首次使用 DBridge Pro。

## 环境要求

### 从源码构建

| 依赖 | 版本要求 | 说明 |
|:-----|:---------|:-----|
| Go | 1.26+ | 后端编译，需要 CGO 支持（SQLite 驱动依赖） |
| Node.js | 18+ | 前端编译 |
| C 编译器 | gcc / clang / zig | CGO 编译 SQLite 驱动 |

### 使用预编译二进制

直接下载对应平台的二进制文件即可运行，无需安装任何依赖。

| 平台 | 二进制文件 | CGO 编译器 |
|:-----|:-----------|:-----------|
| Linux (amd64/arm64) | `dbridge-pro` | gcc / cc / clang |
| macOS (amd64/arm64) | `dbridge-pro` | clang（Xcode CLT） |
| Windows (amd64/arm64) | `dbridge-pro.exe` | gcc 或 zig cc |

## 一键构建与启动

```bash
# 1. 克隆项目
git clone https://github.com/dbridge-ai/dbridge.git
cd dbridge

# 2. 构建全栈（后端 + 前端）
./build.sh

# 3. 启动服务
./start.sh

# 4. 打开浏览器访问
#    http://localhost:8082
```

首次启动时，系统会自动：
1. 创建 SQLite 数据库文件（`./data/dbridge.db`）
2. 初始化表结构
3. 创建默认管理员账号
4. 内置 AI 技能、Agent、MCP 工具等系统数据

## 默认账号

| 认证模式 | 账号 | 密码 |
|:---------|:-----|:-----|
| password（本地账号） | admin | 123456 |
| usercenter（OAuth） | 通过 OAuth 提供商登录 | — |

> 首次登录后请立即修改默认密码。

## 启动脚本详解

DBridge Pro 提供了一套跨平台（Linux / macOS / Windows Git Bash）的管理脚本：

```bash
./build.sh   [release] [backend|frontend|all|force]
./start.sh   [prod|dev] [--port N] [--foreground] [--no-build]
./stop.sh    [--force] [--port N]
./restart.sh [prod|dev] [--port N] [--build|--no-build]
./status.sh  [--port N] [--env]
```

### build.sh — 构建

```bash
./build.sh              # 默认 debug 模式，构建全栈
./build.sh release      # release 模式（去除调试信息，体积更小）
./build.sh release backend   # 仅构建后端
./build.sh release frontend  # 仅构建前端
./build.sh force        # 强制重新安装前端依赖
```

### start.sh — 启动

```bash
./start.sh                      # 默认 prod 模式，后台运行
./start.sh dev                  # dev 模式（端口冲突时仅警告，不拒绝）
./start.sh --port 9090          # 指定端口
./start.sh --foreground         # 前台运行（日志输出到终端）
./start.sh --no-build           # 跳过自动构建
```

端口优先级：`--port` > `PORT` 环境变量 > `configs/config.yaml` 中的 `server.port` > 默认 `8082`。

### stop.sh — 停止

```bash
./stop.sh              # 优雅停止（SIGTERM → 等待 → SIGKILL）
./stop.sh --force      # 强制停止（包括残留进程）
```

### restart.sh — 重启

```bash
./restart.sh                    # 重启（不重新构建）
./restart.sh --build            # 重新构建后端后重启
./restart.sh dev --port 9090    # dev 模式指定端口重启
```

### status.sh — 状态

```bash
./status.sh             # 查看运行状态
./status.sh --env       # 额外输出工具链版本信息
```

状态检查包括：
- 进程状态（PID、CPU、内存、运行时长）
- 端口监听状态
- 健康检查端点（`/health/live`、`/health/ready`）
- 二进制信息（大小、平台匹配）
- SQLite 数据库信息（文件大小、表数量）
- 日志文件信息

## Docker 部署

### Docker Compose（推荐）

```bash
docker-compose up -d
```

Docker Compose 包含 4 个服务：

| 服务 | 端口 | 说明 |
|:-----|:-----|:-----|
| app | 8082 | DBridge Pro 应用 |
| postgres | 5432 | PostgreSQL 14 元数据存储 |
| redis | 6379 | Redis 7 缓存 |
| nats | 4222 | NATS 2.10 消息队列 |

### 单独构建 Docker 镜像

```bash
docker build -t dbridge-pro .
docker run -d -p 8082:8082 -v ./data:/app/data dbridge-pro
```

## 前端开发模式

如果需要开发前端页面，可以单独启动前端开发服务器（支持热更新）：

```bash
cd web
npm install
npm run dev
# 访问 http://localhost:5172
```

开发服务器会自动将 `/api` 请求代理到后端 `http://localhost:8082`。

## 配置文件

配置文件位于 `configs/config.yaml`，主要配置项：

```yaml
# 服务端口
server:
  port: 8082

# 元数据存储
database:
  type: sqlite              # sqlite | mysql | postgresql
  sqlite:
    path: ./data/dbridge.db

# JWT 密钥（生产环境必须修改）
jwt:
  secret: change-me-in-production

# AI 配置
ai:
  provider: openai
  model: gpt-4o
  api_key: "your-api-key"
  base_url: ""              # 自定义 API 地址（兼容 OpenAI 协议）

# 认证模式
subscription:
  auth_mode: password       # password | usercenter
```

## 首次使用流程

1. **启动服务** — 运行 `./start.sh`
2. **打开浏览器** — 访问 `http://localhost:8082`
3. **登录** — 使用默认账号 admin / 123456
4. **添加数据源** — 进入「数据源管理」，点击「新建」，填写数据库连接信息
5. **测试连接** — 点击「测试连接」确认可以连通
6. **开始查询** — 进入「SQL 编辑器」，选择数据源，编写并执行 SQL

## 常见问题

### 启动失败：端口被占用

```bash
# 查看端口占用
lsof -i :8082

# 使用其他端口启动
./start.sh --port 9090
```

### 构建失败：找不到 C 编译器

SQLite 驱动需要 CGO，必须有 C 编译器：

```bash
# Linux
sudo apt install build-essential

# macOS
xcode-select --install

# Windows（使用 zig 作为 C 编译器）
# 下载 zig 并加入 PATH
```

### 前端依赖安装慢

```bash
npm install --registry=https://registry.npmmirror.com --noproxy="*"
```

### Go 模块下载慢

```bash
export GOPROXY=https://goproxy.cn,direct
```

### Windows 下脚本无法执行

确保使用 Git Bash 或 MSYS2 运行 `.sh` 脚本，不要在 CMD 或 PowerShell 中运行。
