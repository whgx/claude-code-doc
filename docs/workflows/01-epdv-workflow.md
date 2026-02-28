# EPDV 工作流指南

## 概述

EPDV 是 Claude Code 推荐的通用开发流程，通过 **Explore（探索）→ Plan（规划）→ Develop（开发）→ Verify（验证）** 四个阶段，帮助你系统化地完成任何软件开发任务，减少返工并提高代码质量。

## 适用场景

- **新功能开发**：从零开始实现一个完整的功能模块
- **大型重构**：需要对现有代码进行结构性调整
- **Bug 修复**：复杂问题的定位和修复
- **技术调研**：评估新技术或方案的可行性
- **代码迁移**：从一个技术栈迁移到另一个

## 前置条件

- Claude Code CLI 已安装
- 无需额外插件或配置
- 建议在 Git 仓库中使用

---

## 完整工作流步骤

### 阶段一：Explore（探索）

**目标**：深入理解代码库结构、现有实现和相关依赖。

#### 使用 Explore 子代理

```bash
# 启动 Explore 子代理进行深度探索
claude-code agent explore "理解用户认证相关的代码结构"
```

#### 使用搜索工具

```bash
# 使用 Grep 搜索相关代码
claude-code grep "authenticate" --type js

# 使用 Glob 查找相关文件
claude-code glob "**/*auth*.{js,ts}"

# 查看特定文件
claude-code read src/auth/login.ts
```

#### 探索清单

- [ ] 理解项目目录结构
- [ ] 找到相关的核心文件
- [ ] 了解现有的设计模式
- [ ] 识别潜在的依赖关系
- [ ] 查看相关的测试文件

---

### 阶段二：Plan（规划）

**目标**：制定详细的实现计划，明确步骤和边界。

#### 使用 EnterPlanMode

```bash
# 进入规划模式
claude-code plan
```

在规划模式下，Claude 会：
1. 分析当前任务需求
2. 评估现有代码结构
3. 生成详细的实现计划
4. 创建任务列表（可选）

#### 使用 Plan 子代理

```bash
# 启动 Plan 子代理
claude-code agent plan "设计用户认证系统的实现方案"
```

#### 规划输出示例

```markdown
## 实现计划

### 1. 数据模型设计
- 创建 User 模型
- 定义认证 Token 结构

### 2. API 接口设计
- POST /auth/login
- POST /auth/logout
- GET /auth/verify

### 3. 安全措施
- 密码加密（bcrypt）
- JWT Token 管理
- 请求限流

### 4. 测试策略
- 单元测试：认证逻辑
- 集成测试：API 端点
- E2E 测试：登录流程
```

---

### 阶段三：Develop（开发）

**目标**：按照规划实现功能代码。

#### 编码实现

```bash
# 创建新文件
claude-code write src/auth/service.ts

# 编辑现有文件
claude-code edit src/api/routes.ts
```

#### 结合 TDD 开发

```bash
# 1. 先写测试
claude-code write tests/auth.test.ts

# 2. 运行测试（预期失败）
npm test -- --grep "auth"

# 3. 实现功能代码
claude-code write src/auth/service.ts

# 4. 再次运行测试（预期通过）
npm test -- --grep "auth"
```

#### 开发最佳实践

- **小步提交**：每完成一个小功能就提交
- **保持专注**：按照规划逐步实现
- **编写注释**：为复杂逻辑添加说明
- **代码格式化**：使用项目配置的 linter

---

### 阶段四：Verify（验证）

**目标**：确保代码质量和功能正确性。

#### 运行测试

```bash
# 运行所有测试
npm test

# 运行特定测试文件
npm test -- tests/auth.test.ts

# 运行测试并生成覆盖率报告
npm test -- --coverage
```

#### 代码审查

```bash
# 使用 Greptile 进行代码审查
claude-code review-pr 123

# 或使用内置审查功能
claude-code agent review
```

#### 手动验证清单

- [ ] 所有测试通过
- [ ] 代码覆盖率达标（建议 >80%）
- [ ] Linter 检查无错误
- [ ] 功能手动测试通过
- [ ] 边界情况已处理
- [ ] 错误处理完善

---

## 真实案例演示

### 案例：实现用户认证功能

#### E - 探索阶段

```bash
# 提示 Claude
> 我需要实现一个用户认证功能。请帮我探索现有的代码结构。

# Claude 执行的探索
1. 搜索现有认证相关代码
2. 查看数据库模型定义
3. 了解 API 路由结构
4. 检查依赖包配置
```

**探索发现：**
- 项目使用 Express.js 框架
- 数据库使用 PostgreSQL + Prisma
- 已有基础的用户模型
- 缺少认证中间件

#### P - 规划阶段

