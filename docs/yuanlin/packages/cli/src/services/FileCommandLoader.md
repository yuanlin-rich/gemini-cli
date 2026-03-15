# FileCommandLoader.ts

> 从用户目录、项目目录和扩展目录的 TOML 文件中发现并加载自定义斜杠命令。

## 概述

`FileCommandLoader` 是 `ICommandLoader` 接口的实现，负责从文件系统中的 `.toml` 文件加载自定义命令定义。它按照固定顺序扫描三类目录：**用户全局命令目录** -> **项目命令目录** -> **扩展命令目录**（按扩展名字母排序），解析每个 TOML 文件的内容，通过 Zod schema 验证后将其适配为可执行的 `SlashCommand` 对象。

该加载器还负责组装**提示词处理管道**，根据命令定义中使用的占位符（`{{args}}`、`!{...}`、`@{...}`）自动注册对应的处理器（`DefaultArgumentProcessor`、`ShellProcessor`、`AtFileProcessor`），在命令执行时对提示词进行流水线式变换。

## 架构图（mermaid）

```mermaid
flowchart TD
    A[FileCommandLoader.loadCommands] --> B[getCommandDirectories]
    B --> C[用户命令目录]
    B --> D[项目命令目录]
    B --> E[扩展命令目录 x N]

    A --> F["glob('**/*.toml')"]
    F --> G[parseAndAdaptFile]

    G --> H[fs.readFile]
    H --> I[toml.parse]
    I --> J[Zod Schema 验证]
    J --> K[计算命令名称]
    K --> L{检测占位符}

    L -->|"@{...}"| M[注册 AtFileProcessor]
    L -->|"!{...} 或 {{args}}"| N[注册 ShellProcessor]
    L -->|无 {{args}}| O[注册 DefaultArgumentProcessor]

    M --> P[构建 SlashCommand]
    N --> P
    O --> P

    P --> Q["action: 执行处理管道"]
    Q --> R[返回 submit_prompt]
    Q -->|ConfirmationRequiredError| S[返回 confirm_shell_commands]
```

## 主要导出

| 导出名称 | 类型 | 说明 |
|---|---|---|
| `FileCommandLoader` | 类 | 从 TOML 文件加载自定义斜杠命令的加载器 |

## 核心逻辑

### 构造函数

```typescript
constructor(private readonly config: Config | null)
```

从 `Config` 中提取：
- `folderTrustEnabled`: 是否启用文件夹信任机制。
- `isTrustedFolder`: 当前文件夹是否受信任。
- `projectRoot`: 项目根目录路径。

### `loadCommands(signal): Promise<SlashCommand[]>`

1. **信任检查**：若启用文件夹信任但当前目录不受信任，直接返回空数组。
2. **目录枚举**：调用 `getCommandDirectories()` 获取所有命令目录。
3. **文件扫描**：对每个目录使用 `glob('**/*.toml')` 递归查找 TOML 文件。
4. **并行解析**：对找到的文件并行调用 `parseAndAdaptFile`。
5. **错误处理**：忽略 `ENOENT`（目录不存在）和中断信号错误，其他错误通过 `coreEvents.emitFeedback` 上报。

### `getCommandDirectories(): CommandDirectory[]`

按固定顺序返回命令目录列表：
1. 用户全局目录 (`Storage.getUserCommandsDir()`) -- `CommandKind.USER_FILE`
2. 项目目录 (`storage.getProjectCommandsDir()`) -- `CommandKind.WORKSPACE_FILE`
3. 扩展目录（按扩展名字母序排列） -- `CommandKind.EXTENSION_FILE`

### `parseAndAdaptFile(filePath, baseDir, kind, extensionName?, extensionId?): Promise<SlashCommand | null>`

1. **读取文件**：`fs.readFile(filePath, 'utf-8')`
2. **解析 TOML**：`toml.parse(fileContent)`
3. **Schema 验证**：使用 `TomlCommandDefSchema`（要求 `prompt` 字段为必填字符串，`description` 可选字符串）
4. **命令名计算**：将文件的相对路径（去除 `.toml` 后缀）按路径分隔符拆分，每段去除非法字符（保留 `a-zA-Z0-9_-.`），超长段截断为 47 字符加 `...`，然后以 `:` 连接
5. **处理管道组装**：
   - 含 `@{` -> `AtFileProcessor`（最先执行，安全优先）
   - 含 `!{` 或 `{{args}}` -> `ShellProcessor`
   - 不含 `{{args}}` -> `DefaultArgumentProcessor`（追加原始输入）
6. **构建 action 闭包**：依次执行处理管道，捕获 `ConfirmationRequiredError` 返回确认请求

### TOML 命令定义 Schema

```typescript
const TomlCommandDefSchema = z.object({
  prompt: z.string(),        // 必填：提示词模板
  description: z.string().optional(),  // 可选：命令描述
});
```

## 内部依赖

| 模块 | 说明 |
|---|---|
| `./types.js` | `ICommandLoader` 接口 |
| `./prompt-processors/argumentProcessor.js` | `DefaultArgumentProcessor` |
| `./prompt-processors/shellProcessor.js` | `ShellProcessor`、`ConfirmationRequiredError` |
| `./prompt-processors/atFileProcessor.js` | `AtFileProcessor` |
| `./prompt-processors/types.js` | 占位符常量、`IPromptProcessor`、`PromptPipelineContent` |
| `../ui/commands/types.js` | `CommandKind`、`SlashCommand` 等类型 |
| `../ui/utils/textUtils.js` | `sanitizeForDisplay` 文本截断工具 |

## 外部依赖

| 包名 | 说明 |
|---|---|
| `node:fs` | 文件读取 |
| `node:path` | 路径操作 |
| `@iarna/toml` | TOML 解析器 |
| `glob` | 文件通配符匹配 |
| `zod` | 运行时类型验证 |
| `@google/gemini-cli-core` | `Storage`、`coreEvents`、`Config` |
