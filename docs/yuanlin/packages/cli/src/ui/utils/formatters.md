# formatters.ts

> 通用格式化工具集：字节大小、时间持续、时间差、引用内容剥离、重置时间显示

## 概述

本文件提供一组用于 UI 显示的格式化函数，涵盖字节大小转换（KB/MB/GB）、毫秒时间格式化（ms/s/m/h）、相对时间显示（"just now"/"5m ago"）、Markdown 引用内容剥离，以及 API 配额重置时间的多种格式化方式。

## 架构图（mermaid）

```mermaid
flowchart LR
    A[formatBytes] --> B[KB / MB / GB]
    C[formatDuration] --> D[ms / s / Xm Ys / Xh Ym Zs]
    E[formatTimeAgo] --> F["just now" / "5m ago"]
    G[stripReferenceContent] --> H[移除标记之间的引用内容]
    I[formatResetTime] --> J[terse / column / full 三种格式]
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `formatBytes` | function | 将字节数格式化为可读字符串（如 `1.5 MB`） |
| `formatDuration` | function | 将毫秒格式化为简洁时间字符串（如 `1h 5s`） |
| `formatTimeAgo` | function | 将日期转为相对时间字符串（如 `just now`） |
| `stripReferenceContent` | function | 移除被引用内容标记包围的文本块 |
| `formatResetTime` | function | 格式化 API 配额重置时间（支持 terse/column/full 三种模式） |

## 核心逻辑

1. **formatBytes**：以 1024 为基数逐级转换，MB 和 KB 保留一位小数，GB 保留两位。
2. **formatDuration**：<1s 显示毫秒，<60s 显示秒（一位小数），>=60s 拆分为 h/m/s 并跳过零值部分。
3. **stripReferenceContent**：使用 `REFERENCE_CONTENT_START/END` 标记的正则匹配移除引用块。
4. **formatResetTime**：`terse` 模式只显示持续时间，`column` 加上具体时间，`full` 包含完整时区信息。

## 内部依赖

无直接内部 UI 模块依赖。

## 外部依赖

| 模块 | 说明 |
|------|------|
| `@google/gemini-cli-core` | `REFERENCE_CONTENT_START`、`REFERENCE_CONTENT_END` 标记常量 |
