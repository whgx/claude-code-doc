# Claude Code 进阶使用手册

> 本手册面向已掌握 Claude Code 基础用法的开发者，深入讲解高级技巧、实战案例和团队协作模式。

---

## 目录

1. [高级工作流模式](#1-高级工作流模式)
2. [实战案例集](#2-实战案例集)
3. [团队协作最佳实践](#3-团队协作最佳实践)
4. [性能调优与成本控制](#4-性能调优与成本控制)
5. [高级技巧集锦](#5-高级技巧集锦)
6. [故障排除进阶](#6-故障排除进阶)
7. [附录：快速参考](#7-附录快速参考)

---

## 1. 高级工作流模式

### 1.1 探索 → 规划 → 编码 → 验证循环

这是处理复杂任务的核心工作流：

```bash
# 步骤 1：探索 - 使用 Explore 子代理
> 使用 Explore 子代理找出项目中所有与用户认证相关的文件
> 查找所有处理 session 管理的代码

# 步骤 2：规划 - 让 Claude 制定计划
> 基于刚才的探索，制定一个将 session 认证迁移到 JWT 的计划
> 不要开始编码，只需要给我详细的实施计划

# 步骤 3：编码 - 执行计划
> 开始执行上述计划，每完成一个步骤告诉我

# 步骤 4：验证 - 运行测试
> 运行所有测试，确保没有破坏现有功能
```

**关键原则：**
- 探索阶段使用 Haiku 子代理（快速、便宜）
- 规划阶段考虑使用 Opus（复杂推理）
- 编码阶段使用 Sonnet（性价比最佳）
- 验证阶段可以并行运行多个测试套件

### 1.2 测试驱动开发 (TDD) 工作流

```bash
# 第 1 轮：编写失败的测试
> 为 UserAuth 类编写测试，覆盖以下场景：
>   - 有效登录成功
>   - 无效密码失败
>   - 账户锁定场景
>   - Token 过期处理
>
> 明确告诉 Claude：不要编写实现代码，只写测试

# 第 2 轮：确认测试失败
> 运行测试，确认所有测试都失败
> 记录失败的原因

# 第 3 轮：实现代码
> 编写代码让这些测试通过
> 不要修改测试，只修改实现代码

# 第 4 轮：迭代直到全部通过
> 继续迭代，直到所有测试都通过
```

### 1.3 视觉迭代工作流

```bash
# 适用于前端/UI 开发
> 这是设计的 Figma 链接：https://figma.com/...
> 实现这个登录页面
>
> 完成后：
> 1. 截图当前实现
> 2. 与设计对比
> 3. 列出差异
> 4. 修复差异
> 5. 重复直到匹配
```

### 1.4 并行工作流 (Git Worktrees)

```bash
# 创建多个并行工作分支
git worktree add ../project-auth-jwt feature-auth-jwt
git worktree add ../project-api-v2 feature-api-v2
git worktree add ../project-bug-fix-123 bug-fix-123

# 在不同终端启动多个 Claude 会话
# 终端 1
cd ../project-auth-jwt && claude --session-id "auth-migration"

# 终端 2
cd ../project-api-v2 && claude --session-id "api-refactor"

# 终端 3
cd ../project-bug-fix-123 && claude --session-id "bug-fix"
```

**优势：**
- 每个会话有独立的上下文
- 可以同时处理不相关的任务
- 避免单个上下文窗口膨胀

---

## 2. 实战案例集

### 案例 1：重构遗留代码系统

**场景：** 一个 5 年历史的 Node.js 项目，包含大量回调函数和过时的模式

```bash
# 第一阶段：全面评估
> 使用 Explore 子代理分析整个代码库：
> - 找出所有使用回调函数的文件
> - 识别重复代码模式
> - 查找没有测试覆盖的核心模块
>
> 生成一份技术债务报告

# 第二阶段：制定重构策略
> 基于评估报告，制定分阶段重构计划：
> - 阶段 1：高风险区域（优先处理）
> - 阶段 2：中等风险区域
> - 阶段 3：低风险区域
>
> 每个阶段包括：
> - 具体文件列表
> - 重构步骤
> - 测试策略
> - 回滚计划

# 第三阶段：执行重构
> 从阶段 1 开始：
> 1. 为现有代码编写测试（作为安全网）
> 2. 小步重构，每次改动后运行测试
> 3. 提交每个成功的改动
> 4. 记录重大决策

# 第四阶段：验证
> 运行完整测试套件
> 执行性能基准测试
> 生成变更报告
```

**配置文件示例 (`.claude/settings.json`)：**

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob",
      "Bash(npm run test:*)",
      "Bash(git:*)",
      "Bash(npm run lint:*)",
      "Edit(src/**)",
      "Write(src/**)"
    ],
    "deny": [
      "Edit(package-lock.json)",
      "Bash(npm install)"
    ],
    "defaultMode": "acceptEdits"
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix \"$FILE_PATH\" || true"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/validate-commands.sh"
          }
        ]
      }
    ]
  }
}
```

### 案例 2：实现完整的 API 端点

**场景：** 从零开始创建一个 RESTful API 端点，包括验证、错误处理和文档

```bash
# 技能文件 (.claude/skills/api-development/SKILL.md)
---
name: api-development
description: RESTful API 开发专家。在创建或修改 API 端点时自动应用。包括路由设计、验证、错误处理、文档和测试。
allowed-tools: Read, Edit, Write, Bash(npm run:*), Bash(git:*)
---

