# 代码审查工作流指南

## 概述

代码审查通过 **系统化检查变更 → 发现潜在问题 → 确保代码质量** 的流程，在代码合并前拦截缺陷、提升可维护性并促进团队知识共享。

## 适用场景

- **Pull Request 审查**：合并前的质量把关
- **代码质量检查**：定期审查关键模块
- **安全审计**：敏感代码的安全审查
- **新人代码指导**：帮助新成员融入团队规范

## 前置条件

- Git 仓库（本地或远程）
- PR/MR 系统（GitHub、GitLab、Azure DevOps、Bitbucket）
- 代码审查清单或规范（可选但推荐）

---

## 完整工作流步骤

### 步骤一：审查前准备

**目标**：理解变更范围和上下文，建立审查基准。

#### 关键问题

- 这个 PR 解决什么问题？
- 涉及哪些模块和文件？
- 是否有相关的设计文档或 Issue？

#### 准备清单

```markdown
## 审查准备

**PR 标题**：feat: 添加用户登录功能
**关联 Issue**：#123
**变更文件**：12 个文件，+450/-120 行
**主要模块**：auth、user、middleware
**风险等级**：中（涉及认证逻辑）
```

#### 获取上下文

```bash
# 查看变更概览
git diff main...feature/login --stat

# 查看完整变更
git diff main...feature/login

# 查看特定文件变更
git diff main...feature/login -- src/auth/
```

---

### 步骤二：审查维度

**目标**：从多个角度系统化检查代码质量。

#### 审查检查清单

| 维度 | 检查项 | 严重程度 |
|------|--------|----------|
| **功能正确性** | 代码是否实现预期功能？是否处理边界情况？ | 高 |
| **安全性** | 是否有 SQL 注入、XSS、敏感信息泄露风险？ | 高 |
| **性能** | 是否有 N+1 查询、内存泄漏、不必要的循环？ | 中 |
| **可维护性** | 命名是否清晰？逻辑是否简洁？是否有重复代码？ | 中 |
| **测试覆盖** | 是否有足够的单元测试？测试用例是否有效？ | 中 |
| **文档** | 复杂逻辑是否有注释？API 是否有文档？ | 低 |

#### 逐项检查示例

```javascript
// 审查：功能正确性
// 问题：未处理用户不存在的情况
async function getUser(id) {
  const user = await db.findUser(id);
  return user.name; // 如果 user 为 null 会崩溃
}

// 建议修复
async function getUser(id) {
  const user = await db.findUser(id);
  if (!user) {
    throw new NotFoundError(`User ${id} not found`);
  }
  return user.name;
}
```

---

### 步骤三：反馈格式

**目标**：提供清晰、可操作、建设性的审查意见。

#### 审查评论分类

| 类型 | 前缀 | 含义 | 示例 |
|------|------|------|------|
| **必须修改** | `[MUST]` | 阻塞合并的问题 | 安全漏洞、功能缺陷 |
| **建议修改** | `[SUGGEST]` | 改进建议但不阻塞 | 代码风格、性能优化 |
| **讨论** | `[DISCUSS]` | 需要进一步讨论 | 设计决策、架构问题 |
| **称赞** | `[NIT]` 或无前缀 | 肯定好的代码 | "这段写得很清晰" |

#### 审查评论模板

```markdown
## 审查评论示例

### [MUST] SQL 注入风险

**位置**：`src/services/user.js:45`

**问题**：直接拼接用户输入到 SQL 查询中，存在 SQL 注入风险。

```javascript
// 当前代码
const query = `SELECT * FROM users WHERE id = ${userId}`;
```

**建议**：使用参数化查询：

```javascript
// 建议修改
const query = 'SELECT * FROM users WHERE id = ?';
db.execute(query, [userId]);
```

---

### [SUGGEST] 可简化逻辑

**位置**：`src/utils/validator.js:23-30`

**问题**：这段条件判断可以简化，提高可读性。

```javascript
// 当前代码
if (value !== null && value !== undefined && value !== '') {
  return true;
} else {
  return false;
}

// 建议修改
return Boolean(value);
```

---

### [DISCUSS] 架构设计

**位置**：整体设计

**问题**：这个 PR 将认证逻辑放在了中间件层，但根据我们的架构规范，
认证应该在专门的 Service 层处理。想确认一下这个设计决策的考虑？
```

---

### 步骤四：审查后跟进

**目标**：确保问题得到解决，完成审查闭环。

#### 跟进清单

