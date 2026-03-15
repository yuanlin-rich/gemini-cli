# index.ts

> 包的根入口文件，负责重导出文件系统测试辅助工具。

## 概述

`index.ts` 位于 `packages/test-utils/` 的根目录，是整个 `test-utils` 包的**主入口点**（对应 `package.json` 中的 `main` / `exports` 字段）。它本身不包含任何业务逻辑，仅通过 `export *` 将 `src/file-system-test-helpers.js` 中的所有导出重新暴露给外部消费者。

设计动机：提供一个简洁的顶层入口，让依赖此包的项目可以直接从包名导入所需工具，而无需了解内部 `src/` 目录结构。

**注意**：此文件只导出了 `file-system-test-helpers`，而 `src/index.ts` 则额外导出了 `test-rig` 和 `mock-utils`。这意味着从包根路径导入与从 `src/` 子路径导入所获得的 API 集合可能不同。

## 架构图

```mermaid
graph LR
    A["packages/test-utils/index.ts"] -->|"export *"| B["src/file-system-test-helpers.ts"]
```

## 主要导出

此文件本身不定义任何新导出，全部来自重导出：

| 来源模块 | 导出内容 |
|---|---|
| `./src/file-system-test-helpers.js` | `FileSystemStructure` (类型), `createTmpDir`, `cleanupTmpDir` |

## 核心逻辑

无自身逻辑。仅包含一行重导出语句：

```typescript
export * from './src/file-system-test-helpers.js';
```

## 内部依赖

| 模块 | 说明 |
|---|---|
| `./src/file-system-test-helpers.js` | 文件系统测试辅助工具（临时目录创建与清理） |

## 外部依赖

无。
