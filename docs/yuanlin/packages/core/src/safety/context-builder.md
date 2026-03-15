# context-builder.ts

> 为安全检查器构建上下文对象，提供环境信息和对话历史，同时过滤敏感数据。

## 概述

`ContextBuilder` 负责将 CLI 的内部状态（`AgentLoopContext`）转换为安全协议所需的标准化上下文结构。它提供两种构建模式：完整上下文（包含环境和对话历史）和最小上下文（仅包含检查器声明需要的字段）。对话历史通过 Google GenAI 的 `Content[]` 格式转换为协议规定的 `ConversationTurn[]` 格式。

## 架构图

```mermaid
flowchart TD
    A[AgentLoopContext] --> B[ContextBuilder]

    B --> C[buildFullContext]
    B --> D[buildMinimalContext]

    C --> E[获取 geminiClient 历史]
    E --> F[convertHistoryToTurns]
    F --> G[ConversationTurn[]]

    C --> H[获取 cwd & workspaces]
    C --> I[检查 pending question]

    D --> C
    D --> J[按 requiredKeys 过滤]

    C --> K["SafetyCheckInput['context']"]
    D --> K
```

## 主要导出

### `class ContextBuilder`

**构造函数**
```typescript
constructor(private readonly context: AgentLoopContext)
```

**`buildFullContext(): SafetyCheckInput['context']`**
构建完整上下文，包含：
- `environment.cwd`: 当前工作目录（`process.cwd()`）
- `environment.workspaces`: 工作区目录列表
- `history.turns`: 从 Gemini 客户端历史转换而来的对话轮次

**`buildMinimalContext(requiredKeys): SafetyCheckInput['context']`**
先构建完整上下文，再根据 `requiredKeys` 数组过滤出检查器所需的字段子集，减少不必要的数据传递。

## 核心逻辑

### 对话历史转换：`convertHistoryToTurns`
将 Google GenAI 的 `Content[]` 格式转换为协议的 `ConversationTurn[]`：

1. 遍历 `Content` 数组，按角色（`user`/`model`）配对
2. 用户消息：提取所有 `parts` 的 `text` 拼接
3. 模型回复：分别提取文本部分和 `functionCall` 部分
4. 配对逻辑：
   - 遇到 `user` 时暂存，等待下一个 `model` 配对
   - 遇到 `model` 时与暂存的 `user` 组成一轮
   - 连续两个 `user` 时，前一个以空模型回复结束
   - 无前置 `user` 的 `model` 以空用户文本填充

### Pending Question 处理
如果对话历史为空但存在待处理的用户问题（`config.getQuestion()`），会将其作为第一轮对话添加，确保检查器在对话开始前也能获取用户意图。

## 内部依赖

| 模块 | 用途 |
|---|---|
| `./protocol.js` | `SafetyCheckInput`、`ConversationTurn` 类型 |
| `../utils/debugLogger.js` | 调试日志输出 |
| `../config/agent-loop-context.js` | `AgentLoopContext` 类型 |

## 外部依赖

| 包 | 用途 |
|---|---|
| `@google/genai` | `Content`、`FunctionCall` 类型 |
