# telemetry-utils.ts

> 遥测工具函数，根据文件路径推断编程语言

## 概述
该文件提供 `getProgrammingLanguage` 工具函数，从工具调用参数中提取文件路径并推断其编程语言。用于在文件操作遥测事件中附加编程语言标签，便于按语言维度分析文件操作数据。

## 架构图
```mermaid
graph LR
    A[工具调用参数] -->|file_path / path / absolute_path| B[getProgrammingLanguage]
    B --> C[getLanguageFromFilePath]
    C --> D["返回语言字符串 (如 'typescript')"]
```

## 主要导出

### `function getProgrammingLanguage(args: Record<string, unknown>): string | undefined`
从工具参数对象中查找文件路径（依次尝试 `file_path`、`path`、`absolute_path` 键），调用语言检测函数返回编程语言名称。若未找到路径或无法识别语言则返回 `undefined`。

## 核心逻辑
简单的属性查找 + 委托到语言检测工具。

## 内部依赖
- `../utils/language-detection.js` — `getLanguageFromFilePath`

## 外部依赖
无
