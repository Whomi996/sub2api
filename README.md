# 子2API

<div align="center">

[![Go](https://img.shields.io/badge/Go-1.25.7-00ADD8.svg)](https://golang.org/)
[![Vue](https://img.shields.io/badge/Vue-3.4+-4FC08D.svg)](https://vuejs.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791.svg)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-7+-DC382D.svg)](https://redis.io/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED.svg)](https://www.docker.com/)

**订阅配额分配的AI API网关平台**

中文文档

</div>

---

## 演示

在线尝试 Sub2API：**https://demo.sub2api.org/**

演示凭据（共享演示环境；**不是**为自托管安装自动创建）：

|电子邮件 |密码 |
|-------|----------|
| admin@sub2api.com |管理员123 |

## Overview

Sub2API 是一个 AI API 网关平台，旨在分配和管理 AI 产品订阅的 API 配额（例如 Claude Code 200 美元/月）。用户可以通过平台生成的API Key访问上游AI服务，而平台则处理身份验证、计费、负载均衡和请求转发。

## Features

- **多账户管理** - 支持多种上游账户类型（OAuth、API Key）
- **API密钥分发** - 为用户生成和管理API密钥
- **精确计费** - 令牌级使用跟踪和成本计算
- **智能调度** - 具有粘性会话的智能帐户选择
- **并发控制** - 每用户和每帐户并发限制
- **速率限制** - 可配置的请求和令牌速率限制
- **管理仪表板** - 用于监控和管理的 Web 界面

## 技术堆栈

|组件|技术 |
|-----------|------------|
|后端| Go 1.25.7，杜松子酒，Ent |
|前端 | Vue 3.4+、Vite 5+、TailwindCSS |
|数据库| PostgreSQL 15+ |
|缓存/队列| Redis 7+ |

---

## 文档

- 依赖安全：`docs/dependency-security.md`

---

## 部署

### 方法一：脚本安装（推荐）

一键安装脚本，从 GitHub Releases 下载预构建的二进制文件。

#### 先决条件

- Linux服务器（amd64或arm64）
- PostgreSQL 15+（已安装并正在运行）
- Redis 7+（已安装并正在运行）
- 根权限

#### 安装步骤

```bash
curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/install.sh | sudo bash
```

该脚本将：
1. 检测您的系统架构
2.下载最新版本
3. 将二进制文件安装到 `/opt/sub2api`
4.创建systemd服务
5.配置系统用户及权限

#### 安装后

```bash
# 1. Start the service
sudo systemctl start sub2api

# 2. Enable auto-start on boot
sudo systemctl enable sub2api

# 3. Open Setup Wizard in browser
# http://YOUR_SERVER_IP:8080
```

设置向导将引导您完成：
- 数据库配置
-Redis配置
- 管理员帐户创建

#### Upgrade

您可以通过单击左上角的“检查更新”按钮直接从“管理仪表板”进行升级。

网络界面将：
- 自动检查新版本
- 一键下载并应用更新
- 如果需要的话支持回滚

#### 有用的命令

```bash
# Check status
sudo systemctl status sub2api

# View logs
sudo journalctl -u sub2api -f

# Restart service
sudo systemctl restart sub2api

# Uninstall
curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/install.sh | sudo bash -s -- uninstall -y
```

---

### 方法2：Docker Compose（推荐）

使用 Docker Compose 进行部署，包括 PostgreSQL 和 Redis 容器。

#### 先决条件

- Docker 20.10+
- Docker Compose v2+

#### 快速入门（一键部署）

使用自动部署脚本轻松设置：

```bash
# Create deployment directory
mkdir -p sub2api-deploy && cd sub2api-deploy

# Download and run deployment preparation script
curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/docker-deploy.sh | bash

# Start services
docker-compose -f docker-compose.local.yml up -d

# View logs
docker-compose -f docker-compose.local.yml logs -f sub2api
```

**脚本的作用：**
- 下载 `docker-compose.local.yml` 和 `.env.example`
- 生成安全凭证（JWT_SECRET、TOTP_ENCRYPTION_KEY、POSTGRES_PASSWORD）
- 使用自动生成的机密创建 `.env` 文件
- 创建数据目录（使用本地目录以便于备份/迁移）
- 显示生成的凭据供您参考

#### 手动部署

如果您更喜欢手动设置：

```bash
# 1. Clone the repository
git clone https://github.com/Wei-Shaw/sub2api.git
cd sub2api/deploy

# 2. Copy environment configuration
cp .env.example .env

# 3. Edit configuration (generate secure passwords)
nano .env
```

**`.env` 中所需的配置：**

```bash
# PostgreSQL password (REQUIRED)
POSTGRES_PASSWORD=your_secure_password_here

# JWT Secret (RECOMMENDED - keeps users logged in after restart)
JWT_SECRET=your_jwt_secret_here

# TOTP Encryption Key (RECOMMENDED - preserves 2FA after restart)
TOTP_ENCRYPTION_KEY=your_totp_key_here

# Optional: Admin account
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=your_admin_password

# Optional: Custom port
SERVER_PORT=8080
```

**生成安全秘密：**
```bash
# Generate JWT_SECRET
openssl rand -hex 32

# Generate TOTP_ENCRYPTION_KEY
openssl rand -hex 32

# Generate POSTGRES_PASSWORD
openssl rand -hex 32
```

```bash
# 4. Create data directories (for local version)
mkdir -p data postgres_data redis_data

# 5. Start all services
# Option A: Local directory version (recommended - easy migration)
docker-compose -f docker-compose.local.yml up -d

# Option B: Named volumes version (simple setup)
docker-compose up -d

# 6. Check status
docker-compose -f docker-compose.local.yml ps

# 7. View logs
docker-compose -f docker-compose.local.yml logs -f sub2api
```

#### 部署版本

|版本 |数据存储|移民|最适合 |
|---------|-------------|-----------|----------|
| **docker-compose.local.yml** |本地目录 | ✅ 简单（tar 整个目录）|生产、频繁备份|
| **docker-compose.yml** |命名卷 | ⚠️ 需要 docker 命令 |简单设置|

**建议：** 使用 `docker-compose.local.yml` （由脚本部署）以更轻松地进行数据管理。

#### Access

在浏览器中打开 `http://YOUR_SERVER_IP:8080`。

如果管理员密码是自动生成的，请在日志中找到它：
```bash
docker-compose -f docker-compose.local.yml logs sub2api | grep "admin password"
```

#### Upgrade

```bash
# Pull latest image and recreate container
docker-compose -f docker-compose.local.yml pull
docker-compose -f docker-compose.local.yml up -d
```

#### 轻松迁移（本地目录版本）

使用 `docker-compose.local.yml` 时，可以轻松迁移到新服务器：

```bash
# On source server
docker-compose -f docker-compose.local.yml down
cd ..
tar czf sub2api-complete.tar.gz sub2api-deploy/

# Transfer to new server
scp sub2api-complete.tar.gz user@new-server:/path/

# On new server
tar xzf sub2api-complete.tar.gz
cd sub2api-deploy/
docker-compose -f docker-compose.local.yml up -d
```

#### 有用的命令

```bash
# Stop all services
docker-compose -f docker-compose.local.yml down

# Restart
docker-compose -f docker-compose.local.yml restart

# View all logs
docker-compose -f docker-compose.local.yml logs -f

# Remove all data (caution!)
docker-compose -f docker-compose.local.yml down
rm -rf data/ postgres_data/ redis_data/
```

---

### 方法 3：从源代码构建

从源代码构建并运行以进行开发或定制。

#### 先决条件

- 去 1.21+
- Node.js 18+
- PostgreSQL 15+
- Redis 7+

#### 构建步骤

```bash
# 1. Clone the repository
git clone https://github.com/Wei-Shaw/sub2api.git
cd sub2api

# 2. Install pnpm (if not already installed)
npm install -g pnpm

# 3. Build frontend
cd frontend
pnpm install
pnpm run build
# Output will be in ../backend/internal/web/dist/

# 4. Build backend with embedded frontend
cd ../backend
go build -tags embed -o sub2api ./cmd/server

# 5. Create configuration file
cp ../deploy/config.example.yaml ./config.yaml

# 6. Edit configuration
nano config.yaml
```

> **注意：** `-tags embed` 标志将前端嵌入到二进制文件中。如果没有此标志，二进制文件将无法为前端 UI 提供服务。

**`config.yaml`中的关键配置：**

```yaml
server:
  host: "0.0.0.0"
  port: 8080
  mode: "release"

database:
  host: "localhost"
  port: 5432
  user: "postgres"
  password: "your_password"
  dbname: "sub2api"

redis:
  host: "localhost"
  port: 6379
  password: ""

jwt:
  secret: "change-this-to-a-secure-random-string"
  expire_hour: 24

default:
  user_concurrency: 5
  user_balance: 0
  api_key_prefix: "sk-"
  rate_multiplier: 1.0
```

`config.yaml` 中提供了其他与安全相关的选项：

- `cors.allowed_origins` 表示 CORS 许可名单
- `security.url_allowlist` 用于上游/定价/CRS 主机白名单
- `security.url_allowlist.enabled` 禁用 URL 验证（谨慎使用）
- `security.url_allowlist.allow_insecure_http` 在禁用验证时允许 HTTP URL
- `security.url_allowlist.allow_private_hosts` 允许私有/本地 IP 地址
- `security.response_headers.enabled` 启用可配置的响应标头过滤（禁用使用默认白名单）
- `security.csp` 控制内容安全策略标头
- `billing.circuit_breaker` 因计费错误而无法关闭
- `server.trusted_proxies` 启用 X-Forwarded-For 解析
- `turnstile.required` 要求旋转门处于释放模式

**⚠️ 安全警告：HTTP URL 配置**

当 `security.url_allowlist.enabled=false` 时，系统默认执行最少的 URL 验证，**拒绝 HTTP URL**，仅允许 HTTPS。要允许 HTTP URL（例如，用于开发或内部测试），您必须显式设置：

```yaml
security:
  url_allowlist:
    enabled: false                # Disable allowlist checks
    allow_insecure_http: true     # Allow HTTP URLs (⚠️ INSECURE)
```

**或通过环境变量：**

```bash
SECURITY_URL_ALLOWLIST_ENABLED=false
SECURITY_URL_ALLOWLIST_ALLOW_INSECURE_HTTP=true
```

**允许 HTTP 的风险：**
- API 密钥和数据以 **明文** 传输（容易被拦截）
- 容易受到**中间人 (MITM) 攻击**
- **不适合生产**环境

**何时使用 HTTP：**
- ✅ 使用本地服务器进行开发/测试 (http://localhost)
- ✅ 具有受信任端点的内部网络
- ✅ 在获取 HTTPS 之前测试帐户连接
- ❌ 生产环境（仅使用 HTTPS）

**没有此设置的错误示例：**
```
Invalid base URL: invalid url scheme: http
```

如果禁用 URL 验证或响应标头过滤，请强化网络层：
- 对上游域/IP 强制执行出口允许列表
- 阻止私有/环回/本地链路范围
- 强制执行仅 TLS 出站流量
- 在代理处剥离敏感的上游响应标头

```bash
# 6. Run the application
./sub2api
```

#### 开发模式

```bash
# Backend (with hot reload)
cd backend
go run ./cmd/server

# Frontend (with hot reload)
cd frontend
pnpm run dev
```

#### 代码生成

编辑`backend/ent/schema`时，重新生成Ent + Wire：

```bash
cd backend
go generate ./ent
go generate ./cmd/server
```

---

## 简单模式

简单模式专为希望快速访问而不需要完整 SaaS 功能的个人开发人员或内部团队而设计。

- 启用：设置环境变量`RUN_MODE=simple`
- 区别：隐藏 SaaS 相关功能并跳过计费流程
- 安全说明：在生产中，您还必须设置 `SIMPLE_MODE_CONFIRM=true` 以允许启动

---

## 反重力支持

Sub2API 支持 [Antigravity](https://antigravity.so/) 帐户。授权后，专用端点可用于 Claude 和 Gemini 型号。

### 专用端点

|端点|型号|
|----------|-------|
| `/antigravity/v1/messages` |克劳德模型|
| `/antigravity/v1beta/` |双子座车型|

### 克劳德代码配置

```bash
export ANTHROPIC_BASE_URL="http://localhost:8080/antigravity"
export ANTHROPIC_AUTH_TOKEN="sk-xxx"
```

### 混合调度模式

反重力帐户支持可选的**混合调度**。启用后，通用端点 `/v1/messages` 和 `/v1beta/` 也会将请求路由到反重力帐户。

> **⚠️警告**：人类克劳德和反重力克劳德**不能在同一对话上下文中混合**。使用组来正确隔离它们。

### 已知问题

在克劳德代码中，计划模式无法自动退出。 （通常使用原生 Claude API 时，规划完成后，Claude Code 会弹出选项供用户批准或拒绝该规划。）

**解决方法**：按 `Shift + Tab` 手动退出计划模式，然后键入您的回复以批准或拒绝该计划。

---

## 项目结构

```
sub2api/
├── backend/                  # Go backend service
│   ├── cmd/server/           # Application entry
│   ├── internal/             # Internal modules
│   │   ├── config/           # Configuration
│   │   ├── model/            # Data models
│   │   ├── service/          # Business logic
│   │   ├── handler/          # HTTP handlers
│   │   └── gateway/          # API gateway core
│   └── resources/            # Static resources
│
├── frontend/                 # Vue 3 frontend
│   └── src/
│       ├── api/              # API calls
│       ├── stores/           # State management
│       ├── views/            # Page components
│       └── components/       # Reusable components
│
└── deploy/                   # Deployment files
    ├── docker-compose.yml    # Docker Compose configuration
    ├── .env.example          # Environment variables for Docker Compose
    ├── config.example.yaml   # Full config file for binary deployment
    └── install.sh            # One-click installation script
```

## License

麻省理工学院许可证

---

<div align="center">

**如果您觉得这个项目有用，请给它一个star！**

</div>
