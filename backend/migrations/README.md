# 数据库迁移

## Overview

此目录包含用于数据库架构更改的 SQL 迁移文件。迁移系统使用 SHA256 校验和来确保跨环境的迁移不变性和一致性。

## 迁移文件命名

格式：`NNN_description.sql`
- `NNN`：序列号（例如，001、002、003）
- `description`：snake_case 中的简要描述

示例：`017_add_gemini_tier_id.sql`

## 迁移文件结构

```sql
-- +goose Up
-- +goose StatementBegin
-- Your forward migration SQL here
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
-- Your rollback migration SQL here
-- +goose StatementEnd
```

## 重要规则

### ⚠️ 不变性原则

**一旦将迁移应用于任何环境（开发、登台、生产），就不得对其进行修改。**

为什么？
- 每个迁移都有一个 SHA256 校验和存储在 `schema_migrations` 表中
- 修改已应用的迁移会导致校验和不匹配错误
- 不同的环境会有不一致的数据库状态
- 破坏审计追踪和再现性

### ✅ 正确的工作流程

1. **创建新的迁移**
   ```bash
   # Create new file with next sequential number
   touch migrations/018_your_change.sql
   ```

2. **写入向上和向下迁移**
- 向上：应用更改
- 向下：恢复更改（应与向上对称）

3. **本地测试**
   ```bash
   # Apply migration
   make migrate-up

   # Test rollback
   make migrate-down
   ```

4. **提交和部署**
   ```bash
   git add migrations/018_your_change.sql
   git commit -m "feat(db): add your change"
   ```

### ❌ 不该做什么

- ❌修改已经应用的迁移文件
- ❌删除迁移文件
- ❌更改迁移文件名
- ❌ 重新排序迁移编号

### 🔧 如果您不小心修改了已应用的迁移

**错误信息：**
```
migration 017_add_gemini_tier_id.sql checksum mismatch (db=abc123... file=def456...)
```

**解决方案：**
```bash
# 1. Find the original version
git log --oneline -- migrations/017_add_gemini_tier_id.sql

# 2. Revert to the commit when it was first applied
git checkout <commit-hash> -- migrations/017_add_gemini_tier_id.sql

# 3. Create a NEW migration for your changes
touch migrations/018_your_new_change.sql
```

## 迁移系统详细信息

- **校验和算法**：修剪文件内容的 SHA256
- **跟踪表**：`schema_migrations`（文件名、校验和、apply_at）
- **跑步者**：`internal/repository/migrations_runner.go`
- **自动运行**：迁移在服务启动时自动运行

## 最佳实践

1. **保持迁移规模小且集中**
- 每次迁移一个逻辑更改
- 更容易审查和回滚

2. **编写可逆迁移**
- 始终提供有效的向下迁移
- 提交前测试回滚

3. **使用交易**
- 尽可能将 DDL 语句包装在事务中
- 确保原子性

4. **添加评论**
- 解释为什么需要改变
- 记录任何特殊考虑因素

5. **首先在开发中进行测试**
- 在本地应用迁移
- 验证数据完整性
- 测试回滚

## 迁移示例

```sql
-- +goose Up
-- +goose StatementBegin
-- Add tier_id field to Gemini OAuth accounts for quota tracking
UPDATE accounts
SET credentials = jsonb_set(
    credentials,
    '{tier_id}',
    '"LEGACY"',
    true
)
WHERE platform = 'gemini'
  AND type = 'oauth'
  AND credentials->>'tier_id' IS NULL;
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
-- Remove tier_id field
UPDATE accounts
SET credentials = credentials - 'tier_id'
WHERE platform = 'gemini'
  AND type = 'oauth'
  AND credentials->>'tier_id' = 'LEGACY';
-- +goose StatementEnd
```

## 故障排除

### 校验和不匹配
请参阅上面的“如果您意外修改了已应用的迁移”。

### 迁移失败
```bash
# Check migration status
psql -d sub2api -c "SELECT * FROM schema_migrations ORDER BY applied_at DESC;"

# Manually rollback if needed (use with caution)
# Better to fix the migration and create a new one
```

### 需要跳过迁移（仅限紧急情况）
```sql
-- DANGEROUS: Only use in development or with extreme caution
INSERT INTO schema_migrations (filename, checksum, applied_at)
VALUES ('NNN_migration.sql', 'calculated_checksum', NOW());
```

## References

- 迁移运行程序：`internal/repository/migrations_runner.go`
- Goose 语法：https://github.com/pressly/goose
- PostgreSQL 文档：https://www.postgresql.org/docs/
