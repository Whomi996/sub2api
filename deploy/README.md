# Sub2API 部署文件

该目录包含用于在 Linux 服务器上部署 Sub2API 的文件。

## 部署方法

|方法|最适合 |设置向导 |
|--------|----------|--------------|
| **Docker 撰写** |快速设置，一体化 |不需要（自动设置）|
| **二进制安装** |生产服务器，systemd |基于网络的向导 |

## 文件

|文件 |描述 |
|------|-------------|
| `docker-compose.yml` | Docker Compose 配置（命名卷）|
| `docker-compose.local.yml` | Docker Compose配置（本地目录，轻松迁移）|
| `docker-deploy.sh` | **一键Docker部署脚本（推荐）** |
| `.env.example` | Docker环境变量模板|
| `DOCKER.md` | Docker Hub 文档 |
| `install.sh` |一键二进制安装脚本 |
| `sub2api.service` | Systemd 服务单元文件 |
| `config.example.yaml` |配置文件示例 |

---

## Docker 部署（推荐）

### 方法一：一键部署（推荐）

使用自动准备脚本进行最简单的设置：

```bash
# Download and run the preparation script
curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/docker-deploy.sh | bash

# Or download first, then run
curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/docker-deploy.sh -o docker-deploy.sh
chmod +x docker-deploy.sh
./docker-deploy.sh
```

**脚本的作用：**
- 下载 `docker-compose.local.yml` 和 `.env.example`
- 自动生成安全秘密（JWT_SECRET、TOTP_ENCRYPTION_KEY、POSTGRES_PASSWORD）
- 使用生成的机密创建 `.env` 文件
- 创建必要的数据目录（data/、postgres_data/、redis_data/）
- **显示生成的凭据**（POSTGRES_PASSWORD、JWT_SECRET 等）

**运行脚本后：**
```bash
# Start services
docker-compose -f docker-compose.local.yml up -d

# View logs
docker-compose -f docker-compose.local.yml logs -f sub2api

# If admin password was auto-generated, find it in logs:
docker-compose -f docker-compose.local.yml logs sub2api | grep "admin password"

# Access Web UI
# http://localhost:8080
```

### 方法二：手动部署

如果您喜欢手动控制：

```bash
# Clone repository
git clone https://github.com/Wei-Shaw/sub2api.git
cd sub2api/deploy

# Configure environment
cp .env.example .env
nano .env  # Set POSTGRES_PASSWORD and other required variables

# Generate secure secrets (recommended)
JWT_SECRET=$(openssl rand -hex 32)
TOTP_ENCRYPTION_KEY=$(openssl rand -hex 32)
echo "JWT_SECRET=${JWT_SECRET}" >> .env
echo "TOTP_ENCRYPTION_KEY=${TOTP_ENCRYPTION_KEY}" >> .env

# Create data directories
mkdir -p data postgres_data redis_data

# Start all services using local directory version
docker-compose -f docker-compose.local.yml up -d

# View logs (check for auto-generated admin password)
docker-compose -f docker-compose.local.yml logs -f sub2api

# Access Web UI
# http://localhost:8080
```

### 部署版本比较

|版本 |数据存储|移民|最适合 |
|---------|-------------|-----------|----------|
| **docker-compose.local.yml** |本地目录（./data、./postgres_data、./redis_data）| ✅ 简单（tar 整个目录）|生产，需要频繁备份/迁移 |
| **docker-compose.yml** |命名卷 (/var/lib/docker/volumes/) | ⚠️ 需要 docker 命令 |设置简单，无需迁移 |

**建议：** 使用`docker-compose.local.yml`（由`docker-deploy.sh`部署）以更轻松地进行数据管理和迁移。

### 自动设置如何工作

当使用带有 `AUTO_SETUP=true` 的 Docker Compose 时：

1. 首次运行时，系统自动：
- 连接到 PostgreSQL 和 Redis
- 应用数据库迁移（`backend/migrations/*.sql` 中的 SQL 文件）并将其记录在 `schema_migrations` 中
- 生成 JWT 秘密（如果未提供）
- 创建管理员帐户（如果未提供则自动生成密码）
- 写入config.yaml

