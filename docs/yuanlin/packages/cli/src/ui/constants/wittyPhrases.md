# wittyPhrases.ts

> 定义 CLI 加载等待时显示的趣味短语列表

## 概述

`wittyPhrases.ts` 导出一个包含 130+ 条幽默短语的字符串数组 `WITTY_LOADING_PHRASES`。这些短语在 CLI 等待 Gemini 模型响应时随机展示，为用户提供轻松愉快的等待体验。内容涵盖编程梗、科幻引用、互联网文化、冷笑话等。

## 架构图（mermaid）

```mermaid
graph LR
    A[加载状态组件] -->|随机选取| B[WITTY_LOADING_PHRASES]
    B --> C[展示趣味短语]
```

## 主要导出

| 名称 | 类型 | 说明 |
|------|------|------|
| `WITTY_LOADING_PHRASES` | `string[]` | 趣味加载短语数组 |

## 核心逻辑

纯数据定义文件。短语示例包括：
- 编程相关：`"Rewriting in Rust for no particular reason..."`, `"Trying to exit Vim..."`
- 科幻引用：`"Calibrating the flux capacitor..."`, `"Don't panic..."`
- 幽默笑话：`"What do you call a fish with no eyes? A fsh..."`
- 互联网文化：`"Rickrolling my boss..."`, `"Pondering the orb..."`

## 内部依赖

无

## 外部依赖

无
