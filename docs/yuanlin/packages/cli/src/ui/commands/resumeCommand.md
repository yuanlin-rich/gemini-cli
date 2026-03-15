# resumeCommand.ts

> 浏览自动保存的对话并管理聊天检查点

## 概述

`resumeCommand` 实现了 `/resume` 斜杠命令，默认打开会话浏览器对话框，并复用 `chatCommand` 中导出的 `chatResumeSubCommands` 作为子命令集（包含 `list`、`save`、`resume`、`delete`、`share` 等检查点操作）。

## 架构图（mermaid）

```mermaid
flowchart TD
    A["/resume"] --> B["返回 sessionBrowser 对话框"]
    A --> C{子命令}
    C -->|list| D["列出检查点"]
    C -->|save| E["保存检查点"]
    C -->|resume/load| F["恢复检查点"]
    C -->|delete| G["删除检查点"]
    C -->|share| H["导出对话"]
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `resumeCommand` | `SlashCommand` | `/resume` 命令，自动执行 |

## 核心逻辑

1. 默认 action 返回 `sessionBrowser` 对话框，让用户浏览自动保存的对话。
2. 子命令集来自 `chatCommand.ts` 的 `chatResumeSubCommands`（详见 chatCommand 文档）。

## 内部依赖

| 模块 | 用途 |
|------|------|
| `./types.js` | `OpenDialogActionReturn`、`CommandContext`、`SlashCommand`、`CommandKind` |
| `./chatCommand.js` | `chatResumeSubCommands` |

## 外部依赖

无