# API 开发最佳实践

## 端点设计原则

### RESTful 约定
- GET: 获取资源（幂等）
- POST: 创建资源
- PUT/PATCH: 更新资源（PUT 幂等，PATCH 部分更新）
- DELETE: 删除资源（幂等）

### 路由结构
```
GET    /api/users           # 列表
GET    /api/users/:id       # 详情
POST   /api/users           # 创建
PUT    /api/users/:id       # 完整更新
PATCH  /api/users/:id       # 部分更新
DELETE /api/users/:id       # 删除
```

## 请求验证

使用 Zod schema 验证：
```typescript
import { z } from 'zod';

export const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).regex(/[A-Z]/, "必须包含大写字母"),
  name: z.string().min(2).max(100)
});
```

## 错误处理

标准错误响应格式：
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "输入验证失败",
    "details": [
      {
        "field": "email",
        "message": "无效的邮箱格式"
      }
    ]
  }
}
```

## 响应格式

成功响应：
```json
{
  "data": { ... },
  "meta": {
    "timestamp": "2026-02-02T10:00:00Z",
    "version": "1.0.0"
  }
}
```

## 必须包含的测试

1. 成功场景测试
2. 验证失败测试（每个字段）
3. 权限测试
4. 边界条件测试
5. 并发测试（如适用）

## 文档要求

每个端点必须包含：
- OpenAPI/Swagger 规范
- 请求/响应示例
- 错误代码列表
- 认证要求
```

**使用流程：**

```bash
# 触发技能（Claude 会自动识别上下文）
> 创建一个用户管理 API，包括：
> - 创建用户
> - 获取用户列表（带分页）
> - 获取单个用户
> - 更新用户
> - 删除用户
>
# Claude 会自动应用 api-development 技能

# 验证实现
> 运行测试并确保所有场景都被覆盖
> 生成 OpenAPI 文档

# 代码审查
> 使用另一个 Claude 会话审查刚才的代码
> 检查是否有安全漏洞、性能问题
```

### 案例 3：调试生产环境问题

**场景：** 生产环境出现间歇性错误，需要快速定位和修复

```bash
# 配置 Sentry MCP 集成
claude mcp add --transport http sentry https://sentry.io/mcp/

# 开始调试流程
> 从 Sentry 获取过去 24 小时的所有错误
> 按频率排序

# 深入分析最频繁的错误
> 获取错误 ABC-123 的完整堆栈跟踪
> 分析以下内容：
> 1. 错误发生的模式（时间、用户、请求类型）
> 2. 相关代码位置
> 3. 可能的根本原因
>
> 使用扩展思考模式进行深度分析

# 提出修复方案
> 基于分析，提出 3 个可能的修复方案：
> - 方案 A：快速修复（治标）
> - 方案 B：中期修复（治本）
> - 方案 C：长期重构（预防）
>
> 对每个方案评估：
> - 实施时间
> - 风险等级
> - 预期效果

# 实施修复
> 先实施方案 A（紧急修复）
> 编写回归测试
> 提交并创建 PR
>
> 计划方案 B 和 C 的实施时间
```

**调试钩子配置：**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/track-changes.sh \"$FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# .claude/hooks/track-changes.sh
# 记录所有代码变更，便于调试时追溯

FILE_PATH="$1"
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
BRANCH=$(git branch --show-current)

echo "[${TIMESTAMP}] [${BRANCH}] Edited: ${FILE_PATH}" >> ~/.claude/debug-changes.log
```

### 案例 4：数据库迁移

**场景：** 大规模数据库 schema 迁移，需要零停机

```bash
# 技能：数据库迁移专家
# .claude/skills/database-migration/SKILL.md

---
name: database-migration
description: 数据库迁移专家。处理 schema 变更、数据迁移、零停机部署策略。
allowed-tools: Read, Write, Bash(git:*), Bash(npm run:*), Bash(psql:*)
---

# 零停机迁移策略

## 阶段 1：准备阶段

### 1.1 分析影响
```sql
-- 查找所有依赖目标表的查询
SELECT query, calls, total_time
FROM pg_stat_statements
WHERE query LIKE '%old_table_name%';
```

### 1.2 创建新表结构
```sql
-- 创建新表（与旧表并存）
CREATE TABLE users_v2 (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    -- 新字段和优化
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 创建索引
CREATE INDEX idx_users_v2_email ON users_v2(email);
```

## 阶段 2：双写阶段

### 2.1 触发器实现双写
```sql
CREATE OR REPLACE FUNCTION sync_user_data()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO users_v2 (id, email, created_at, updated_at)
    VALUES (NEW.id, NEW.email, NEW.created_at, NEW.updated_at)
    ON CONFLICT (id) DO UPDATE SET
        email = EXCLUDED.email,
        updated_at = EXCLUDED.updated_at;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER sync_users_to_v2
AFTER INSERT OR UPDATE ON users
FOR EACH ROW EXECUTE FUNCTION sync_user_data();
```

