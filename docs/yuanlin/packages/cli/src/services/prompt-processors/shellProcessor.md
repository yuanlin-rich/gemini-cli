# shellProcessor.ts

> 处理提示词模板中的 Shell 命令注入（`!{...}`）和上下文感知的参数替换（`{{args}}`）。

## 概述

`ShellProcessor` 是 `IPromptProcessor` 接口中最复杂的实现，负责处理自定义命令提示词模板中的两种动态内容：

1. **Shell 命令注入** (`!{command}`) -- 解析并执行嵌入的 shell 命令，将输出内联到提示词中。
2. **参数替换** (`{{args}}`) -- 在 shell 注入块内部使用 shell 转义后的参数，在外部使用原始参数。

该处理器集成了完整的**安全策略检查**流程：在执行任何 shell 命令之前，通过策略引擎（`PolicyEngine`）验证命令权限。被拒绝的命令直接抛出错误，需要用户确认的命令抛出 `ConfirmationRequiredError`，由上层 UI 处理确认流程。

## 架构图（mermaid）

```mermaid
flowchart TD
    A["ShellProcessor.process(prompt, context)"] --> B["flatMapTextParts(prompt, processString)"]
    B --> C["processString(text, context)"]

    C --> D{文本包含 '!\{' ?}
    D -->|否| E["替换 {{args}} 为原始参数"]
    D -->|是| F["extractInjections(text, '!\{')"]

    F --> G["解析 {{args}}: shell 转义"]
    G --> H[resolvedInjections]

    H --> I{安全策略检查}
    I --> J{遍历每个命令}
    J --> K["PolicyEngine.check(command)"]

    K --> L{决策结果}
    L -->|DENY| M[抛出 Error: 被策略阻止]
    L -->|ASK_USER| N[加入待确认集合]
    L -->|ALLOW| O[跳过]

    N --> P{有待确认命令?}
    P -->|是| Q[抛出 ConfirmationRequiredError]
    P -->|否| R[执行 Shell 命令]

    R --> S[遍历注入点]
    S --> T["注入前文本: {{args}} -> 原始参数"]
    S --> U["ShellExecutionService.execute(command)"]
    U --> V[追加命令输出]
    V --> W{退出状态}
    W -->|成功| X[仅追加输出]
    W -->|非零退出码| Y[追加退出码信息]
    W -->|被中断| Z[追加中断信息]
    W -->|信号终止| AA[追加信号信息]

    S --> AB["注入后文本: {{args}} -> 原始参数"]
    AB --> AC["返回 [{ text: processedPrompt }]"]
```

## 主要导出

| 导出名称 | 类型 | 说明 |
|---|---|---|
| `ConfirmationRequiredError` | 类 | 自定义错误类型，携带需要用户确认的命令列表 |
| `ShellProcessor` | 类 | 实现 `IPromptProcessor`，处理 shell 注入和参数替换 |

## 核心逻辑

### `ConfirmationRequiredError`

```typescript
class ConfirmationRequiredError extends Error {
  constructor(message: string, public commandsToConfirm: string[])
}
```

自定义错误类型，当策略引擎返回 `ASK_USER` 决策时抛出。`commandsToConfirm` 包含所有需要用户确认的 shell 命令字符串，由 `FileCommandLoader` 的 action 闭包捕获并转换为 `confirm_shell_commands` 类型的返回值。

### `ResolvedShellInjection` 接口

扩展 `Injection` 接口，增加 `resolvedCommand?: string` 字段，存储 `{{args}}` 替换为 shell 转义参数后的最终命令。

### `processString(prompt, context): Promise<PromptPipelineContent>`

核心处理方法，执行以下步骤：

#### 1. 快速路径
若文本不含 `!{`，直接将所有 `{{args}}` 替换为原始参数并返回。

#### 2. 安全配置检查
若 `config` 不可用，抛出错误阻止执行。

#### 3. 注入解析与参数替换
- 调用 `extractInjections` 提取所有 `!{...}` 块。
- 获取 shell 配置并对用户参数进行 shell 转义。
- 将每个注入块中的 `{{args}}` 替换为转义后的参数，生成 `resolvedCommand`。

#### 4. 安全策略批量检查
对每个 `resolvedCommand`：
- 先检查 `session.sessionShellAllowlist`，已允许的命令跳过策略检查。
- 调用 `config.getPolicyEngine().check({ name: 'run_shell_command', args: { command } })` 获取策略决策。
- `DENY` -> 直接抛出 `Error`。
- `ASK_USER` -> 收集到 `commandsToConfirm` 集合。
- 所有命令检查完毕后，若有待确认命令，抛出 `ConfirmationRequiredError`。

#### 5. 命令执行与结果拼接
逐个处理注入点：
- **注入前文本**：`{{args}}` 替换为**原始**参数（非转义）。
- **命令执行**：通过 `ShellExecutionService.execute()` 执行，配置包含主题颜色。
- **输出追加**：将命令的 stdout/stderr 输出追加到结果字符串。
- **状态附注**：
  - 命令被中断：追加 `[Shell command '...' aborted]`。
  - 非零退出码：追加 `[Shell command '...' exited with code X]`。
  - 信号终止：追加 `[Shell command '...' terminated by signal X]`。
  - 启动失败：抛出 `Error`。
- **注入后文本**：`{{args}}` 同样替换为**原始**参数。

### `{{args}}` 的双重语义

| 上下文 | 替换内容 | 原因 |
|---|---|---|
| `!{...}` 内部 | shell 转义后的参数 | 防止命令注入攻击 |
| `!{...}` 外部 | 原始参数 | 直接作为提示词文本传递给模型 |

## 内部依赖

| 模块 | 说明 |
|---|---|
| `./types.js` | `SHELL_INJECTION_TRIGGER`、`SHORTHAND_ARGS_PLACEHOLDER`、`IPromptProcessor`、`PromptPipelineContent` |
| `./injectionParser.js` | `extractInjections` 函数和 `Injection` 接口 |
| `../../ui/themes/theme-manager.js` | `themeManager`，获取当前主题颜色用于 shell 输出渲染 |

## 外部依赖

| 包名 | 说明 |
|---|---|
| `@google/gemini-cli-core` | `escapeShellArg`（shell 参数转义）、`getShellConfiguration`（shell 配置）、`ShellExecutionService`（shell 命令执行服务）、`flatMapTextParts`（多模态内容遍历）、`PolicyDecision`（策略决策枚举） |
