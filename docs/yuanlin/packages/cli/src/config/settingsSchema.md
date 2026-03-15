# settingsSchema.ts

> 设置系统的声明式 Schema 定义，约 2802 行，定义了 Gemini CLI 全部配置项的类型、默认值、描述、分类和合并策略。

## 概述

`settingsSchema.ts` 是 Gemini CLI 设置系统的"唯一真相源"（single source of truth）。它以声明式方式定义了所有配置项的元数据，驱动以下功能：

1. **TypeScript 类型**：导出 `Settings` 和 `MergedSettings` 接口。
2. **运行时校验**：被 `settings-validation.ts` 消费生成 Zod 校验器。
3. **UI 渲染**：设置对话框根据 Schema 动态生成编辑控件。
4. **文档生成**：通过 `npm run docs:settings` 自动生成配置参考文档。
5. **合并策略**：每个设置项可指定 `mergeStrategy`（replace / concat / union / shallow_merge）。
6. **JSON Schema 生成**：支持 `$ref` 引用和 `SETTINGS_SCHEMA_DEFINITIONS` 定义块。

全部配置按分类组织为十余个顶级组：`general`、`model`、`tools`、`mcp`、`mcpServers`、`context`、`security`、`telemetry`、`privacy`、`ui`、`experimental`、`advanced`、`admin`、`agents`、`skills`、`billing`、`output`、`hooksConfig`、`hooks`、`modelConfigs` 等。

## 架构图（mermaid）

```mermaid
flowchart TD
    A[settingsSchema.ts] --> B[SETTINGS_SCHEMA 常量]
    B --> C[getSettingsSchema 函数]

    A --> D[SETTINGS_SCHEMA_DEFINITIONS]
    D --> E[引用类型定义]
    E --> E1[TelemetrySettings]
    E --> E2[MCPServerConfig]
    E --> E3[AgentOverride]
    E --> E4[SandboxConfig]
    E --> E5[CustomTheme]

    A --> F[SettingDefinition 接口]
    F --> F1[type: SettingsType]
    F --> F2[label: string]
    F --> F3[category: string]
    F --> F4[description: string]
    F --> F5[default: SettingsValue]
    F --> F6[properties: SettingsSchema]
    F --> F7[mergeStrategy: MergeStrategy]
    F --> F8[options: SettingEnumOption[]]
    F --> F9[deprecated / hidden / readOnly]

    A --> G[Settings 接口]
    A --> H[MergedSettings 接口]
    H -->|所有字段必填| G
```

## 主要导出

| 导出名称 | 类型 | 说明 |
|---------|------|------|
| `SettingsType` | `type` | 设置值类型：`boolean`/`string`/`number`/`array`/`object`/`enum` |
| `SettingsValue` | `type` | 设置值联合类型 |
| `TOGGLE_TYPES` | `ReadonlySet` | 可切换类型集合（`boolean`、`enum`） |
| `SettingEnumOption` | `interface` | 枚举选项：`value` + `label` |
| `SettingCollectionDefinition` | `interface` | 集合类型定义（用于数组元素、additionalProperties） |
| `MergeStrategy` | `enum` | 合并策略：`REPLACE`/`CONCAT`/`UNION`/`SHALLOW_MERGE` |
| `SettingDefinition` | `interface` | 设置项完整定义 |
| `SettingsSchema` | `type` | `Record<string, SettingDefinition>` |
| `Settings` | `interface` | 设置接口（所有字段可选） |
| `MergedSettings` | `interface` | 合并后设置接口（顶级字段必选，通过 `Required<>` 和交叉类型） |
| `MemoryImportFormat` | `type` | 内存导入格式：`tree`/`flat`/`hierarchical` |
| `DnsResolutionOrder` | `type` | DNS 解析顺序 |
| `getSettingsSchema` | `() => SettingsSchema` | 获取完整 Schema |
| `SETTINGS_SCHEMA_DEFINITIONS` | `Record<string, unknown>` | JSON Schema 引用类型定义块 |

## 核心逻辑

### SettingDefinition 接口

每个设置项包含以下元数据：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `SettingsType` | 值类型 |
| `label` | `string` | 显示标签 |
| `category` | `string` | 分类名（用于 UI 分组） |
| `description` | `string` | 详细描述 |
| `default` | `SettingsValue` | 默认值 |
| `properties` | `SettingsSchema` | 子属性（嵌套对象） |
| `additionalProperties` | `SettingCollectionDefinition` | 动态属性定义 |
| `items` | `SettingCollectionDefinition` | 数组元素定义 |
| `options` | `SettingEnumOption[]` | 枚举选项 |
| `mergeStrategy` | `MergeStrategy` | 合并策略 |
| `deprecated` | `boolean` | 是否废弃 |
| `hidden` | `boolean` | 是否对 UI 隐藏 |
| `readOnly` | `boolean` | 是否只读 |
| `ref` | `string` | JSON Schema 引用标识 |
| `scope` | `string` | 适用作用域 |

