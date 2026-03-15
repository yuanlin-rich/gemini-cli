# useConfirmingTool.ts

> 从待处理历史项队列中选取第一个需要用户确认的工具调用

## 概述

`useConfirmingTool` 是一个轻量级 React Hook，作为确认队列的"头部选择器"。它从 UI 状态上下文中获取 `pendingHistoryItems`（待处理的历史项列表），然后通过 `getConfirmingToolState` 工具函数提取队列中第一个需要用户确认的工具调用状态。

此 Hook 使用 `useMemo` 优化性能，仅在 `pendingHistoryItems` 变化时重新计算。

## 架构图（mermaid）

```mermaid
graph TD
    A[useConfirmingTool] --> B[useUIState - pendingHistoryItems]
    B --> C[useMemo]
    C --> D[getConfirmingToolState]
    D --> E[ConfirmingToolState | null]
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `ConfirmingToolState` | `type` (re-export) | 确认中工具的状态类型 |
| `useConfirmingTool` | `() => ConfirmingToolState \| null` | 返回队首的确认工具状态，无则返回 null |

## 核心逻辑

1. 通过 `useUIState()` 获取 `pendingHistoryItems`，这包含来自 Gemini 响应和 Slash 命令的工具调用。
2. `useMemo` 包裹 `getConfirmingToolState(pendingHistoryItems)` 调用，仅在数据变化时重新计算。
3. 返回第一个处于待确认状态的工具，若无则返回 `null`。

## 内部依赖

| 依赖 | 路径 | 说明 |
|------|------|------|
| `useUIState` | `../contexts/UIStateContext.js` | UI 全局状态上下文 |
| `getConfirmingToolState` | `../utils/confirmingTool.js` | 从历史项中提取确认状态的工具函数 |

## 外部依赖

| 依赖 | 说明 |
|------|------|
| `react` | `useMemo` |
