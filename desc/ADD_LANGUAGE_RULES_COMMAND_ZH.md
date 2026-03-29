# /add-language-rules 中文版

> 这是 [add-language-rules.md](/Users/bill/code/everything-claude-code/.claude/commands/add-language-rules.md) 的中文说明版，放在 `desc/` 下用于阅读和理解。  
> 它是文档，不是会被 Claude Code 直接加载的 command 文件。

## 命令名称

`/add-language-rules`

## 适用场景

当你在 `everything-claude-code` 中处理“为规则系统新增一种编程语言支持”这类任务时，可以参考这个工作流。

## 目标

为 `rules/` 系统新增一种编程语言支持，补齐该语言对应的规则文件，包括：

- 编码风格
- hooks 约定
- 常见模式
- 安全要求
- 测试规范

## 常见涉及文件

- `rules/*/coding-style.md`
- `rules/*/hooks.md`
- `rules/*/patterns.md`
- `rules/*/security.md`
- `rules/*/testing.md`

## 建议执行顺序

1. 先理解当前规则系统的现状，以及为什么需要新增这门语言。
2. 在满足目标的前提下，尽量做最小且完整的改动。
3. 对改动过的文件运行最相关的验证。
4. 总结本次修改内容，以及仍然需要人工复核的部分。

## 常见改动信号

- 在 `rules/{language}/` 下新建一个语言目录
- 新增以下文件：
  - `coding-style.md`
  - `hooks.md`
  - `patterns.md`
  - `security.md`
  - `testing.md`
- 如有必要，补充或引用相关技能文档

## 说明

- 这个命令是一个 workflow scaffold，也就是工作流脚手架，不是硬编码脚本。
- 它提供的是方向和结构，不会自动完成所有安装链路和文档同步工作。
- 如果规则系统的实现方式发生明显变化，这条命令本身也应该同步更新。

## 一句话理解

`/add-language-rules` 的作用不是“自动加一门语言”，而是告诉 Claude：

> 当我要为 ECC 的 `rules/` 系统新增一种语言时，应该按什么结构、什么顺序、改哪些核心文件去完成。