```markdown
## 审查跟进

- [ ] 所有 [MUST] 问题已修复
- [ ] [SUGGEST] 问题已处理或有合理说明
- [ ] [DISCUSS] 问题已达成共识
- [ ] CI/CD 检查全部通过
- [ ] 至少一位审查者批准
- [ ] 代码已重新自我审查
```

#### 状态更新示例

```markdown
## 更新日志

**2024-03-15 14:30** - 提交审查意见（3 个 MUST，2 个 SUGGEST）
**2024-03-15 16:00** - 作者修复所有 MUST 问题
**2024-03-15 16:30** - 确认修复，批准合并
```

---

## 与 Claude Code 集成

### pr-review-toolkit 插件

```bash
# 获取 PR 信息
> 获取 PR #123 的详细信息

# 触发代码审查
> 触发对 PR #123 的代码审查

# 查看审查状态
> 查看代码审查列表
```

### superpowers:requesting-code-review 技能

```bash
# 请求代码审查
> /skill superpowers:requesting-code-review

# Claude 会帮助你：
# - 整理变更摘要
# - 识别需要重点审查的代码
# - 生成审查请求描述
```

### superpowers:receiving-code-review 技能

```bash
# 处理审查反馈
> /skill superpowers:receiving-code-review

# Claude 会帮助你：
# - 分析审查意见
# - 生成修复方案
# - 解释设计决策
```

### code-reviewer 代理

```bash
# 启动代码审查代理
> 请作为 code-reviewer 审查当前的变更

# Claude 会自动：
# - 分析代码变更
# - 检查常见问题模式
# - 提供结构化的审查意见
```

---

## 真实案例演示：审查一个 Pull Request

### PR 背景

```
PR #456: feat: 添加订单导出功能
作者: zhangsan
变更: +320/-45 行，8 个文件
风险: 中（涉及数据库查询和数据导出）
```

### Step 1：审查前准备

```bash
# 查看变更概览
gh pr view 456 --json title,body,files

# 本地检出分支测试
gh pr checkout 456
```

### Step 2：逐文件审查

**文件 1：`src/services/export.js`**

```javascript
// 审查发现的问题
async function exportOrders(userId, format) {
  const orders = await Order.findAll(); // [MUST] 缺少分页，可能 OOM

  if (format === 'csv') {
    let csv = 'id,amount,status\n';
    for (const order of orders) {
      csv += `${order.id},${order.amount},${order.status}\n`; // [SUGGEST] 使用 CSV 库
    }
    return csv;
  }

  return JSON.stringify(orders); // [MUST] 未处理敏感字段
}
```

### Step 3：提交审查意见

```markdown
## 代码审查：PR #456

### [MUST] 内存溢出风险

**位置**：`src/services/export.js:5`

**问题**：`Order.findAll()` 一次性加载所有订单，数据量大时会导致内存溢出。

**建议**：使用流式处理或分页查询：

```javascript
async function* exportOrdersStream(userId) {
  const pageSize = 1000;
  let page = 0;

  while (true) {
    const orders = await Order.findAll({
      where: { userId },
      limit: pageSize,
      offset: page * pageSize
    });

    if (orders.length === 0) break;

    for (const order of orders) {
      yield order;
    }
    page++;
  }
}
```

---

### [MUST] 敏感数据泄露

**位置**：`src/services/export.js:15`

**问题**：直接返回订单 JSON 可能包含用户敏感信息（如手机号、地址）。

**建议**：过滤敏感字段：

```javascript
const sanitizeOrder = (order) => ({
  id: order.id,
  amount: order.amount,
  status: order.status,
  createdAt: order.createdAt
});

return JSON.stringify(orders.map(sanitizeOrder));
```

---

### [SUGGEST] 使用专业 CSV 库

**位置**：`src/services/export.js:9-12`

**问题**：手动拼接 CSV 存在注入风险（字段中包含逗号或换行）。

**建议**：使用 `csv-writer` 或类似库：

```javascript
import createCsvWriter from 'csv-writer';

const csvWriter = createCsvWriter({
  header: [
    {id: 'id', title: 'ID'},
    {id: 'amount', title: 'Amount'},
    {id: 'status', title: 'Status'}
  ]
});
```

---

### [NIT] 良好的错误处理

**位置**：`src/services/export.js:20-25`

这段错误处理写得很规范，日志记录详细，值得学习！

---

## 审查总结

| 类型 | 数量 | 状态 |
|------|------|------|
| MUST | 2 | 待修复 |
| SUGGEST | 1 | 可选 |
| NIT | 1 | - |

**整体评价**：功能实现思路正确，但需要解决内存和数据安全问题后才能合并。
```

