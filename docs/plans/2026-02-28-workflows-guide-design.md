# 高级工作流指南设计文档

> Claude Code 四大核心工作流文档的设计方案

**日期：** 2026-02-28
**状态：** 已批准
**类型：** 新增文档内容

---

## 1. 背景

### 1.1 项目现状

当前项目包含以下文档：
- `claude-code-manual.md` - 基础使用手册
- `claude-code-advanced-manual.md` - 进阶使用手册
- `claude-code-essentials.md` - 必装插件与技能指南
- `plugins-guide.md` - 常用插件指南 (2026)

### 1.2 需求

用户希望新增**高级工作流模式**专题指南，覆盖：
- TDD 工作流
- 调试工作流
- 代码审查流程
- EPDV 工作流

**文档深度：** 实用级（详细步骤 + 示例代码）
**文档结构：** 多文件模式（每个工作流一个独立文件，统一索引）

---

## 2. 文件结构设计

```
docs/
└── workflows/
    ├── README.md                # 工作流总览索引
    ├── 01-epdv-workflow.md      # EPDV：探索-规划-开发-验证
    ├── 02-tdd-workflow.md       # TDD：测试驱动开发
    ├── 03-debugging-workflow.md # 系统化调试
    └── 04-code-review-workflow.md # 代码审查流程
```

**编号说明：** 使用 `01-`、`02-` 等前缀表示推荐的学习/使用顺序

---

## 3. 文档模板设计

每个工作流文档遵循统一模板：

```markdown
# [工作流名称] 工作流

> 一句话描述这个工作流的核心价值

## 适用场景
- 场景 1
- 场景 2

## 前置条件
- 需要安装的插件/工具
- 需要的配置

## 完整工作流步骤

### 步骤 1: [动作]
[详细说明 + 示例命令/代码]

### 步骤 2: [动作]
...

## 真实案例演示
[一个完整的端到端案例]

## 常见问题
Q1: ...
A1: ...

## 技巧与最佳实践
- 技巧 1
- 技巧 2
```

---

## 4. 各工作流核心内容

### 4.1 EPDV 工作流 (01-epdv-workflow.md)

**核心流程：**
- **E (Explore):** 使用 Explore 子代理快速理解代码库
- **P (Plan):** 使用 Plan 子代理或 EnterPlanMode 规划实现
- **D (Develop):** 编码实现，遵循 TDD 或直接开发
- **V (Verify):** 运行测试、代码审查、验证功能

**案例：** 实现一个完整的用户认证功能

### 4.2 TDD 工作流 (02-tdd-workflow.md)

**核心流程：**
- **Red:** 先写失败的测试
- **Green:** 最小代码使测试通过
- **Refactor:** 重构优化代码

**与 Claude Code 集成：**
- Superpowers TDD 技能使用
- 测试运行自动化

**案例：** 使用 TDD 开发一个表单验证工具

### 4.3 系统化调试工作流 (03-debugging-workflow.md)

**核心流程：**
1. 定义问题：明确错误现象
2. 收集信息：日志、堆栈、环境
3. 生成假设：可能的原因
4. 验证假设：逐一排查
5. 修复并验证：应用修复，确认解决

**与 Claude Code 集成：**
- Superpowers systematic-debugging 技能
- Sentry MCP 集成

**案例：** 调试一个生产环境的 API 超时问题

### 4.4 代码审查流程 (04-code-review-workflow.md)

**核心流程：**
1. 审查前准备：变更范围、相关上下文
2. 审查维度：功能、安全、性能、可维护性
3. 反馈格式：清晰、可操作的建议
4. 审查后跟进：确认修复

**与 Claude Code 集成：**
- Code Review 插件使用
- Superpowers code-reviewer 代理

**案例：** 审查一个 Pull Request

---

## 5. 与现有文档集成

### 5.1 README.md 更新

在主 README.md 中添加新章节引用：

```markdown
### 📓 [高级工作流指南](./docs/workflows/README.md)

深入讲解 Claude Code 的四大核心工作流模式：

- **EPDV 工作流** - 探索-规划-开发-验证全流程
- **TDD 工作流** - 测试驱动开发完整实践
- **系统化调试** - 结构化的问题解决流程
- **代码审查流程** - 高效的代码质量检查方法

**适合人群：** 希望掌握高级开发工作流的开发者
```

### 5.2 docs/workflows/README.md 索引文件

包含：
- 工作流目录列表
- 学习路径建议
- 快速导航表格

---

## 6. 实现计划

### 文件创建顺序

1. `docs/workflows/README.md` - 索引文件
2. `docs/workflows/01-epdv-workflow.md`
3. `docs/workflows/02-tdd-workflow.md`
4. `docs/workflows/03-debugging-workflow.md`
5. `docs/workflows/04-code-review-workflow.md`
6. 更新根目录 `README.md`

### 预估工作量

| 文件 | 预估行数 | 复杂度 |
|------|----------|--------|
| README.md (索引) | ~50 行 | 低 |
| EPDV 工作流 | ~300 行 | 中 |
| TDD 工作流 | ~250 行 | 中 |
| 调试工作流 | ~250 行 | 中 |
| 代码审查 | ~200 行 | 低 |
| README.md 更新 | ~20 行 | 低 |

---

## 7. 批准记录

- **设计批准日期：** 2026-02-28
- **批准人：** 用户

---

**文档版本：** 1.0
**最后更新：** 2026-02-28
