# Agents 使用指南

## 三种使用方式

```
┌─────────────────────────────────────────────────────────────────┐
│                     使用 Agents 的三种方式                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐   │
│  │   方式 1      │    │   方式 2      │    │   方式 3      │   │
│  │  命令触发     │    │  自动委托     │    │  Hook 触发    │   │
│  │  (显式)       │    │  (隐式)       │    │  (自动)       │   │
│  └───────┬───────┘    └───────┬───────┘    └───────┬───────┘   │
│          │                    │                    │           │
│          ▼                    ▼                    ▼           │
│     /plan "..."         "帮我设计..."          保存文件后      │
│          │                    │                    │           │
│          └────────────────────┴────────────────────┘           │
│                               │                                │
│                               ▼                                │
│                    ┌─────────────────────┐                     │
│                    │   加载对应 Agent    │                     │
│                    │   执行专业任务      │                     │
│                    └─────────────────────┘                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 方式 1: 命令触发 (推荐)

通过斜杠命令显式调用 Agent，最直接可控。

### 核心命令速查

| 命令 | 调用的 Agent | 用途 |
|------|-------------|------|
| `/plan "描述"` | `planner` | 功能规划 |
| `/tdd "描述"` | `tdd-guide` | TDD 开发 |
| `/code-review` | `code-reviewer` | 代码审查 |
| `/build-fix` | `build-error-resolver` | 修复构建错误 |
| `/e2e "流程"` | `e2e-runner` | E2E 测试 |
| `/security-scan` | `security-reviewer` | 安全扫描 |
| `/refactor-clean` | `refactor-cleaner` | 代码清理 |
| `/update-docs` | `doc-updater` | 更新文档 |

### 详细使用示例

#### 1. `/plan` - 功能规划

```bash
# 基础用法
/plan "添加用户认证系统"

# 详细描述
/plan "添加 Stripe 支付订阅功能，包括：
- 三种套餐：免费/专业版($29)/企业版($99)
- Stripe Checkout 支付流程
- Webhook 处理订阅状态变更
- 基于套餐的功能限制"

# Claude 会加载 planner agent，输出：
# Implementation Plan: Stripe Subscription Billing
#
# ## Overview
# ## Requirements  
# ## Architecture Changes
# ## Implementation Steps (分 Phase)
# ## Testing Strategy
# ## Risks & Mitigations
```

**使用时机**: 开始新功能前、技术方案设计时

---

#### 2. `/tdd` - TDD 开发

```bash
# 新功能开发
/tdd "实现用户注册功能，包括邮箱验证和密码强度检查"

# Bug 修复 (先写测试复现)
/tdd "修复登录时的空指针异常"

# 重构 (保护现有行为)
/tdd "重构订单服务，提取支付方式策略"
```

**Claude 执行流程**:
```
1. 写测试 (RED) ← 你应该看到测试失败
2. 运行测试确认失败
3. 写最小实现 (GREEN)
4. 运行测试确认通过
5. 重构 (IMPROVE)
6. 检查覆盖率 80%+
```

**使用时机**: 任何代码变更，特别是复杂逻辑

---

#### 3. `/code-review` - 代码审查

```bash
# 审查最近的修改
code-review

# 审查特定文件
/code-review src/auth/login.ts

# 审查特定目录
/code-review src/components/
```

**Claude 会检查**:
- CRITICAL: 硬编码密钥、SQL 注入、XSS
- HIGH: 大函数(>50行)、深嵌套(>4层)、缺失错误处理
- MEDIUM: 性能问题、React 反模式
- LOW: 代码风格、TODO 无票据

**输出示例**:
```
[CRITICAL] Hardcoded API key in source
File: src/api/client.ts:42
Issue: API key "sk-abc..." exposed in source
Fix: Move to environment variable

## Review Summary
| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 1     | block  |
| HIGH     | 2     | warn   |

Verdict: BLOCK — CRITICAL issues must be fixed before merge.
```

**使用时机**: 代码提交前、PR 创建后、合并前

---

#### 4. `/build-fix` - 修复构建错误

```bash
# 当 TypeScript 报错时
npx tsc --noEmit
# 显示: error TS2345: Argument of type 'string' is not assignable...

/build-fix

# Claude 会:
# 1. 运行 tsc 收集所有错误
# 2. 分类错误类型
# 3. 逐个修复（最小化变更）
# 4. 验证构建通过
```

**使用时机**: CI 失败、类型检查报错、构建失败时

---

#### 5. `/security-scan` - 安全扫描

```bash
# 全面安全扫描
/security-scan

