# /feature-development 中文版

> 这是 [feature-development.md](/Users/bill/code/everything-claude-code/.claude/commands/feature-development.md) 的中文说明版，放在 `desc/` 下用于阅读和理解。  
> 它是文档，不是会被 Claude Code 直接加载的 command 文件。

## 命令名称

`/feature-development`

## 适用场景

当你在 `everything-claude-code` 中处理标准功能开发任务时，可以参考这个工作流。

## 目标

执行一套标准的功能实现流程。

## 常见涉及文件

- `manifests/*`
- `schemas/*`
- `**/*.test.*`
- `**/api/**`

## 建议执行顺序

1. 先理解当前功能现状、已有实现方式以及当前失败或需求背景。
2. 在满足功能目标的前提下，尽量做最小且完整的改动。
3. 对改动过的实现、测试和接口相关文件运行最相关的验证。
4. 总结本次修改内容，以及仍然需要人工复核的部分。

## 常见改动信号

- 新增功能实现
- 为功能补充测试
- 更新相关文档

## 说明

- 这个命令是一个 workflow scaffold，也就是工作流脚手架，不是硬编码脚本。
- 它提供的是标准功能开发的基本结构，但不会自动替你完成具体实现、测试或发布流程。
- 如果仓库里的功能开发流程、测试策略或文档要求发生明显变化，这条命令本身也应该同步更新。

## 一句话理解

`/feature-development` 的作用不是“自动开发一个功能”，而是告诉 Claude：

> 当我要在 ECC 项目里做标准功能开发时，应该按什么顺序理解、修改、验证和总结。
