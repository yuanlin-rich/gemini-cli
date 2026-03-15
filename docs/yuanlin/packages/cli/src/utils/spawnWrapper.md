# spawnWrapper.ts

> 对 `child_process.spawn` 的简单封装，便于测试时 mock

## 概述

`spawnWrapper.ts` 是一个极简模块，仅将 `node:child_process` 的 `spawn` 函数重新导出为 `spawnWrapper`。这种间接引用的设计模式使得在单元测试中可以方便地对子进程创建进行 mock，而无需直接修补 Node.js 内置模块。

## 架构图（mermaid）

```mermaid
flowchart LR
    A[node:child_process.spawn] --> B[spawnWrapper]
    B --> C[使用方引用]
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `spawnWrapper` | `typeof spawn` | `child_process.spawn` 的重导出，用于方便测试 mock |

## 核心逻辑

直接赋值导出：`export const spawnWrapper = spawn;`

## 内部依赖

无。

## 外部依赖

| 包名 | 用途 |
|------|------|
| `node:child_process` | `spawn` - 创建子进程 |