### 2.2 验证双写
```bash
# 对比两个表的数据一致性
SELECT 'users' as table_name, COUNT(*) as count FROM users
UNION ALL
SELECT 'users_v2', COUNT(*) FROM users_v2;
```

## 阶段 3：数据回填

### 3.1 分批回填历史数据
```sql
-- 分批处理，避免锁表
DO $$
DECLARE
    batch_size INT := 10000;
    max_id INT := 0;
    rows_affected INT;
BEGIN
    LOOP
        INSERT INTO users_v2 (id, email, created_at, updated_at)
        SELECT id, email, created_at, updated_at
        FROM users
        WHERE id > max_id
        ORDER BY id
        LIMIT batch_size
        ON CONFLICT (id) DO NOTHING;

        GET DIAGNOSTICS rows_affected = ROW_COUNT;
        EXIT WHEN rows_affected = 0;

        SELECT COALESCE(MAX(id), 0) INTO max_id
        FROM users_v2;

        RAISE NOTICE 'Processed batch, current max_id: %', max_id;
    END LOOP;
END $$;
```

## 阶段 4：切换读流量

### 4.1 灰度切换
```bash
# 第 1 天：10% 流量
# 第 3 天：50% 流量
# 第 7 天：100% 流量
```

## 阶段 5：清理

### 5.1 移除双写触发器
```sql
DROP TRIGGER IF EXISTS sync_users_to_v2 ON users;
DROP FUNCTION IF EXISTS sync_user_data();
```

### 5.2 重命名表
```sql
BEGIN;
ALTER TABLE users RENAME TO users_old;
ALTER TABLE users_v2 RENAME TO users;
COMMIT;
```

### 5.3 监控后删除旧表
```sql
-- 保留 30 天后删除
DROP TABLE IF EXISTS users_old;
```
```

**执行迁移：**

```bash
# 使用 Plan 模式制定详细计划
> /model opus
> 制定 users 表迁移计划
> 包含回滚步骤和每个阶段的验证方法

# 执行阶段 1
> 执行迁移阶段 1：准备阶段
> 验证新表创建成功

# 执行阶段 2
> 执行迁移阶段 2：设置双写
> 监控触发器性能

# 继续各阶段...
> 每个阶段完成后进行验证
> 遇到问题立即回滚
```

### 案例 5：自动化 CI/CD 流程

**场景：** 为 GitHub Actions 工作流添加 Claude Code 自动化

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  claude-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install Claude Code
        run: |
          curl -fsSL https://claude.ai/install.sh | bash

      - name: Configure Claude
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          echo "{\"apiKey\": \"$ANTHROPIC_API_KEY\"}" > ~/.claude.json

      - name: Run Code Review
        run: |
          claude -p "Review this PR for:
          - Security vulnerabilities
          - Performance issues
          - Code style violations
          - Missing error handling
          - Test coverage gaps

          Focus on files changed in this PR.
          Provide findings in markdown format with severity levels.

          PR Number: ${{ github.event.number }}
          Base Branch: ${{ github.event.pull_request.base.ref }}
          " \
          --output-format json \
          --allowedTools "Read,Grep,Glob" \
          --permission-mode plan \
          --max-turns 10 \
          > review-output.json

      - name: Parse Review Results
        id: parse
        run: |
          REVIEW=$(jq -r '.result' review-output.json)
          echo "review<<EOF" >> $GITHUB_OUTPUT
          echo "$REVIEW" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT

          # Check for critical issues
          if echo "$REVIEW" | grep -qi "critical"; then
            echo "has_critical=true" >> $GITHUB_OUTPUT
          else
            echo "has_critical=false" >> $GITHUB_OUTPUT
          fi

      - name: Comment on PR
        uses: actions/github-script@v7
        with:
          script: |
            const review = `${{ steps.parse.outputs.review }}`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## 🤖 Claude Code Review\n\n${review}`
            })

      - name: Fail on Critical Issues
        if: steps.parse.outputs.has_critical == 'true'
        run: exit 1
```

**本地测试 CI 工作流：**

```bash
# 使用 act 本地测试 GitHub Actions
act pull_request --eventpath test-pr-event.json

# 或者直接运行 Claude 审查
claude -p "$(cat .claude/commands/pr-review-prompt.txt)" \
  --allowedTools "Read,Grep,Glob" \
  --permission-mode plan
```

### 案例 6：多语言项目协作

**场景：** Monorepo 包含多个语言（TypeScript、Python、Go）

```bash
# 项目结构
# /apps
#   /frontend (TypeScript/React)
#   /api (Python/FastAPI)
#   /worker (Go)
# /packages
#   /shared-types (TypeScript)
#   /protobuf (Protobuf definitions)

# 技能：多语言专家
# .claude/skills/polyglot/SKILL.md

---
name: polyglot
description: 多语言项目专家。理解 TypeScript、Python、Go 之间的交互，处理类型共享、API 契约、跨语言测试。
allowed-tools: Read, Write, Bash
---

# 多语言项目模式

## 类型共享

### Protocol Buffers 优先
```protobuf
// packages/protobuf/user.proto
syntax = "proto3";

message User {
  string id = 1;
  string email = 2;
  string name = 3;
}
```

