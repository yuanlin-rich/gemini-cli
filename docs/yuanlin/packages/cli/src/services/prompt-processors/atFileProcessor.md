# atFileProcessor.ts

> 处理提示词模板中的 `@{path}` 文件内容注入语法，将文件内容内联替换到提示词中。

## 概述

`AtFileProcessor` 是 `IPromptProcessor` 接口的实现，负责解析提示词模板中的 `@{...}` 语法，读取指定路径的文件内容，并将其作为多模态部分注入到提示词中。它在提示词处理管道中被设计为**最先执行的处理器**，这是出于安全考虑 -- 确保文件内容在 shell 命令注入之前已被安全处理，避免动态生成恶意路径的攻击向量。

该处理器尊重 `.gitignore` 和 `.geminiignore` 配置，被忽略的文件不会被注入，并通过 UI 向用户发出通知。

## 架构图（mermaid）

```mermaid
flowchart TD
    A["AtFileProcessor.process(input, context)"] --> B["flatMapTextParts(input, callback)"]
    B --> C{文本包含 '@\{' ?}
    C -->|否| D["返回 [{ text }]"]
    C -->|是| E["extractInjections(text, '@{', commandName)"]
    E --> F{有注入点?}
    F -->|否| D
    F -->|是| G[遍历每个注入点]

    G --> H[提取前缀文本]
    G --> I["readPathFromWorkspace(path, config)"]

    I --> J{读取成功?}
    J -->|成功且有内容| K[追加文件内容部分]
    J -->|成功但被忽略| L[UI 通知: 文件被忽略]
    J -->|失败| M[UI 显示错误]
    M --> N[保留原始占位符]

    G --> O[追加后缀文本]
    K --> P[返回替换后的 PromptPipelineContent]
    L --> P
    N --> P
```

## 主要导出

| 导出名称 | 类型 | 说明 |
|---|---|---|
| `AtFileProcessor` | 类 | 实现 `IPromptProcessor`，处理 `@{path}` 文件内容注入 |

## 核心逻辑

### 构造函数

```typescript
constructor(private readonly commandName?: string)
```

- `commandName`: 可选的命令名称，用于在 `extractInjections` 解析错误时提供上下文信息。

### `process(input, context): Promise<PromptPipelineContent>`

使用 `flatMapTextParts` 对输入中的每个文本部分进行处理：

1. **快速跳过**：若文本不含 `@{` 触发器，直接返回原始文本部分。
2. **解析注入点**：调用 `extractInjections(text, AT_FILE_INJECTION_TRIGGER, commandName)` 提取所有 `@{...}` 块。
3. **逐一处理注入点**：
   - **提取前缀**：将注入点之前的文本作为独立的文本部分追加到输出。
   - **读取文件**：调用 `readPathFromWorkspace(pathStr, config)` 读取文件内容。
   - **成功处理**：
     - 若返回内容非空，将文件内容部分（可能是多模态的）追加到输出。
     - 若返回空数组（文件被 `.gitignore` / `.geminiignore` 忽略），通过 `context.ui.addItem` 发送 INFO 级别通知。
   - **失败处理**：
     - 记录调试日志。
     - 通过 `context.ui.addItem` 发送 ERROR 级别通知。
     - 将原始 `@{...}` 占位符文本保留在输出中，确保提示词不会因错误而断裂。
4. **追加后缀**：将最后一个注入点之后的文本追加到输出。

### 安全设计

- **管道顺序优先**：`FileCommandLoader` 将此处理器注册为管道第一个处理器，确保在 `ShellProcessor` 之前运行。这防止了通过 shell 命令动态生成恶意 `@{...}` 路径的攻击。
- **忽略规则尊重**：通过 `readPathFromWorkspace` 自动遵守 `.gitignore` 和 `.geminiignore` 规则。
- **错误隔离**：单个文件读取失败不影响其他注入点和整体提示词。

## 内部依赖

| 模块 | 说明 |
|---|---|
| `./types.js` | `AT_FILE_INJECTION_TRIGGER` 常量、`IPromptProcessor`、`PromptPipelineContent` |
| `./injectionParser.js` | `extractInjections` 注入语法解析函数 |
| `../../ui/types.js` | `MessageType` 枚举（INFO、ERROR） |

## 外部依赖

| 包名 | 说明 |
|---|---|
| `@google/gemini-cli-core` | `debugLogger`（调试日志）、`flatMapTextParts`（多模态内容遍历工具）、`readPathFromWorkspace`（工作区文件读取） |
