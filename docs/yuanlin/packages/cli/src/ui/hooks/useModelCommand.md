# useModelCommand.ts

> 管理模型选择对话框的打开/关闭状态

## 概述

`useModelCommand` 是一个简单的 React Hook，提供模型选择对话框的状态管理。它仅维护一个布尔状态 `isModelDialogOpen`，以及两个用 `useCallback` 包装的稳定操作函数。

## 架构图（mermaid）

```mermaid
graph TD
    A[useModelCommand] --> B[isModelDialogOpen 状态]
    C[openModelDialog] --> D[set true]
    E[closeModelDialog] --> F[set false]
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `useModelCommand` | `() => { isModelDialogOpen, openModelDialog, closeModelDialog }` | 返回对话框状态和控制函数 |

## 核心逻辑

1. `useState(false)` 初始化对话框关闭状态。
2. `openModelDialog` 和 `closeModelDialog` 使用空依赖的 `useCallback` 保持引用稳定。

## 内部依赖

无。

## 外部依赖

| 依赖 | 说明 |
|------|------|
| `react` | `useState`, `useCallback` |
