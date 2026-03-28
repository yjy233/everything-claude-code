# Everything Claude Code (ECC) 项目介绍

## 📋 项目概述

**Everything Claude Code (ECC)** 是一个生产级的 AI 编码插件系统，由 Anthropic 黑客马拉松获奖者开发。它提供了一个完整的 AI 智能体性能优化系统，包含 28 个专业智能体、125 个工作流技能、60 个斜杠命令和自动化钩子工作流。

> **核心理念**: 不仅是配置文件，而是一套完整的系统，包括技能、本能、内存优化、持续学习和安全扫描。

---

## 🏆 项目亮点

| 指标 | 数据 |
|------|------|
| GitHub Stars | 50K+ |
| Forks | 6K+ |
| 贡献者 | 30+ |
| 支持语言 | 12 种编程语言 |
| 智能体数量 | 28 个 |
| 技能数量 | 125 个 |
| 命令数量 | 60 个 |
| 内部测试 | 997+ |
| 版本 | v1.9.0 |

---

## 🎯 核心功能

### 1. 专业智能体 (Agents) - 28 个

按功能分类的专业智能体系统：

**开发流程类:**
- `planner` - 功能实现规划
- `architect` - 系统架构设计
- `tdd-guide` - 测试驱动开发
- `code-reviewer` - 代码质量审查

**语言专用类:**
- `typescript-reviewer` - TypeScript/JavaScript
- `python-reviewer` - Python
- `java-reviewer` / `java-build-resolver` - Java/Spring Boot
- `kotlin-reviewer` / `kotlin-build-resolver` - Kotlin/Android
- `go-reviewer` / `go-build-resolver` - Go
- `rust-reviewer` / `rust-build-resolver` - Rust
- `cpp-reviewer` / `cpp-build-resolver` - C++
- `pytorch-build-resolver` - PyTorch/CUDA

**专项功能类:**
- `security-reviewer` - 安全漏洞检测
- `database-reviewer` - PostgreSQL/Supabase 专家
- `e2e-runner` - Playwright E2E 测试
- `refactor-cleaner` - 死代码清理
- `doc-updater` - 文档同步
- `docs-lookup` - 文档/API 查询
- `build-error-resolver` - 构建错误修复
- `chief-of-staff` - 通信处理和草稿
- `loop-operator` - 自主循环执行
- `harness-optimizer` - 性能配置优化

### 2. 工作流技能 (Skills) - 125 个

**框架/技术栈技能:**
| 技能 | 描述 |
|------|------|
| `nextjs-turbopack` | Next.js 16+ 和 Turbopack 配置 |
| `bun-runtime` | Bun 运行时、包管理器、打包器 |
| `django-patterns` | Django 架构模式 |
| `springboot-patterns` | Spring Boot 开发模式 |
| `laravel-patterns` | Laravel 架构模式 |
| `golang-patterns` | Go 语言惯用法 |
| `kotlin-patterns` | Kotlin 开发模式 |
| `swiftui-patterns` | SwiftUI 开发 |
| `rust-patterns` | Rust 开发模式 |

**开发方法论技能:**
| 技能 | 描述 |
|------|------|
| `tdd-workflow` | TDD 工作流程 |
| `security-review` | 安全审查清单 |
| `backend-patterns` | 后端架构模式 |
| `frontend-patterns` | 前端开发模式 |
| `api-design` | REST API 设计 |
| `e2e-testing` | Playwright E2E 测试 |

**AI/内容创作技能:**
| 技能 | 描述 |
|------|------|
| `claude-api` | Anthropic Claude API 集成 |
| `content-engine` | 多平台内容创作 |
| `article-writing` | 长文写作 |
| `frontend-slides` | HTML 演示文稿生成 |
| `fal-ai-media` | AI 媒体生成（图像/视频/音频） |
| `video-editing` | AI 辅助视频编辑 |
| `deep-research` | 深度研究 |

