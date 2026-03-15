# hookRunner.ts

> 实际执行 Hook（命令行和运行时两种类型）的执行引擎。

## 概述

`HookRunner` 负责 Hook 执行管道中的"执行"阶段，支持两种执行模式：**命令行 Hook**（通过子进程 `spawn` 执行 shell 命令）和**运行时 Hook**（直接调用 JavaScript/TypeScript 函数）。它还提供并行和串行两种批量执行策略，串行执行时支持将前一个 Hook 的输出作为下一个 Hook 的输入修改。

**设计动机：** 命令行 Hook 允许用户使用任意语言编写 Hook 脚本，而运行时 Hook 为核心系统和扩展提供高性能的内联 Hook 能力。统一的执行器屏蔽了这两种模式的差异。

**在模块中的角色：** 被 `HookEventHandler` 在执行计划确定后调用，是管道中直接与操作系统交互的层。

## 架构图

```mermaid
flowchart TD
    HEH[HookEventHandler] --> |executeHooksParallel/Sequential| HR[HookRunner]

    HR --> |type === runtime| RH[executeRuntimeHook]
    HR --> |type === command| CH[executeCommandHook]

    RH --> |action() + timeout race| RR[HookExecutionResult]

    CH --> |spawn shell| CP[子进程]
    CP --> |stdin: JSON input| CP
    CP --> |stdout/stderr| Parse[解析输出]
    Parse --> |JSON| JO[结构化 HookOutput]
    Parse --> |纯文本| TO[convertPlainTextToHookOutput]
    JO --> CR[HookExecutionResult]
    TO --> CR

    subgraph 安全检查
        SC[项目 Hook + 不可信文件夹 = 阻止]
    end
    HR --> SC
```

## 主要导出

### `class HookRunner`

#### 构造函数

```typescript
constructor(config: Config)
```

#### 公开方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `executeHook` | `(hookConfig, eventName, input): Promise<HookExecutionResult>` | 执行单个 Hook |
| `executeHooksParallel` | `(configs[], eventName, input, onStart?, onEnd?)` | 并行执行多个 Hook |
| `executeHooksSequential` | `(configs[], eventName, input, onStart?, onEnd?)` | 串行执行多个 Hook（支持输入链式修改） |

## 核心逻辑

### 安全检查

每次执行前，对 `source === ConfigSource.Project` 的 Hook 验证 `config.isTrustedFolder()`。不可信文件夹中的项目 Hook 直接返回失败结果。

### 命令行 Hook 执行（`executeCommandHook`）

1. **Shell 配置**：通过 `getShellConfiguration()` 获取当前 shell 类型（bash/zsh/powershell）
2. **命令扩展**：替换 `$GEMINI_PROJECT_DIR` 和 `$CLAUDE_PROJECT_DIR`（兼容性别名）为当前工作目录，使用 shell 类型对应的转义
3. **环境变量**：
   - 基于 `sanitizeEnvironment()` 清理后的 `process.env`
   - 注入 `GEMINI_PROJECT_DIR` 和 `CLAUDE_PROJECT_DIR`
   - 合并 Hook 自定义环境变量
4. **子进程管理**：
   - 通过 `spawn` 启动，stdin 传入 JSON 格式的 `HookInput`
   - 收集 stdout/stderr
   - 超时处理：默认 60 秒，超时后 SIGTERM -> 5 秒 -> SIGKILL
   - Windows 兼容：使用 `taskkill` 替代信号
5. **输出解析**：
   - 优先尝试 JSON 解析 stdout（支持双重 JSON 编码）
   - 回退到 `convertPlainTextToHookOutput()`

### 纯文本输出转换

| 退出码 | 结果 |
|--------|------|
| 0 | `decision: 'allow'`, `systemMessage: text` |
| 1 | `decision: 'allow'`, `systemMessage: 'Warning: text'` |
| 其他 | `decision: 'deny'`, `reason: text` |

### 运行时 Hook 执行（`executeRuntimeHook`）

1. 使用 `AbortController` 支持取消
2. `Promise.race` 竞争执行和超时
3. 超时或错误时自动 abort
4. 返回值为 `null`/`undefined` 视为无输出成功

### 串行执行的输入链式修改（`applyHookOutputToInput`）

在串行执行中，前一个 Hook 的输出可以修改后续 Hook 的输入：

| 事件类型 | 修改行为 |
|----------|----------|
| BeforeAgent | `additionalContext` 追加到 `prompt` |
| BeforeModel | `llm_request` 合并到输入的 `llm_request` |
| BeforeTool | `tool_input` 合并到输入的 `tool_input` |
| 其他 | 无修改 |

### EPIPE 容错

stdin 写入时对 EPIPE 错误做了两层处理（异步 `error` 事件和同步 `try-catch`），避免子进程提前退出导致的崩溃。

## 内部依赖

| 模块 | 说明 |
|------|------|
| `./types.js` | HookConfig、HookInput、HookOutput 等 |
| `./hookTranslator.js` | LLMRequest 类型 |
| `../config/config.js` | Config 类型 |
| `../utils/debugLogger.js` | 调试日志 |
| `../utils/shell-utils.js` | Shell 配置和参数转义 |
| `../services/environmentSanitization.js` | 环境变量清理 |

## 外部依赖

| 包 | 说明 |
|------|------|
| `node:child_process` | `spawn`、`execSync` |
