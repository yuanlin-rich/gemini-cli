# ideContext.ts

> IDE 上下文状态的响应式存储，管理打开文件列表、活动文件和选中文本

## 概述

本文件实现了 `IdeContextStore` 类，一个具有发布-订阅能力的状态容器，用于存储从 IDE 伴侣扩展推送过来的工作区上下文信息。该存储在 `IdeClient` 收到 `ide/contextUpdate` 通知时被更新，供 CLI 的其他组件（如提示构建器）读取当前 IDE 状态。

文件同时导出了全局共享的 `ideContextStore` 单例实例。

## 架构图

```mermaid
graph LR
    IDEServer["IDE 伴侣扩展"] -->|contextUpdate 通知| IdeClient["IdeClient"]
    IdeClient -->|set()| IdeContextStore["IdeContextStore"]
    IdeContextStore -->|subscribe()| PromptBuilder["提示构建器"]
    IdeContextStore -->|get()| OtherModules["其他模块"]
    IdeClient -->|clear()| IdeContextStore
```

## 主要导出

### `IdeContextStore` (类)

| 方法 | 签名 | 用途 |
|------|------|------|
| `set` | `set(newIdeContext: IdeContext): void` | 设置新的 IDE 上下文并通知所有订阅者 |
| `clear` | `clear(): void` | 清除上下文（断开连接时调用） |
| `get` | `get(): IdeContext \| undefined` | 获取当前上下文 |
| `subscribe` | `subscribe(subscriber): () => void` | 注册变化监听器，返回取消订阅函数 |

### `ideContextStore`

```typescript
export const ideContextStore = new IdeContextStore();
```

应用全局共享的 `IdeContextStore` 单例。

## 核心逻辑

`set()` 方法在存储上下文前执行以下数据清洗：

1. **按时间排序**: 将 `openFiles` 按 `timestamp` 降序排列（最新文件在前）
2. **活动文件唯一性**: 只有索引 0 的文件可以是活动文件。如果最新文件不是活动文件，则清除所有文件的 `isActive`、`cursor`、`selectedText`
3. **选中文本截断**: 活动文件的 `selectedText` 超过 `IDE_MAX_SELECTED_TEXT_LENGTH`（16 KiB）时截断并追加 `... [TRUNCATED]`
4. **文件数量限制**: 超过 `IDE_MAX_OPEN_FILES`（10）的文件被丢弃

## 内部依赖

| 模块 | 用途 |
|------|------|
| `constants.ts` | `IDE_MAX_OPEN_FILES`, `IDE_MAX_SELECTED_TEXT_LENGTH` |
| `types.ts` | `IdeContext` 类型 |

## 外部依赖

无。
