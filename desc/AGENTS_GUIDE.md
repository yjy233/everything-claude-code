# ECC Agents 子包详解

## 概述

**Agents** 是 ECC (Everything Claude Code) 的核心组件，由 **28 个专业智能体**组成。每个智能体都是针对特定任务的专家，通过"委托模式"实现专业分工。

> **核心理念**: 不是让一个 AI 做所有事，而是将任务委托给领域专家，就像软件团队中的角色分工。

---

## Agent 分类与职责

### 🔧 一、开发流程类 Agent (4 个)

负责软件开发生命周期中的关键决策和质量把控。

#### 1. `planner` - 规划专家
| 属性 | 说明 |
|------|------|
| **模型** | opus (最强推理) |
| **触发条件** | 功能实现请求、架构变更、复杂重构 |
| **核心能力** | 需求分析、架构审查、步骤分解、风险评估 |

**工作流程:**
```
需求分析 → 架构审查 → 步骤分解 → 制定实施顺序 → 测试策略 → 风险评估
```

**输出格式:**
```markdown
# Implementation Plan: [功能名]

## Overview
## Requirements
## Architecture Changes
## Implementation Steps (分 Phase)
## Testing Strategy
## Risks & Mitigations
## Success Criteria
```

**使用场景:**
- 添加新功能前制定详细计划
- 大规模重构前的规划
- 复杂 Bug 修复前的分析

---

#### 2. `architect` - 架构师
| 属性 | 说明 |
|------|------|
| **模型** | opus |
| **触发条件** | 新系统设计、技术决策、可扩展性规划 |
| **核心能力** | 系统设计、技术权衡、模式推荐、可扩展性分析 |

**关注维度:**
- **功能性需求**: 用户故事、API 契约、数据模型
- **非功能性需求**: 性能目标、可扩展性、安全性、可用性
- **技术设计**: 架构图、组件职责、数据流
- **运维**: 部署策略、监控告警、灾备方案

**输出:** 架构决策记录 (ADR)

**使用场景:**
- 设计新系统或模块
- 评估技术选型
- 制定扩展路线图 (10K→100K→1M→10M 用户)

---

#### 3. `tdd-guide` - TDD 导师
| 属性 | 说明 |
|------|------|
| **模型** | sonnet |
| **触发条件** | 新功能开发、Bug 修复、重构 |
| **核心能力** | 红绿重构循环、测试覆盖保障 (80%+) |

**TDD 工作流程 (Red-Green-Refactor):**
```
1. 写测试 (RED) → 2. 运行测试 → 验证失败
3. 最小实现 (GREEN) → 4. 运行测试 → 验证通过
5. 重构 (IMPROVE) → 6. 验证覆盖 80%+
```

**必测边界情况:**
- Null/Undefined 输入
- 空数组/字符串
- 无效类型
- 边界值 (最小/最大)
- 错误路径 (网络失败、DB 错误)
- 竞态条件
- 大数据量性能
- 特殊字符 (Unicode、SQL 注入字符)

**使用场景:**
- 任何新功能开发
- Bug 修复 (先写复现测试)
- 重构 (确保测试保护)

---

#### 4. `code-reviewer` - 代码审查员
| 属性 | 说明 |
|------|------|
| **模型** | sonnet |
| **触发条件** | 代码编写/修改后 |
| **核心能力** | 质量检查、安全审查、最佳实践 |

**审查清单 (按严重程度):**

| 级别 | 检查项 |
|------|--------|
| **CRITICAL** | 硬编码密钥、SQL 注入、XSS、路径遍历、CSRF、认证绕过、依赖漏洞 |
| **HIGH** | 大函数 (>50行)、深嵌套 (>4层)、缺失错误处理、可变模式、未测试代码 |
| **MEDIUM** | 性能问题 (N+1 查询)、React 依赖数组缺失、内存泄漏 |
| **LOW** | TODO 无票据、命名不佳、魔法数字 |