生成各语言类型：
```bash
# TypeScript
protoc --ts_out=./packages/shared-types user.proto

# Python
protoc --python_out=./apps/api user.proto

# Go
protoc --go_out=./apps/worker user.proto
```

## API 契约测试

使用 Pact 进行跨语言契约测试：
```typescript
// frontend/src/tests/consumer.spec.ts
import { Pact } from '@pactfoundation/pact';

describe('User API', () => {
  const pact = new Pact({
    consumer: 'frontend',
    provider: 'api',
    port: 1234
  });

  beforeAll(() => pact.setup());
  afterAll(() => pact.finalize());

  it('returns user data', async () => {
    await pact.addInteraction({
      uponReceiving: 'a request for user',
      withRequest: {
        method: 'GET',
        path: '/api/users/123'
      },
      willRespondWith: {
        status: 200,
        body: {
          id: '123',
          email: 'test@example.com',
          name: 'Test User'
        }
      }
    });

    // 运行实际测试
  });
});
```

## 错误处理统一

所有语言使用相同的错误代码：
```typescript
// packages/shared-types/errors.ts
export enum ErrorCode {
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  NOT_FOUND = 'NOT_FOUND',
  UNAUTHORIZED = 'UNAUTHORIZED',
  INTERNAL_ERROR = 'INTERNAL_ERROR'
}
```

```python
# apps/api/shared/errors.py
class ErrorCode(Enum):
    VALIDATION_ERROR = "VALIDATION_ERROR"
    NOT_FOUND = "NOT_FOUND"
    UNAUTHORIZED = "UNAUTHORIZED"
    INTERNAL_ERROR = "INTERNAL_ERROR"
```

## 日志关联

使用 trace ID 关联跨语言请求：
```typescript
// frontend/src/lib/trace.ts
export function generateTraceId(): string {
  return `trace-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}
```

```python
# apps/api/middleware/trace.py
@app.middleware("http")
async def add_trace_id(request: Request, call_next):
    trace_id = request.headers.get("X-Trace-ID", str(uuid4()))
    response = await call_next(request)
    response.headers["X-Trace-ID"] = trace_id
    return response
```
```

**跨语言任务示例：**

```bash
# 任务：添加新的 user preferences 功能
> 需要在所有三个服务中实现 user preferences：
>
> 1. 更新 protobuf 定义
> 2. 在 frontend 中添加 preferences UI
> 3. 在 api 中添加 preferences 端点
> 4. 在 worker 中添加 preferences 处理
> 5. 确保类型一致性
> 6. 添加跨语言集成测试
>
# Claude 会自动应用 polyglot 技能

# 验证类型一致性
> 生成各语言的类型并进行对比
> 确保 API 契约一致
```

---

## 3. 团队协作最佳实践

### 3.1 团队共享配置

**项目级 CLAUDE.md：**

```markdown
# 项目：电商后端系统

## 架构概览
- Monorepo 结构，使用 Turbo
- API: Node.js + Fastify + PostgreSQL
- Worker: Go + Redis
- Frontend: Next.js 14

## 团队约定

### 分支命名
- `feature/` - 新功能
- `fix/` - Bug 修复
- `refactor/` - 重构
- `hotfix/` - 生产紧急修复

### 提交信息规范
使用 Conventional Commits：
```
feat: add user preferences API
fix: resolve race condition in cart service
refactor: simplify order processing logic
test: add integration tests for payment flow
docs: update API documentation
```

### 代码审查清单
- [ ] 测试覆盖率 > 80%
- [ ] 无 ESLint/Prettier 错误
- [ ] API 文档已更新
- [ ] 性能测试通过
- [ ] 安全扫描无问题

### 常用命令
```bash
npm run dev          # 启动开发服务器
npm run test         # 运行所有测试
npm run test:watch   # 监听模式
npm run lint         # 检查代码质量
npm run typecheck    # TypeScript 类型检查
npm run build        # 生产构建
```

## 重要约定

### 数据库变更
- 所有 schema 变更必须经过 db-migrate
- 迁移文件命名：`YYYY-MM-DD-description.sql`
- 生产迁移需要 DBA 审批

### API 设计
- RESTful 风格
- 使用 OpenAPI 3.0 规范
- 所有端点需要认证
- 错误响应统一格式

### 安全要求
- 不在代码中硬编码密钥
- 使用参数化查询
- 输入验证使用 Zod
- 敏感操作需要审计日志

# 压缩说明
压缩时重点保留：
- 最后一次讨论的架构决策
- 当前任务的上下文
- 相关文件的引用
```

**个人配置 (`.claude/settings.local.json`)：**

```json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  },
  "env": {
    "GIT_AUTHOR_NAME": "Your Name",
    "GIT_AUTHOR_EMAIL": "your.email@company.com"
  }
}
```

**.gitignore 配置：**

```gitignore
# Claude Code
.claude/settings.local.json
.claude/local-*
CLAUDE.local.md
```

### 3.2 团队技能库

**创建团队技能目录：**

```bash
.claude/skills/
├── security/
│   └── SKILL.md
├── api-design/
│   └── SKILL.md
├── database/
│   └── SKILL.md
├── testing/
│   └── SKILL.md
└── deployment/
    └── SKILL.md
```

