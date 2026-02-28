# 高级工作流指南实现计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 创建 Claude Code 四大核心工作流的实用级中文文档指南

**Architecture:** 多文件模式组织，每个工作流一个独立 Markdown 文件，统一索引文件，与现有文档通过 README.md 集成

**Tech Stack:** Markdown, 中文文档, Claude Code 技能/插件生态

---

## Task 1: 创建目录结构

**Files:**
- Create: `docs/workflows/` (目录)

**Step 1: 创建 workflows 目录**

```bash
mkdir -p docs/workflows
```

**Step 2: 验证目录创建成功**

```bash
ls -la docs/
```
Expected: 看到 `workflows` 目录

---

## Task 2: 创建索引文件

**Files:**
- Create: `docs/workflows/README.md`

**Step 1: 写入索引文件内容**

创建文件 `docs/workflows/README.md`，包含：
- 工作流概述
- 目录列表（4 个工作流）
- 学习路径建议
- 快速导航表格

**Step 2: 验证文件创建**

```bash
wc -l docs/workflows/README.md
```
Expected: ~50 行

---

## Task 3: 创建 EPDV 工作流文档

**Files:**
- Create: `docs/workflows/01-epdv-workflow.md`

**Step 1: 写入 EPDV 工作流文档**

创建文件，包含：
- 概述：EPDV 是 Claude Code 推荐的通用开发流程
- 适用场景：新功能开发、大型重构
- 前置条件：无需额外插件
- 完整步骤：
  - E (Explore): 使用 Explore 子代理或 Grep/Glob 工具
  - P (Plan): 使用 EnterPlanMode 或 Plan 子代理
  - D (Develop): 编码实现
  - V (Verify): 测试、代码审查
- 真实案例：实现用户认证功能
- 常见问题
- 技巧与最佳实践

**Step 2: 验证文件创建**

```bash
wc -l docs/workflows/01-epdv-workflow.md
```
Expected: ~300 行

---

## Task 4: 创建 TDD 工作流文档

**Files:**
- Create: `docs/workflows/02-tdd-workflow.md`

**Step 1: 写入 TDD 工作流文档**

创建文件，包含：
- 概述：测试驱动开发的 Red-Green-Refactor 循环
- 适用场景：功能开发、Bug 修复
- 前置条件：测试框架已配置
- 完整步骤：
  - Red: 写失败的测试
  - Green: 最小代码通过测试
  - Refactor: 重构优化
- 与 Claude Code 集成：
  - superpowers:test-driven-development 技能
  - 测试命令自动化
- 真实案例：开发表单验证工具
- 常见问题
- 技巧与最佳实践

**Step 2: 验证文件创建**

```bash
wc -l docs/workflows/02-tdd-workflow.md
```
Expected: ~250 行

---

## Task 5: 创建调试工作流文档

**Files:**
- Create: `docs/workflows/03-debugging-workflow.md`

**Step 1: 写入调试工作流文档**

创建文件，包含：
- 概述：系统化调试方法论
- 适用场景：Bug 修复、性能问题排查
- 前置条件：日志系统、错误追踪工具
- 完整步骤：
  1. 定义问题：明确错误现象
  2. 收集信息：日志、堆栈、环境
  3. 生成假设：可能的原因列表
  4. 验证假设：逐一排查
  5. 修复并验证：应用修复，确认解决
- 与 Claude Code 集成：
  - superpowers:systematic-debugging 技能
  - Sentry MCP 集成
- 真实案例：调试 API 超时问题
- 常见问题
- 技巧与最佳实践

**Step 2: 验证文件创建**

```bash
wc -l docs/workflows/03-debugging-workflow.md
```
Expected: ~250 行

---

## Task 6: 创建代码审查流程文档

**Files:**
- Create: `docs/workflows/04-code-review-workflow.md`

**Step 1: 写入代码审查文档**

创建文件，包含：
- 概述：高效代码审查流程
- 适用场景：PR 审查、代码质量检查
- 前置条件：Git 仓库、PR 系统
- 完整步骤：
  1. 审查前准备：变更范围、相关上下文
  2. 审查维度：功能、安全、性能、可维护性
  3. 反馈格式：清晰、可操作的建议
  4. 审查后跟进：确认修复
- 与 Claude Code 集成：
  - pr-review-toolkit 插件
  - superpowers:requesting-code-review 技能
  - superpowers:receiving-code-review 技能
- 真实案例：审查一个 Pull Request
- 常见问题
- 技巧与最佳实践

**Step 2: 验证文件创建**

```bash
wc -l docs/workflows/04-code-review-workflow.md
```
Expected: ~200 行

---

## Task 7: 更新根目录 README.md

**Files:**
- Modify: `README.md`

**Step 1: 读取现有 README.md**

```bash
cat README.md
```

**Step 2: 添加高级工作流指南章节**

在适当位置添加：

```markdown
### 📓 [高级工作流指南](./docs/workflows/README.md)

深入讲解 Claude Code 的四大核心工作流模式：

- **EPDV 工作流** - 探索-规划-开发-验证全流程
- **TDD 工作流** - 测试驱动开发完整实践
- **系统化调试** - 结构化的问题解决流程
- **代码审查流程** - 高效的代码质量检查方法

**适合人群：** 希望掌握高级开发工作流的开发者
```

**Step 3: 验证更新**

```bash
grep -A 10 "高级工作流指南" README.md
```
Expected: 看到新增的章节内容

---

## Task 8: 最终验证

**Step 1: 验证所有文件存在**

```bash
ls -la docs/workflows/
```
Expected: 5 个文件（README.md + 4 个工作流文档）

**Step 2: 验证文档结构**

```bash
wc -l docs/workflows/*.md
```
Expected: 每个文件的行数符合预期

**Step 3: 验证链接有效性**

检查 README.md 中的链接指向正确的文件

---

## 执行顺序

```
Task 1 (创建目录) → Task 2 (索引) → Task 3 (EPDV) → Task 4 (TDD) → Task 5 (调试) → Task 6 (代码审查) → Task 7 (更新 README) → Task 8 (验证)
```

---

## 文件清单

| 文件 | 操作 | 预估行数 |
|------|------|----------|
| `docs/workflows/` | 创建目录 | - |
| `docs/workflows/README.md` | 创建 | ~50 |
| `docs/workflows/01-epdv-workflow.md` | 创建 | ~300 |
| `docs/workflows/02-tdd-workflow.md` | 创建 | ~250 |
| `docs/workflows/03-debugging-workflow.md` | 创建 | ~250 |
| `docs/workflows/04-code-review-workflow.md` | 创建 | ~200 |
| `README.md` | 修改 | +20 |

---

**计划版本：** 1.0
**创建日期：** 2026-02-28
