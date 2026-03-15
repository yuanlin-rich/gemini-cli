# consent.ts

> 扩展安装与更新过程中的用户同意（consent）交互管理模块。

## 概述

`consent.ts` 负责在安装或更新 Gemini CLI 扩展时，向用户展示安全警告信息并获取用户的明确同意。该模块支持两种交互模式：交互式（interactive，在 CLI UI 中弹出确认请求）和非交互式（non-interactive，通过 stdin 读取 Y/n 输入）。它还会根据扩展配置（MCP 服务器、上下文文件、Hooks、Agent Skills 等）生成详细的同意描述文本，确保用户在安装前充分了解扩展的行为和权限。

## 架构图（mermaid）

```mermaid
flowchart TD
    A[调用方: ExtensionManager / CLI 命令] -->|安装或更新| B[maybeRequestConsentOrFail]
    B --> C{是否存在旧配置?}
    C -->|是| D[对比新旧 extensionConsentString]
    C -->|否| E[生成 extensionConsentString]
    D -->|相同| F[跳过同意, 直接返回]
    D -->|不同| G[调用 requestConsent 回调]
    E --> G
    G -->|交互式| H[promptForConsentInteractive]
    G -->|非交互式| I[promptForConsentNonInteractive]
    H --> J[UI ConfirmationRequest]
    I --> K[readline stdin Y/n]
    J --> L{用户是否同意?}
    K --> L
    L -->|是| M[继续安装]
    L -->|否| N[抛出错误, 取消安装]

    subgraph 同意文本生成
        B --> O[extensionConsentString]
        O --> P[MCP 服务器列表]
        O --> Q[上下文文件名]
        O --> R[排除工具列表]
        O --> S[Hooks 警告]
        O --> T[renderSkillsList]
    end
```

## 主要导出

| 导出名称 | 类型 | 说明 |
|---------|------|------|
| `INSTALL_WARNING_MESSAGE` | `string` | 扩展安装的第三方安全警告常量（黄色 chalk 文本） |
| `SKILLS_WARNING_MESSAGE` | `string` | Agent Skills 注入系统提示的安全警告常量（黄色 chalk 文本） |
| `skillsConsentString` | `async function` | 为安装 Agent Skills 生成同意描述文本 |
| `requestConsentNonInteractive` | `async function` | 非交互模式下通过 stdin 请求用户同意 |
| `requestConsentInteractive` | `async function` | 交互模式下通过 UI 确认请求获取用户同意 |
| `promptForConsentNonInteractive` | `async function` | 底层非交互式 Y/n 提示函数 |
| `maybeRequestConsentOrFail` | `async function` | 核心入口：比较新旧配置差异，仅在必要时请求同意，否则抛出异常取消安装 |

## 核心逻辑

1. **`extensionConsentString`**（内部函数）：根据 `ExtensionConfig` 的各字段（`mcpServers`、`contextFileName`、`excludeTools`、hooks、skills）以及迁移/重命名状态，拼接出完整的同意描述文本。所有输入先通过 `escapeAnsiCtrlCodes` 进行 ANSI 控制码转义以防注入。

2. **`renderSkillsList`**（内部函数）：格式化 Agent Skills 列表，包括技能名称、描述、源路径以及目录中的文件数量统计。

3. **`maybeRequestConsentOrFail`**：核心流程控制函数。当存在旧配置时，同时生成新旧两份同意文本进行比较——若完全相同则静默跳过；否则调用传入的 `requestConsent` 回调函数请求用户确认。用户拒绝时抛出 `Error`。

4. **两种提示模式**：
   - `promptForConsentNonInteractive`：使用 Node.js `readline` 模块从 stdin 读取输入，默认值为 `true`（直接回车即同意）。
   - `promptForConsentInteractive`：通过回调函数将 `ConfirmationRequest` 对象传递给 UI 层，由 UI 渲染确认对话框并通过 `onConfirm` 回调返回结果。

## 内部依赖

| 模块路径 | 用途 |
|---------|------|
| `../../ui/types.js` | `ConfirmationRequest` 类型定义 |
| `../../ui/utils/textUtils.js` | `escapeAnsiCtrlCodes` 函数，防止 ANSI 注入 |
| `../extension.js` | `ExtensionConfig` 类型定义 |

## 外部依赖

| 包名 | 用途 |
|------|------|
| `node:fs/promises` | 读取技能目录文件列表 |
| `node:path` | 路径解析（获取技能所在目录） |
| `@google/gemini-cli-core` | `debugLogger` 日志工具、`SkillDefinition` 类型 |
| `chalk` | 终端文本着色（警告信息使用黄色） |
