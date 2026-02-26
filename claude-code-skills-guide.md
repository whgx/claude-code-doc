# Claude Code Skills 使用指南

> 更新日期: 2026-02-26

## 目录

1. [什么是 Skills](#什么是-skills)
2. [如何使用 Skills](#如何使用-skills)
3. [内置 Skills 详解](#内置-skills-详解)
4. [社区 Skills 推荐](#社区-skills-推荐)
5. [最佳实践](#最佳实践)

---

## 什么是 Skills

Skills 是 Claude Code 的可扩展能力模块，可以通过 `/skill-name` 的方式调用。每个 Skill 封装了特定领域的工作流程和最佳实践。

---

## 如何使用 Skills

### 基本调用方式

```bash
# 在对话中直接输入斜杠命令
/brainstorming
/commit
/review-pr
```

### 在代码中调用

Claude 可以通过 `Skill` 工具自动调用相关 skill：

```
用户: 帮我创建一个新功能
Claude: [自动调用 brainstorming skill]
```

---

## 内置 Skills 详解

### 🚀 代码提交与版本控制

#### 1. `commit-commands:commit`

**用途**: 创建规范的 git commit

**使用场景**:
- 完成代码修改后需要提交
- 需要生成符合规范的 commit message

**使用案例**:
```
用户: /commit
或: 帮我提交这些更改

Claude 会:
1. 检查 git status 查看更改
2. 分析 diff 理解变更内容
3. 生成符合规范的 commit message
4. 执行 git commit
```

**示例 Commit Message**:
```
feat(auth): add JWT token refresh mechanism

- Implement automatic token refresh before expiration
- Add refresh token storage in httpOnly cookie
- Include unit tests for token refresh flow
```

---

#### 2. `commit-commands:commit-push-pr`

**用途**: 一键完成提交、推送、创建 PR

**使用案例**:
```
用户: /commit-push-pr

Claude 会:
1. 提交本地更改
2. 推送到远程分支
3. 使用 gh 命令创建 PR
4. 生成 PR 标题和描述
```

**PR 描述模板**:
```markdown
## Summary
- 添加了用户认证功能
- 实现了 JWT token 刷新机制

## Test plan
- [ ] 运行单元测试: npm test
- [ ] 手动测试登录流程
- [ ] 验证 token 刷新功能
```

---

#### 3. `commit-commands:clean_gone`

**用途**: 清理已删除的远程分支

**使用案例**:
```
用户: /clean_gone

# 清理前
$ git branch
  main
  feature/auth
  feature/old-deleted-remote

# 清理后
$ git branch
  main
  feature/auth
```

---

### 🧪 开发流程

#### 4. `superpowers:brainstorming`

**用途**: 创建功能前的需求探索和设计

**使用场景**:
- 开始任何新功能开发前
- 需要理清需求和设计方案

**使用案例**:
```
用户: 我想添加一个用户通知系统

Claude (使用 brainstorming skill) 会:
1. 探索用户意图和具体需求
2. 询问关键问题:
   - 通知类型有哪些？
   - 实时性要求？
   - 存储需求？
3. 提出多种设计方案
4. 记录决策和权衡
```

**输出示例**:
```markdown
## 通知系统设计

### 需求确认
- 支持邮件、站内信、推送通知
- 需要实时性 (WebSocket)
- 保留30天历史记录

### 方案选择
✅ 方案 A: WebSocket + Redis Pub/Sub
❌ 方案 B: 轮询 (实时性差)
❌ 方案 C: SSE (不支持双向通信)
```

---

#### 5. `superpowers:test-driven-development`

**用途**: TDD 测试驱动开发

**使用案例**:
```
用户: 用 TDD 方式实现用户注册功能

Claude 会:
1. 先写测试用例
2. 运行测试 (应失败)
3. 编写最小实现代码
4. 运行测试 (应通过)
5. 重构代码
6. 重复 1-5
```

**测试优先示例**:
```typescript
// 1. 先写测试
describe('UserRegistration', () => {
  it('should register user with valid email', () => {
    const result = register('user@example.com', 'password123');
    expect(result.success).toBe(true);
  });

  it('should reject invalid email', () => {
    const result = register('invalid-email', 'password123');
    expect(result.success).toBe(false);
  });
});

// 2. 实现最小代码
function register(email: string, password: string) {
  if (!email.includes('@')) return { success: false };
  // 最小实现...
  return { success: true };
}
```

---

#### 6. `superpowers:systematic-debugging`

**用途**: 系统化调试流程

**使用案例**:
```
用户: 我的 API 返回 500 错误，帮我调试

Claude (使用 systematic-debugging skill) 会:
1. 收集错误信息和日志
2. 分析错误堆栈
3. 定位问题根源
4. 提出修复方案
5. 验证修复
```

**调试报告示例**:
```markdown
## 调试报告

### 问题现象
- API: GET /api/users
- 错误: 500 Internal Server Error
- 频率: 持续发生

### 根因分析
- 文件: src/handlers/user.ts:45
- 原因: 数据库连接未正确关闭
- 影响: 连接池耗尽

### 修复方案
```typescript
// 修复前
async function getUser(id: string) {
  const conn = await db.connect();
  return conn.query('SELECT * FROM users WHERE id = ?', [id]);
}

// 修复后
async function getUser(id: string) {
  const conn = await db.connect();
  try {
    return await conn.query('SELECT * FROM users WHERE id = ?', [id]);
  } finally {
    conn.release();
  }
}
```
```

---

#### 7. `superpowers:verification-before-completion`

**用途**: 完成工作前的验证检查

**使用案例**:
```
用户: 功能完成了，准备提交

Claude (使用 verification skill) 会:
1. 运行测试套件
2. 检查 lint 错误
3. 验证构建成功
4. 检查类型错误
5. 确认所有检查通过后才允许提交
```

---

### 📝 规划与计划

#### 8. `planning-with-files:plan`

**用途**: 文件式任务规划 (适合复杂任务)

**使用案例**:
```
用户: 我要重构整个认证系统

Claude 会创建:
- task_plan.md (任务计划)
- findings.md (发现和调研)
- progress.md (进度追踪)
```

**task_plan.md 示例**:
```markdown
# 认证系统重构计划

## 目标
将现有的 session 认证迁移到 JWT

## 任务列表
- [ ] 1. 调研现有认证流程
- [ ] 2. 设计 JWT 结构
- [ ] 3. 实现 token 生成
- [ ] 4. 实现 token 验证
- [ ] 5. 迁移现有用户
- [ ] 6. 更新前端适配
- [ ] 7. 编写迁移脚本
- [ ] 8. 测试和验证

## 风险
- 用户需要重新登录
- 旧 session 可能存在
```

---

#### 9. `superpowers:writing-plans`

**用途**: 编写详细实施计划

**使用案例**:
```
用户: 帮我写一个实现计划，要添加支付功能

Claude 会生成:
1. 技术选型分析
2. 架构设计
3. 分步骤实施计划
4. 风险评估
5. 测试策略
```

---

### 🎨 前端开发

#### 10. `frontend-design:frontend-design`

**用途**: 创建高质量前端界面

**使用案例**:
```
用户: 创建一个登录页面

Claude 会:
1. 询问设计偏好 (风格、配色)
2. 生成生产级代码
3. 确保响应式设计
4. 添加适当的动画效果
```

**生成代码示例**:
```tsx
// 登录页面组件
export function LoginPage() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100">
      <Card className="w-full max-w-md p-8 shadow-xl">
        <h1 className="text-2xl font-bold text-center mb-6">欢迎回来</h1>
        <LoginForm />
      </Card>
    </div>
  );
}
```

---

#### 11. `ui-ux-pro-max:ui-ux-pro-max`

**用途**: 专业级 UI/UX 设计

**支持**:
- 50 种设计风格 (glassmorphism, brutalism, minimalism...)
- 21 种配色方案
- 50 种字体搭配
- 多框架支持 (React, Vue, Svelte, Flutter...)

**使用案例**:
```
用户: 用 glassmorphism 风格创建一个仪表盘

Claude 会:
1. 应用毛玻璃效果
2. 选择合适的配色
3. 生成高质量组件代码
```

---

### 🔧 插件开发

#### 12. `plugin-dev:create-plugin`

**用途**: 创建 Claude Code 插件

**使用案例**:
```
用户: 我想创建一个自动生成 API 文档的插件

Claude 会引导:
1. 设计插件结构
2. 创建 plugin.json
3. 添加命令和钩子
4. 测试插件功能
```

**插件结构**:
```
my-plugin/
├── plugin.json          # 插件配置
├── commands/
│   └── gen-docs.md      # 斜杠命令
├── agents/
│   └── doc-writer.md    # 子代理
└── hooks/
    └── pre-commit.md    # 钩子
```

---

### 🛡️ 代码质量

#### 13. `hookify:hookify`

**用途**: 创建钩子防止不良行为

**使用案例**:
```
用户: 防止我提交包含 console.log 的代码

Claude 会创建钩子:
- 在 PreToolUse 时检查
- 如果发现 console.log，阻止提交
- 提示用户移除调试代码
```

---

#### 14. `code-review:code-review`

**用途**: 代码审查

**使用案例**:
```
用户: 审查 PR #123

Claude 会:
1. 获取 PR 变更
2. 分析代码质量
3. 检查安全漏洞
4. 提供改进建议
```

---

## 社区 Skills 推荐

> 以下是搜索到的社区优质 Skills 资源

### 🌟 推荐资源库

| 资源 | 描述 | 热度 |
|------|------|------|
| [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | 精选 Claude Skills 列表 | ~21.6k ⭐ |
| [anthropics/agent-skills](https://github.com/anthropics/agent-skills) | Anthropic 官方 Skills 框架 | 34.7k ⭐ |
| [obra/superpowers](https://github.com/obra/superpowers) | 社区驱动的 Skills 集合 | 16k+ ⭐ |
| [OpenSkills Project](https://github.com/openskills) | 跨平台 AI Skills 标准 | 7.2k ⭐ |

---

### 📦 官方插件 (28-53个可用)

**位置**: `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/`
**访问**: 在 Claude Code 中输入 `/plugins`

| 插件 | 用途 |
|------|------|
| `typescript-lsp` | 实时类型检查 (不再是猜测) |
| `security-guidance` | 被动安全漏洞扫描 |
| `context7` | 获取最新文档 |
| `playwright` | 浏览器自动化测试 |
| `frontend-design` | 设计系统和可访问性 |
| `code-review` | 5个并行代理代码审查 |
| `plugin-dev` | 插件开发工具包 |

---

### 🔧 安装方法

```bash
# 方法1: 使用 Claude Code CLI
/install-github-app

# 方法2: 使用 skills.sh CLI
npx skills list
npx skills install github.com/obra/superpowers

# 方法3: 手动安装
# 下载 skill 到 ~/.claude/skills/ 或项目的 .claude/skills/
```

---

### 🏆 Hackathon 冠军推荐配置

**作者**: @affaanmustafa (Anthropic x Forum Ventures hackathon $15k 获胜者)
**仓库**: [github.com/affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)

包含:
- **9个专业代理**: planner, architect, code-reviewer, security-reviewer 等
- **多类别技能**: coding-standards, backend-patterns, frontend-patterns, TDD, security
- **自定义命令**: 快速执行常用操作

---

### 📋 热门社区 Skills

#### 15. Superpowers (强烈推荐!)

**来源**: [github.com/obra/superpowers](https://github.com/obra/superpowers)
**用途**: 完整工作流覆盖 - 从头脑风暴到文档到开发到测试

**包含 Skills**:
- `brainstorming` - 创建功能前的需求探索
- `systematic-debugging` - 系统化调试
- `test-driven-development` - TDD 流程
- `writing-plans` - 编写实施计划
- `verification-before-completion` - 完成前验证

```bash
# 安装
npx skills install github.com/obra/superpowers

# 使用
/brainstorming
/systematic-debugging
```

---

#### 16. Planning-with-files

**用途**: 文件式任务规划，适合复杂项目

**创建文件**:
- `task_plan.md` - 任务计划
- `findings.md` - 发现和调研
- `progress.md` - 进度追踪

```bash
# 使用
用户: 我要重构整个认证系统
/plan
```

---

#### 17. UI/UX Pro Max

**用途**: 专业级 UI/UX 设计

**特性**:
- 50 种设计风格
- 21 种配色方案
- 50 种字体搭配
- 9 种技术栈支持

```bash
# 使用示例
用户: 用 glassmorphism 风格创建仪表盘
```

---

#### 18. Context7 (MCP 服务器)

**用途**: 获取任何库的最新文档

```bash
# Claude 可以自动调用获取最新文档
用户: React 19 有什么新特性？
# Claude 会通过 context7 获取最新文档
```

---

#### 19. Playwright 浏览器自动化

**用途**: 浏览器自动化测试

**可用操作**:
- 导航到 URL
- 点击元素
- 填写表单
- 截图
- 获取网络请求

```bash
用户: 打开 https://example.com 并截图
```

---

#### 20. TypeScript LSP

**用途**: 真正的类型检查，而不是猜测

**优势**:
- 实时类型错误检测
- 跳转到定义
- 查找引用
- 悬停信息

---

### 🏗️ 插件/Skill 目录结构

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json       # 插件元数据
├── commands/             # 斜杠命令
│   └── my-command.md
├── agents/               # 专用代理
│   └── my-agent.md
├── skills/               # 技能
│   └── my-skill.md
├── hooks/                # 事件处理
│   └── pre-commit.md
├── .mcp.json             # 外部工具配置
└── README.md
```

---

### 📝 Skill 文件格式

```markdown
---
name: my-skill
description: 简短描述 skill 的用途
---

# Skill 名称

详细的 skill 说明和使用指南...

## 使用场景
- 场景1
- 场景2

## 工作流程
1. 步骤1
2. 步骤2
```

---

### 💡 最佳实践提示

1. **使用 `/init` 生成初始 `CLAUDE.md`**，然后手动完善
2. **安装 Skills 后重启 Claude Code** 才能生效
3. **Skills 可以存储在**:
   - 全局: `~/.claude/skills/`
   - 项目: `.claude/skills/`
4. **为 Skills 添加中文描述**: 编辑 `SKILL.md` 的 frontmatter

---

### 📚 更多学习资源

| 资源 | 描述 |
|------|------|
| [Agent Skills 教程 (CSDN)](https://blog.csdn.net/m0_71746299/article/details/158382152) | AI Agent Skills 安装教程 |
| [Claude Code Skills 指南 (Apifox)](https://apifox.com/apiskills/claude-code-agent-skills-tutorial/) | 完整使用指南和示例 |
| [Skills 最佳实践 (掘金)](https://juejin.cn/post/7606523741611065363) | 编写有效 skills 的官方最佳实践 |
| [GitHub: Skill_Seekers](https://github.com/yusufkaraaslan/Skill_Seekers) | 将文档转换为 skills 格式的工具 |

---

### 🎯 自定义 Skill 示例

#### API 文档生成器 Skill

```markdown
---
name: api-doc-generator
description: 从代码生成全面的 API 文档。用于创建 API 文档、记录端点或生成 OpenAPI 规范。
---

# API 文档生成器

生成 API 文档时:
1. 分析代码结构和路由定义
2. 提取请求/响应模型
3. 生成 OpenAPI 3.0 规范
4. 包含示例请求和响应
5. 添加认证要求说明

## 输出格式
- Markdown 文档
- OpenAPI YAML/JSON
- Postman Collection
```

#### 数据库设计 Skill

```markdown
---
name: database-designer
description: 根据业务需求设计优化的数据库 Schema。
---

# 数据库设计器

设计数据库时:
1. 分析实体和关系
2. 应用规范化原则 (3NF)
3. 考虑查询性能添加索引
4. 设计迁移脚本
5. 生成 ER 图

## 最佳实践
- 使用有意义的主键
- 避免过度规范化
- 考虑分区策略
```

---

### 📁 Skill 目录结构建议

```
skills/
├── SKILL.md           # 主 skill 文件 (始终加载)
├── reference.md       # API 参考 (按需加载)
├── examples.md        # 使用示例 (按需加载)
└── scripts/
    └── analyze.py     # 工具脚本
```

---

### ✅ Skill 命名规范 (动名词形式)

| ✅ 好的命名 | ❌ 不好的命名 |
|------------|--------------|
| `processing-pdfs` | `pdf-skill` |
| `analyzing-spreadsheets` | `excel` |
| `managing-databases` | `db` |
| `testing-code` | `test` |
| `writing-documentation` | `docs` |

---

### 🔌 通过 API 使用 Skills

```javascript
// 使用 beta headers 调用 Skills API
const response = await client.beta.messages.create({
  model: "claude-opus-4-6",
  max_tokens: 4096,
  betas: ["code-execution-2025-08-25", "skills-2025-10-02"]
});
```

---

### 📊 Skills 安装位置汇总

| 工具 | 位置 | 作用域 |
|------|------|--------|
| **Claude Code** | `~/.claude/skills/` | 全局 |
| **Cursor** | `.cursor/skills/` | 项目 |
| **VS Code / Copilot** | `.github/skills/` | 项目 |

---

## 最佳实践

### 推荐工作流

```
新功能开发:
1. /brainstorming          # 需求探索
2. /plan                   # 创建计划
3. /test-driven-development # TDD 开发
4. /verification-before-completion # 验证
5. /commit-push-pr         # 提交

Bug 修复:
1. /systematic-debugging   # 系统化调试
2. /test-driven-development # TDD 修复
3. /verification-before-completion # 验证
4. /commit                 # 提交

代码审查:
1. /code-review            # 审查代码
2. 根据反馈修改
3. /verification-before-completion # 验证
```

### 技巧

1. **组合使用**: 多个 skills 可以串联使用
2. **自动化**: Claude 会根据上下文自动调用合适的 skill
3. **自定义**: 可以创建自己的 skills 满足特定需求

---

## 快速参考卡片

| 场景 | 推荐 Skill |
|------|-----------|
| 开始新功能 | `brainstorming` |
| 写测试 | `test-driven-development` |
| 调试问题 | `systematic-debugging` |
| 提交代码 | `commit` |
| 创建 PR | `commit-push-pr` |
| 审查代码 | `code-review` |
| 前端开发 | `frontend-design` |
| 复杂任务 | `plan` |
| 完成验证 | `verification-before-completion` |

---

*文档持续更新中...*