2. 无需手动设置向导 - 只需配置 `.env` 并启动

3. 如果未设置 `ADMIN_PASSWORD`，请检查日志中是否有生成的密码：
   ```bash
   docker-compose logs sub2api | grep "admin password"
   ```

### 数据库迁移笔记（PostgreSQL）

- 迁移按字典顺序应用（例如 `001_...sql`、`002_...sql`）。
- `schema_migrations` 跟踪应用的迁移（文件名+校验和）。
- 迁移是向前的；回滚需要数据库备份恢复或手动补偿SQL脚本。

**验证 `users.allowed_groups` → `user_allowed_groups` 回填**

在增量 GORM→Ent 迁移期间，`users.allowed_groups`（旧版 `BIGINT[]`）被规范化连接表 `user_allowed_groups(user_id, group_id)` 替换。

运行此查询来比较旧数据与连接表：

```sql
WITH old_pairs AS (
  SELECT DISTINCT u.id AS user_id, x.group_id
  FROM users u
  CROSS JOIN LATERAL unnest(u.allowed_groups) AS x(group_id)
  WHERE u.allowed_groups IS NOT NULL
)
SELECT
  (SELECT COUNT(*) FROM old_pairs)           AS old_pair_count,
  (SELECT COUNT(*) FROM user_allowed_groups) AS new_pair_count;
```

### 命令

对于**本地目录版本**（docker-compose.local.yml）：

```bash
# Start services
docker-compose -f docker-compose.local.yml up -d

# Stop services
docker-compose -f docker-compose.local.yml down

# View logs
docker-compose -f docker-compose.local.yml logs -f sub2api

# Restart Sub2API only
docker-compose -f docker-compose.local.yml restart sub2api

# Update to latest version
docker-compose -f docker-compose.local.yml pull
docker-compose -f docker-compose.local.yml up -d

# Remove all data (caution!)
docker-compose -f docker-compose.local.yml down
rm -rf data/ postgres_data/ redis_data/
```

对于 **命名卷版本** (docker-compose.yml)：

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f sub2api

# Restart Sub2API only
docker-compose restart sub2api

# Update to latest version
docker-compose pull
docker-compose up -d

# Remove all data (caution!)
docker-compose down -v
```

### 环境变量

|变量|必填|默认 |描述 |
|----------|----------|---------|-------------|
| `POSTGRES_PASSWORD` | **是** | - | PostgreSQL 密码 |
| `JWT_SECRET` | **推荐** | *（自动生成）* | JWT 秘密（针对持久会话进行了修复）|
| `TOTP_ENCRYPTION_KEY` | **推荐** | *（自动生成）* | TOTP 加密密钥（针对持久 2FA 进行了修复）|
| `SERVER_PORT` |没有 | `8080` |服务器端口|
| `ADMIN_EMAIL` |没有 | `admin@sub2api.local` |管理员邮箱|
| `ADMIN_PASSWORD` |没有 | *（自动生成）* |管理员密码 |
| `TZ` |没有 | `Asia/Shanghai` |时区 |
| `GEMINI_OAUTH_CLIENT_ID` |没有 | *（内置）* | Google OAuth 客户端 ID (Gemini OAuth)。留空以使用内置 Gemini CLI 客户端。 |
| `GEMINI_OAUTH_CLIENT_SECRET` |没有 | *（内置）* | Google OAuth 客户端密钥（Gemini OAuth）。留空以使用内置 Gemini CLI 客户端。 |
| `GEMINI_OAUTH_SCOPES` |没有 | *（默认）* | OAuth 范围 (Gemini OAuth) |
| `GEMINI_QUOTA_POLICY` |没有 | *（空）* |用于 Gemini 本地配额模拟的 JSON 覆盖（仅限 Code Assist）。 |

请参阅 `.env.example` 了解所有可用选项。

> **注意：** `docker-deploy.sh` 脚本会自动为您生成 `JWT_SECRET`、`TOTP_ENCRYPTION_KEY` 和 `POSTGRES_PASSWORD`。

### 轻松迁移（本地目录版本）

当使用 `docker-compose.local.yml` 时，所有数据都存储在本地目录中，使迁移变得简单：

```bash
# On source server: Stop services and create archive
cd /path/to/deployment
docker-compose -f docker-compose.local.yml down
cd ..
tar czf sub2api-complete.tar.gz deployment/