**审查输出:**
```
[CRITICAL] 硬编码 API Key
File: src/api/client.ts:42
Issue: API key "sk-abc..." 暴露在源码中
Fix: 移至环境变量

## Review Summary
| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 0 | pass |
| HIGH | 2 | warn |
```

**使用场景:**
- 每次代码提交前
- PR 审查
- 合并前最终检查

---

### 🔒 二、安全类 Agent (1 个)

#### 5. `security-reviewer` - 安全审查员
| 属性 | 说明 |
|------|------|
| **模型** | sonnet |
| **触发条件** | 用户输入处理、认证代码、API 端点、敏感数据 |
| **核心能力** | OWASP Top 10、密钥检测、输入验证、依赖审计 |

**OWASP Top 10 检查:**
1. **注入攻击** - 查询是否参数化
2. **失效认证** - 密码哈希、JWT 验证
3. **敏感数据泄露** - HTTPS、加密、日志脱敏
4. **XML 外部实体** - XML 解析器配置
5. **访问控制失效** - 路由认证检查
6. **安全配置错误** - 默认凭证、调试模式
7. **XSS** - 输出转义、CSP
8. **不安全的反序列化** - 用户输入反序列化
9. **已知漏洞** - 依赖更新
10. **日志监控不足** - 安全事件记录

**危险代码模式:**
```javascript
// 立即标记为 CRITICAL
const query = `SELECT * FROM users WHERE id = ${userId}`;  // SQL 注入
child_process.exec(userInput);                              // 命令注入
innerHTML = userInput;                                      // XSS
fetch(userProvidedUrl);                                     // SSRF
```

**使用场景:**
- 新 API 端点开发
- 认证/授权代码变更
- 文件上传功能
- 支付代码
- 生产环境发布前

---

### 🔨 三、错误修复类 Agent (1 个)

#### 6. `build-error-resolver` - 构建错误修复专家
| 属性 | 说明 |
|------|------|
| **模型** | sonnet |
| **触发条件** | 构建失败、类型错误 |
| **核心能力** | TypeScript 错误修复、依赖解决、配置修复 |

**修复策略 (最小化变更):**
- ✅ 添加类型注解
- ✅ 添加 null 检查
- ✅ 修复导入/导出
- ❌ 不重构无关代码
- ❌ 不改变架构
- ❌ 不优化性能

**常见错误与修复:**

| 错误 | 修复 |
|------|------|
| `implicitly has 'any' type` | 添加类型注解 |
| `Object is possibly 'undefined'` | 可选链 `?.` 或 null 检查 |
| `Cannot find module` | 检查 tsconfig paths、安装包 |
| `Type 'X' not assignable to 'Y'` | 类型转换或修正 |
| `Hook called conditionally` | 将 hooks 移至顶层 |

**使用场景:**
- 构建失败时的紧急修复
- 类型错误清理
- CI/CD 管道修复

---

### 🌐 四、语言专用 Reviewer Agent (10 组)

为不同编程语言提供专业的代码审查。

| Agent | 语言/框架 | 核心关注点 |
|-------|-----------|------------|
| `typescript-reviewer` | TypeScript/JavaScript | 类型安全、异步正确性、React/Node 模式 |
| `python-reviewer` | Python | 类型注解、PEP8、Django/Flask 模式 |
| `go-reviewer` | Go | 惯用法、并发模式、错误处理 |
| `java-reviewer` | Java/Spring Boot | SOLID、Spring 模式、JPA |
| `kotlin-reviewer` | Kotlin/Android/KMP | 协程、空安全、Compose |
| `rust-reviewer` | Rust | 所有权、生命周期、错误处理 |
| `cpp-reviewer` | C++ | 内存安全、现代 C++、RAII |
| `flutter-reviewer` | Flutter/Dart | Widget 树、状态管理 |
| `database-reviewer` | PostgreSQL/Supabase | 查询优化、索引、RLS |
| `pytorch-build-resolver` | PyTorch/CUDA | 训练错误、CUDA 问题 |

#### 示例: `typescript-reviewer`

**审查优先级:**