**团队共享技能示例：**

```markdown
# .claude/skills/security/SKILL.md
---
name: security
description: 安全代码审查专家。处理认证、授权、数据保护、常见漏洞。
allowed-tools: Read, Grep, Glob, Bash(git:*)
---

# 安全审查清单

## 认证与授权
- [ ] 所有端点都有认证检查
- [ ] 使用 RBAC 或 ABAC 进行授权
- [ ] Token 有适当的过期时间
- [ ] 敏感操作需要重新认证

## 数据保护
- [ ] 敏感数据加密存储
- [ ] 日志中不包含敏感信息
- [ ] 使用 HTTPS 传输
- [ ] 密码使用 bcrypt/argon2

## 常见漏洞 (OWASP Top 10)
- [ ] SQL 注入：使用参数化查询
- [ ] XSS：输出转义和 CSP
- [ ] CSRF：使用 CSRF token
- [ ] SSRF：验证和限制 URL
- [ ] IDOR：验证资源所有权

## 依赖管理
- [ ] 定期更新依赖
- [ ] 使用 `npm audit` 检查漏洞
- [ ] 锁定依赖版本
```

### 3.3 代码审查工作流

```bash
# 审查者技能
# .claude/skills/code-reviewer/SKILL.md

---
name: code-reviewer
description: 代码审查专家。检查代码质量、可维护性、性能、安全。
allowed-tools: Read, Grep, Glob, Bash(npm run:*)
---

# 代码审查标准

## 审查维度

### 1. 正确性
- 代码是否实现了预期功能？
- 边界条件是否处理？
- 错误处理是否完善？

### 2. 可读性
- 变量和函数命名是否清晰？
- 复杂逻辑是否有注释？
- 代码结构是否合理？

### 3. 可维护性
- 是否遵循 DRY 原则？
- 是否容易扩展？
- 测试是否充分？

### 4. 性能
- 是否有明显的性能问题？
- 数据库查询是否优化？
- 是否有不必要的计算？

### 5. 安全
- 是否有安全漏洞？
- 敏感信息是否暴露？
- 输入是否验证？

## 审查输出格式

对于每个问题：
```markdown
## [严重性] 问题标题

**文件**: `path/to/file.ts:123`

**问题描述**: 详细说明问题

**建议**: 如何修复

**参考**: 相关文档或最佳实践链接
```

严重性级别：
- 🔴 Critical: 必须修复
- 🟠 High: 应该修复
- 🟡 Medium: 建议修复
- 🔵 Low: 可选改进
```

**使用流程：**

```bash
# 1. 审查者获取 PR
> 从 PR #456 获取变更
> 分析变更范围和影响

# 2. 执行审查
> 使用 code-reviewer 技能全面审查
> 生成详细的审查报告

# 3. 提供反馈
> 将审查结果格式化为 PR 评论
> 优先处理 Critical 和 High 问题

# 4. 跟进修复
> 作者修复后重新审查
> 确保所有问题都得到解决
```

### 3.4 知识管理系统

**创建团队知识库技能：**

```markdown
# .claude/skills/knowledge-base/SKILL.md
---
name: knowledge-base
description: 团队知识库。访问项目文档、架构决策记录、常见问题解答。
---

# 团队知识库

## 架构决策记录 (ADRs

### ADR-001: 选择 PostgreSQL 作为主数据库
**日期**: 2024-01-15
**状态**: 已接受

**决策**: 使用 PostgreSQL 作为主数据库

**原因**:
- 成熟稳定
- 优秀的 JSON 支持
- 强大的全文搜索
- 优秀的性能

**后果**:
- 需要学习 PostgreSQL 特性
- 可以利用高级查询功能

### ADR-002: 采用事件驱动架构
**日期**: 2024-02-01
**状态**: 已接受

**决策**: 使用事件总线解耦服务

**原因**:
- 提高可扩展性
- 允许异步处理
- 更容易添加新功能

## 常见问题

### Q: 如何添加新的 API 端点？
A: 参考 `docs/api-development-guide.md`

### Q: 如何部署到生产环境？
A: 参考 `docs/deployment-guide.md`

### Q: 如何处理数据库迁移？
A: 参考 `docs/database-migrations.md`
```

---

## 4. 性能调优与成本控制

### 4.1 上下文管理策略

**监控上下文使用：**

```bash
# 定期检查上下文使用情况
> /context
# 输出：
# 系统提示: 20% (40,000 tokens)
# 对话历史: 35% (70,000 tokens)
# 文件内容: 30% (60,000 tokens)
# 工具输出: 15% (30,000 tokens)
# 总计: 200,000 tokens / 200,000 (100%)
```

**上下文优化技巧：**

```bash
# 1. 明确指定文件范围
> 只搜索 src/api/ 目录中的认证相关代码
# 而不是：
> 搜索整个项目中的认证代码

# 2. 使用具体引用
> 查看 @src/auth/middleware.ts 中的认证逻辑
# 而不是：
> 找到并查看认证中间件文件

# 3. 分阶段处理复杂任务
> 第一阶段：只分析问题，不读取代码
> 第二阶段：读取相关文件
> 第三阶段：提出解决方案
> 第四阶段：实现修复

# 4. 定期压缩
> 在上下文超过 50% 时自动提醒我压缩
# 或设置自动压缩
```

