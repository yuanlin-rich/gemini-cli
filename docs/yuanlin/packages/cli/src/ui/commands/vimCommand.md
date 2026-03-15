# vimCommand.ts

> 切换 Vim 编辑模式

## 概述

`vimCommand` 实现了 `/vim` 斜杠命令，切换输入框的 Vim 键绑定模式。标记为并发安全，可在 Agent 运行时使用。

## 架构图（mermaid）

```mermaid
flowchart TD
    A["/vim"] --> B["toggleVimEnabled()"]
    B --> C{新状态}
    C -->|启用| D["提示已进入 Vim 模式"]
    C -->|禁用| E["提示已退出 Vim 模式"]
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `vimCommand` | `SlashCommand` | `/vim` 命令，自动执行，并发安全 |

## 核心逻辑

1. 调用 `context.ui.toggleVimEnabled()` 异步切换 Vim 模式。
2. 根据返回的新状态布尔值显示相应的提示消息。

## 内部依赖

| 模块 | 用途 |
|------|------|
| `./types.js` | `CommandKind`、`SlashCommand` |

## 外部依赖

无
