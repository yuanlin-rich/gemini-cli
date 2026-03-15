# trackerService.ts

> 任务追踪服务，提供基于文件系统的任务 CRUD 操作，支持依赖关系管理和循环依赖检测。

## 概述

`TrackerService` 是一个轻量级的任务追踪系统，将每个任务作为独立的 JSON 文件存储在磁盘上。它支持任务的创建、读取、更新和列表操作，并提供完整的依赖关系管理能力——包括父任务验证、关闭前依赖检查（所有依赖必须已关闭）和循环依赖检测。每个任务拥有 6 字符的十六进制 ID，支持 `epic`、`task`、`bug` 三种类型和 `open`、`in_progress`、`blocked`、`closed` 四种状态。该模块在架构中为 AI 代理提供任务规划和进度跟踪的持久化基础设施。

## 架构图

```mermaid
graph TD
    A[Task Tracker 工具] -->|createTask / updateTask / listTasks| B[TrackerService]
    B --> C[tasksDir 目录]
    C --> D[{id}.json 任务文件]
    B --> E[Zod 验证<br>TrackerTaskSchema]
    B --> F[依赖管理]
    F --> G[父任务存在性检查]
    F --> H[关闭前依赖状态检查]
    F --> I[循环依赖 DFS 检测]
```

## 主要导出

### `class TrackerService`
- **构造函数**: `constructor(trackerDir: string)` - `trackerDir` 为任务文件存储目录。
- `createTask(taskData: Omit<TrackerTask, 'id'>): Promise<TrackerTask>` - 创建任务，自动生成 6 字符 ID，验证父任务存在性。
- `getTask(id: string): Promise<TrackerTask | null>` - 按 ID 读取任务。
- `listTasks(): Promise<TrackerTask[]>` - 列出所有任务。
- `updateTask(id: string, updates: Partial<TrackerTask>): Promise<TrackerTask>` - 更新任务，执行依赖验证。

## 核心逻辑

1. **惰性目录创建**: 首次操作时确保 `tasksDir` 目录存在。
2. **ID 生成**: 使用 `crypto.randomBytes(3)` 生成 6 字符十六进制 ID。
3. **Zod 验证**: 创建和更新时都通过 `TrackerTaskSchema` 进行数据验证。
4. **关闭验证 (`validateCanClose`)**: 关闭任务前检查所有依赖任务是否已关闭，未关闭则抛出错误。
5. **循环依赖检测 (`validateNoCircularDependencies`)**: 修改依赖关系时，使用 DFS（深度优先搜索）+ 递归栈检测循环，发现循环则抛出错误。
6. **性能优化**: 关闭或修改依赖时一次性加载所有任务到 Map，避免多次磁盘读取。
7. **错误处理**: 文件读取失败时通过 `coreEvents.emitFeedback` 发送警告反馈，`ENOENT` 错误返回 `null`。

## 内部依赖

| 模块 | 用途 |
|------|------|
| `./trackerTypes.js` | `TrackerTaskSchema`, `TaskStatus`, `TrackerTask` 类型 |
| `../utils/debugLogger.js` | 调试日志 |
| `../utils/events.js` | `coreEvents` 反馈事件 |

## 外部依赖

| 包 | 用途 |
|----|------|
| `node:fs/promises` | 异步文件操作 |
| `node:path` | 路径处理 |
| `node:crypto` | 随机 ID 生成 |
| `zod` | 数据验证（通过 `TrackerTaskSchema`） |
