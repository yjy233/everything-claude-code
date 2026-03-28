# ECC 快速入门指南

## 什么是 ECC?

**Everything Claude Code (ECC)** 是 AI 智能体开发的生产级性能优化系统。

```
不是配置文件，而是完整的系统
├── 28 个专业智能体
├── 125 个工作流技能
├── 60 个斜杠命令
├── 自动化钩子工作流
└── 跨平台支持 (Claude Code / Cursor / OpenCode / Codex)
```

---

## 一分钟安装

### 方式 1: 插件安装 (推荐)
```bash
/plugin marketplace add affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

### 方式 2: 手动安装
```bash
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code
npm install
./install.sh typescript   # 或 python, golang, java, kotlin, rust, cpp...
```

### 方式 3: npm
```bash
npm install ecc-universal
npx ecc typescript
```

---

## 核心命令速查

| 命令 | 用途 |
|------|------|
| `/plan "功能"` | 智能体规划功能实现 |
| `/tdd "功能"` | 测试驱动开发 |
| `/code-review` | 代码质量审查 |
| `/build-fix` | 自动修复构建错误 |
| `/e2e "用户流程"` | 生成 E2E 测试 |
| `/security-scan` | 安全漏洞扫描 |
| `/learn` | 提取会话中的模式 |

---

## 智能体委托

ECC 会自动委托给专业智能体：

```
复杂功能请求 → planner (规划)
代码编写后   → code-reviewer (审查)
Bug 修复    → tdd-guide (TDD)
构建错误    → build-error-resolver (修复)
安全敏感    → security-reviewer (安全检查)
架构决策    → architect (架构设计)
```

---

## 项目结构速览

```
agents/      # 28 个智能体定义 (planner, code-reviewer, security-reviewer...)
skills/      # 125 个技能 (tdd-workflow, api-design, security-review...)
commands/    # 60 个命令 (/plan, /tdd, /code-review...)
rules/       # 12 种语言规则 (common, typescript, python, golang...)
hooks/       # 自动化钩子 (会话启动/结束, 工具使用前后)
scripts/     # Node.js 跨平台脚本
```

---

## 下一步

1. **阅读指南**: `the-shortform-guide.md` (简短) 或 `the-longform-guide.md` (详细)
2. **尝试命令**: 从 `/plan "你的功能"` 开始
3. **探索技能**: 查看 `skills/` 目录了解可用工作流
4. **配置规则**: 根据你的技术栈安装对应的规则

---

**版本**: v1.9.0 | **测试**: 997+ | **许可**: MIT
