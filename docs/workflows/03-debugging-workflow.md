# 系统化调试工作流指南

## 概述

系统化调试通过 **定义问题 → 收集信息 → 生成假设 → 验证假设 → 修复验证** 五步法，将混乱的 Bug 排查过程转化为有序、可复现的科学方法论。

## 适用场景

- **Bug 修复**：生产环境或测试环境发现的功能缺陷
- **性能问题排查**：响应慢、内存泄漏、CPU 飙高
- **生产环境问题**：服务不可用、数据不一致、异常崩溃

## 前置条件

- 日志系统已配置（如 Winston、Pino、Log4j）
- 错误追踪工具可选（Sentry、Bugsnag、DataDog）
- 基本的监控告警（可选但推荐）

---

## 完整工作流步骤

### 步骤一：定义问题

**目标**：清晰描述问题现象，建立可复现的基准。

#### 关键问题

- 错误现象是什么？
- 何时开始发生？
- 影响范围有多大？
- 能否稳定复现？

#### 问题记录模板

```markdown
## Bug 报告

**现象描述**：用户登录后页面白屏
**发生时间**：2024-03-15 14:30 开始
**影响范围**：约 30% 的移动端用户
**复现步骤**：
1. 打开移动端浏览器
2. 输入账号密码登录
3. 等待跳转 -> 白屏

**环境信息**：
- iOS Safari / Android Chrome
- 生产环境 v2.3.1
```

---

### 步骤二：收集信息

**目标**：获取足够的诊断数据，不凭猜测下结论。

#### 日志收集

```bash
# 按时间范围过滤日志
grep "2024-03-15 14:3" /var/log/app.log | grep -i error

# 查看特定请求的完整日志链路
grep "request-id-12345" /var/log/app.log
```

#### 堆栈跟踪分析

```javascript
// 典型的错误堆栈
TypeError: Cannot read property 'id' of undefined
    at UserService.getProfile (src/services/user.js:45:22)
    at loginController (src/controllers/auth.js:28:15)
    at Layer.handle [as handle_request] (node_modules/express/lib/router/layer.js:95:5)
```

#### 环境信息收集

```bash
# 系统资源状态
top -bn1 | head -20
free -h
df -h

# 进程状态
ps aux | grep node

# 网络连接
netstat -tlnp
```

---

### 步骤三：生成假设

**目标**：基于收集的信息，列出所有可能的原因。

#### 假设列表示例

```markdown
## 假设列表

| # | 假设 | 可能性 | 验证方法 |
|---|------|--------|----------|
| 1 | 用户数据中缺少 profile 字段 | 高 | 检查数据库记录 |
| 2 | 移动端 JS 兼容性问题 | 中 | 检查浏览器日志 |
| 3 | API 响应格式变更 | 中 | 对比 API 返回 |
| 4 | CDN 缓存了旧版本代码 | 低 | 检查版本号 |
```

#### 优先级排序

按以下因素排序假设：
1. **可能性**：基于证据的置信度
2. **影响面**：能解释多少观察到的现象
3. **验证成本**：验证所需的时间精力

---

### 步骤四：验证假设

**目标**：逐一排查假设，使用二分法快速定位。

#### 二分法定位

```bash
# 在代码中插入日志定位问题位置
console.log('[DEBUG] Step 1: entering function');
// ... 代码
console.log('[DEBUG] Step 2: data fetched', data);
// ... 代码
console.log('[DEBUG] Step 3: processing complete');
```

#### 假设验证示例

```javascript
// 假设 1：用户数据缺少 profile 字段
// 验证代码
const user = await db.user.findUnique({ where: { id: userId } });
console.log('User data:', JSON.stringify(user, null, 2));
// 结果：user.profile 为 null -> 假设成立
```

```bash
# 假设 2：移动端兼容性问题
# 验证方法：检查 Safari 控制台
# 结果：发现 SyntaxError "Unexpected token ..." -> 需要进一步调查
```

---

### 步骤五：修复并验证

**目标**：应用修复方案，确保问题彻底解决。

#### 修复实施

```javascript
// 修复前
function getProfile(user) {
  return user.profile.id; // 崩溃点
}

// 修复后
function getProfile(user) {
  if (!user.profile) {
    console.warn(`User ${user.id} missing profile`);
    return null;
  }
  return user.profile.id;
}
```

#### 验证清单

- [ ] 本地环境复现问题成功
- [ ] 应用修复后本地测试通过
- [ ] 单元测试/集成测试通过
- [ ] 预发布环境验证通过
- [ ] 生产环境部署后监控正常
- [ ] 添加回归测试防止复发

---

## 与 Claude Code 集成

### 系统化调试技能

```bash
# 启动调试模式
> /skill superpowers:systematic-debugging
```

### Sentry MCP 集成

```bash
# 查看 Sentry 中的错误详情
> 使用 Sentry MCP 获取错误 event-12345 的详细信息

# 分析错误趋势
> 查看过去 24 小时 TypeError 的发生频率
```

### 日志分析技巧

```bash
# 让 Claude 帮助分析日志
> 分析这段日志，找出可能的错误原因：
```
[2024-03-15 14:32:15] ERROR: Connection timeout to database
[2024-03-15 14:32:16] WARN: Retrying connection... attempt 1
[2024-03-15 14:32:18] ERROR: Connection timeout to database
[2024-03-15 14:32:19] WARN: Retrying connection... attempt 2
[2024-03-15 14:32:21] FATAL: Max retries exceeded
```
```

### Claude 辅助调试流程

```bash
# 启动系统化调试
> 我遇到了一个 Bug：用户登录后偶尔会白屏。帮我系统化地排查。

# Claude 会引导你完成五步调试法
```