**自定义压缩指令 (CLAUDE.md)：**

```markdown
# 压缩指令

压缩对话时，优先保留：
1. 当前任务的上下文
2. 最近讨论的代码变更
3. 重要的架构决策
4. 待办事项列表

可以丢弃：
1. 已完成任务的细节
2. 探索性代码的中间结果
3. 重复的确认对话
```

### 4.2 成本优化策略

**智能模型选择：**

```bash
# 配置子代理使用更便宜的模型
export CLAUDE_CODE_SUBAGENT_MODEL=haiku

# 在设置中配置
{
  "env": {
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

**成本对比：**

| 任务类型 | 推荐模型 | 成本 | 理由 |
|---------|---------|------|------|
| 文件搜索 | Haiku | $0.03 | 简单模式匹配 |
| 代码重构 | Sonnet | $0.75 | 平衡质量和成本 |
| 架构设计 | Opus | $2.00 | 需要深度推理 |
| 测试生成 | Sonnet | $0.50 | 不需要最强推理 |
| 文档编写 | Haiku | $0.10 | 主要是模板填充 |

**批量处理策略：**

```bash
# 错误：逐个处理文件
> 修复 src/components/ 中的所有组件
# 会导致多次上下文加载

# 正确：批量处理
> 生成 src/components/ 目录的文件列表
> 批量处理这些组件：
> 1. 先扫描所有文件
> 2. 识别需要修复的模式
> 3. 一次性应用修复
```

**缓存利用：**

```bash
# 确保提示缓存启用（默认开启）
# 检查是否被禁用
echo $DISABLE_PROMPT_CACHING  # 应该为空或未设置

# 针对重复任务的优化
> 对于这类重复的代码审查任务：
> 1. 保持系统提示和工具定义不变
> 2. 只变更需要审查的文件
> 这样可以最大化缓存命中率
```

### 4.3 性能基准测试

**创建性能测试技能：**

```markdown
# .claude/skills/performance-testing/SKILL.md
---
name: performance-testing
description: 性能测试专家。执行基准测试、识别瓶颈、优化建议。
allowed-tools: Read, Write, Bash(npm run:*), Bash(git:*)
---

# 性能测试标准

## API 端点性能

### 目标指标
- P50 响应时间: < 100ms
- P95 响应时间: < 500ms
- P99 响应时间: < 1000ms
- 吞吐量: > 1000 req/s

### 测试工具
```bash
# 使用 autocannon 进行负载测试
npm run load-test

# 使用 clinic.js 进行性能分析
npm run performance-analyze
```

## 数据库查询性能

### 目标指标
- 查询时间: < 10ms (简单查询)
- 查询时间: < 100ms (复杂查询)
- 慢查询: > 1s (需要优化)

### 分析工具
```sql
-- 查找慢查询
SELECT query, calls, mean_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- 查看索引使用情况
SELECT schemaname, tablename, indexname
FROM pg_stat_user_indexes
WHERE idx_scan = 0;
```

## 内存使用

### 目标指标
- 堆内存: < 512MB
- 无内存泄漏
- GC 暂停: < 100ms

### 监控工具
```bash
# 使用 clinic.js memory 分析
npm run memory-profile

# 使用 heapdump
npm run heapdump
```
```

**执行性能测试：**

```bash
# 1. 建立基准
> 为当前 API 端点建立性能基准
> 记录 P50、P95、P99 响应时间

# 2. 识别瓶颈
> 分析慢查询和阻塞操作
> 生成性能报告

# 3. 优化
> 基于分析结果进行优化
> 添加索引、缓存、查询优化

# 4. 验证
> 重新运行性能测试
> 对比优化前后的改进
```

---

## 5. 高级技巧集锦

### 5.1 多 Claude 协作模式

**模式 1：编码-审查循环**

```bash
# 终端 1：编码者
cd project && claude --session-id "coder-1"
> 实现用户认证功能

# 终端 2：审查者
cd project && claude --session-id "reviewer-1"
> 审查 coder-1 会话中的最新变更
> 提供改进建议

# 终端 3：整合者
cd project && claude --session-id "integrator-1"
> 基于审查建议，整合改进
```

**模式 2：并行开发**

```bash
# 使用 git worktree
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b

# 并行开发
# 终端 1
cd ../project-feature-a && claude --session-id "feature-a-dev"

# 终端 2
cd ../project-feature-b && claude --session-id "feature-b-dev"
```

### 5.2 自定义钩子模式

**自动格式化链：**

```bash
#!/bin/bash
# .claude/hooks/format-chain.sh

FILE_PATH="$1"
EXT="${FILE_PATH##*.}"

case "$EXT" in
  ts|tsx|js|jsx)
    npx prettier --write "$FILE_PATH"
    npx eslint --fix "$FILE_PATH"
    ;;
  py)
    black "$FILE_PATH"
    pylama "$FILE_PATH"
    ;;
  go)
    gofmt -w "$FILE_PATH"
    ;;
esac
```

**智能测试触发：**

```bash
#!/bin/bash
# .claude/hooks/smart-test.sh

FILE_PATH="$1"