**专业领域技能:**
| 技能 | 描述 |
|------|------|
| `market-research` | 市场研究 |
| `investor-materials` | 投资人材料 |
| `investor-outreach` | 投资人外联 |
| `continuous-learning` | 持续学习 |
| `eval-harness` | 评估框架 |

### 3. 斜杠命令 (Commands) - 60 个

**开发命令:**
```
/plan "功能描述"          # 实现规划
/tdd "功能描述"           # TDD 开发
/code-review              # 代码审查
/build-fix                # 修复构建错误
/e2e "用户流程"           # E2E 测试生成
/refactor-clean           # 死代码清理
```

**多智能体命令:**
```
/multi-plan "任务"        # 多智能体任务分解
/multi-execute            # 多智能体工作流执行
/multi-backend            # 后端多服务编排
/multi-frontend           # 前端多服务编排
/orchestrate              # 多智能体协调
```

**学习/优化命令:**
```
/learn                    # 从会话中提取模式
/learn-eval               # 提取并评估模式
/instinct-status          # 查看学习到的本能
/instinct-export          # 导出本能
/evolve                   # 将本能聚合成技能
/checkpoint               # 保存验证状态
/verify                   # 运行验证循环
```

**项目管理命令:**
```
/sessions                 # 会话历史管理
/save-session             # 保存会话
/resume-session           # 恢复会话
/pm2                      # PM2 服务管理
/setup-pm                 # 配置包管理器
```

### 4. 规则系统 (Rules)

**12 种编程语言支持:**

```
rules/
├── common/          # 通用原则
│   ├── coding-style.md      # 编码风格
│   ├── git-workflow.md      # Git 工作流
│   ├── testing.md           # 测试要求
│   ├── performance.md       # 性能优化
│   ├── security.md          # 安全检查
│   └── agents.md            # 智能体委托
├── typescript/      # TypeScript/JavaScript
├── python/          # Python
├── golang/          # Go
├── java/            # Java
├── kotlin/          # Kotlin
├── rust/            # Rust
├── cpp/             # C++
├── swift/           # Swift
├── php/             # PHP
├── perl/            # Perl
└── csharp/          # C#
```

**核心原则:**
1. **不可变性** - 始终创建新对象，永不修改现有对象
2. **文件组织** - 小文件优于大文件（200-400 行典型，800 行最大）
3. **错误处理** - 每个层级都要处理错误
4. **输入验证** - 在系统边界验证所有用户输入
5. **TDD** - 测试覆盖率要求 80%+

### 5. 钩子系统 (Hooks)

自动化触发器系统：

```
hooks/
├── hooks.json                    # 钩子配置
├── memory-persistence/           # 会话生命周期钩子
│   ├── session-start/            # 会话开始
│   ├── pre-tool-use/             # 工具使用前
│   ├── post-tool-use/            # 工具使用后
│   └── stop/                     # 会话结束
└── strategic-compact/            # 压缩建议钩子
```

**钩子类型:**
- `PreToolUse` - 工具执行前触发
- `PostToolUse` - 工具执行后触发
- `Stop` - 会话结束时触发
- `SessionStart` - 会话开始时触发

---

## 📁 项目结构

```
everything-claude-code/
├── agents/              # 28 个专业智能体定义
├── skills/              # 125 个工作流技能
├── commands/            # 60 个斜杠命令
├── rules/               # 12 种语言的编码规则
├── hooks/               # 自动化钩子
├── scripts/             # Node.js 跨平台脚本
├── tests/               # 测试套件
├── contexts/            # 动态系统提示注入上下文
├── examples/            # 示例配置
├── mcp-configs/         # MCP 服务器配置
├── manifests/           # 安装清单
├── docs/                # 多语言文档
├── .claude-plugin/      # Claude Code 插件配置
├── .cursor/             # Cursor IDE 配置
├── .opencode/           # OpenCode 配置
├── .codex/              # Codex CLI 配置
└── desc/                # 本介绍文档
```

---

## 🛠️ 技术栈