# 扫描特定文件
/security-scan src/auth/
```

**Claude 会检查 OWASP Top 10**:
1. 注入攻击 (SQL/NoSQL/命令)
2. 失效认证
3. 敏感数据泄露
4. XXE
5. 访问控制失效
6. 安全配置错误
7. XSS
8. 不安全的反序列化
9. 已知漏洞 (依赖)
10. 日志监控不足

**使用时机**: 新 API 开发、认证代码变更、生产发布前

---

#### 6. `/e2e` - E2E 测试

```bash
# 生成特定流程的测试
/e2e "用户注册并订阅付费套餐"

# 生成多个关键流程
/e2e "用户登录、创建项目、邀请成员"
```

**Claude 会**:
1. 识别关键用户旅程
2. 使用 Playwright 或 Agent Browser 创建测试
3. 应用 Page Object Model 模式
4. 添加适当的等待和断言

**使用时机**: 核心功能完成后、回归测试、CI/CD 集成

---

## 方式 2: 自动委托 (智能识别)

Claude 会自动识别任务类型，无需显式命令。

### 自动委托规则

```yaml
# AGENTS.md 中定义的规则

用户说: "帮我设计一个微服务架构"
  └── Claude 识别: "复杂架构请求"
  └── 自动委托给: architect

用户说: "这段代码有问题"
  └── Claude 识别: "代码审查需求"
  └── 自动委托给: code-reviewer

用户说: "帮我修复这个 bug"
  └── Claude 识别: "Bug 修复"
  └── 自动委托给: tdd-guide

用户写代码后保存
  └── Claude 识别: "代码变更完成"
  └── 自动委托给: code-reviewer
```

### 实际对话示例

#### 示例 1: 自动调用 planner

```
用户: 我想添加一个购物车功能，用户可以添加商品、修改数量、应用优惠券

Claude: [自动加载 planner agent]

我来帮你规划这个功能。让我分析一下需求并制定实施计划。

# Implementation Plan: Shopping Cart

## Overview
Add a shopping cart system with item management, quantity updates, 
and coupon code support.

## Requirements
- Add items to cart
- Update quantities
- Remove items
- Apply coupon codes
- Calculate totals with discounts

