# generateContentResponseUtilities.ts

> 解析和转换 Gemini API 的 GenerateContentResponse，提取文本、函数调用和引用信息

## 概述
`generateContentResponseUtilities.ts` 是 Gemini API 响应处理的核心工具集，负责将 `GenerateContentResponse` 和 `Part[]` 转换为工具系统可消费的结构。它处理三大类内容：纯文本响应、函数调用（FunctionCall）以及工具执行结果的回传（FunctionResponse）。该文件的设计动机是将复杂的多模态响应解析逻辑从业务代码中剥离，提供统一的响应处理接口。在模块中它是"响应解析层"，被聊天核心和工具执行器广泛调用。

## 架构图
```mermaid
flowchart TD
    A[GenerateContentResponse / Part[]] --> B{响应类型}
    B --> C[getResponseTextFromParts]
    B --> D[getFunctionCalls / getFunctionCallsFromParts]
    B --> E[getCitations]

    F[工具执行结果] --> G[convertToFunctionResponse]
    G --> H{内容类型}
    H -->|纯文本| I[FunctionResponse Part]
    H -->|二进制 + 文本| J{模型支持多模态 FR?}
    J -->|是| K[嵌套 inlineData 到 FR]
    J -->|否| L[inlineData 作为兄弟 Part]
    H -->|已有 FR| M[透传 FunctionResponse]

    N[getStructuredResponse] --> C
    N --> D
    N --> O["文本 + JSON 函数调用"]
```

## 主要导出

### 函数
- **`convertToFunctionResponse(toolName, callId, llmContent, model): Part[]`** — 将工具输出转换为 Gemini `FunctionResponse` 格式，处理文本/二进制/fileData 混合内容，根据模型能力决定 inlineData 嵌套或平铺
- **`getResponseTextFromParts(parts: Part[]): string | undefined`** — 从 Part 数组中提取并拼接所有文本部分
- **`getFunctionCalls(response: GenerateContentResponse): FunctionCall[] | undefined`** — 从完整响应中提取函数调用
- **`getFunctionCallsFromParts(parts: Part[]): FunctionCall[] | undefined`** — 从 Part 数组中提取函数调用
- **`getFunctionCallsAsJson(response): string | undefined`** — 将响应中的函数调用序列化为 JSON
- **`getFunctionCallsFromPartsAsJson(parts): string | undefined`** — 将 Part 数组中的函数调用序列化为 JSON
- **`getStructuredResponse(response): string | undefined`** — 获取文本 + 函数调用的组合响应
- **`getStructuredResponseFromParts(parts): string | undefined`** — 从 Part 数组获取组合响应
- **`getCitations(resp): string[]`** — 提取响应中的引用链接列表

## 核心逻辑
1. **FunctionResponse 构建**：`convertToFunctionResponse` 是最复杂的函数，它将工具输出的 `PartListUnion`（可能是字符串、Part 数组或混合类型）分类为 textParts、inlineDataParts、fileDataParts，然后根据模型是否支持多模态 FunctionResponse（`supportsMultimodalFunctionResponse`）决定 inlineData 的处理方式。
2. **透传机制**：如果输入中已包含 `functionResponse`（某些工具可能直接返回 FR），直接用新的 callId/toolName 包装透传。
3. **空文本回退**：当只有二进制内容没有文本时，自动生成描述性文本 `"Binary content provided (N item(s))."`。

## 内部依赖
- `./partUtils.js` — `getResponseText` 从完整响应提取文本
- `../config/models.js` — `supportsMultimodalFunctionResponse` 模型能力检查
- `./debugLogger.js` — 调试日志

## 外部依赖
- `@google/genai` — `GenerateContentResponse`、`Part`、`FunctionCall`、`PartListUnion` 类型