# 确定相关测试
if [[ "$FILE_PATH" == *"test"* ]]; then
  # 文件本身就是测试
  TEST_FILE="$FILE_PATH"
elif [[ "$FILE_PATH" == *"src"* ]]; then
  # 源文件，查找对应测试
  TEST_FILE="${FILE_PATH/src/test}"
  TEST_FILE="${TEST_FILE%.ts}.test.ts"
fi

# 运行测试
if [[ -f "$TEST_FILE" ]]; then
  npm test -- "$TEST_FILE"
fi
```

### 5.3 MCP 高级用法

**链式 MCP 调用：**

```bash
# 使用 GitHub MCP 获取 PR 信息
> 使用 GitHub MCP 获取 PR #456 的信息

# 使用 Sentry MCP 检查相关错误
> 使用 Sentry MCP 检查 PR #456 中修改文件的错误

# 使用 Jira MCP 创建跟踪任务
> 使用 Jira MCP 为这些问题创建跟踪任务
```

**自定义 MCP 工作流：**

```bash
# 创建自动化工作流技能
# .claude/skills/mcp-workflow/SKILL.md

---
name: mcp-workflow
description: MCP 自动化工作流。协调多个 MCP 服务器完成复杂任务。
allowed-tools: mcp__github, mcp__sentry, mcp__jira, mcp__slack
---

# Bug 修复自动化流程

## 触发条件
当 Sentry 报告新的高优先级错误时

## 工作流步骤

### 1. 错误分析
从 Sentry 获取错误详情：
- 堆栈跟踪
- 影响用户数
- 首次出现时间

### 2. 代码定位
使用 GitHub MCP：
- 查找相关的最近代码变更
- 识别可能的引入点

### 3. 任务创建
在 Jira 中创建 bug 任务：
- 自动填充错误信息
- 设置优先级
- 分配给相应团队

### 4. 团队通知
在 Slack 通知：
- 发送到相关频道
- 包含错误摘要和 Jira 链接

### 5. 进度跟踪
定期检查：
- Jira 任务状态
- 相关 PR 状态
- Sentry 错误趋势
```

### 5.4 会话恢复技巧

**命名会话：**

```bash
# 创建有意义的会话 ID
claude --session-id "auth-migration-$(date +%Y%m%d)"
claude --session-id "bug-fix-${ISSUE_ID}"
claude --session-id "feature-${FEATURE_NAME}-$(git branch --show-current)"
```

**会话分支：**

```bash
# 创建会话分支点
> /rename session-main-branch
> 完成第一阶段工作

# 创建分支探索
> /fork explore-alternative-approach
> 尝试不同的实现方法

# 返回主线
> /resume session-main-branch
```

### 5.5 大型代码库处理

**分块策略：**

```bash
# 对于超大型代码库

# 策略 1：按模块划分
> 将项目分为以下模块分别处理：
> - 认证模块
> - 数据访问层
> - API 路由
> - 业务逻辑

# 策略 2：按依赖层次
> 从底层开始：
> 1. 工具函数
> 2. 数据模型
> 3. 服务层
> 4. 控制器
> 5. 视图层

# 策略 3：按功能垂直切片
> 按功能切片：
> - 用户管理（全栈）
> - 订单处理（全栈）
> - 支付集成（全栈）
```

---

## 6. 故障排除进阶

### 6.1 常见问题诊断

**问题：上下文快速膨胀**

```bash
# 诊断
> /context
# 查看 token 分布

# 解决方案
> 检查是否读取了过多完整文件
> 考虑使用子代理进行探索

# 预防
> 在 CLAUDE.md 中设置上下文限制建议
```

**问题：权限提示过多**

```bash
# 诊断
> /permissions
# 查看当前权限配置

# 解决方案
> 将常用工具添加到允许列表
{
  "permissions": {
    "allow": [
      "Read",
      "Edit(src/**)",
      "Bash(npm run test:*)"
    ],
    "defaultMode": "acceptEdits"
  }
}

# 或使用沙箱模式
> /sandbox
```

**问题：MCP 服务器连接失败**

```bash
# 诊断
claude mcp get <server-name>

# 测试连接
claude --mcp-debug

# 常见问题
# 1. 端点 URL 错误
# 2. 认证令牌过期
# 3. 网络代理问题

# 解决方案
export HTTPS_PROXY=http://proxy:8080
claude mcp remove <server-name>
claude mcp add --transport http <name> <url> --header "Authorization: Bearer $TOKEN"
```

### 6.2 调试工具

**钩子调试：**

```bash
# 启用钩子调试
claude --debug

# 查看钩子执行日志
# 钩子输入/输出会显示在调试输出中

# 测试钩子
echo '{"tool_name":"Edit","tool_input":{"file_path":"test.ts"}}' | \
  .claude/hooks/test-hook.sh
```

**性能分析：**

```bash
# 记录详细性能日志
export ANTHROPIC_LOG=debug
claude --session-id "performance-test" > perf.log 2>&1

# 分析日志
grep "duration_ms" perf.log | \
  awk '{print $2}' | \
  sort -n | \
  tail -10
```

### 6.3 恢复策略

**会话损坏恢复：**

```bash
# 如果会话无法恢复
# 1. 检查会话文件
ls -la ~/.claude/sessions/

