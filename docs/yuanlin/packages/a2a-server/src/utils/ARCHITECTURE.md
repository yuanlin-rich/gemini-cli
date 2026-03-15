# a2a-server/src/utils 架构

> 工具函数模块，提供日志、错误处理和测试辅助功能。

## 概述

`utils` 目录提供 A2A 服务器的公共工具函数。`logger.ts` 基于 winston 创建格式化的日志记录器，在整个服务端代码中广泛使用。`executor_utils.ts` 提供将任务状态设为失败的辅助函数。`testing_utils.ts` 提供测试专用的 Mock 配置创建和断言辅助函数。

## 关键文件

| 文件 | 功能 |
|------|------|
| `logger.ts` | 基于 winston 创建日志实例，输出格式为 `[LEVEL] timestamp -- message`，支持附加元数据的 JSON 输出 |
| `executor_utils.ts` | `pushTaskStateFailed()` 函数：向 EventBus 发布 failed 状态更新事件，用于任务创建失败等场景 |
| `testing_utils.ts` | 测试工具：`createMockConfig()` 创建完整的 Mock Config 对象；`createStreamMessageRequest()` 构建测试用 JSON-RPC 请求；`assertUniqueFinalEventIsLast()` 和 `assertTaskCreationAndWorkingStatus()` 提供事件流断言 |

## 内部依赖

- `../types.ts` - CoderAgentEvent、StateChange（executor_utils 使用）

## 外部依赖

| 包名 | 用途 |
|------|------|
| `winston` | 日志框架 |
| `@a2a-js/sdk` | Message 类型（executor_utils） |
| `@a2a-js/sdk/server` | ExecutionEventBus 类型 |
| `uuid` | UUID 生成 |
| `@google/gemini-cli-core` | Config、GeminiClient 等（testing_utils） |
| `vitest` | 测试框架 Mock 工具（testing_utils） |
