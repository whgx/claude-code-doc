# Claude Code 使用手册

> Claude Code 是 Anthropic 官方开发的终端 AI 编程助手，可直接读取代码库、执行命令、修改文件、管理 Git 工作流，并通过 MCP 连接外部服务。

---

## 目录

1. [快速入门](#1-快速入门)
2. [核心功能](#2-核心功能)
3. [配置系统](#3-配置系统)
4. [权限管理](#4-权限管理)
5. [MCP 集成](#5-mcp-集成)
6. [子代理系统](#6-子代理系统)
7. [技能系统](#7-技能系统)
8. [钩子系统](#8-钩子系统)
9. [最佳实践](#9-最佳实践)
10. [常见命令](#10-常见命令)
11. [成本优化](#11-成本优化)
12. [故障排除](#12-故障排除)

---

## 1. 快速入门

### 系统要求

- **操作系统**: macOS 10.15+、Ubuntu 20.04+/Debian 10+、Windows 10+ (WSL 或 Git Bash)
- **内存**: 最低 4GB RAM
- **账户**: Claude 订阅（Pro、Max、Teams 或 Enterprise）或 Claude Console 账户

### 安装方式

**原生安装（推荐）**

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd

# Homebrew
brew install --cask claude-code

# WinGet
winget install Anthropic.ClaudeCode
```

**首次使用**

```bash
cd your-project
claude
```

系统会提示登录，完成后即可开始使用。

---

## 2. 核心功能

### 2.1 主要能力

| 功能 | 说明 |
|------|------|
| **构建功能** | 用自然语言描述需求，Claude 会制定计划、编写代码并确保正常工作 |
| **调试修复** | 描述 bug 或粘贴错误信息，Claude 会分析代码库并实施修复 |
| **代码库导航** | 询问任何关于项目的问题，Claude 会保持对整个项目结构的了解 |
| **自动化任务** | 修复 lint 问题、解决合并冲突、编写发布说明等 |

### 2.2 模型选择

| 模型 | 价格（每百万 token） | 适用场景 |
|------|---------------------|----------|
| **Haiku 4.5** | 输入 $1 / 输出 $5 | 简单任务、快速操作、子代理探索 |
| **Sonnet 4.5** | 输入 $3 / 输出 $15 | 日常开发（默认推荐） |
| **Opus 4.5** | 输入 $5 / 输出 $25 | 复杂推理、架构决策、安全分析 |

**决策规则**
```
简单任务？ → Haiku
需要深度推理？ → Opus
其他情况 → Sonnet
```

### 2.3 交互模式

**交互式 REPL**

```bash
claude                                    # 启动交互式会话
claude "解释项目中的认证流程"              # 带初始提示启动
```

**非交互模式**

```bash
claude -p "列出项目中的所有 TODO 注释"
cat error.log | claude -p "识别失败的根因"
claude -p "生成 README" > README.md       # JSON 输出
claude -p "统计代码行数" --output-format json
```

**会话管理**

```bash
claude -c                                 # 继续最近的会话
claude -c -p "现在添加错误处理"            # 继续并添加新提示
claude -r "session-id" "继续实现测试"     # 恢复特定会话
```

---

## 3. 配置系统

### 3.1 配置文件层次

| 级别 | 位置 | 作用域 | 是否可覆盖 |
|------|------|--------|------------|
| **企业级** | `/etc/claude-code/managed-settings.json` | 所有用户 | 否 |
| **项目共享** | `.claude/settings.json` | 团队（通过 git） | 是 |
| **项目本地** | `.claude/settings.local.json` | 个人、当前项目 | 是 |
| **用户级** | `~/.claude/settings.json` | 所有项目 | 是 |
| **CLI 标志** | 命令行参数 | 当前会话 | 是 |

### 3.2 配置文件示例

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Bash(npm run:*)",
      "Bash(git:*)",
      "Edit(src/**)",
      "Write(src/**)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(secrets/**)",
      "Bash(rm -rf:*)",
      "Bash(sudo:*)"
    ],
    "defaultMode": "acceptEdits"
  },
  "env": {
    "NODE_ENV": "development"
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$FILE_PATH\""
          }
        ]
      }
    ]
  },
  "outputStyle": "Explanatory",
  "respectGitignore": true,
  "showTurnDuration": true
}
```

### 3.3 环境变量

```bash
# 认证和 API
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-opus-4-5

# 模型配置
MAX_THINKING_TOKENS=10000                 # 扩展思考预算
CLAUDE_CODE_MAX_OUTPUT_TOKENS=4000        # 限制输出长度
CLAUDE_CODE_SUBAGENT_MODEL=sonnet         # 子代理模型

# 行为控制
DISABLE_AUTOUPDATER=1                     # 禁用自动更新
DISABLE_TELEMETRY=1                       # 禁用遥测
BASH_DEFAULT_TIMEOUT_MS=30000             # Bash 超时

# 网络和代理
HTTP_PROXY=http://proxy:8080
HTTPS_PROXY=https://proxy:8080

# 云提供商
CLAUDE_CODE_USE_BEDROCK=1                 # AWS Bedrock
CLAUDE_CODE_USE_VERTEX=1                  # Google Vertex AI
```

---

## 4. 权限管理

### 4.1 权限模式

| 模式 | 行为 | 使用场景 |
|------|------|----------|
| `default` | 首次使用时提示 | 正常开发 |
| `acceptEdits` | 自动批准文件编辑，bash 需确认 | 受信任的项目 |
| `plan` | 不允许执行或编辑 | 仅分析 |
| `bypassPermissions` | 跳过所有提示 | CI/CD 自动化 |

### 4.2 权限规则语法

**Bash 命令模式**

```json
{
  "allow": [
    "Bash(npm run build)",           // 精确匹配
    "Bash(npm run test:*)",          // 前缀匹配
    "Bash(git commit:*)",
    "Bash(make:*)"
  ],
  "deny": [
    "Bash(rm -rf:*)",
    "Bash(sudo:*)",
    "Bash(curl:*)"                   // 阻止所有 curl
  ]
}
```

**文件操作模式**

```json
{
  "allow": [
    "Edit(src/**)",
    "Write(src/**)",
    "Read(docs/**)"
  ],
  "deny": [
    "Read(.env*)",
    "Read(secrets/**)",
    "Edit(.git/**)",
    "Edit(node_modules/**)"
  ]
}
```

**路径语法**
- 相对路径: `Edit(src/**)` - 相对于工作目录
- 绝对路径: `Edit(//tmp/**)` - 以 `//` 开头
- 主目录: `Read(~/.zshrc)`

**MCP 工具模式**

```json
{
  "allow": [
    "mcp__github",
    "mcp__database__query",
    "mcp__myserver__*"               // 通配符
  ]
}
```

### 4.3 沙箱模式

```bash
> /sandbox
```

或配置：

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["git", "docker"]
  }
}
```

---

## 5. MCP 集成

### 5.1 什么是 MCP

MCP（Model Context Protocol）扩展 Claude Code 连接外部工具、数据库、API 和服务。截至 2026 年 1 月，MCP 拥有 **1 亿月下载量** 和 **3000+ 服务器**。

### 5.2 MCP 服务器配置

**项目级配置 (`.mcp.json`)**

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
    },
    "sentry": {
      "type": "http",
      "url": "https://mcp.sentry.dev/mcp",
      "headers": {
        "Authorization": "Bearer ${SENTRY_API_KEY}"
      }
    }
  }
}
```

### 5.3 MCP 管理命令

```bash
claude mcp list                          # 查看所有服务器
claude mcp add                           # 交互式添加
claude mcp add --transport http github https://...
claude mcp get github                    # 获取详情
claude mcp remove github                 # 删除
```

### 5.4 使用 MCP

```bash
> Review PR #456
> What errors occurred today in Sentry?
> Show the schema for the users table
```

### 5.5 常用 MCP 服务器

| 服务器 | 用途 | 关键能力 |
|--------|------|----------|
| **GitHub** | 代码仓库管理 | PR、Issue、CI/CD、代码审查 |
| **PostgreSQL** | 数据库访问 | 查询、模式检查、数据分析 |
| **Sentry** | 错误监控 | 错误查找、堆栈跟踪 |
| **Linear/Jira** | 项目管理 | Issue、项目、Sprint |
| **Playwright** | Web 自动化 | E2E 测试、可访问性 |
| **Figma Dev Mode** | 设计转代码 | 图层层次、自动布局 |
| **Stripe** | 支付 | 交易查询、客户数据 |

---

## 6. 子代理系统

### 6.1 为什么使用子代理

- **防止上下文膨胀**: 探索结果不会污染主对话
- **并行执行**: 可同时运行最多 10 个子代理
- **专注工作**: 每个子代理有独立的上下文窗口

### 6.2 内置子代理类型

| 类型 | 模型 | 模式 | 工具 | 用途 |
|------|------|------|------|------|
| **Explore** | Haiku | 只读 | Glob, Grep, Read, 安全 bash | 代码库探索 |
| **General-purpose** | Sonnet | 读写 | 所有可用工具 | 复杂研究+修改 |
| **Plan** | Sonnet/Opus | 只读 | Read, Glob, Grep, Bash | 实现前规划 |

### 6.3 触发子代理

```bash
> Use the explore agent to find all authentication-related files
> Have a subagent analyze the database schema thoroughly
> Spawn an agent to research how error handling works
```

### 6.4 创建自定义子代理

在 `.claude/agents/` 创建文件：

```markdown
---
name: security-reviewer
description: 专家级安全代码审查员。在任何认证、授权或数据处理代码变更后主动使用。
tools: Read, Grep, Glob, Bash
model: opus
permissionMode: plan
---

你是负责审查代码漏洞的高级安全工程师。

当被调用时：
1. 识别最近更改的文件
2. 分析 OWASP Top 10 漏洞
3. 检查密钥、硬编码凭证、SQL 注入
4. 报告发现及严重程度和修复步骤

专注于可操作的安全发现，而非样式问题。
```

### 6.5 子代理管理

```bash
> /agents                    # 交互式管理
> /agents create             # 创建新子代理
> /agents list               # 查看所有
```

---

## 7. 技能系统

### 7.1 技能 vs 命令 vs 子代理

| 特性 | 斜杠命令 | 技能 | 子代理 |
|------|----------|------|--------|
| **调用方式** | 用户调用 (`/command`) | 模型调用（自动） | 显式或自动委托 |
| **触发条件** | 你记得使用它 | Claude 识别上下文 | 任务需要委托 |
| **结构** | 单个 markdown 文件 | 带资源的目录 | 带 frontmatter 的 markdown |
| **上下文** | 主对话 | 主对话 | 独立上下文窗口 |

### 7.2 技能结构

```
.claude/skills/
├── code-reviewer/           # 项目技能（通过 git 共享）
│   ├── SKILL.md
│   ├── SECURITY_PATTERNS.md
│   └── PERFORMANCE_CHECKLIST.md
└── payments-domain/         # 个人技能（~/.claude/skills/）
    ├── SKILL.md
    ├── BUSINESS_RULES.md
    └── COMPLIANCE.md
```

### 7.3 SKILL.md 格式

```markdown
---
name: payments-domain
description: 支付处理领域专家知识，包括交易流程、合规要求和业务规则。在处理支付代码、交易处理、退款、争议或财务计算时使用。
allowed-tools: Read, Grep, Glob, Bash
---

# 支付处理领域专业知识

## 核心概念

### 交易状态
```
PENDING → AUTHORIZED → CAPTURED → SETTLED
↘ VOIDED
AUTHORIZED → DECLINED
```

### 货币处理规则
- 所有货币值存储为整数（美分，而非美元）
- 货币始终显式跟踪（从不假设 USD）
- 绝不使用浮点数处理货币

## 快速参考

### 费用计算
- 交换费: 借记卡 1.5% + $0.10，信用卡 2.1% + $0.10
- 平台费: 标准 2.9% + $0.30
- 国际: +1% 跨境费

### 合规阈值
- $3,000: 增强尽职调查触发
- $10,000: CTR 备案要求
```

### 7.4 技能调试

**技能未激活**
1. 检查 description 是否匹配你的请求
2. 验证文件位置: `ls .claude/skills/my-skill/SKILL.md`
3. 验证 YAML frontmatter 格式

**技能意外激活**
- 缩小 description 范围，使其更具体

---

## 8. 钩子系统

### 8.1 为什么使用钩子

钩子保证执行，无论模型行为如何。对于合规、安全和团队标准，**确定性胜过概率性**。

### 8.2 可用事件

| 事件 | 时机 | 可阻止 | 用途 |
|------|------|--------|------|
| `PreToolUse` | 工具执行前 | 是 | 验证、日志、阻止 |
| `PostToolUse` | 工具完成后 | 否 | 格式化、lint、触发构建 |
| `UserPromptSubmit` | 用户发送提示 | 是 | 添加上下文、验证输入 |
| `SessionStart` | 会话开始 | 是 | 环境设置、验证 |
| `SessionEnd` | 会话结束 | 否 | 清理、日志 |
| `Stop` | Claude 完成响应 | 否 | 清理、指标 |

### 8.3 钩子配置

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$FILE_PATH\""
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
            "command": ".claude/hooks/validate-bash.sh"
          }
        ]
      }
    ]
  }
}
```

### 8.4 匹配器语法

```json
{"matcher": "*"}              // 匹配所有工具
{"matcher": "Bash"}           // 仅匹配 Bash
{"matcher": "Edit|Write"}     // 匹配 Edit 或 Write
{"matcher": ""}               // 事件无工具（如 UserPromptSubmit）
```

### 8.5 钩子输入/输出协议

钩子接收 JSON 输入：

```json
{
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test",
    "description": "Run test suite"
  },
  "session_id": "abc-123",
  "file_path": "/path/to/file.ts"  // PostToolUse 时可用
}
```

退出代码：
- **0**: 成功 - 操作继续
- **2**: 阻止错误 - 操作停止，stderr 成为错误消息
- **1, 3, ...**: 非阻止错误 - 操作继续

### 8.6 实用钩子示例

**自动格式化 TypeScript**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c '[[ \"$FILE_PATH\" == *.ts ]] && npx prettier --write \"$FILE_PATH\" || true'"
          }
        ]
      }
    ]
  }
}
```

**阻止访问敏感文件**

```bash
#!/bin/bash
# .claude/hooks/protect-files.sh
data=$(cat)
path=$(echo "$data" | jq -r '.tool_input.file_path // empty')

if [[ "$path" == *".env"* ]] || [[ "$path" == *"secrets/"* ]]; then
  echo "Blocked: Cannot access sensitive file $path" >&2
  exit 2
fi
exit 0
```

---

## 9. 最佳实践

### 9.1 CLAUDE.md 设计

```markdown
# 项目上下文

## 架构
- Monorepo，包位于 /packages
- React 前端在 /packages/ui
- Node.js API 在 /packages/api
- 通过 Prisma 使用 PostgreSQL

## 代码标准
- 所有地方使用 TypeScript 严格模式
- ESLint + Prettier（预提交钩子强制执行）
- 无默认导出
- 所有公共 API 上有 JSDoc

## 命令
- `npm test` - 运行所有测试
- `npm run lint` - 检查 lint
- `npm run build` - 生产构建

## 重要说明
- 绝不提交 .env 文件
- API 运行在 :3000，UI 在 :3001

# 压缩说明
使用压缩时，专注于：
- 最近的代码更改
- 测试结果
- 本次会议做出的架构决策
```

### 9.2 工作流模式

**探索 → 规划 → 编码 → 提交**

```
1. 让 Claude 阅读相关文件、图片或 URL
   - 明确告诉它暂不编写代码
   - 考虑使用子代理进行验证

2. 让 Claude 制定解决问题的计划
   - 使用 "think" 触发扩展思考模式

3. 让 Claude 实现解决方案

4. 让 Claude 提交结果并创建 PR
```

**测试驱动开发**

```
1. 询问 Claude 基于预期输入/输出对编写测试
2. 告诉 Claude 运行测试并确认失败
3. 让 Claude 提交测试
4. 让 Claude 编写通过测试的代码
5. 让 Claude 提交代码
```

**视觉迭代**

```
1. 给 Claude 一种截取浏览器截图的方法
2. 给 Claude 视觉模型（复制粘贴或拖放图片）
3. 让 Claude 在代码中实现设计，截图结果，迭代直到匹配
4. 让 Claude 提交
```

### 9.3 提示技巧

| 差 | 好 |
|---|---|
| 添加 foo.py 的测试 | 为 foo.py 编写新的测试用例，覆盖用户登出的边缘情况。避免使用 mock |
| 为什么 ExecutionFactory API 这么奇怪？ | 查看 ExecutionFactory 的 git 历史并总结其 API 如何演变 |
| 添加日历小部件 | 查看主页上现有小部件的实现...然后遵循该模式实现... |

**关键原则**
- 具体性 → 更好的预期对齐
- 直接引用文件 → Claude 不需要猜测路径
- 提供约束 → Claude 遵循特定模式
- 使用子代理进行探索 → 防止上下文膨胀

### 9.4 成本控制

| 策略 | 说明 |
|------|------|
| 使用 Haiku 子代理 | 探索通常不需要 Sonnet |
| 定期检查 `/cost` | 在主要任务后查看支出 |
| 仅在需要时设置 `MAX_THINKING_TOKENS` | 扩展思考增加成本 |
| 使用 `--max-turns` | 防止自动化脚本中的失控对话 |
| 主动压缩 | 在长会话中 `/compact` |

### 9.5 安全实践

- 配置 `.claude/settings.json` 拒绝规则阻止敏感文件
- 对不受信任的项目使用沙箱模式
- 绝不允许 `Bash(rm -rf:*)` 或 `Bash(sudo:*)`
- 使用钩子阻止访问密钥
- 为企业部署托管设置

---

## 10. 常见命令

### 10.1 斜杠命令

| 命令 | 用途 |
|------|------|
| `/init` | 初始化项目并创建 CLAUDE.md |
| `/memory` | 编辑内存文件 |
| `/context` | 查看上下文窗口使用情况 |
| `/compact` | 压缩对话历史 |
| `/cost` | 显示 token 使用和成本 |
| `/stats` | 使用统计 |
| `/permissions` | 管理权限设置 |
| `/mcp` | 配置 MCP 服务器 |
| `/hooks` | 查看钩子配置 |
| `/config` | 打开设置界面 |
| `/resume` | 恢复命名会话 |
| `/rename` | 命名当前会话 |
| `/clear` | 清除对话历史 |
| `/model` | 切换 AI 模型 |
| `/agents` | 管理子代理 |
| `/bashes` | 列出后台任务 |
| `/tasks` | 列出后台代理 |
| `/doctor` | 检查安装健康状态 |
| `/status` | 查看会话状态 |
| `/sandbox` | 启用沙箱模式 |
| `/vim` | 启用 vim 编辑模式 |

### 10.2 快捷键

| 快捷键 | 动作 |
|--------|------|
| `Ctrl+C` | 取消当前操作 |
| `Ctrl+D` | 退出会话 |
| `Ctrl+L` | 清除屏幕 |
| `Ctrl+O` | 切换详细输出 |
| `Ctrl+V` | 粘贴图片 |
| `Ctrl+B` | 后台运行当前操作 |
| `Esc Esc` | 撤销上次更改 |
| `Tab` | 接受提示建议 / 切换扩展思考 |
| `Shift+Tab` | 循环权限模式 |
| `Alt+T` | 切换思考模式 |
| `Ctrl+T` | 在 `/theme` 中切换语法高亮 |

### 10.3 快速前缀

| 前缀 | 动作 | 示例 |
|------|------|------|
| `#` | 添加到内存 | `# 始终使用 TypeScript` |
| `/` | 斜杠命令 | `/review` |
| `!` | 直接 bash | `! git status` |
| `@` | 文件引用 | `@src/index.ts` |
| `&` | 发送到云端 | `& 构建该 API` |

---

## 11. 成本优化

### 11.1 查看成本

```bash
> /cost
```

输出：
```
Total cost:            $0.55
Total duration (API):  6m 19.7s
Total duration (wall): 6h 33m 10.2s
Total code changes:    247 lines added, 89 lines removed
```

### 11.2 定价参考

| 模型 | 输入 | 输出 |
|------|------|------|
| Haiku 4.5 | $1/M | $5/M |
| Sonnet 4.5 | $3/M | $15/M |
| Opus 4.5 | $5/M | $25/M |

### 11.3 实际成本示例

| 任务 | 模型 | 输入 | 输出 | 成本 |
|------|------|------|------|------|
| 快速文件搜索 | Haiku | 20K | 2K | $0.03 |
| 带测试的 bug 修复 | Sonnet | 100K | 30K | $0.75 |
| 架构审查 | Opus | 150K | 50K | $2.00 |
| 全天会话 | Sonnet | 500K | 150K | $3.75 |

### 11.4 成本节省策略

1. **对子代理使用 Haiku** - 探索通常不需要 Sonnet
2. **启用提示缓存** - 默认启用，验证是否未被禁用
3. **设置最大轮次** - `claude --max-turns 5` 防止失控对话
4. **对探索使用计划模式** - 无执行 = 无意外昂贵操作
5. **主动压缩** - 较小的上下文 = 较少的 token
6. **限制输出** - `export CLAUDE_CODE_MAX_OUTPUT_TOKENS=2000`

---

## 12. 故障排除

### 12.1 安装问题

**WSL 路径问题**

```bash
npm config set os linux
npm install -g @anthropic-ai/claude-code --force --no-os-check
```

**权限错误**

```bash
# 使用原生安装代替 npm
curl -fsSL https://claude.ai/install.sh | bash
```

### 12.2 认证问题

```bash
# 完全重置
/logout
rm -rf ~/.config/claude-code/auth.json
claude  # 新登录
```

### 12.3 性能问题

**高 CPU/内存**

- 使用 `/compact` 减少上下文
- 在主要任务之间重启
- 将大目录添加到 `.gitignore`
- 运行 `claude doctor`

**搜索慢**

```bash
# macOS
brew install ripgrep

# 安装系统 ripgrep 后
export USE_BUILTIN_RIPGREP=0
```

### 12.4 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| "Rate limit exceeded" | 请求过多 | 等待或减少频率 |
| "Context length exceeded" | 对话太长 | 使用 `/compact` 或 `/clear` |
| "Authentication failed" | 令牌无效或过期 | 运行 `/login` |
| "Tool not permitted" | 权限被拒绝 | 检查 settings.json 权限 |
| "MCP server failed to start" | 服务器配置错误 | 检查 `claude mcp get <name>` |

### 12.5 调试模式

```bash
claude --debug                    # 完整调试输出
ANTHROPIC_LOG=debug claude       # API 请求日志
claude doctor                    # 健康检查
```

---

## 附录

### IDE 集成

**VS Code 扩展**
- 搜索 "Claude Code" 安装
- 侧边栏面板（Spark 图标）
- 计划模式与差异预览
- 文件附件和图片粘贴

**JetBrains 插件**
- 支持 IntelliJ IDEA、PyCharm、WebStorm 等
- 设置 → 插件 → 搜索 "Claude Code"

### 企业部署

**托管设置部署**

位置：
- macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`
- Linux: `/etc/claude-code/managed-settings.json`
- Windows: `C:\ProgramFiles\ClaudeCode\managed-settings.json`

示例策略：

```json
{
  "permissions": {
    "allow": ["Read", "Glob", "Grep", "Bash(npm run:*)", "Bash(git:*)"],
    "deny": ["Bash(rm -rf:*)", "Bash(curl:*)", "Read(.env*)", "WebFetch"],
    "defaultMode": "default"
  },
  "model": "claude-sonnet-4-5-20250929",
  "disableBypassPermissionsMode": "disable"
}
```

### 参考资源

- **官方文档**: https://code.claude.com/docs
- **GitHub**: https://github.com/anthropics/claude-code
- **最佳实践**: https://www.anthropic.com/engineering/claude-code-best-practices
- **技术参考**: https://blakecrosley.com/en/guides/claude-code

---

**文档版本**: 2026-02-02

**Claude Code 版本**: v2.1.x 系列
