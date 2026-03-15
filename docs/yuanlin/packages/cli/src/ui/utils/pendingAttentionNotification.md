# pendingAttentionNotification.ts

> 综合判断当前是否有需要用户注意的待处理请求，并生成通知事件

## 概述

本文件实现了统一的"需要关注"通知逻辑。它检查多种待处理状态（工具确认、命令确认、认证同意、文件系统权限、扩展更新、循环检测），按优先级返回第一个需要用户注意的通知。该通知用于驱动终端标题栏闪烁、桌面通知等关注提醒机制。

## 架构图（mermaid）

```mermaid
flowchart TD
    A[getPendingAttentionNotification] --> B{有工具确认?}
    B -- 是 --> C{类型是 ask_user?}
    C -- 是 --> D[返回"Agent 需要回答"]
    C -- 否 --> E[返回"审批工具操作"]
    B -- 否 --> F{命令确认?}
    F -- 是 --> G[返回"命令等待确认"]
    F -- 否 --> H{认证同意?}
    H -- 是 --> I[返回"认证等待确认"]
    H -- 否 --> J{文件权限?}
    J -- 是 --> K[返回"文件系统权限"]
    J -- 否 --> L{扩展更新?}
    L -- 是 --> M[返回"扩展更新确认"]
    L -- 否 --> N{循环检测?}
    N -- 是 --> O[返回"循环检测确认"]
    N -- 否 --> P[返回 null]
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `PendingAttentionNotification` | interface | 通知对象，包含 key（唯一标识）和 event（通知事件） |
| `getPendingAttentionNotification` | function | 返回当前最高优先级的待关注通知，或 null |

## 核心逻辑

1. **优先级顺序**：工具确认 > 命令确认 > 认证同意 > 文件权限 > 扩展更新 > 循环检测。
2. **唯一 key 生成**：每种通知类型使用不同前缀 + 具体标识符（如 callId、文件列表）生成唯一 key，供消费者去重。
3. **ask_user 特殊处理**：当工具确认类型为 `ask_user` 时，提取第一个问题的 header 作为通知详情。

## 内部依赖

| 模块 | 说明 |
|------|------|
| `../types.js` | `ConfirmationRequest`、`HistoryItemWithoutId`、`PermissionConfirmationRequest` |
| `../../utils/terminalNotifications.js` | `RunEventNotificationEvent` 类型 |
| `./confirmingTool.js` | `getConfirmingToolState` 函数 |

## 外部依赖

| 模块 | 说明 |
|------|------|
| `react` | `ReactNode` 类型（用于 key 生成） |