```bash
# 提示 Claude 进入规划模式
> /plan 实现基于 JWT 的用户认证功能

# Claude 生成的计划
```

```markdown
## 用户认证实现计划

### 文件结构
src/
├── auth/
│   ├── controller.ts    # 认证控制器
│   ├── service.ts       # 业务逻辑
│   ├── middleware.ts    # 认证中间件
│   └── types.ts         # 类型定义
└── routes/
    └── auth.ts          # 路由定义

### 实现步骤
1. 安装依赖：jsonwebtoken, bcryptjs
2. 定义类型和接口
3. 实现 service 层
4. 实现 controller 层
5. 创建认证中间件
6. 配置路由
7. 编写测试
```

#### D - 开发阶段

```bash
# 按步骤实现
> 请按照计划第一步开始实现

# Claude 会逐步创建文件
```

**service.ts 示例：**

```typescript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';
import { prisma } from '../db';

export class AuthService {
  async login(email: string, password: string) {
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user) {
      throw new Error('用户不存在');
    }

    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      throw new Error('密码错误');
    }

    const token = jwt.sign(
      { userId: user.id },
      process.env.JWT_SECRET!,
      { expiresIn: '7d' }
    );

    return { token, user: { id: user.id, email: user.email } };
  }
}
```

#### V - 验证阶段

```bash
# 运行测试
> 运行认证相关的测试

# Claude 执行
npm test -- tests/auth/

# 输出
✓ AuthService.login - 成功登录
✓ AuthService.login - 用户不存在
✓ AuthService.login - 密码错误
✓ 认证中间件 - 有效 token
✓ 认证中间件 - 无效 token

Test Suites: 1 passed
Coverage: 92%
```

---

## 常见问题

### Q1: 什么时候应该使用 EPDV 工作流？

**A:** 当任务满足以下条件时推荐使用：
- 涉及多个文件的修改
- 需要理解现有代码结构
- 任务复杂度较高
- 需要保证代码质量

对于简单的单文件修改，可以直接开发。

### Q2: 探索阶段要花多少时间？

**A:** 建议探索阶段占整体时间的 20-30%。充分的探索可以：
- 避免后期返工
- 发现潜在问题
- 选择最优实现方案

### Q3: 规划模式必须使用吗？

**A:** 不是强制的，但强烈建议使用。规划模式可以：
- 帮助理清思路
- 生成可追踪的任务列表
- 便于团队协作和讨论

### Q4: 如果验证失败怎么办？

**A:** 遵循以下流程：
1. 分析失败原因
2. 回到开发阶段修复
3. 如果发现设计问题，返回规划阶段
4. 严重理解偏差时，重新探索

### Q5: 可以跳过某些阶段吗？

**A:** 可以根据实际情况调整：
- 熟悉的小改动：P → D → V
- 快速原型：E → D（省略验证）
- 紧急修复：D → V（直接修复验证）

---

## 技巧与最佳实践

### 1. 使用任务列表追踪进度

```bash
# 创建任务
> /todo 创建认证服务
> /todo 编写认证中间件
> /todo 添加测试用例

# 查看任务列表
> /todo list

# 完成任务
> /todo done 1
```

### 2. 利用 Git 检查点

每个阶段完成后创建提交：

```bash
# 探索完成后
git add . && git commit -m "docs: 记录认证模块探索结果"

# 规划完成后
git add . && git commit -m "docs: 添加认证实现计划"

# 开发完成后
git add . && git commit -m "feat: 实现用户认证功能"

# 验证通过后
git add . && git commit -m "test: 添加认证模块测试"
```

### 3. 善用上下文管理

```bash
# 查看当前上下文
> /context

# 清理不必要的上下文
> /compact

# 保存重要上下文
> /memory add "认证使用 JWT，密钥在 .env 中"
```

### 4. 组合使用子代理

```bash
# 探索 + 规划组合
> 使用 explore 子代理理解代码库，然后用 plan 子代理生成实现方案

# 开发 + 验证组合
> 实现功能后自动运行测试验证
```

### 5. 保持迭代思维

EPDV 不是线性流程，可以迭代循环：

```
E → P → D → V → (发现问题) → E → P → D → V
```

关键是在每个阶段保持专注，出现问题及时回溯。

---

## 总结

EPDV 工作流是一个经过验证的高效开发流程：

| 阶段 | 核心目标 | 关键工具 |
|------|----------|----------|
| Explore | 理解现状 | Grep, Glob, Read, Explore Agent |
| Plan | 制定方案 | Plan Mode, Plan Agent, Todo |
| Develop | 实现功能 | Edit, Write, Bash |
| Verify | 确保质量 | Test, Review, Lint |

掌握 EPDV 工作流，让你的开发过程更加系统化、可预测、高质量。
