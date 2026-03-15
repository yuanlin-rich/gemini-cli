# injectionParser.ts

> 通用的注入语法解析器，从提示词字符串中提取 `!{...}` 或 `@{...}` 注入块。

## 概述

本文件提供了一个通用的注入语法解析函数 `extractInjections`，用于从提示词模板字符串中提取所有匹配指定触发器（如 `!{` 或 `@{`）的注入块。解析器通过**花括号计数**正确处理嵌套花括号（例如 shell 命令中的 `if { ... }` 结构），但不支持转义机制。

该解析器同时被 `ShellProcessor` 和 `AtFileProcessor` 使用，是提示词处理管道的底层基础设施。

## 架构图（mermaid）

```mermaid
flowchart TD
    A["extractInjections(prompt, trigger, contextName?)"] --> B[初始化: index = 0]
    B --> C{在 prompt 中查找 trigger}
    C -->|未找到| D[返回 injections 数组]
    C -->|找到 startIndex| E[从 trigger 后开始扫描]
    E --> F[braceCount = 1]
    F --> G{扫描当前字符}
    G -->|"'{'| H[braceCount++]"
    G -->|"'}'| I[braceCount--]"
    I --> J{braceCount === 0?}
    J -->|是| K[提取内容, 记录注入]
    K --> C
    J -->|否| G
    H --> G
    G -->|其他字符| G
    G -->|到达字符串末尾| L{braceCount > 0?}
    L -->|是| M[抛出错误: 未闭合的注入]
```

## 主要导出

| 导出名称 | 类型 | 说明 |
|---|---|---|
| `Injection` | 接口 | 描述单个注入点的数据结构 |
| `extractInjections` | 函数 | 从字符串中提取所有注入块的解析函数 |

## 核心逻辑

### `Injection` 接口

```typescript
export interface Injection {
  content: string;    // 花括号内部的内容（已 trim）
  startIndex: number; // 注入块起始位置（含触发器）
  endIndex: number;   // 注入块结束位置（含闭合花括号之后）
}
```

- `startIndex` 指向触发器的第一个字符（如 `!` 或 `@`）。
- `endIndex` 指向闭合 `}` 的下一个位置（独占式）。
- `content` 是花括号内部内容的 `trim()` 结果。

### `extractInjections(prompt, trigger, contextName?): Injection[]`

迭代式解析算法：

1. **查找触发器**：使用 `prompt.indexOf(trigger, index)` 逐个查找触发器位置。
2. **花括号计数**：从触发器后的位置开始，维护 `braceCount` 计数器：
   - 遇到 `{` 时计数 +1。
   - 遇到 `}` 时计数 -1。
   - 计数归零时表示找到匹配的闭合花括号。
3. **提取内容**：将触发器之后到闭合花括号之前的子字符串提取为 `content`，经过 `trim()` 处理。
4. **继续扫描**：将扫描位置移至闭合花括号之后，继续查找下一个触发器。
5. **错误处理**：若扫描到字符串末尾仍未找到匹配的闭合花括号，抛出包含位置信息和上下文名称的错误。

### 嵌套花括号支持

```
!{echo ${VAR:-default}}
```

- 遇到 `!{` 时 `braceCount = 1`
- 遇到 `${` 中的 `{` 时 `braceCount = 2`
- 遇到 `default}` 中的 `}` 时 `braceCount = 1`
- 遇到最后的 `}` 时 `braceCount = 0`，匹配完成
- 提取内容：`echo ${VAR:-default}`

### 限制

- **不支持转义**：无法通过转义符跳过花括号（如 `\{` 不会被特殊处理）。
- **不支持不平衡花括号**：若命令或路径本身包含不配对的花括号，将导致解析错误。
- **严格模式**：未闭合的注入会直接抛出异常，而非静默跳过。

## 内部依赖

无。

## 外部依赖

无。
