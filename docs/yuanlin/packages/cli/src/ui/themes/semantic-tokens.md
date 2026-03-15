# semantic-tokens.ts

> 定义语义化颜色接口以及明/暗主题的语义颜色预设实例

## 概述

`semantic-tokens.ts` 定义了 `SemanticColors` 接口，将颜色按用途（而非具体色值）分为五大类：文本（`text`）、背景（`background`）、边框（`border`）、UI 元素（`ui`）和状态（`status`）。同时提供 `lightSemanticColors` 和 `darkSemanticColors` 两套预设实例。

## 架构图（mermaid）

```mermaid
graph TD
    A[SemanticColors] --> B[text]
    A --> C[background]
    A --> D[border]
    A --> E[ui]
    A --> F[status]

    B --> B1[primary: 主要文本色]
    B --> B2[secondary: 次要文本色]
    B --> B3[link: 链接色]
    B --> B4[accent: 强调色]
    B --> B5[response: 响应文本色]

    C --> C1[primary: 主背景]
    C --> C2[message: 消息背景]
    C --> C3[input: 输入框背景]
    C --> C4[focus: 焦点背景]
    C --> C5[diff.added / diff.removed]

    D --> D1[default: 默认边框色]

    E --> E1[comment / symbol / active / dark / focus]
    E --> E2[gradient: 渐变色数组]

    F --> F1[error / success / warning]
```

## 主要导出

| 名称 | 类型 | 说明 |
|------|------|------|
| `SemanticColors` | `interface` | 语义化颜色结构定义 |
| `lightSemanticColors` | `SemanticColors` | 浅色主题语义颜色预设 |
| `darkSemanticColors` | `SemanticColors` | 深色主题语义颜色预设 |

## 核心逻辑

- `lightSemanticColors` 从 `lightTheme` (ColorsTheme) 映射而来
- `darkSemanticColors` 从 `darkTheme` (ColorsTheme) 映射而来
- 映射规则一致：`Foreground`→`text.primary`、`Gray`→`text.secondary`、`AccentBlue`→`text.link` 等

## 内部依赖

| 模块 | 用途 |
|------|------|
| `./theme.js` → `lightTheme`, `darkTheme` | 内置颜色预设 |

## 外部依赖

无
