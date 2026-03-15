# coreTools.ts

> 工具定义编排器：解析模型族工具集、提供遗留导出和动态工具定义。

## 概述
本文件是工具定义的编排中心。它根据模型 ID 解析对应的工具集（`CoreToolSet`），将每个工具包装为 `ToolDefinition`（含 base 声明和 model-specific overrides），并以遗留导出形式提供所有工具定义常量（如 `READ_FILE_DEFINITION`、`WRITE_FILE_DEFINITION` 等）。同时重导出所有名称/参数常量和动态工具定义函数。

## 架构图
```mermaid
graph TD
    A[coreTools.ts] -->|getToolSet| B{modelId}
    B -->|gemini-3| C[GEMINI_3_SET]
    B -->|default| D[DEFAULT_LEGACY_SET]
    A -->|TOOL_DEFINITION| E[base: DEFAULT_LEGACY_SET.tool]
    E -->|overrides| F[getToolSet(modelId).tool]
    A -->|dynamic| G[getShellDefinition]
    A -->|dynamic| H[getExitPlanModeDefinition]
    A -->|dynamic| I[getActivateSkillDefinition]
```

## 主要导出

### 函数
- `getToolSet(modelId?)`: 根据模型 ID 返回对应的 `CoreToolSet`（gemini-3 或 default-legacy）
- `getShellDefinition(enableInteractiveShell, enableEfficiency)`: 动态生成 Shell 工具定义
- `getExitPlanModeDefinition(plansDir)`: 动态生成退出计划模式工具定义
- `getActivateSkillDefinition(skillNames)`: 动态生成技能激活工具定义
- `getShellToolDescription / getCommandDescription`: Shell 描述生成（重导出）

### 常量（ToolDefinition 类型，共 15 个）
`READ_FILE_DEFINITION`, `WRITE_FILE_DEFINITION`, `GREP_DEFINITION`, `RIP_GREP_DEFINITION`, `WEB_SEARCH_DEFINITION`, `EDIT_DEFINITION`, `GLOB_DEFINITION`, `LS_DEFINITION`, `WEB_FETCH_DEFINITION`, `READ_MANY_FILES_DEFINITION`, `MEMORY_DEFINITION`, `WRITE_TODOS_DEFINITION`, `GET_INTERNAL_DOCS_DEFINITION`, `ASK_USER_DEFINITION`, `ENTER_PLAN_MODE_DEFINITION`

每个 ToolDefinition 的 `base` 取自 `DEFAULT_LEGACY_SET`，`overrides` 通过 `getToolSet(modelId)` 动态解析。

## 核心逻辑
使用 getter 延迟求值 `base` 属性，避免模块加载时循环引用。

## 内部依赖
- `./types.ts` - `ToolDefinition`, `CoreToolSet`
- `./modelFamilyService.ts` - `getToolFamily`
- `./model-family-sets/default-legacy.ts` - `DEFAULT_LEGACY_SET`
- `./model-family-sets/gemini-3.ts` - `GEMINI_3_SET`
- `./dynamic-declaration-helpers.ts` - 动态声明函数
- `./base-declarations.ts` - 所有名称/参数常量

## 外部依赖
无