# Transfer to new server
scp sub2api-complete.tar.gz user@new-server:/path/to/destination/

# On new server: Extract and start
tar xzf sub2api-complete.tar.gz
cd deployment/
docker-compose -f docker-compose.local.yml up -d
```

您的整个部署（配置+数据）已迁移！

---

## Gemini OAuth 配置

Sub2API 支持三种连接 Gemini 的方法：

### 方法 1：Code Assist OAuth（推荐给 GCP 用户）

**无需配置** - 始终使用内置的 Gemini CLI OAuth 客户端（公共）。

1. 将 `GEMINI_OAUTH_CLIENT_ID` 和 `GEMINI_OAUTH_CLIENT_SECRET` 留空
2. 在管理 UI 中，创建一个 Gemini OAuth 帐户并选择 **“Code Assist”** 类型
3. 在浏览器中完成 OAuth 流程

> 注意：即使您为 AI Studio OAuth 配置了 `GEMINI_OAUTH_CLIENT_ID` / `GEMINI_OAUTH_CLIENT_SECRET`，
> Code Assist OAuth 仍将使用内置的 Gemini CLI 客户端。

**要求：**
- 可访问 Google Cloud Platform 的 Google 帐户
- GCP项目（自动检测或手动指定）

**如何获取项目ID（如果自动检测失败）：**
1. 前往[Google Cloud Console](https://console.cloud.google.com/)
2. 单击页面顶部的项目下拉列表
3. 从列表中复制项目 ID（不是项目名称）
4. 常用格式：`my-project-123456` 或 `cloud-ai-companion-xxxxx`

### 方法 2：AI Studio OAuth（适用于常规 Google 帐户）

需要您自己的 OAuth 客户端凭据。

**步骤 1：在 Google Cloud Console 中创建 OAuth 客户端**

1. 前往[Google Cloud Console - Credentials](https://console.cloud.google.com/apis/credentials)
2. 创建一个新项目或选择现有项目
3. **启用生成语言API：**
- 转到“API 和服务”→“库”
- 搜索“生成语言 API”
- 单击“启用”
4. **配置 OAuth 同意屏幕**（如果未完成）：
- 转到“API 和服务”→“OAuth 同意屏幕”
- 选择“外部”用户类型
- 填写应用名称、用户支持电子邮件、开发者联系方式
- 添加范围：`https://www.googleapis.com/auth/generative-language.retriever`（以及可选的 `https://www.googleapis.com/auth/cloud-platform`）
- 添加测试用户（您的Google帐户电子邮件）
5. **创建 OAuth 2.0 凭据：**
- 转到“API 和服务”→“凭据”
- 单击“创建凭据”→“OAuth 客户端 ID”
- 应用程序类型：**Web 应用程序**（或 **桌面应用程序**）
- 名称：例如“Sub2API Gemini”
- 授权重定向 URI：添加 `http://localhost:1455/auth/callback`
6. 复制 **客户端 ID** 和 **客户端密钥**
7. **⚠️ 发布到生产环境（重要）：**
- 转到“API 和服务”→“OAuth 同意屏幕”
- 单击“发布应用程序”从测试转移到生产
- **测试模​​式限制：**
- 只有手动添加的测试用户才能进行身份验证（最多 100 个用户）
- 刷新令牌将在 7 天后过期
- 必须定期重新添加用户
- **生产模式：**任何Google用户都可以进行身份​​验证，令牌不会过期
- 注意：对于敏感范围，Google 可能需要验证（演示视频、隐私政策）

**步骤2：配置环境变量**

