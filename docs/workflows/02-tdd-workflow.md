# TDD（测试驱动开发）工作流指南

## 概述

TDD（Test-Driven Development）是一种以测试为中心的开发方法，通过 **Red-Green-Refactor** 循环确保代码质量和设计合理性：先写失败的测试，再写最小代码使测试通过，最后重构优化。

## 适用场景

- **功能开发**：实现新的业务功能
- **Bug 修复**：先写测试复现 Bug，再修复
- **重构**：为现有代码添加测试保护后安全重构

## 前置条件

- 测试框架已配置（Jest、pytest、Go test 等）
- 项目已有测试运行脚本

---

## 完整工作流步骤

### Red 阶段：编写失败的测试

**目标**：定义期望行为，测试必须失败。

```javascript
// tests/user.register.test.js
describe('用户注册', () => {
  test('应该成功注册有效用户', async () => {
    const result = await registerUser({
      email: 'test@example.com',
      password: 'Secure123!'
    });
    expect(result.success).toBe(true);
  });

  test('应该拒绝无效邮箱', async () => {
    await expect(registerUser({
      email: 'invalid',
      password: 'Secure123!'
    })).rejects.toThrow('邮箱格式无效');
  });
});
```

```bash
npm test -- tests/user.register.test.js
# FAIL - registerUser is not defined
```

---

### Green 阶段：最小代码使测试通过

**目标**：用最少的代码让测试通过。

```javascript
// src/user.service.js
async function registerUser({ email, password }) {
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    throw new Error('邮箱格式无效');
  }
  return { success: true, user: { email } };
}
```

```bash
npm test
# PASS - 所有测试通过
```

---

### Refactor 阶段：重构优化代码

**目标**：在测试保护下优化代码结构。

```javascript
// src/user.service.js - 重构后
const validators = {
  email: (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v)
};

class UserService {
  async register(data) {
    if (!validators.email(data.email)) {
      throw new Error('邮箱格式无效');
    }
    return { success: true, user: { email: data.email } };
  }
}
```

---

### 循环迭代

```
┌─────────────────────────────────┐
│  ┌─────┐   ┌─────┐   ┌────────┐ │
│  │ Red │──▶│Green│──▶│Refactor│ │
│  └─────┘   └─────┘   └────────┘ │
│      ▲                       │  │
│      └───────────────────────┘  │
└─────────────────────────────────┘
```

---

## 与 Claude Code 集成

### 测试驱动开发技能

```bash
# 启动 TDD 模式
> /skill test-driven-development
```

### 自动化测试命令

```json
// .claude/settings.json
{
  "commands": {
    "test": "npm test",
    "test:watch": "npm test -- --watch"
  }
}
```

### Claude 辅助 TDD 流程

```bash
> 使用 TDD 方式开发表单验证功能

# Claude 自动执行：
# 1. 创建测试 (Red)
# 2. 确认失败
# 3. 实现代码 (Green)
# 4. 确认通过
# 5. 重构优化
```

---

## 真实案例：表单验证工具

### 第一轮：必填验证

**Red**
```javascript
test('应该拒绝空值', () => {
  const result = validate({ name: '' }, { name: { required: true } });
  expect(result.valid).toBe(false);
});
```

**Green**
```javascript
function validate(data, rules) {
  const errors = {};
  for (const [field, rule] of Object.entries(rules)) {
    if (rule.required && !data[field]) {
      errors[field] = '此字段为必填项';
    }
  }
  return { valid: !Object.keys(errors).length, errors };
}
```

### 第二轮：邮箱验证

**Red**
```javascript
test('应该拒绝无效邮箱', () => {
  const result = validate({ email: 'abc' }, { email: { email: true } });
  expect(result.valid).toBe(false);
});
```

**Green**
```javascript
// 添加邮箱验证器
validators.email = (value) => {
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
    return '请输入有效的邮箱地址';
  }
};
```

### 第三轮：最小长度验证

**Red**
```javascript
test('应该拒绝短密码', () => {
  const result = validate({ pwd: '123' }, { pwd: { minLength: 8 } });
  expect(result.valid).toBe(false);
});
```

**Green**
```javascript
validators.minLength = (value, min) => {
  if (value.length < min) return `长度不能少于 ${min} 个字符`;
};
```

### 最终代码

```javascript
// src/validator.js
const validators = {
  required: (v) => !v && '此字段为必填项',
  email: (v) => !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v) && '邮箱格式无效',
  minLength: (v, min) => v.length < min && `长度不能少于 ${min} 个字符`
};

function validate(data, rules) {
  const errors = {};
  for (const [field, fieldRules] of Object.entries(rules)) {
    for (const [rule, param] of Object.entries(fieldRules)) {
      const error = validators[rule]?.(data[field], param);
      if (error) { errors[field] = error; break; }
    }
  }
  return { valid: !Object.keys(errors).length, errors };
}

module.exports = { validate };
```

---

## 常见问题

### Q1: 什么时候使用 TDD？

**A:** 适合业务逻辑复杂、长期维护、团队协作的项目。简单脚本可不使用。

### Q2: 测试太详细会拖慢开发吗？

**A:** 初期稍慢，但减少 Debug 时间、支持重构，长期效率更高。

### Q3: 如何处理难测试的代码？

**A:** 使用依赖注入、Mock/Stub，将业务逻辑与 I/O 分离。

### Q4: 覆盖率要多少？

**A:** 核心逻辑 90%+，一般功能 70-80%。关键是覆盖关键路径。

---

## 技巧与最佳实践

### 1. 测试命名清晰

```javascript
// 好
test('应该在密码错误时抛出异常', () => {...});
// 差
test('test1', () => {...});
```

### 2. 每个测试单一职责

```javascript
// 好 - 一个测试一个行为
test('应该拒绝空邮箱', () => {...});
test('应该拒绝无效格式', () => {...});
```

### 3. AAA 模式组织

```javascript
test('计算订单总价', () => {
  // Arrange
  const order = { items: [{ price: 100, quantity: 2 }] };
  // Act
  const total = calculateTotal(order);
  // Assert
  expect(total).toBe(200);
});
```

### 4. 利用 Claude Code

```bash
> 使用 TDD 开发日期格式化函数
# Claude 自动遵循 Red-Green-Refactor
```

### 5. 保持测试快速

```bash
npm test -- --grep "注册"  # 只运行相关测试
npm test -- --watch        # 监听模式
```

---

## 总结

| 阶段 | 目标 | 关键点 |
|------|------|--------|
| Red | 定义期望 | 测试必须失败 |
| Green | 最小实现 | 只写必要代码 |
| Refactor | 优化结构 | 测试保护下重构 |

开始使用：`/skill test-driven-development`