---

## 真实案例演示：调试 API 超时问题

### 问题背景

```
现象：/api/orders 接口偶尔超时（30s）
频率：每天约 50 次
影响：用户无法查看订单列表
```

### Step 1：定义问题

```markdown
**错误现象**：API 返回 504 Gateway Timeout
**发生频率**：峰值时段（10:00-12:00）更频繁
**复现条件**：无法稳定复现，随机发生
**错误日志**：
[ERROR] Request timeout after 30000ms - GET /api/orders
```

### Step 2：收集信息

```bash
# 查看数据库慢查询日志
grep "slow_query" /var/log/postgresql.log | grep "orders"

# 输出
2024-03-15 10:23:45 [SLOW] duration: 28534ms query: SELECT * FROM orders WHERE user_id = ? AND status = ?
```

```javascript
// 检查 API 代码
router.get('/api/orders', async (req, res) => {
  const orders = await Order.findAll({ where: { userId: req.user.id } });
  // 问题：N+1 查询
  for (const order of orders) {
    order.items = await OrderItem.findAll({ where: { orderId: order.id } });
  }
  res.json(orders);
});
```

### Step 3：生成假设

| # | 假设 | 可能性 | 验证方法 |
|---|------|--------|----------|
| 1 | N+1 查询导致慢查询 | 高 | 分析 SQL 执行计划 |
| 2 | 数据库连接池耗尽 | 中 | 监控连接池状态 |
| 3 | 缺少索引 | 中 | 检查表索引 |

### Step 4：验证假设

```bash
# 验证假设 1：N+1 查询
# 分析：用户有 100+ 订单时，会执行 101 次 SQL 查询
# 结论：假设成立

# 验证假设 3：缺少索引
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123 AND status = 'pending';
# 结果：Seq Scan（全表扫描）-> 缺少索引
```

### Step 5：修复并验证

```javascript
// 修复方案：使用 JOIN 一次性获取数据
router.get('/api/orders', async (req, res) => {
  const orders = await Order.findAll({
    where: { userId: req.user.id },
    include: [{ model: OrderItem, as: 'items' }]
  });
  res.json(orders);
});
```

```sql
-- 添加索引
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

```bash
# 验证结果
curl -w "Time: %{time_total}s\n" http://api.example.com/api/orders
# Time: 0.234s（修复前：28s+）
```

---

## 常见问题

### Q1: 无法复现的 Bug 怎么调试？

**A:** 采用以下策略：
1. 增加详细日志，捕获下次发生时的上下文
2. 使用错误追踪工具（Sentry）收集完整堆栈
3. 分析错误发生的模式（时间、用户类型、操作）
4. 在测试环境模拟相似条件

### Q2: 生产环境问题如何安全调试？

**A:** 遵循以下原则：
1. 优先查看日志和监控，避免直接操作生产环境
2. 必要时在只读副本上执行查询
3. 使用 feature flag 控制调试代码的开关
4. 修复先在预发布环境验证

### Q3: 调试时陷入僵局怎么办？

**A:** 尝试以下方法：
1. 休息一下，换个角度思考
2. 向同事解释问题（小黄鸭调试法）
3. 回顾假设列表，是否有遗漏
4. 从"什么没发生"的角度分析
5. 请 Claude 帮助分析

### Q4: 如何防止 Bug 再次发生？

**A:** 每次修复后：
1. 添加回归测试覆盖该场景
2. 更新文档记录问题和解决方案
3. 考虑是否需要架构改进
4. 在团队中分享经验教训

---

## 技巧与最佳实践

### 1. 日志分级记录

```javascript
// 使用不同日志级别
logger.debug('详细调试信息', { userId, requestId });
logger.info('关键业务事件', { action: 'login', userId });
logger.warn('潜在问题警告', { latency: 5000 });
logger.error('错误发生', { error: err.message, stack: err.stack });
```

### 2. 请求链路追踪

```javascript
// 为每个请求生成唯一 ID
app.use((req, res, next) => {
  req.requestId = uuid();
  res.setHeader('X-Request-Id', req.requestId);
  next();
});

// 所有日志带上 requestId
logger.info('Processing request', { requestId: req.requestId });
```

### 3. 时间戳标记法

```javascript
// 在关键步骤打点计时
const start = Date.now();
await fetchData();
console.log(`[DEBUG] fetchData took ${Date.now() - start}ms`);

await processData();
console.log(`[DEBUG] processData took ${Date.now() - start}ms`);
```

### 4. 二分注释法

```javascript
// 快速定位问题代码
// 注释掉一半代码，看问题是否消失
// 如果消失，问题在注释部分；否则在未注释部分
function complexFunction() {
  /* 注释开始
  step1();
  step2();
  step3();
  注释结束 */
  step4();
  step5();
  step6();
}
```

### 5. 利用 Claude Code 分析

```bash
# 让 Claude 分析错误模式
> 分析这个目录下的错误日志，找出最常见的前 5 种错误

# 让 Claude 帮助定位代码
> 找到所有可能导致 TypeError: Cannot read property 'x' of undefined 的代码位置
```

---

## 总结

系统化调试五步法：

| 步骤 | 目标 | 关键动作 |
|------|------|----------|
| 定义问题 | 明确目标 | 描述现象、复现步骤 |
| 收集信息 | 获取证据 | 日志、堆栈、环境 |
| 生成假设 | 列出可能 | 优先级排序 |
| 验证假设 | 定位根因 | 二分法排查 |
| 修复验证 | 彻底解决 | 测试、监控、回归测试 |

开始使用：`/skill superpowers:systematic-debugging`
