# markdownUtils.ts

> 将 JSON 兼容的数据结构转换为可读的 Markdown 格式

## 概述
`markdownUtils.ts` 实现了一个递归的 JSON-to-Markdown 转换器，能够将任意深度的 JSON 数据智能地转换为 Markdown 列表、表格或内联文本。其设计动机是让工具输出的结构化 JSON 数据能以人类友好的 Markdown 格式展示给用户。该文件在模块中作为"数据展示格式化器"被工具输出处理链路调用。

## 架构图
```mermaid
flowchart TD
    A[jsonToMarkdown data] --> B{类型?}
    B -->|null/undefined| C[返回字面量]
    B -->|数组| D{是同构对象数组?}
    D -->|是| E[renderTable 表格]
    D -->|否| F[递归列表 - item]
    B -->|对象| G[递归列表 - key: value]
    B -->|字符串| H[多行缩进处理]
    B -->|其他| I[String 转换]

    J[safeJsonToMarkdown] --> K{JSON.parse 成功?}
    K -->|是| L[jsonToMarkdown]
    K -->|否| M[返回原文本]

    subgraph 表格渲染
        E --> N[camelCase → Space Case 表头]
        E --> O[| 管道分隔 | 表格行]
    end
```

## 主要导出

### 函数
- **`jsonToMarkdown(data: unknown, indent?: number): string`** — 递归将 JSON 数据转换为 Markdown 字符串
- **`safeJsonToMarkdown(text: string): string`** — 安全版本，解析失败返回原文本
- **`isRecord(value: unknown): value is Record<string, unknown>`** — 类型守卫，判断值是否为普通对象

## 核心逻辑
1. **智能表格检测**：`isArrayOfSimilarObjects` 检测数组元素是否为具有相同键集的对象，是则渲染为 Markdown 表格（更紧凑）。
2. **camelCase 转换**：`camelToSpace` 将 camelCase 键名转换为 Space Case 作为表头和列表标签（如 `quotaId` → `Quota Id`）。
3. **递归缩进**：每层嵌套增加 2 空格缩进，对象键名加粗（`**Key**:`），复杂值换行递归。
4. **表格转义**：`renderTable` 对管道符和反斜杠进行转义，换行符替换为空格。
5. **多行字符串处理**：字符串内的非首行添加当前缩进对齐。

## 内部依赖
无

## 外部依赖
无
