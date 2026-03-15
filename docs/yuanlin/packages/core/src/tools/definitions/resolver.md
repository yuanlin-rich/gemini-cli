# resolver.ts

> 工具声明解析器：将 ToolDefinition 的 base 和 model-specific overrides 合并为最终 FunctionDeclaration。

## 概述
本文件提供 `resolveToolDeclaration` 函数，是工具定义解析的终端步骤。它接受一个 `ToolDefinition`（含 base 声明和可选的 overrides 函数），根据传入的 modelId 调用 overrides 获取模型特定的覆盖属性，然后与 base 合并（overrides 优先）返回最终的 `FunctionDeclaration`。

## 架构图
```mermaid
graph LR
    A[ToolDefinition] --> B{有 overrides 且有 modelId?}
    B -->|no| C[返回 base]
    B -->|yes| D[调用 overrides(modelId)]
    D --> E{返回值非空?}
    E -->|no| C
    E -->|yes| F[合并 base + override]
    F --> G[FunctionDeclaration]
```

## 主要导出

### `resolveToolDeclaration(definition: ToolDefinition, modelId?: string): FunctionDeclaration`
- 无 modelId 或无 overrides 时直接返回 `definition.base`
- 有 override 时浅合并：`{ ...base, ...override }`

## 核心逻辑
简单的浅合并策略，override 的属性覆盖 base 中的同名属性。

## 内部依赖
- `./types.ts` - `ToolDefinition`

## 外部依赖
- `@google/genai` - `FunctionDeclaration`
