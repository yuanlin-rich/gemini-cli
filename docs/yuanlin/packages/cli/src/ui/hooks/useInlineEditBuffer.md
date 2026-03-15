# useInlineEditBuffer.ts

> 提供行内编辑缓冲区的状态管理，支持光标移动、字符插入/删除和闪烁光标

## 概述

`useInlineEditBuffer` 是一个 React Hook，为设置页面等场景中的行内文本编辑提供完整的编辑缓冲区功能。它使用 `useReducer` 管理编辑状态，支持：

- 开始/提交编辑
- 光标左/右移动、Home/End 跳转
- 退格删除（DELETE_LEFT）和 Delete 键删除（DELETE_RIGHT）
- 字符插入（支持数字类型验证）
- 闪烁光标动画（500ms 间隔）

所有字符串操作使用 `cpLen`/`cpSlice` 函数处理 Unicode 码点，确保多字节字符的正确处理。

## 架构图（mermaid）

```mermaid
graph TD
    A[useInlineEditBuffer] --> B[useReducer - editBufferReducer]
    A --> C[useState - cursorVisible]

    B --> D[EditBufferState]
    D --> E[editingKey: string | null]
    D --> F[buffer: string]
    D --> G[cursorPos: number]

    H[startEditing] --> I[dispatch START_EDIT]
    J[commitEdit] --> K[onCommit callback]
    J --> L[dispatch COMMIT_EDIT]

    M[useEffect - 光标闪烁] --> N[setInterval 500ms]
    N --> O[toggleCursorVisible]

    subgraph Actions
        P[MOVE_LEFT / MOVE_RIGHT]
        Q[HOME / END]
        R[DELETE_LEFT / DELETE_RIGHT]
        S[INSERT_CHAR]
    end
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `EditBufferState` | `interface` | 编辑状态：editingKey, buffer, cursorPos |
| `EditBufferAction` | `type` | 所有编辑操作的联合类型 |
| `UseEditBufferProps` | `interface` | `{ onCommit: (key, value) => void }` |
| `useInlineEditBuffer` | `(props) => { editState, editDispatch, startEditing, commitEdit, cursorVisible }` | 返回状态和操作函数 |

## 核心逻辑

1. `editBufferReducer` 处理 8 种操作类型，所有字符串操作基于码点（codepoint）。
2. `INSERT_CHAR`：数字类型仅允许 `[0-9\-+.]`；文本类型需 `charCodeAt(0) >= 32` 且经过 `stripUnsafeCharacters` 过滤。
3. 光标闪烁：当 `editingKey` 存在时启动 500ms 间隔定时器，每次文本或光标变化时重置为可见。
4. `commitEdit` 调用 `onCommit(editingKey, buffer)` 并 dispatch `COMMIT_EDIT` 重置状态。

## 内部依赖

| 依赖 | 路径 | 说明 |
|------|------|------|
| `cpSlice`, `cpLen`, `stripUnsafeCharacters` | `../utils/textUtils.js` | Unicode 感知的字符串操作 |

## 外部依赖

| 依赖 | 说明 |
|------|------|
| `react` | `useReducer`, `useCallback`, `useEffect`, `useState` |
