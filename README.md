# Claude Code 完整使用手册

> 全面掌握 Anthropic Claude Code CLI 工具 —— 从入门到精通的完整指南

![Claude Code](https://img.shields.io/badge/Claude_Code-v2.1.x-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Language](https://img.shields.io/badge/lang-Chinese-red)

---

## 📚 文档目录

本仓库包含三份完整的 Claude Code 使用手册，涵盖从基础到高级的全方位内容：

### 📘 [基础使用手册](./claude-code-manual.md)

适合初学者，涵盖 Claude Code 的核心概念和基础功能。

**内容包括：**
- 快速入门与安装
- 核心功能介绍
- 配置系统详解
- 权限管理
- MCP 集成
- 子代理系统
- 技能系统
- 钩子系统
- 最佳实践
- 常见命令
- 成本优化
- 故障排除

**适合人群：** Claude Code 新手、需要快速上手的开发者

---

### 📗 [进阶使用手册](./claude-code-advanced-manual.md)

面向有经验的开发者，深入讲解高级技巧和实战案例。

**内容包括：**
- 高级工作流模式（探索→规划→编码→验证）
- 6 个完整实战案例：
  - 重构遗留代码系统
  - 实现完整的 API 端点
  - 调试生产环境问题
  - 数据库迁移（零停机）
  - 自动化 CI/CD 流程
  - 多语言项目协作
- 团队协作最佳实践
- 性能调优与成本控制
- 高级技巧集锦
- 故障排除进阶

**适合人群：** 有 Claude Code 基础、希望提升技能的开发者

---

### 📙 [必装插件与技能指南](./claude-code-essentials.md)

详细介绍 Claude Code 生态系统中的扩展工具。

**内容包括：**
- 必装 MCP 服务器（GitHub、Postgres、Sentry、Sequential Thinking、Context7、Playwright）
- Superpowers 技能框架详解
- 热门技能合集
- 官方插件推荐
- 安装管理指南
- 实战使用案例

**适合人群：** 所有 Claude Code 用户

---

## 🚀 快速开始

### 安装 Claude Code

```bash
# macOS / Linux
curl -fsSL https://claude.ai/install.sh | bash

# macOS Homebrew
brew install --cask claude-code

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex
```

### 首次使用

```bash
cd your-project
claude
```

### 必装 MCP 服务器

```bash
# GitHub - 代码仓库管理
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# Postgres - 数据库操作
claude mcp add --transport stdio postgres -- npx -y @anthropic-ai/mcp-server-postgres

# Sentry - 错误监控
claude mcp add --transport http sentry https://sentry.io/mcp/

# Sequential Thinking - 深度推理
claude mcp add --transport stdio sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

### 安装 Superpowers 框架

```bash
# 添加 Superpowers 市场
/plugin marketplace add obra/superpowers-marketplace

# 安装核心插件
/plugin install superpowers-core
/plugin install superpowers-tdd
/plugin install superpowers-debug
/plugin install superpowers-review
```

---

## 🎯 核心概念

### MCP 服务器

MCP (Model Context Protocol) 扩展 Claude Code 连接外部服务的能力：

| 服务器 | 功能 | 星级 |
|--------|------|------|
| GitHub | PR/Issue/代码管理 | ⭐⭐⭐⭐⭐ |
| Postgres | 数据库查询优化 | ⭐⭐⭐⭐⭐ |
| Sentry | 错误监控分析 | ⭐⭐⭐⭐⭐ |
| Sequential Thinking | 结构化推理 | ⭐⭐⭐⭐ |
| Context7 | 实时文档查询 | ⭐⭐⭐⭐ |
| Playwright | 浏览器自动化 | ⭐⭐⭐⭐ |

### 技能 (Skills)

技能是 Claude 可以自动识别并应用的领域知识：

- **系统化调试** - 结构化的问题解决流程
- **TDD 工作流** - 测试驱动开发循环
- **代码审查** - 全面的代码质量检查
- **前端设计** - 从设计稿到组件代码
- **需求分析** - 完整的需求处理流程

### 子代理 (Subagents)

专注处理特定任务的 Claude 实例：

- **Explore** - 快速代码库探索（Haiku）
- **General-purpose** - 复杂研究+修改（Sonnet）
- **Plan** - 实现前规划（Sonnet/Opus）

---

## 📖 使用示例

### 基础使用

```bash
# 启动交互式会话
claude

# 单次查询
claude -p "列出项目中的所有 TODO 注释"

# 继续上一次会话
claude -c

# 查看使用成本
> /cost

# 压缩对话历史
> /compact
```

### MCP 集成使用

```bash
# 使用 GitHub MCP
> 查看 PR #456 的所有评论
> 为当前更改创建 PR

# 使用 Postgres MCP
> 显示 users 表的 schema
> 优化这个查询的性能

# 使用 Sentry MCP
> 获取今天发生的所有错误
> 分析错误 ABC-123 的根本原因
```

### Superpowers 技能使用

```bash
# 系统化调试
> 应用出现错误，使用 systematic debugging 分析

# TDD 开发
> 使用 TDD 工作流开发用户认证功能

# 代码审查
> 使用 code review 模式审查这段代码

# 前端开发
> [拖拽设计截图]
> 根据这个设计实现 React 组件
```

---

## 🛠️ 配置文件

### 项目配置 (`.claude/settings.json`)

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": ["Read", "Edit(src/**)", "Bash(npm run:*)"],
    "deny": ["Read(.env*)", "Bash(rm -rf:*)"],
    "defaultMode": "default"
  },
  "mcpToolSearchAutoEnable": "auto:15"
}
```

### MCP 配置 (`.mcp.json`)

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

---

## 📊 模型选择指南

| 模型 | 输入价格 | 输出价格 | 适用场景 |
|------|----------|----------|----------|
| **Haiku 4.5** | $1/M | $5/M | 简单任务、文件搜索、子代理探索 |
| **Sonnet 4.5** | $3/M | $15/M | 日常开发（默认推荐） |
| **Opus 4.5** | $5/M | $25/M | 复杂推理、架构决策、安全分析 |

**决策规则：**
```
简单任务？ → Haiku
需要深度推理？ → Opus
其他情况 → Sonnet
```

---

## 🔗 相关资源

### 官方资源

- [Claude Code 官方文档](https://code.claude.com/docs)
- [Claude API 文档](https://docs.anthropic.com)
- [MCP 官方服务器](https://github.com/modelcontextprotocol/servers)

### 社区资源

- [Superpowers 框架](https://github.com/obra/superpowers-developing-for-claude-code) ⭐ 34k+
- [Awesome MCP Servers](https://github.com/wong2/awesome-mcp-servers)
- [Awesome Claude Skills](https://github.com/travisvn/awesome-claude-skills)
- [MCP Servers 目录](https://mcp-awesome.com/)

### 推荐阅读

- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [The Claude Code Survival Guide for 2026](https://www.linkedin.com/pulse/claude-code-survival-guide-2026-skills-agents-mcp-servers-rob-foster-lq9we)

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

如果你有任何改进建议或发现文档中的错误，请随时贡献。

---

## 📄 许可证

MIT License

---

## 🌟 Star History

如果这份文档对你有帮助，请给个 Star ⭐

---

**文档版本：** 1.0
**最后更新：** 2026-02-02
**Claude Code 版本：** v2.1.x 系列