```bash
GEMINI_OAUTH_CLIENT_ID=your-client-id.apps.googleusercontent.com
GEMINI_OAUTH_CLIENT_SECRET=GOCSPX-your-client-secret
```

**第 3 步：在管理 UI 中创建帐户**

1.创建Gemini OAuth账户并选择**“AI Studio”**类型
2. 完成OAuth流程
- 同意后，您的浏览器将被重定向到`http://localhost:1455/auth/callback?code=...&state=...`
- 复制完整的回调 URL（推荐）或仅复制 `code` 并将其粘贴回管理 UI

### 方法 3：API 密钥（最简单）

1. 前往[Google AI Studio](https://aistudio.google.com/app/apikey)
2. 点击“创建API密钥”
3. 在管理界面中，创建一个 Gemini **API Key** 帐户
4. 粘贴​​您的 API 密钥（以 `AIza...` 开头）

### 比较表

|特色 |代码辅助 OAuth | AI Studio OAuth | API 密钥 |
|---------|-------------------|-----------------|---------|
|设置复杂性 |简单（无需配置）|中（OAuth 客户端）|简单|
|需要 GCP 项目 |是的 |没有 |没有 |
|自定义 OAuth 客户端 |否（内置）|是（必填）|不适用 |
|速率限制 | GCP 配额 |标准|标准|
|最适合 | GCP 开发者 |需要 OAuth 的普通用户 |快速测试 |

---

## 二进制安装

对于使用 systemd 的生产服务器。

### 一行安装

```bash
curl -sSL https://raw.githubusercontent.com/Wei-Shaw/sub2api/main/deploy/install.sh | sudo bash
```

### 手动安装

1. 从[GitHub Releases](https://github.com/Wei-Shaw/sub2api/releases)下载最新版本
2. 提取二进制文件并将其复制到 `/opt/sub2api/`
3. 将 `sub2api.service` 复制到 `/etc/systemd/system/`
4. 运行：
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable sub2api
   sudo systemctl start sub2api
   ```
5. 在浏览器中打开设置向导完成配置

### 命令

```bash
# Install
sudo ./install.sh

# Upgrade
sudo ./install.sh upgrade

# Uninstall
sudo ./install.sh uninstall
```

### 服务管理

```bash
# Start the service
sudo systemctl start sub2api

# Stop the service
sudo systemctl stop sub2api

# Restart the service
sudo systemctl restart sub2api

# Check status
sudo systemctl status sub2api

# View logs
sudo journalctl -u sub2api -f

# Enable auto-start on boot
sudo systemctl enable sub2api
```

### Configuration

#### 服务器地址和端口

安装过程中会提示配置服务器监听地址和端口。这些设置作为环境变量存储在 systemd 服务文件中。

安装后更改：

1.编辑systemd服务：
   ```bash
   sudo systemctl edit sub2api
   ```

2.添加或修改：
   ```ini
   [Service]
   Environment=SERVER_HOST=0.0.0.0
   Environment=SERVER_PORT=3000
   ```

3. 重新加载并重启：
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart sub2api
   ```

#### Gemini OAuth 配置

如果您需要对 Gemini 帐户使用 AI Studio OAuth，请将 OAuth 客户端凭据添加到 systemd 服务文件中：

1.编辑服务文件：
   ```bash
   sudo nano /etc/systemd/system/sub2api.service
   ```

2. 在 `[Service]` 部分添加您的 OAuth 凭据（在现有 `Environment=` 行之后）：
   ```ini
   Environment=GEMINI_OAUTH_CLIENT_ID=your-client-id.apps.googleusercontent.com
   Environment=GEMINI_OAUTH_CLIENT_SECRET=GOCSPX-your-client-secret
   ```

3. 重新加载并重启：
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart sub2api
   ```

> **注意：** Code Assist OAuth 不需要任何配置 - 它使用内置的 Gemini CLI 客户端。
> 有关详细设置说明，请参阅上面的 [Gemini OAuth Configuration](#gemini-oauth-configuration) 部分。

#### 应用程序配置

主配置文件位于 `/etc/sub2api/config.yaml`（由安装向导创建）。

### 先决条件

- Linux 服务器（Ubuntu 20.04+、Debian 11+、CentOS 8+ 等）
- PostgreSQL 14+
- 雷迪斯 6+
- 系统

### 目录结构

```
/opt/sub2api/
├── sub2api              # Main binary
├── sub2api.backup       # Backup (after upgrade)
└── data/                # Runtime data

/etc/sub2api/
└── config.yaml          # Configuration file
```

---

## 故障排除

### 码头工人

对于**本地目录版本**：

```bash
# Check container status
docker-compose -f docker-compose.local.yml ps

# View detailed logs
docker-compose -f docker-compose.local.yml logs --tail=100 sub2api

# Check database connection
docker-compose -f docker-compose.local.yml exec postgres pg_isready

# Check Redis connection
docker-compose -f docker-compose.local.yml exec redis redis-cli ping

# Restart all services
docker-compose -f docker-compose.local.yml restart

# Check data directories
ls -la data/ postgres_data/ redis_data/
```

对于**命名卷版本**：

```bash
# Check container status
docker-compose ps

# View detailed logs
docker-compose logs --tail=100 sub2api

# Check database connection
docker-compose exec postgres pg_isready

# Check Redis connection
docker-compose exec redis redis-cli ping

# Restart all services
docker-compose restart
```

### 二进制安装

```bash
# Check service status
sudo systemctl status sub2api

# View recent logs
sudo journalctl -u sub2api -n 50

# Check config file
sudo cat /etc/sub2api/config.yaml

# Check PostgreSQL
sudo systemctl status postgresql

# Check Redis
sudo systemctl status redis
```

### 常见问题

1. **端口已在使用**：更改 `.env` 或 systemd 配置中的 `SERVER_PORT`
2. **数据库连接失败**：检查 PostgreSQL 是否正在运行并且凭据是否正确
3. **Redis连接失败**：检查Redis是否正在运行且密码是否正确
4. **权限被拒绝**：确保二进制安装的正确文件所有权

---

## TLS 指纹配置

Sub2API 支持 TLS 指纹模拟，使请求看起来就像来自官方 Claude CLI（Node.js 客户端）。

> **💡提示：**访问**[tls.sub2api.org](https://tls.sub2api.org/)**可获取不同设备和浏览器的TLS指纹信息。

### 默认行为

- 内置 `claude_cli_v2` 配置文件模拟 Node.js 20.x + OpenSSL 3.x
- JA3 哈希值：`1a28e69016765d92e3b381168d68922c`
- JA4：`t13d5911h1_a33745022dd6_1f22a2ca17c4`
- 配置文件选择：`accountID % profileCount`

### Configuration

```yaml
gateway:
  tls_fingerprint:
    enabled: true  # Global switch
    profiles:
      # Simple profile (uses default cipher suites)
      profile_1:
        name: "Profile 1"

      # Profile with custom cipher suites (use compact array format)
      profile_2:
        name: "Profile 2"
        cipher_suites: [4866, 4867, 4865, 49199, 49195, 49200, 49196]
        curves: [29, 23, 24]
        point_formats: [0]

      # Another custom profile
      profile_3:
        name: "Profile 3"
        cipher_suites: [4865, 4866, 4867, 49199, 49200]
        curves: [29, 23, 24, 25]
```

### 配置文件字段

|领域|类型 |描述 |
|-------|------|-------------|
| `name` |字符串|显示名称（必填）|
| `cipher_suites` | []uint16 |十进制密码套件。空=默认|
| `curves` | []uint16 |十进制椭圆曲线。空=默认|
| `point_formats` | []uint8 | EC 点格式。空=默认|

### 通用值参考

**密码套件 (TLS 1.3)：** `4865` (AES_128_GCM)、`4866` (AES_256_GCM)、`4867` (CHACHA20)

**密码套件 (TLS 1.2)：** `49195`、`49196`、`49199`、`49200`（ECDHE 变体）

**曲线：** `29` (X25519)、`23` (P-256)、`24` (P-384)、`25` (P-521)