### 主要配置分类

| 分类 | 键 | 核心配置项 |
|------|-----|-----------|
| General | `general` | `defaultApprovalMode`、`enableAutoUpdate`、`checkpointing`、`retryFetchErrors`、`maxAttempts`、`plan`、`sessionRetention`、`preferredEditor` |
| Model | `model` | `name`、`maxSessionTurns`、`disableLoopDetection`、`compressionThreshold`、`summarizeToolOutput`、`skipNextSpeakerCheck` |
| Tools | `tools` | `allowed`、`exclude`、`sandbox`、`useRipgrep`、`shell`（交互式 shell、超时）、`truncateToolOutputThreshold`、`discoveryCommand`、`callCommand` |
| MCP | `mcp` | `allowed`、`excluded`、`serverCommand` |
| MCP Servers | `mcpServers` | 动态键值对：每个键为服务器名，值为 `MCPServerConfig` |
| Context | `context` | `fileName`、`importFormat`、`includeDirectoryTree`、`includeDirectories`、`fileFiltering`、`discoveryMaxDirs` |
| Security | `security` | `folderTrust`、`disableYoloMode`、`disableAlwaysAllow`、`toolSandboxing`、`blockGitExtensions`、`allowedExtensions`、`environmentVariableRedaction`、`enableConseca` |
| Telemetry | `telemetry` | `enabled`/`disabled`、`endpoint` 等（引用 `TelemetrySettings`） |
| Privacy | `privacy` | `usageStatisticsEnabled` |
| UI | `ui` | `theme`、`footer`（项目列表、旧版 hide 标志）、`showMemoryUsage`、`accessibility`、`loadingPhrases`、`useBackgroundColor`、`useAlternateBuffer` |
| Experimental | `experimental` | `jitContext`、`modelSteering`、`toolOutputMasking`、`extensionManagement`、`extensionReloading`、`extensionRegistryURI`、`extensionConfig`、`plan`、`taskTracker`、`directWebFetch`、`enableAgents`、`gemmaModelRouter` |
| Admin | `admin` | `secureModeEnabled`、`mcp`（enabled、config）、`extensions`（enabled）、`skills`（enabled） |
| Agents | `agents` | `overrides`（每个 agent 的 enabled、runConfig、modelConfig） |
| Skills | `skills` | `enabled`、`disabled` |
| Hooks | `hooksConfig` | `enabled`、`disabled` |
| Billing | `billing` | `billingProject` |
| Output | `output` | `format`（text/json/stream-json） |
| IDE | `ide` | `enabled` |

### SETTINGS_SCHEMA_DEFINITIONS

为 JSON Schema 生成提供共享定义，当前包括：
- `TelemetrySettings`：遥测配置的 JSON Schema 表示
- `MCPServerConfig`：MCP 服务器配置
- `AgentOverride`：Agent 覆盖配置
- `SandboxConfig`：沙箱配置
- `CustomTheme`：自定义主题

### Settings 与 MergedSettings 的关系

- `Settings`：所有字段可选，表示单个设置文件的内容。
- `MergedSettings`：通过 `Required<Pick<Settings, ...>>` + 类型交叉，确保顶级字段都存在（合并后不应有缺失）。
- `MergedSettings` 的嵌套属性保持可选性，但所有直属分类字段（`general`、`tools`、`ui` 等）为必选。

### oneLine 辅助函数

模板字符串工具，将多行描述压缩为单行（替换所有空白为单个空格），便于在 Schema 定义中编写长描述。

## 内部依赖

| 模块 | 导入内容 | 用途 |
|------|---------|------|
| `./settings.js` | `SessionRetentionSettings`（类型） | 会话保留设置类型 |
| `../utils/sessionCleanup.js` | `DEFAULT_MIN_RETENTION` | 最小保留时间默认值 |

## 外部依赖

| 模块 | 导入内容 | 用途 |
|------|---------|------|
| `@google/gemini-cli-core` | `DEFAULT_TRUNCATE_TOOL_OUTPUT_THRESHOLD`, `DEFAULT_MODEL_CONFIGS`, `MCPServerConfig`, `BugCommandSettings`, `TelemetrySettings`, `AuthType`, `AgentOverride`, `CustomTheme`, `SandboxConfig` | 核心类型常量与接口 |
