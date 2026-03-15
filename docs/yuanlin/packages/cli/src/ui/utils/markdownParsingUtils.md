# markdownParsingUtils.ts

> 将 Markdown 内联语法（粗体、斜体、删除线、代码、链接等）转换为 ANSI 终端着色字符串

## 概述

本文件实现了一个 Markdown-to-ANSI 解析器，将 Markdown 内联格式转换为终端可显示的 ANSI 转义序列。支持 `***粗斜体***`、`**粗体**`、`*斜体*`、`~~删除线~~`、`` `代码` ``、`[链接](url)`、`<u>下划线</u>` 以及裸 URL 的着色。解析逻辑与 `InlineMarkdownRenderer.tsx` 组件保持一致。

## 架构图（mermaid）

```mermaid
flowchart TD
    A[parseMarkdownToANSI] --> B{包含 Markdown 标记?}
    B -- 否 --> C[直接着色返回]
    B -- 是 --> D[inlineRegex 正则遍历]
    D --> E{匹配类型}
    E --> |***...***| F[chalk.bold + chalk.italic 递归]
    E --> |**...**| G[chalk.bold 递归]
    E --> |*...*/_..._| H[chalk.italic 递归]
    E --> |~~...~~| I[chalk.strikethrough 递归]
    E --> |`...`| J[accent 颜色着色]
    E --> |[text](url)| K[text + link 颜色 url]
    E --> |<u>...</u>| L[chalk.underline 递归]
    E --> |https://...| M[link 颜色着色]
    D --> N[拼接结果字符串]
```

## 主要导出

| 导出名 | 类型 | 说明 |
|--------|------|------|
| `parseMarkdownToANSI` | function | 将 Markdown 文本转换为 ANSI 着色字符串 |

## 核心逻辑

1. **快速路径**：不包含任何 Markdown 标记字符时直接返回着色文本，避免正则开销。
2. **递归解析**：粗体/斜体/删除线/下划线等支持嵌套，通过递归调用 `parseMarkdownToANSI` 处理内部格式。
3. **斜体防误判**：对 `*` 和 `_` 斜体做额外边界检查，排除路径分隔符（`./\\`）和单词内部的情况。
4. **颜色系统**：`ansiColorize` 支持 HEX 颜色、Ink 命名颜色和主题映射。

## 内部依赖

| 模块 | 说明 |
|------|------|
| `../themes/color-utils.js` | `resolveColor`、`INK_SUPPORTED_NAMES`、`INK_NAME_TO_HEX_MAP` |
| `../semantic-colors.js` | `theme` 主题颜色 |

## 外部依赖

| 模块 | 说明 |
|------|------|
| `chalk` | ANSI 样式（粗体、斜体、下划线、删除线、HEX 颜色） |
| `@google/gemini-cli-core` | `debugLogger` |
