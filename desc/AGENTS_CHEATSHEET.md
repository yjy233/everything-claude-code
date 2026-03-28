# Agents 速查卡

## 28 个智能体一览

### 🎯 核心开发 (4 个)
| Agent | 用途 | 命令 |
|-------|------|------|
| `planner` | 功能规划、任务分解 | `/plan` |
| `architect` | 架构设计、技术决策 | 自动委托 |
| `tdd-guide` | TDD 开发、测试驱动 | `/tdd` |
| `code-reviewer` | 代码审查、质量检查 | `/code-review` |

### 🔒 安全 (1 个)
| Agent | 用途 | 命令 |
|-------|------|------|
| `security-reviewer` | 安全漏洞扫描 | `/security-scan` |

### 🔨 修复 (1 个)
| Agent | 用途 | 命令 |
|-------|------|------|
| `build-error-resolver` | 构建/类型错误修复 | `/build-fix` |

### 🌐 语言专用 (10 组 = 18 个)
| Agent | 语言 | Agent | 用途 |
|-------|------|-------|------|
| `typescript-reviewer` | TS/JS | - | 代码审查 |
| `python-reviewer` | Python | - | 代码审查 |
| `go-reviewer` | Go | `go-build-resolver` | 审查+构建 |
| `java-reviewer` | Java | `java-build-resolver` | 审查+构建 |
| `kotlin-reviewer` | Kotlin | `kotlin-build-resolver` | 审查+构建 |
| `rust-reviewer` | Rust | `rust-build-resolver` | 审查+构建 |
| `cpp-reviewer` | C++ | `cpp-build-resolver` | 审查+构建 |
| `flutter-reviewer` | Flutter | - | 代码审查 |
| `database-reviewer` | PostgreSQL | - | 数据库审查 |
| `pytorch-build-resolver` | PyTorch | - | 训练错误修复 |

### 🧪 测试 (1 个)
| Agent | 用途 | 命令 |
|-------|------|------|
| `e2e-runner` | E2E 测试 | `/e2e` |

### 📝 文档 (2 个)
| Agent | 用途 | 命令 |
|-------|------|------|
| `doc-updater` | 文档更新 | `/update-docs` |
| `docs-lookup` | 文档查询 | 自动委托 |

### 🧹 维护 (1 个)
| Agent | 用途 | 命令 |
|-------|------|------|
| `refactor-cleaner` | 死代码清理 | `/refactor-clean` |

### 🤖 自动化 (2 个)
| Agent | 用途 |
|-------|------|
| `loop-operator` | 自主循环执行、监控 |
| `harness-optimizer` | Harness 配置优化 |

### 💼 业务 (1 个)
| Agent | 用途 |
|-------|------|
| `chief-of-staff` | 邮件/Slack 分流、日程管理 |

---

## 常用工作流

### 新功能开发流程
```bash
/plan "功能描述"        # planner
/tdd "功能描述"         # tdd-guide
/code-review            # code-reviewer
/security-scan          # security-reviewer
```

### Bug 修复流程
```bash
/tdd "修复 Bug"         # tdd-guide (先写测试)
/build-fix              # build-error-resolver (如有错误)
/code-review            # code-reviewer
```

### 重构流程
```bash
/plan "重构方案"        # planner
/tdd "保持行为"         # tdd-guide
/refactor-clean         # refactor-cleaner
/code-review            # code-reviewer
```

### 发布前检查
```bash
/code-review            # code-reviewer
/security-scan          # security-reviewer
/e2e "关键流程"         # e2e-runner
/update-docs            # doc-updater
```

---

## 模型分配

| 模型 | Agent |
|------|-------|
| **opus** (最强) | planner, architect, chief-of-staff |
| **sonnet** (平衡) | code-reviewer, security-reviewer, tdd-guide, *-reviewer |
| **haiku** (轻量) | doc-updater |

---

## 自动委托触发条件

```
复杂功能请求     → planner
代码编写/修改后  → code-reviewer
Bug 修复/新功能  → tdd-guide
架构决策         → architect
安全敏感代码     → security-reviewer
多通道通信       → chief-of-staff
自主循环         → loop-operator
Harness 优化     → harness-optimizer
```

---

## Agent 配置文件格式

```yaml
---
name: agent-name
description: 简短描述。Use PROACTIVELY when... 
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet  # opus | sonnet | haiku
color: blue    # 可选: blue, green, orange, teal, purple
---

# Agent 指令内容 (角色、流程、检查清单、示例)
```

---

**完整文档**: `AGENTS_GUIDE.md`
