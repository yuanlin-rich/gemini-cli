# write-todos.ts

> 待办事项列表管理工具，用于跟踪复杂任务的子任务进度。

## 概述
`WriteTodosTool` 实现了 `write_todos` 工具，允许 AI 创建和更新结构化的待办事项列表，帮助跟踪复杂任务的执行进度。每次调用会完全替换现有列表。支持四种状态：`pending / in_progress / completed / cancelled`，且同一时间只允许一个任务处于 `in_progress` 状态。

## 架构图
```mermaid
graph LR
    A[WriteTodosTool] -->|build| B[WriteTodosToolInvocation]
    B -->|execute| C[格式化 TodoList]
    C --> D[returnDisplay: TodoList 对象]
```

## 主要导出

### 接口
- `WriteTodosToolParams` - 参数：`todos`(必选, Todo 数组)

### 类
- `WriteTodosTool extends BaseDeclarativeTool` - 待办列表工具，Kind 为 Other

## 核心逻辑
1. 参数验证：检查数组格式、每项必须有非空 description 和有效 status
2. 业务规则：最多一个 `in_progress` 状态的任务
3. 返回结构化的 `TodoList` 对象供 UI 渲染

## 内部依赖
- `./tools.ts` - `BaseDeclarativeTool`, `Todo`, `ToolResult`
- `./tool-names.ts` - `WRITE_TODOS_TOOL_NAME`
- `./definitions/coreTools.ts`, `./definitions/resolver.ts`

## 外部依赖
无
