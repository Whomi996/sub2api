# Sub2API Docker 镜像

Sub2API是一个AI API网关平台，用于分配和管理AI产品订阅API配额。

## 快速入门

```bash
docker run -d \
  --name sub2api \
  -p 8080:8080 \
  -e DATABASE_URL="postgres://user:pass@host:5432/sub2api" \
  -e REDIS_URL="redis://host:6379" \
  weishaw/sub2api:latest
```

## Docker 组合

```yaml
version: '3.8'

services:
  sub2api:
    image: weishaw/sub2api:latest
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/sub2api?sslmode=disable
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=sub2api
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## 环境变量

|变量|描述 |必填|默认 |
|----------|-------------|----------|---------|
| `DATABASE_URL` | PostgreSQL 连接字符串 |是的 | - |
| `REDIS_URL` | Redis 连接字符串 |是的 | - |
| `PORT` |服务器端口|没有 | `8080` |
| `GIN_MODE` | Gin 框架模式 (`debug`/`release`) |没有 | `release` |

## 支持的架构

- `linux/amd64`
- `linux/arm64`

## 标签

- `latest` - 最新稳定版本
- `x.y.z` - 特定版本
- `x.y` - 小版本最新补丁
- `x` - 主要版本的最新次要版本

## 链接

- [GitHub Repository](https://github.com/weishaw/sub2api)
- [Documentation](https://github.com/weishaw/sub2api#readme)