## Architecture Changes
- New table: `cart_items`
- New API routes: /api/cart/*
- New components: Cart, CartItem, CouponInput

## Implementation Steps

### Phase 1: Database & API
1. **Create cart_items table** (File: db/migrations/...)
   ...

### Phase 2: Frontend Components
...

## Testing Strategy
...
```

#### 示例 2: 自动调用 code-reviewer

```
用户: 我刚写完了登录功能，帮我看看

Claude: [自动加载 code-reviewer agent]

我来审查你的登录功能代码。

[运行 git diff 查看修改]
[读取相关文件]

## Code Review Results

### src/auth/login.ts

[HIGH] Missing input validation
Line 15: const { email, password } = req.body
Issue: No validation that email and password exist and are strings
Fix: Add Zod schema validation

[CRITICAL] Plaintext password comparison
Line 22: if (user.password === password)
Issue: Comparing plaintext passwords
Fix: Use bcrypt.compare()

...
```

---

## 方式 3: Hook 触发 (自动化)

特定事件发生时，自动触发 Agent。

### Hook 类型

```
SessionStart    ──► 会话开始时
PreToolUse      ──► 使用工具前
PostToolUse     ──► 使用工具后
Stop            ──► 会话结束时
```

### 实际应用场景

#### 场景 1: 代码保存后自动审查

```json
// hooks/hooks.json
{
  "hooks": [
    {
      "type": "PostToolUse",
      "tool": "WriteFile",
      "condition": "file.match(/\.(ts|tsx|js|jsx)$/)",
      "action": "suggest_code_review"
    }
  ]
}
```

**效果**: 用户保存代码文件后 → Claude 提示 "代码已修改，建议运行 /code-review"

---

#### 场景 2: 会话开始时加载上下文

```json
{
  "type": "SessionStart",
  "action": "load_context",
  "files": ["CLAUDE.md", "AGENTS.md", "docs/CODEMAPS/*.md"]
}
```

**效果**: 每次打开 Claude Code → 自动读取项目配置和代码地图

---

#### 场景 3: 会话结束时保存状态

```json
{
  "type": "Stop",
  "action": "save_session",
  "data": ["todo_list", "learned_patterns", "context"]
}
```

**效果**: 关闭会话时 → 自动保存当前任务状态和学习到的模式

---

## 高级使用技巧

### 技巧 1: Agent 组合

**完整开发流程**:
```bash
# 1. 规划
/plan "添加支付功能"

# 2. 开发 (TDD)
/tdd "实现支付服务"

# 3. 审查
/code-review

# 4. 安全扫描
/security-scan

# 5. E2E 测试
/e2e "支付流程"

# 6. 更新文档
/update-docs
```

---

### 技巧 2: 指定语言 Agent

```bash
# TypeScript 项目
/typescript-review src/components/

# Python 项目  
/python-review src/services/

# Go 项目
/go-review cmd/server/

# Java 项目
/java-review src/main/java/
```

---

### 技巧 3: 多 Agent 并行

```bash
# 同时审查多个方面
/code-review & /security-scan

# 同时检查多种语言
/typescript-review & /python-review
```

---

### 技巧 4: 自定义工作流

创建自定义命令组合：

```markdown
# commands/my-deploy.md

---
name: my-deploy
description: Pre-deployment checklist
---

## Execution
1. Run `/code-review`
2. Run `/security-scan`
3. Run `/e2e "critical flows"`
4. Run `/update-docs`
5. Confirm all checks pass before deploying
```

使用:
```bash
/my-deploy
```

---

## 常见使用场景

### 场景 A: 新功能开发

```
Step 1: /plan "功能描述"
        ↓
Step 2: /tdd "第一步"
        ↓
Step 3: [实现代码]
        ↓
Step 4: /code-review
        ↓
Step 5: /security-scan
        ↓
Step 6: /e2e "关键流程"
        ↓
Step 7: /update-docs
```

### 场景 B: Bug 修复

```
Step 1: /tdd "修复 XXX Bug"
        (先写复现测试)
        ↓
Step 2: [修复代码]
        ↓
Step 3: /code-review
        ↓
Step 4: /build-fix (如果需要)
```

### 场景 C: 代码重构

```
Step 1: /plan "重构方案"
        ↓
Step 2: /tdd "保持现有行为"
        ↓
Step 3: [重构代码]
        ↓
Step 4: /code-review
        ↓
Step 5: /refactor-clean
```

### 场景 D: 生产发布前

```
Step 1: /code-review
        ↓
Step 2: /security-scan
        ↓
Step 3: /e2e "所有关键流程"
        ↓
Step 4: /update-docs
        ↓
Step 5: 发布
```

---

## 使用建议

### ✅ 最佳实践

1. **始终用 /plan 开始新功能** - 避免返工
2. **用 /tdd 写任何代码** - 保证质量
3. **提交前 /code-review** - 防止问题进入仓库
4. **敏感代码 /security-scan** - 安全第一
5. **定期 /refactor-clean** - 保持代码健康

### ❌ 避免的错误

1. **不要跳过 /plan 直接写代码** - 容易走偏
2. **不要修复后再写测试** - 失去 TDD 意义
3. **不要忽视 CRITICAL 审查问题** - 技术债务
4. **不要在功能开发中运行 /refactor-clean** - 容易出错
5. **不要过度依赖自动委托** - 显式命令更可控

---

## 故障排除

### 问题 1: Agent 没有被调用

```
症状: 我说了"/plan"但 Claude 只是普通回复

解决:
1. 确认 ECC 已安装: /plugin list
2. 确认 CLAUDE.md 存在
3. 手动触发: "请使用 planner agent 帮我规划..."
```

### 问题 2: Agent 行为不符合预期

```
症状: code-reviewer 没有检查到明显问题

解决:
1. 检查 agents/code-reviewer.md 是否存在
2. 明确指定范围: "/code-review src/auth/login.ts"
3. 提示检查项: "请重点检查安全问题"
```

### 问题 3: 命令不存在

```
症状: 说 "/xxx" 提示命令不存在

解决:
1. 查看可用命令: /plugin list everything-claude-code
2. 确认命令拼写正确
3. 使用手动方式: "请用 xxx agent 帮我..."
```

---

## 总结

| 使用方式 | 适用场景 | 优点 | 缺点 |
|----------|----------|------|------|
| **命令触发** | 明确知道需要什么 | 可控、精确 | 需要记住命令 |
| **自动委托** | 日常对话 | 自然、无缝 | 可能识别错误 |
| **Hook 触发** | 自动化流程 | 无需干预 | 配置复杂 |

**推荐组合**: 命令触发为主 + 自动委托为辅 + Hook 自动化为补充