**CRITICAL - 安全:**
- `eval` / `new Function` 注入
- XSS (innerHTML = userInput)
- SQL/NoSQL 注入
- 路径遍历
- 硬编码密钥

**HIGH - 类型安全:**
- 无理由的 `any`
- 非空断言滥用 `value!`
- 危险的 `as` 类型转换

**HIGH - 异步:**
- 未处理的 Promise
- 顺序 await 独立操作
- Floating promises
- `forEach` 中使用 `async`

**HIGH - 错误处理:**
- 空的 catch 块
- `JSON.parse` 无 try/catch
- 抛出非 Error 对象

---

### 🧪 五、测试类 Agent (1 个)

#### 17. `e2e-runner` - E2E 测试专家
| 属性 | 说明 |
|------|------|
| **模型** | sonnet |
| **工具** | Agent Browser (首选) / Playwright (备选) |
| **核心能力** | 测试旅程创建、不稳定测试管理、产物管理 |

**工作流程:**
```
1. Plan → 确定关键用户旅程
2. Create → 使用 POM 模式创建测试
3. Execute → 本地运行 3-5 次检查不稳定性
```

**关键原则:**
- 使用语义选择器 `[data-testid="..."]`
- 等待条件而非时间
- 每个测试独立，无共享状态
- 关键步骤使用 `expect()` 断言

**不稳定测试处理:**
```typescript
// 隔离不稳定测试
test('flaky: market search', async ({ page }) => {
  test.fixme(true, 'Flaky - Issue #123')
})
```

**使用场景:**
- 关键用户流程 (认证、支付、CRUD)
- 回归测试
- CI/CD 集成

---

### 📝 六、文档类 Agent (2 个)

#### 18. `doc-updater` - 文档更新专家
| 属性 | 说明 |
|------|------|
| **模型** | haiku (轻量) |
| **触发条件** | 文档同步、Codemap 生成 |
| **核心能力** | 架构图生成、文档刷新、AST 分析 |

**核心职责:**
1. **Codemap 生成** - 从代码结构创建架构图
2. **文档更新** - 从代码刷新 README 和指南
3. **依赖映射** - 跟踪模块间的导入/导出

**Codemap 结构:**
```
docs/CODEMAPS/
├── INDEX.md          # 总览
├── frontend.md       # 前端结构
├── backend.md        # 后端/API 结构
├── database.md       # 数据库模式
└── integrations.md   # 外部服务
```

**使用场景:**
- 新主要功能添加后
- API 路由变更后
- 架构变更后
- 定期文档同步

---

#### 19. `docs-lookup` - 文档查询专家
| 属性 | 说明 |
|------|------|
| **触发条件** | 库/API 文档问题 |
| **核心能力** | 官方文档搜索、API 参考、代码示例 |

**使用场景:**
- 查找库的最新 API
- 获取代码示例
- 验证用法

---

### 🧹 七、维护类 Agent (1 个)

#### 20. `refactor-cleaner` - 重构清理专家
| 属性 | 说明 |
|------|------|
| **模型** | sonnet |
| **触发条件** | 代码清理、死代码移除 |
| **核心能力** | 死代码检测、重复消除、依赖清理 |

**检测工具:**
```bash
npx knip          # 未使用文件、导出、依赖
npx depcheck      # 未使用 npm 依赖
npx ts-prune      # 未使用 TypeScript 导出
```

**风险分类:**
- **SAFE**: 未使用导出/依赖
- **CAREFUL**: 动态导入
- **RISKY**: 公共 API

**工作流程:**
```
分析 → 验证 → 安全移除 → 测试 → 提交
```

**使用场景:**
- 定期代码清理
- 减少包体积
- 技术债务偿还

---

### 🤖 八、自动化操作类 Agent (2 个)

#### 21. `loop-operator` - 循环操作员
| 属性 | 说明 |
|------|------|
| **模型** | sonnet |
| **颜色** | orange |
| **核心能力** | 自主循环执行、进度监控、安全干预 |

**使命:** 安全地运行自主循环，具备清晰的停止条件、可观察性和恢复操作。

