# Claude Code 必装插件与技能完全指南

> 全面覆盖 2026 年最受推荐的 MCP 服务器、技能框架和开发插件，含详细安装与使用教程。

---

## 目录

1. [核心概念辨析](#1-核心概念辨析)
2. [必装 MCP 服务器](#2-必装-mcp-服务器)
3. [Superpowers 技能框架](#3-superpowers-技能框架)
4. [热门技能合集](#4-热门技能合集)
5. [官方插件推荐](#5-官方插件推荐)
6. [安装管理指南](#6-安装管理指南)
7. [实战使用案例](#7-实战使用案例)

---

## 1. 核心概念辨析

### 1.1 三大扩展类型对比

| 类型 | 触发方式 | 适用场景 | 存储位置 |
|------|----------|----------|----------|
| **MCP 服务器** | Claude 自动调用 | 外部服务集成（数据库、API） | `.mcp.json` 或全局 |
| **技能 (Skills)** | Claude 自动识别 | 领域知识、编码规范 | `.claude/skills/` |
| **插件 (Plugins)** | 用户安装 | 打包的技能+命令+代理集合 | `.claude/plugins/` |

### 1.2 安装优先级建议

```
🔥 绝对必装（核心开发必备）
├── GitHub MCP Server
├── Postgres MCP Server
├── Sentry MCP Server
└── Superpowers 框架

⭐ 强烈推荐（提升效率）
├── Sequential Thinking MCP
├── Context7 MCP
├── Filesystem MCP
└── Browser MCP (Playwright)

💡 可选安装（特定需求）
├── Slack/Notion/Jira MCP
├── Linear MCP
├── Stripe MCP
└── Figma Dev Mode MCP
```

---

## 2. 必装 MCP 服务器

### 2.1 GitHub MCP Server ⭐⭐⭐⭐⭐

**功能：** 直接与 GitHub 仓库交互，管理 PR、Issue、Commit

**安装方式：**

```bash
# 方式一：HTTP 传输（推荐）
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 方式二：使用配置文件
cat > .mcp.json << EOF
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
EOF

# 需要 GitHub Copilot 订阅
```

**使用示例：**

```bash
# 在 Claude Code 中
> 查看 PR #456 的所有评论
> 列出所有分配给我的 Issues
> 为当前更改创建 PR
> 审查 PR #123 并给出建议
> 获取仓库 main 分支的最近 10 次提交
```

**可用的 MCP 工具：**

| 工具名 | 功能 |
|--------|------|
| `github__create_pr` | 创建 Pull Request |
| `github__list_issues` | 列出 Issues |
| `github__get_pr` | 获取 PR 详情 |
| `github__create_comment` | 添加评论 |
| `github__search_repos` | 搜索仓库 |

---

### 2.2 Postgres MCP Server ⭐⭐⭐⭐⭐

**功能：** 安全查询 PostgreSQL 数据库，分析 schema，执行优化查询

**安装方式：**

```bash
# 方式一：官方服务器（npm）
claude mcp add --transport stdio postgres \
  --env "DATABASE_URL=postgresql://user:password@localhost:5432/dbname" \
  -- npx -y @anthropic-ai/mcp-server-postgres

# 方式二：第三方增强版（推荐）
claude mcp add --transport stdio postgres-pro \
  --env "POSTGRES_CONNECTION_STRING=postgresql://..." \
  -- npx -y postgres-mcp-server

# 配置文件方式
cat >> .mcp.json << EOF
{
  "mcpServers": {
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
EOF
```

**使用示例：**

```bash
# 数据库查询
> 显示 users 表的 schema
> 查找所有过去 7 天注册的用户
> 分析 orders 表的索引使用情况
> 优化这个查询的性能：SELECT * FROM orders WHERE ...

# Schema 分析
> 列出数据库所有表及其关系
> 找出没有索引的外键
> 分析表大小和行数
```

**高级功能（postgres-mcp-server）：**

| 功能 | 说明 |
|------|------|
| 查询分析 | EXPLAIN ANALYZE 结果解析 |
| 索引建议 | 自动推荐索引优化 |
| 健康检查 | 数据库性能监控 |
| 安全执行 | 只读模式保护 |

---

### 2.3 Sentry MCP Server ⭐⭐⭐⭐⭐

**功能：** 实时监控和分析错误，获取生产环境问题详情

**安装方式：**

```bash
# 方式一：HTTP 传输（推荐）
claude mcp add --transport http sentry https://sentry.io/mcp/

# 方式二：配置环境变量
export SENTRY_API_KEY="your-api-key"
export SENTRY_ORGANIZATION="your-org"
export SENTRY_PROJECT="your-project"

# 添加到配置
cat >> .mcp.json << EOF
{
  "mcpServers": {
    "sentry": {
      "type": "http",
      "url": "https://sentry.io/api/",
      "headers": {
        "Authorization": "Bearer ${SENTRY_API_KEY}"
      }
    }
  }
}
EOF
```

**使用示例：**

```bash
# 错误分析
> 获取今天发生的所有错误
> 显示错误 ABC-123 的完整堆栈跟踪
> 分析过去一周的高频错误
> 查找与特定用户相关的所有错误

# 错误处理
> 获取错误发生时的用户上下文
> 分析错误的浏览器/操作系统分布
> 查看错误首次出现时间
```

---

### 2.4 Sequential Thinking MCP ⭐⭐⭐⭐

**功能：** 结构化推理模式，让 Claude 进行更深入的分析

**安装方式：**

```bash
# 添加 Sequential Thinking MCP
claude mcp add --transport stdio sequential-thinking \
  -- npx -y @modelcontextprotocol/server-sequential-thinking

# 或使用配置文件
cat >> .mcp.json << EOF
{
  "mcpServers": {
    "sequential-thinking": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
EOF
```

**使用示例：**

```bash
# 启用结构化思考
> 使用 sequential thinking 分析这个架构决策
> 分步骤思考这个问题：
> 1. 分析当前架构的瓶颈
> 2. 评估可能的解决方案
> 3. 比较每个方案的优劣
> 4. 给出最终建议
```

---

### 2.5 Context7 MCP ⭐⭐⭐⭐

**功能：** 实时获取特定版本的库文档，避免过时信息

**安装方式：**

```bash
# 添加 Context7 MCP
claude mcp add --transport http context7 https://context7.com/mcp/

# 使用配置文件
cat >> .mcp.json << EOF
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://api.context7.com/mcp/"
    }
  }
}
EOF
```

**使用示例：**

```bash
# 获取实时文档
> 查询 React 18.3.0 的 useEffect 文档
> 获取 Next.js 14 App Router 的最新文档
> 查询 TypeScript 5.4 的新特性
```

---

### 2.6 Playwright MCP (Browser) ⭐⭐⭐⭐

**功能：** 浏览器自动化，E2E 测试，UI 验证

**安装方式：**

```bash
# 添加 Playwright MCP
claude mcp add --transport stdio playwright \
  -- npx -y @executeautomation/playwright-mcp-server

# 配置示例
cat >> .mcp.json << EOF
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@executeautomation/playwright-mcp-server"],
      "env": {
        "HEADLESS": "true"
      }
    }
  }
}
EOF
```

**使用示例：**

```bash
# 浏览器操作
> 打开 https://example.com 并截图
> 填写登录表单并提交
> 运行 E2E 测试：从登录到购买流程
> 检查页面的可访问性问题
```

---

### 2.7 Filesystem MCP ⭐⭐⭐⭐

**功能：** 增强的文件系统操作，支持更多文件格式和操作

**安装方式：**

```bash
# 添加 Filesystem MCP
claude mcp add --transport stdio filesystem \
  -- npx -y @modelcontextprotocol/server-filesystem /path/to/allowed/directory

# 配置示例
cat >> .mcp.json << EOF
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/yourname/projects"]
    }
  }
}
EOF
```

---

## 3. Superpowers 技能框架

### 3.1 什么是 Superpowers

**Superpowers** 是目前最受欢迎的 Claude Code 技能框架（GitHub 34k+ stars），它提供了：

- 系统化的调试工作流
- TDD（测试驱动开发）完整流程
- 代码审查模式
- 前端设计工具
- 需求分析技能
- 完整的软件开发生命周期支持

**项目地址：** [obra/superpowers-developing-for-claude-code](https://github.com/obra/superpowers-developing-for-claude-code)

### 3.2 安装 Superpowers

**方式一：通过插件市场（推荐）**

```bash
# 启动 Claude Code
claude

# 添加 Superpowers 市场
> /plugin marketplace add obra/superpowers-marketplace

# 查看可用插件
> /plugin list

# 安装核心插件
> /plugin install superpowers-core
> /plugin install superpowers-tdd
> /plugin install superpowers-debug
> /plugin install superpowers-review
> /plugin install superpowers-frontend
```

**方式二：手动克隆**

```bash
# 克隆仓库
git clone https://github.com/obra/superpowers-developing-for-claude-code.git ~/.claude/plugins/superpowers

# 链接技能
ln -s ~/.claude/plugins/superpowers/skills/* ~/.claude/skills/

# 重启 Claude Code
```

### 3.3 Superpowers 核心技能

#### 技能 1：系统化调试 (Systematic Debugging)

**触发方式：** Claude 自动识别错误场景

**使用示例：**

```bash
# 当出现 bug 时
> 应用出现错误：TypeError: Cannot read property 'x' of undefined
# Claude 会自动应用调试技能

# 明确调用
> 使用 systematic debugging 分析这个问题：
> 1. 收集错误信息
> 2. 分析堆栈跟踪
> 3. 定位根本原因
> 4. 提出修复方案
```

**技能内容：**

```markdown
# Systematic Debugging Skill

## 调试协议

### 阶段 1：信息收集
- 收集完整的错误消息
- 获取堆栈跟踪
- 确定错误发生的上下文
- 识别相关代码路径

### 阶段 2：假设生成
- 基于信息生成可能的原因
- 按概率排序假设
- 识别关键验证点

### 阶段 3：假设验证
- 设计验证实验
- 执行验证步骤
- 记录结果

### 阶段 4：解决方案实施
- 选择最佳修复方案
- 实施修复
- 添加回归测试

### 阶段 5：预防措施
- 识别根本原因
- 提出预防建议
- 更新文档
```

#### 技能 2：TDD 工作流

**使用示例：**

```bash
# 开始 TDD 开发
> 使用 TDD 工作流开发用户认证功能

# Claude 会引导：
# 1. 先编写测试（红色）
# 2. 运行测试确认失败
# 3. 编写最少代码让测试通过（绿色）
# 4. 重构代码（重构）
# 5. 重复循环
```

**TDD 检查清单：**

```markdown
## TDD 循环

### 红色阶段
- [ ] 编写失败的测试
- [ ] 确认测试失败原因明确
- [ ] 验证测试失败消息有意义

### 绿色阶段
- [ ] 编写最少代码让测试通过
- [ ] 不添加额外功能
- [ ] 保持代码简单

### 重构阶段
- [ ] 改善代码结构
- [ ] 确保测试仍然通过
- [ ] 保持功能不变

### 验证阶段
- [ ] 所有测试通过
- [ ] 代码覆盖率满意
- [ ] 无明显代码异味
```

#### 技能 3：代码审查模式

**使用示例：**

```bash
> 使用 code review 模式审查这段代码
# 或
> 审查当前分支的所有变更

# Claude 会检查：
# - 正确性
# - 可读性
# - 可维护性
# - 性能
# - 安全性
# - 测试覆盖
```

#### 技能 4：前端设计工具

**使用示例：**

```bash
# 从设计稿生成代码
> [粘贴或拖拽 Figma 设计截图]
> 根据这个设计实现 React 组件

# Claude 会：
# 1. 分析设计元素
# 2. 选择合适的组件结构
# 3. 实现样式（Tailwind/CSS Modules）
# 4. 添加响应式支持
# 5. 实现交互逻辑
```

#### 技能 5：需求分析

**使用示例：**

```bash
> 使用需求分析技能处理这个需求：
> "用户希望能够保存和加载他们的工作进度"

# Claude 会：
# 1. 澄清需求细节
# 2. 识别边界条件
# 3. 提出技术方案
# 4. 估算开发时间
# 5. 识别潜在风险
```

### 3.4 Superpowers 技能结构

```
superpowers/
├── skills/
│   ├── systematic-debugging/
│   │   └── SKILL.md
│   ├── tdd-workflow/
│   │   └── SKILL.md
│   ├── code-review/
│   │   └── SKILL.md
│   ├── frontend-design/
│   │   └── SKILL.md
│   └── requirements-analysis/
│       └── SKILL.md
├── commands/
│   ├── debug.md
│   ├── tdd.md
│   └── review.md
└── agents/
    ├── debugger.md
    └── reviewer.md
```

---

## 4. 热门技能合集

### 4.1 awesome-claude-skills

**仓库地址：** [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)

**精选技能：**

| 技能名称 | 功能 | 安装方式 |
|---------|------|----------|
| **React Expert** | React 最佳实践、Hook 模式 | `git clone` 到 skills 目录 |
| **TypeScript Pro** | 高级 TypeScript 类型操作 | 同上 |
| **Python Mastery** | Python 异步、装饰器、类型提示 | 同上 |
| **Go Patterns** | Go 并发模式、错误处理 | 同上 |
| **Database Design** | 数据库建模、索引优化 | 同上 |
| **API Design** | REST/GraphQL 设计规范 | 同上 |
| **Testing Strategies** | 单元测试、集成测试策略 | 同上 |

**安装方式：**

```bash
# 克隆技能仓库
git clone https://github.com/travisvn/awesome-claude-skills.git ~/tmp/awesome-skills

# 复制你需要的技能
cp -r ~/tmp/awesome-skills/skills/react-expert ~/.claude/skills/
cp -r ~/tmp/awesome-skills/skills/typescript-pro ~/.claude/skills/

# 或者链接整个技能库
ln -s ~/tmp/awesome-skills/skills/* ~/.claude/skills/
```

### 4.2 创建自定义技能

**技能模板：**

```markdown
# .claude/skills/my-custom-skill/SKILL.md

---
name: my-custom-skill
description: 简洁描述这个技能的用途。在什么情况下 Claude 应该使用这个技能？
allowed-tools: Read, Edit, Write, Bash(npm run:*)
---

# 技能名称

## 使用场景

当以下情况时，Claude 应该使用此技能：
- 场景 1
- 场景 2
- 场景 3

## 核心原则

### 原则 1
详细说明...

### 原则 2
详细说明...

## 快速参考

### 常用模式
```typescript
// 代码示例
```

### 常用命令
```bash
# 命令示例
```

## 检查清单

- [ ] 检查项 1
- [ ] 检查项 2
- [ ] 检查项 3

## 相关资源

- [文档链接](https://...)
- [相关技能](./other-skill)
```

---

## 5. 官方插件推荐

### 5.1 LSP 插件 ⭐⭐⭐⭐⭐

**功能：** 提供 IDE 级别的代码智能（跳转定义、查找引用、悬停文档）

**安装：**

```bash
# LSP 功能内置在 Claude Code v2.0.74+
# 确保安装了语言服务器

# TypeScript
npm install -g typescript-language-server
npm install -g vscode-langservers-extracted

# Python
pip install python-lsp-server

# Go
go install golang.org/x/tools/gopls@latest
```

**使用：**

```bash
# LSP 工具自动可用
> 跳转到 useEffect 的定义
> 查找所有使用 UserContext 的地方
> 显示这个函数的类型信息
```

### 5.2 插件市场访问

**官方插件市场：**

```bash
# 列出所有可用插件
> /plugin marketplace list

# 搜索插件
> /plugin search react

# 安装插件
> /plugin install react-helper

# 查看已安装插件
> /plugin list

# 启用/禁用插件
> /plugin enable react-helper
> /plugin disable react-helper

# 更新插件
> /plugin update react-helper

# 卸载插件
> /plugin uninstall react-helper
```

---

## 6. 安装管理指南

### 6.1 MCP 服务器管理

```bash
# 查看所有 MCP 服务器
claude mcp list

# 获取特定服务器详情
claude mcp get github

# 添加服务器
claude mcp add --transport http <name> <url>

# 删除服务器
claude mcp remove <name>

# 重置项目配置
claude mcp reset-project-choices

# 添加本地市场
claude mcp add-from-claude-desktop

# JSON 方式添加
claude mcp add-json my-server '{"type":"http","url":"https://..."}'
```

### 6.2 技能管理

```bash
# 技能目录结构
.claude/skills/
├── personal/           # 个人技能（~/.claude/skills/）
├── project/           # 项目技能（.claude/skills/）
└── shared/            # 共享技能（通过 git 共享）

# 刷新技能（技能修改后）
> /plugin reload

# 验证技能
# 技能应该包含有效的 YAML frontmatter
```

### 6.3 配置文件管理

**全局配置 (`~/.claude/settings.json`)：**

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "defaultMode": "default"
  },
  "mcpToolSearchAutoEnable": "auto:15"
}
```

**项目配置 (`.claude/settings.json`)：**

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Edit(src/**)",
      "Bash(npm run:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

**MCP 配置 (`.mcp.json`)：**

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

### 6.4 故障排除

**MCP 服务器无法启动：**

```bash
# 检查配置
claude mcp get <server-name>

# 启用调试模式
claude --mcp-debug

# 检查环境变量
echo $DATABASE_URL
echo $GITHUB_TOKEN

# 测试连接
curl -I https://api.githubcopilot.com/mcp/
```

**技能未激活：**

```bash
# 检查技能文件
ls -la ~/.claude/skills/my-skill/SKILL.md

# 验证 YAML 格式
cat ~/.claude/skills/my-skill/SKILL.md | head -20

# 检查 description 字段
grep "description:" ~/.claude/skills/my-skill/SKILL.md

# 重新加载
# 重启 Claude Code
```

---

## 7. 实战使用案例

### 案例 1：完整的 Bug 修复工作流

```bash
# 使用 Superpowers 调试技能 + Sentry MCP

# 步骤 1：获取错误信息
> 使用 Sentry MCP 获取今天的错误
> 获取错误 ERR-123 的详细信息

# 步骤 2：应用调试技能
# Claude 自动激活 systematic debugging 技能
> 分析这个错误：
> 1. 收集堆栈跟踪和用户上下文
> 2. 识别可能的根本原因
> 3. 提出修复假设

# 步骤 3：定位代码
> 使用 GitHub MCP 查看相关代码的最近变更
> 找出可能引入 bug 的 commit

# 步骤 4：实施修复
> 使用 TDD 工作流修复这个问题：
> 1. 先编写失败的测试
> 2. 实施修复
> 3. 确保测试通过

# 步骤 5：代码审查
> 使用 code review 模式审查修复
> 创建 PR 并推送
```

### 案例 2：新功能开发

```bash
# 使用 Superpowers TDD + 需求分析技能

# 步骤 1：需求分析
> 使用 requirements analysis 技能分析需求：
> "实现用户个人资料编辑功能，包括头像上传"

# Claude 会：
# - 澄清细节（图片格式、大小限制、存储方式）
# - 识别技术方案（前端组件、API 端点、数据库变更）
# - 提出开发计划

# 步骤 2：TDD 开发
> 使用 TDD 工作流实现用户头像上传

# Claude 会引导：
# - 编写测试（上传成功、失败场景）
# - 实现后端 API
# - 实现前端组件
# - 集成测试

# 步骤 3：数据库变更
> 使用 Postgres MCP 添加 avatar_url 列到 users 表
> 创建数据库迁移文件

# 步骤 4：代码审查
> 审查所有变更
> 使用 code review 模式检查安全性和性能

# 步骤 5：创建 PR
> 使用 GitHub MCP 创建 Pull Request
> 包含详细的变更说明
```

### 案例 3：数据库性能优化

```bash
# 使用 Postgres MCP + Context7

# 步骤 1：分析慢查询
> 使用 Postgres MCP 获取慢查询列表
> 分析最慢的 10 个查询

# 步骤 2：查询优化
> 优化这个查询的性能：
> SELECT * FROM orders o
> JOIN users u ON o.user_id = u.id
> WHERE o.created_at > NOW() - INTERVAL '7 days'

# Claude 会：
# - 使用 EXPLAIN ANALYZE
# - 识别缺失的索引
# - 推荐索引优化方案
# - 重写查询

# 步骤 3：参考最佳实践
> 使用 Context7 查询 PostgreSQL 16 的索引最佳实践
> 查询关于复合索引的使用建议

# 步骤 4：实施优化
> 创建推荐的索引
> 验证性能改进
> 记录优化结果
```

### 案例 4：前端组件开发

```bash
# 使用 Superpowers 前端设计工具 + Playwright MCP

# 步骤 1：分析设计
> [拖拽 Figma 设计截图]
> 分析这个登录页面的设计元素
> 识别所需的组件结构

# 步骤 2：实现组件
> 使用 frontend design 工具实现这个登录页面
> 使用 React + Tailwind CSS
> 确保响应式设计

# Claude 会：
# - 创建组件结构
# - 实现 TypeScript 类型
# - 添加样式
# - 实现表单验证
# - 添加可访问性支持

# 步骤 3：E2E 测试
> 使用 Playwright MCP 编写 E2E 测试
> 测试登录流程
> 验证错误处理
> 检查可访问性

# 步骤 4：视觉回归测试
> 使用 Playwright 截图
> 与设计对比
> 迭代调整
```

---

## 附录：快速参考

### A. 必装清单（快速复制）

```bash
# MCP 服务器
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
claude mcp add --transport http sentry https://sentry.io/mcp/
claude mcp add --transport http context7 https://context7.com/mcp/
claude mcp add --transport stdio postgres -- npx -y @anthropic-ai/mcp-server-postgres
claude mcp add --transport stdio sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
claude mcp add --transport stdio filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/projects

# Superpowers
claude
> /plugin marketplace add obra/superpowers-marketplace
> /plugin install superpowers-core
> /plugin install superpowers-tdd
> /plugin install superpowers-debug

# awesome-skills
git clone https://github.com/travisvn/awesome-claude-skills.git ~/tmp/awesome-skills
cp -r ~/tmp/awesome-skills/skills/* ~/.claude/skills/
```

### B. 配置模板

**完整的项目配置 (`.claude/settings.json` + `.mcp.json`)：**

```json
// .claude/settings.json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read",
      "Edit(src/**)",
      "Write(src/**)",
      "Bash(npm run:*)",
      "Bash(git:*)"
    ],
    "deny": [
      "Read(.env*)",
      "Bash(rm -rf:*)"
    ],
    "defaultMode": "default"
  },
  "mcpToolSearchAutoEnable": "auto:15"
}
```

```json
// .mcp.json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "sentry": {
      "type": "http",
      "url": "https://sentry.io/api/",
      "headers": {
        "Authorization": "Bearer ${SENTRY_API_KEY}"
      }
    }
  }
}
```

### C. 有用的链接

| 资源 | 链接 |
|------|------|
| **MCP 官方服务器** | [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) |
| **Awesome MCP Servers** | [wong2/awesome-mcp-servers](https://github.com/wong2/awesome-mcp-servers) |
| **Superpowers 框架** | [obra/superpowers](https://github.com/obra/superpowers-developing-for-claude-code) |
| **Awesome Skills** | [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) |
| **MCP Server 目录** | [mcp-awesome.com](https://mcp-awesome.com/) |

---

**更新日期：** 2026-02-02

**文档版本：** 1.0
