# state-manager.ts

> 工具调用的状态管理中心，维护调用的完整生命周期状态并通过 MessageBus 广播变更。

## 概述

`SchedulerStateManager` 是调度器的状态管理核心，维护三个数据结构：活跃调用映射（`activeCalls`）、等待队列（`queue`）和已完成批次（`_completedBatch`）。它实现了严格的状态机转换逻辑，确保工具调用只能按照合法路径在各状态之间转换。每次状态变更都会通过 MessageBus 发布 `TOOL_CALLS_UPDATE` 事件，驱动 UI 层实时更新。

## 架构图

```mermaid
flowchart LR
    subgraph 数据结构
        Q[queue: ToolCall[]] -->|dequeue| A[activeCalls: Map]
        A -->|finalize| C["completedBatch: CompletedToolCall[]"]
    end

    subgraph 状态转换
        V[Validating] --> S[Scheduled]
        V --> AW[AwaitingApproval]
        V --> E1[Error]
        AW --> S
        AW --> CA[Cancelled]
        S --> EX[Executing]
        EX --> SU[Success]
        EX --> E1
        EX --> CA
    end

    subgraph 广播
        A -->|emitUpdate| MB["MessageBus: TOOL_CALLS_UPDATE"]
    end
```

## 主要导出

### `type TerminalCallHandler = (call: CompletedToolCall) => void`
终态调用的回调处理器，用于遥测日志等。

### `class SchedulerStateManager`

**构造函数**
```typescript
constructor(
  messageBus: MessageBus,
  schedulerId: string = ROOT_SCHEDULER_ID,
  onTerminalCall?: TerminalCallHandler,
)
```

#### 队列操作
| 方法 | 说明 |
|---|---|
| `enqueue(calls)` | 批量入队 |
| `dequeue()` | 出队并移入活跃调用 |
| `peekQueue()` | 查看队首（不移除） |
| `addToolCalls(calls)` | `enqueue` 的别名 |

#### 查询方法
| 属性/方法 | 说明 |
|---|---|
| `isActive` | 是否有活跃调用 |
| `allActiveCalls` | 所有活跃调用列表 |
| `activeCallCount` | 活跃调用数量 |
| `queueLength` | 队列长度 |
| `firstActiveCall` | 第一个活跃调用 |
| `getToolCall(callId)` | 按 ID 查找（活跃/队列/已完成） |
| `completedBatch` | 已完成调用列表（只读副本） |
| `getSnapshot()` | 所有调用的完整快照 |

#### 状态更新（重载方法）
**`updateStatus(callId, status, data?)`**
根据目标状态的不同，`data` 参数具有不同的类型约束：
| 目标状态 | data 类型 |
|---|---|
| `Success` | `ToolCallResponseInfo`（必填） |
| `Error` | `ToolCallResponseInfo`（必填） |
| `AwaitingApproval` | 确认详情对象（必填） |
| `Cancelled` | `string` 或 `ToolCallResponseInfo`（必填） |
| `Executing` | `Partial<ExecutingToolCall>`（可选） |
| `Scheduled` / `Validating` | 无 |

#### 其他修改方法
| 方法 | 说明 |
|---|---|
| `finalizeCall(callId)` | 将终态调用从活跃移入已完成批次 |
| `updateArgs(callId, newArgs, newInvocation)` | 更新调用参数（用于用户修改） |
| `setOutcome(callId, outcome)` | 设置确认结果 |
| `replaceActiveCallWithTailCall(callId, nextCall)` | 替换为尾调用 |
| `cancelAllQueued(reason)` | 取消所有排队调用 |
| `clearBatch()` | 清空已完成批次 |

## 核心逻辑

### 状态机转换：`transitionCall`
通过 `switch` 语句实现严格的状态转换，每个分支调用专用的转换辅助方法：
- `toSuccess`: 校验必须有 `tool` 和 `invocation`，计算 `durationMs`
- `toError`: `tool` 可选（工具未找到时无 tool）
- `toAwaitingApproval`: 支持事件驱动（带 `correlationId`）和旧版回调两种格式
- `toScheduled`: 校验必须有 `tool` 和 `invocation`
- `toExecuting`: 合并增量更新（`liveOutput`、`pid`、`progress*`）
- `toCancelled`: 保留等待中的确认详情（如编辑差异）和执行中的实时输出

### 广播机制
`emitUpdate` 在每次状态变更后调用，通过 MessageBus 发布完整快照：
```typescript
void this.messageBus.publish({
  type: MessageBusType.TOOL_CALLS_UPDATE,
  toolCalls: this.getSnapshot(),
  schedulerId: this.schedulerId,
});
```
使用 `void` 表示 fire-and-forget 语义。

### 尾调用支持
`replaceActiveCallWithTailCall` 将当前活跃调用替换为新调用，新调用插入队列头部（`unshift`），确保下一轮循环立即处理。

### 取消时的数据保留
`toCancelled` 方法特别注意保留用户可见的数据：
- 等待审批中的编辑差异（`fileDiff`、`fileName` 等）
- 执行中的实时输出（`liveOutput`）

## 内部依赖

| 模块 | 用途 |
|---|---|
| `./types.js` | 全部工具调用状态类型 |
| `../tools/tools.js` | 工具相关类型 |
| `../confirmation-bus/message-bus.js` | `MessageBus` |
| `../confirmation-bus/types.js` | `MessageBusType`、`SerializableConfirmationDetails` |
| `../utils/tool-utils.js` | `isToolCallResponseInfo` 类型守卫 |

## 外部依赖

无直接外部依赖。
