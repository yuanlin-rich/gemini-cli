# protocol.ts

> 定义安全检查器与 CLI 之间通信的标准协议数据结构与枚举。

## 概述

`protocol.ts` 是整个 safety 模块的类型基石，定义了安全检查输入（`SafetyCheckInput`）、输出（`SafetyCheckResult`）以及决策枚举（`SafetyCheckDecision`）。所有内置和外部安全检查器都必须遵循此协议进行数据交换。文件还定义了对话轮次结构 `ConversationTurn`，用于向检查器提供语义上下文。此协议采用显式版本号（`protocolVersion`），便于未来引入不兼容变更时保持向后兼容。

## 架构图

```mermaid
classDiagram
    class ConversationTurn {
        +user: { text: string }
        +model: { text?: string, toolCalls?: FunctionCall[] }
    }

    class SafetyCheckInput {
        +protocolVersion: "1.0.0"
        +toolCall: FunctionCall
        +context: ContextObject
        +config?: unknown
    }

    class SafetyCheckDecision {
        <<enumeration>>
        ALLOW = "allow"
        DENY = "deny"
        ASK_USER = "ask_user"
    }

    class SafetyCheckResult {
        +decision: SafetyCheckDecision
        +reason?: string
        +error?: string
    }

    SafetyCheckInput --> ConversationTurn : context.history.turns
    SafetyCheckResult --> SafetyCheckDecision : decision
```

## 主要导出

### `interface ConversationTurn`
表示用户与模型之间的单轮对话，包含用户文本和模型回复（可含工具调用）。为安全检查器提供工具调用发生的语义背景。

### `interface SafetyCheckInput`
通过 stdin 传递给安全检查器进程的完整输入数据结构：
- `protocolVersion`: 固定为 `"1.0.0"`，用于版本管理
- `toolCall`: 待验证的具体工具调用（`FunctionCall`）
- `context.environment`: 当前工作目录和工作区列表
- `context.history`: 对话历史轮次
- `config`: 可选的检查器配置参数

### `enum SafetyCheckDecision`
安全检查器的三种决策：
| 值 | 含义 |
|---|---|
| `ALLOW` | 允许工具调用执行 |
| `DENY` | 拒绝工具调用 |
| `ASK_USER` | 需要用户确认 |

### `type SafetyCheckResult`
安全检查器通过 stdout 返回的判定结果，是一个可辨识联合类型：
- `ALLOW` 时 `reason` 可选
- `DENY` 和 `ASK_USER` 时 `reason` 必填

## 核心逻辑

此文件为纯类型定义文件，不含运行时逻辑。其核心设计要点：

1. **协议版本化**：`protocolVersion` 字段确保检查器与 CLI 之间的接口契约明确，便于未来升级。
2. **上下文分层**：`context` 对象将数据按类别（环境、历史）分组，避免扁平化结构膨胀。
3. **可辨识联合**：`SafetyCheckResult` 根据 `decision` 字段区分不同分支，`DENY`/`ASK_USER` 强制要求提供 `reason`。

## 内部依赖

无内部依赖。

## 外部依赖

| 包 | 用途 |
|---|---|
| `@google/genai` | 使用 `FunctionCall` 类型表示工具调用 |