**工作流程:**
```
1. 从显式模式和模式启动循环
2. 跟踪进度检查点
3. 检测停滞和重试风暴
4. 失败重复时暂停并缩小范围
5. 验证通过后恢复
```

**必需检查:**
- ✅ 质量门控激活
- ✅ 评估基线存在
- ✅ 回滚路径存在
- ✅ 分支/worktree 隔离已配置

**升级条件:**
- 连续两个检查点无进展
- 相同堆栈跟踪的重复失败
- 成本超出预算窗口
- 合并冲突阻塞队列

---

#### 22. `harness-optimizer` - Harness 优化器
| 属性 | 说明 |
|------|------|
| **模型** | sonnet |
| **颜色** | teal |
| **核心能力** | Harness 配置分析、性能优化 |

**使命:** 通过改进 Harness 配置提升智能体完成质量。

**工作流程:**
```
1. 运行 /harness-audit 收集基线分数
2. 确定前 3 个杠杆领域 (hooks, evals, routing, context, safety)
3. 提出最小化、可逆的配置更改
4. 应用更改并运行验证
5. 报告前后对比
```

**约束:**
- 偏好小变更、可衡量效果
- 保持跨平台行为一致
- 保持与 Claude Code、Cursor、OpenCode、Codex 的兼容性

---

### 💼 九、业务支持类 Agent (1 个)

#### 23. `chief-of-staff` - 幕僚长
| 属性 | 说明 |
|------|------|
| **模型** | opus |
| **触发条件** | 多通道通信工作流 |
| **核心能力** | 邮件/Slack/LINE/Messenger 分流、草稿回复、日程管理 |

**4 级分类系统:**

| 级别 | 描述 | 操作 |
|------|------|------|
| **skip** | 自动归档 (noreply、bot 消息) | 立即归档 |
| **info_only** | 仅摘要 (收据、群聊) | 单行摘要 |
| **meeting_info** | 会议信息 (含 Zoom/Meet 链接) | 交叉引用日历 |
| **action_required** | 需要操作 (直接消息、@提及) | 生成草稿回复 |

**工作流程:**
```
并行获取所有通道 → 分类 → 执行操作 → 草稿回复 → 发送后跟进
```

**发送后检查清单:**
1. 日历 - 创建临时事件
2. 关系 - 更新关系档案
3. Todo - 更新待办事项
4. 待回复 - 设置跟进截止日期
5. 归档 - 从收件箱移除
6. 版本控制 - Git commit & push

**使用场景:**
- 每日邮件分流
- Slack 消息处理
- 日程安排管理
- 多通道通信统一处理

---

## Agent 使用矩阵

### 按开发阶段使用

| 开发阶段 | 推荐 Agent |
|----------|------------|
| 需求分析 | `planner`, `architect` |
| 编码实现 | `tdd-guide` |
| 代码审查 | `code-reviewer`, `{language}-reviewer` |
| 构建测试 | `build-error-resolver`, `tdd-guide` |
| 安全审查 | `security-reviewer` |
| 集成测试 | `e2e-runner` |
| 文档更新 | `doc-updater` |
| 代码清理 | `refactor-cleaner` |
| 部署发布 | `security-reviewer`, `e2e-runner` |

### 按任务类型使用

| 任务类型 | Agent |
|----------|-------|
| 功能规划 | `/plan` → `planner` |
| Bug 修复 | `/tdd` → `tdd-guide` |
| 构建失败 | `/build-fix` → `build-error-resolver` |
| 代码审查 | `/code-review` → `code-reviewer` |
| 安全扫描 | `/security-scan` → `security-reviewer` |
| E2E 测试 | `/e2e` → `e2e-runner` |
| 更新文档 | `/update-docs` → `doc-updater` |
| 清理代码 | `/refactor-clean` → `refactor-cleaner` |
| 多智能体任务 | `/multi-plan` → `planner` + 其他 |

---

## Agent 委托机制

### 自动委托规则