### Step 4：跟进修复

```markdown
## 修复确认

- [x] 添加分页查询（commit: abc1234）
- [x] 过滤敏感字段（commit: def5678）
- [ ] CSV 库替换（作者说明：下个迭代处理）

审查通过，可以合并。
```

---

## 常见问题

### Q1: 审查时发现代码太复杂看不懂怎么办？

**A:** 采用以下策略：
1. 先阅读 PR 描述和关联 Issue 理解背景
2. 让 Claude 帮助解释复杂逻辑
3. 在评论中标记 `[DISCUSS]` 请作者解释
4. 建议添加注释或拆分函数提高可读性

### Q2: 作者对审查意见有异议怎么处理？

**A:** 建议的处理流程：
1. 理解作者的观点和上下文
2. 解释提出问题的原因（安全/性能/可维护性）
3. 寻找双方都能接受的替代方案
4. 必要时邀请第三方审查者参与讨论
5. 对于非阻塞性问题，可以标记为 "下个迭代处理"

### Q3: 如何避免审查遗漏重要问题？

**A:** 建立系统化审查流程：
1. 使用审查清单逐项检查
2. 优先审查高风险模块（认证、支付、数据操作）
3. 运行自动化工具（ESLint、安全扫描）辅助
4. 让 Claude 帮助检查常见问题模式

### Q4: 审查速度慢影响开发效率怎么办？

**A:** 优化审查效率的方法：
1. 小批量审查：鼓励小 PR（< 400 行）
2. 使用 Claude 快速生成初步审查意见
3. 设置审查 SLA（如 24 小时内响应）
4. 培训团队成员提高审查能力

---

## 技巧与最佳实践

### 1. 使用代码审查模板

```markdown
## 审查模板

### 功能正确性
- [ ] 代码实现符合需求描述
- [ ] 边界情况处理完整
- [ ] 错误处理得当

### 代码质量
- [ ] 命名清晰、有意义
- [ ] 无重复代码
- [ ] 函数职责单一

### 安全性
- [ ] 无 SQL 注入风险
- [ ] 无 XSS 漏洞
- [ ] 敏感数据已脱敏

### 测试
- [ ] 有足够的测试覆盖
- [ ] 测试用例有效
```

### 2. 分层审查法

```bash
# 第一层：快速浏览变更范围
git diff main...feature --stat

# 第二层：审查关键文件
git diff main...feature -- src/auth/ src/payment/

# 第三层：深入审查复杂逻辑
git diff main...feature -- src/services/order.js
```

### 3. 让 Claude 辅助审查

```bash
# 自动检查代码问题
> 审查 src/services/ 目录下的代码变更，检查以下问题：
> - 安全漏洞
> - 性能问题
> - 代码规范

# 生成审查意见
> 根据这个 diff 生成代码审查意见，使用 [MUST]/[SUGGEST] 格式
```

### 4. 审查评论三明治法

```markdown
## 三明治评论结构

1. **肯定**：先指出做得好的地方
   > 这里的错误处理写得很规范

2. **建议**：提出改进意见
   > [SUGGEST] 可以考虑将这段逻辑抽取为独立函数

3. **鼓励**：以积极的语气结束
   > 整体实现思路很清晰，修复后就可以合并了
```

### 5. 善用 GitHub/GitLab 功能

```bash
# GitHub: 提交带建议的审查
gh api repos/:owner/:repo/pulls/:number/reviews \
  -f event='COMMENT' \
  -f body='[MUST] 安全问题需要修复' \
  -F comments='[{"path":"src/auth.js","line":45,"body":"建议使用参数化查询"}]'

# GitLab: 使用讨论线程
# 将评论标记为 "需要解决" 确保跟进
```

---

## 总结

代码审查四步法：

| 步骤 | 目标 | 关键动作 |
|------|------|----------|
| 审查前准备 | 理解上下文 | 阅读 PR 描述、查看变更范围 |
| 审查维度 | 系统检查 | 功能、安全、性能、可维护性 |
| 反馈格式 | 清晰沟通 | [MUST]/[SUGGEST]/[DISCUSS] 分类 |
| 审查后跟进 | 确保闭环 | 确认修复、批准合并 |

开始使用：`/skill superpowers:requesting-code-review` 或 `/skill superpowers:receiving-code-review`