| 类别 | 技术 |
|------|------|
| **核心语言** | TypeScript, JavaScript, Node.js |
| **包管理器** | npm, pnpm, yarn, Bun |
| **测试** | 内部测试框架，c8 覆盖率 |
| **代码质量** | ESLint, markdownlint |
| **数据存储** | SQLite (sql.js) |
| **配置验证** | AJV JSON Schema |
| **跨平台** | Node.js 脚本 (Windows/macOS/Linux) |

---

## 🚀 快速开始

### 安装方式 1: 插件安装

```bash
# 添加市场
/plugin marketplace add affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code@everything-claude-code
```

### 安装方式 2: 手动安装

```bash
# 克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# 安装依赖
npm install

# 安装规则 (选择你需要的语言)
./install.sh typescript          # macOS/Linux
# 或
.\install.ps1 typescript         # Windows PowerShell

# 支持多语言同时安装
./install.sh typescript python golang
```

### npm 安装

```bash
npm install ecc-universal
npx ecc typescript
```

---

## 📖 使用示例

### 1. 功能规划
```bash
/plan "添加用户认证系统，包括登录、注册、密码重置"
```

### 2. TDD 开发
```bash
/tdd "实现购物车功能，包括添加商品、计算总价、应用优惠码"
```

### 3. 代码审查
```bash
/code-review
# 或针对特定文件
/code-review src/auth/login.ts
```

### 4. 安全扫描
```bash
/security-scan
```

### 5. 多智能体协作
```bash
/multi-plan "重构整个后端 API，迁移到微服务架构"
```

---

## 🌐 跨平台支持

| 平台 | 支持状态 |
|------|---------|
| Claude Code | ✅ 完全支持 |
| Cursor | ✅ 完全支持 |
| OpenCode | ✅ 完全支持 |
| Codex CLI | ✅ 完全支持 |
| Antigravity | ✅ 完全支持 |
| Windows | ✅ PowerShell + Node.js |
| macOS | ✅ Shell + Node.js |
| Linux | ✅ Shell + Node.js |

---

## 📚 相关文档

- **简短指南** (`the-shortform-guide.md`) - 设置、基础、理念
- **详细指南** (`the-longform-guide.md`) - Token 优化、内存持久化、评估
- **安全指南** (`the-security-guide.md`) - 攻击向量、沙箱、CVE、AgentShield
- **故障排除** (`TROUBLESHOOTING.md`) - 常见问题解决
- **贡献指南** (`CONTRIBUTING.md`) - 如何贡献

---

## 🔄 版本历史

### v1.9.0 (2026年3月) - 选择性安装与语言扩展
- 清单驱动安装管道
- 6 个新智能体 (TypeScript, PyTorch, Java, Kotlin 等)
- 会话和状态基础设施
- 12 种语言生态系统支持

### v1.8.0 (2026年3月) - Harness 性能系统
- Harness 优先架构
- 钩子可靠性重构
- NanoClaw v2
- 997 个内部测试

### v1.7.0 (2026年2月) - 跨平台扩展
- Codex app + CLI 支持
- `frontend-slides` 技能
- 5 个新的业务/内容技能

---

## 👥 作者与许可

- **作者**: Affaan Mustafa ([@affaanmustafa](https://x.com/affaanmustafa))
- **许可**: MIT
- **GitHub**: [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)
- **NPM**: [ecc-universal](https://www.npmjs.com/package/ecc-universal)

---

## 🎯 总结

Everything Claude Code 是一个企业级的 AI 编码助手系统，通过智能体、技能、命令和规则的完整体系，为开发者提供:

1. **标准化开发流程** - TDD、代码审查、安全扫描
2. **多语言支持** - 12 种编程语言的专业规则
3. **智能协作** - 28 个专业智能体按需调用
4. **持续学习** - 自动提取和保存开发模式
5. **性能优化** - Token 优化、内存持久化、验证循环

无论是个人开发者还是团队，ECC 都能显著提升 AI 辅助编码的效率和质量。