```yaml
# AGENTS.md 中定义的委托规则

复杂功能请求:
  委托给: planner
  
代码编写/修改后:
  委托给: code-reviewer
  
Bug 修复或新功能:
  委托给: tdd-guide
  
架构决策:
  委托给: architect
  
安全敏感代码:
  委托给: security-reviewer
  
多通道通信分流:
  委托给: chief-of-staff
  
自主循环/循环监控:
  委托给: loop-operator
  
Harness 配置可靠性和成本:
  委托给: harness-optimizer
```

### 模型选择策略

| Agent | 模型 | 原因 |
|-------|------|------|
| `planner`, `architect`, `chief-of-staff` | opus | 需要最强推理能力 |
| `code-reviewer`, `security-reviewer`, `tdd-guide` | sonnet | 平衡性能和成本 |
| `doc-updater` | haiku | 轻量任务，成本敏感 |

---

## Agent 文件结构

```
agents/
├── planner.md              # 规划专家
├── architect.md            # 架构师
├── tdd-guide.md            # TDD 导师
├── code-reviewer.md        # 代码审查员
├── security-reviewer.md    # 安全审查员
├── build-error-resolver.md # 构建错误修复
├── e2e-runner.md           # E2E 测试专家
├── doc-updater.md          # 文档更新
├── docs-lookup.md          # 文档查询
├── refactor-cleaner.md     # 重构清理
├── loop-operator.md        # 循环操作员
├── harness-optimizer.md    # Harness 优化器
├── chief-of-staff.md       # 幕僚长
├── typescript-reviewer.md  # TypeScript 审查
├── python-reviewer.md      # Python 审查
├── go-reviewer.md          # Go 审查
├── go-build-resolver.md    # Go 构建修复
├── java-reviewer.md        # Java 审查
├── java-build-resolver.md  # Java 构建修复
├── kotlin-reviewer.md      # Kotlin 审查
├── kotlin-build-resolver.md# Kotlin 构建修复
├── rust-reviewer.md        # Rust 审查
├── rust-build-resolver.md  # Rust 构建修复
├── cpp-reviewer.md         # C++ 审查
├── cpp-build-resolver.md   # C++ 构建修复
├── flutter-reviewer.md     # Flutter 审查
├── database-reviewer.md    # 数据库审查
└── pytorch-build-resolver.md # PyTorch 修复
```

---

## 最佳实践

### 1. 委托而非重复

```markdown
❌ 错误: 在对话中手动审查代码
✅ 正确: "/code-review" 委托给 code-reviewer

❌ 错误: 自己规划功能实现
✅ 正确: "/plan '添加支付功能'" 委托给 planner
```

### 2. 组合使用 Agent

```markdown
# 完整开发流程示例

1. /plan "添加用户认证"              → planner
2. /tdd "实现登录功能"               → tdd-guide
3. /code-review                      → code-reviewer
4. /security-scan                    → security-reviewer
5. /e2e "用户登录流程"               → e2e-runner
6. /update-docs                      → doc-updater
```

### 3. 信任 Agent 的专业性

每个 Agent 都有:
- 明确的角色定义
- 详细的检查清单
- 具体的输出格式
- 成功标准

不要试图覆盖 Agent 的判断，而是提供上下文让它更好地工作。

---

## 总结

ECC 的 28 个 Agent 构成了一个完整的 AI 辅助开发团队:

| 角色 | Agent | 数量 |
|------|-------|------|
| 项目经理 | planner, architect | 2 |
| 开发工程师 | tdd-guide | 1 |
| 代码审查员 | code-reviewer, *-reviewer | 11 |
| 安全专家 | security-reviewer | 1 |
| 测试工程师 | e2e-runner | 1 |
| 技术作家 | doc-updater, docs-lookup | 2 |
| 维护工程师 | refactor-cleaner, build-error-resolver | 2 |
| 运维工程师 | loop-operator, harness-optimizer | 2 |
| 行政助理 | chief-of-staff | 1 |
| **总计** | | **28** |

通过智能委托，ECC 能够像经验丰富的开发团队一样工作，确保代码质量、安全性和可维护性。