# 2. 备份损坏的会话
cp ~/.claude/sessions/bad-session.jsonl ~/backup/

# 3. 创建新会话
claude --session-id "recovery-$(date +%s)"

# 4. 从损坏会话中提取关键信息
grep "## " ~/.claude/sessions/bad-session.jsonl
```

**配置重置：**

```bash
# 如果配置出现问题
# 1. 备份当前配置
cp ~/.claude/settings.json ~/backup/

# 2. 重置为默认
rm ~/.claude/settings.json
claude  # 将创建新的默认配置

# 3. 逐步恢复设置
```

---

## 7. 附录：快速参考

### 7.1 常用命令速查

```bash
# 会话管理
claude                                          # 启动新会话
claude -c                                       # 继续最近会话
claude -c -p "添加功能"                          # 继续并添加提示
claude --session-id "my-task"                   # 命名会话
claude --resume "previous-session"              # 恢复特定会话

# 模型选择
claude --model haiku                            # 使用 Haiku
claude --model sonnet                           # 使用 Sonnet
claude --model opus                             # 使用 Opus
> /model opus                                   # 会话中切换模型

# 权限模式
claude --permission-mode plan                   # 只读模式
claude --permission-mode acceptEdits            # 自动接受编辑
claude --dangerously-skip-permissions           # 跳过所有权限（危险！）

# 非交互模式
claude -p "检查代码质量"                         # 单次查询
echo "输入" | claude -p "处理"                   # 管道输入
cat file.log | claude -p "分析错误"              # 处理文件

# 输出格式
claude -p "生成报告" --output-format json        # JSON 输出
claude -p "分析" --output-format stream-json    # 流式 JSON

# 诊断
claude --debug                                  # 调试模式
claude --mcp-debug                              # MCP 调试
claude doctor                                   # 健康检查
```

### 7.2 斜杠命令速查

| 命令 | 功能 |
|------|------|
| `/help` | 显示帮助 |
| `/init` | 初始化项目 |
| `/memory` | 编辑记忆文件 |
| `/context` | 查看上下文使用 |
| `/compact` | 压缩对话历史 |
| `/cost` | 查看成本统计 |
| `/stats` | 使用统计 |
| `/model <model>` | 切换模型 |
| `/permissions` | 权限设置 |
| `/mcp` | MCP 配置 |
| `/hooks` | 钩子配置 |
| `/config` | 打开配置 |
| `/agents` | 管理子代理 |
| `/clear` | 清除对话 |
| `/status` | 会话状态 |
| `/resume` | 恢复会话 |
| `/rename` | 重命名会话 |
| `/fork` | 分支会话 |
| `/sandbox` | 沙箱模式 |
| `/vim` | Vim 模式 |
| `/theme` | 主题设置 |
| `/plugin` | 插件管理 |
| `/doctor` | 诊断检查 |

### 7.3 快捷键速查

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | 取消当前操作 |
| `Ctrl+D` | 退出会话 |
| `Ctrl+L` | 清屏 |
| `Ctrl+O` | 切换详细输出 |
| `Ctrl+B` | 后台运行 |
| `Ctrl+V` | 粘贴图片 |
| `Esc Esc` | 撤销上次更改 |
| `Tab` | 接受建议 / 切换思考 |
| `Shift+Tab` | 切换权限模式 |
| `Alt+T` | 切换思考模式 |
| `Ctrl+T` | 切换语法高亮 |

### 7.4 前缀速查

| 前缀 | 用途 | 示例 |
|------|------|------|
| `#` | 添加到记忆 | `# 记住使用 TypeScript` |
| `/` | 斜杠命令 | `/review` |
| `!` | 直接 bash | `! git status` |
| `@` | 文件引用 | `@src/app.ts` |
| `&` | 云端执行 | `& 构建项目` |

### 7.5 环境变量速查

```bash
# 认证
ANTHROPIC_API_KEY=sk-ant-...

# 模型配置
ANTHROPIC_MODEL=claude-sonnet-4-5
CLAUDE_CODE_SUBAGENT_MODEL=haiku
MAX_THINKING_TOKENS=10000

# 行为控制
DISABLE_AUTOUPDATER=1
DISABLE_TELEMETRY=1
BASH_DEFAULT_TIMEOUT_MS=30000

# 网络配置
HTTP_PROXY=http://proxy:8080
HTTPS_PROXY=https://proxy:8080

# 云提供商
CLAUDE_CODE_USE_BEDROCK=1
CLAUDE_CODE_USE_VERTEX=1
AWS_REGION=us-east-1
```

---

## 结语

Claude Code 是一个强大的工具，但掌握它需要理解其工作原理和最佳实践。本手册涵盖了：

1. **高效工作流**：从探索到验证的完整循环
2. **实战案例**：真实场景的解决方案
3. **团队协作**：共享配置和知识管理
4. **性能优化**：成本控制和性能调优
5. **高级技巧**：多 Claude 协作和自定义钩子

记住：**熟能生巧**。多使用、多实验、多总结，你会发现 Claude Code 能极大地提升你的开发效率。

**更新日期：** 2026-02-02

**Claude Code 版本：** v2.1.x 系列
