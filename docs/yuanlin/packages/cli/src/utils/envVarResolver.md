# envVarResolver.ts

> 递归解析字符串和对象中的环境变量占位符（`$VAR` 和 `${VAR}`），支持循环引用保护。

## 概述

`envVarResolver.ts` 提供了两个层次的环境变量解析功能：字符串级别的 `resolveEnvVarsInString` 和对象级别的 `resolveEnvVarsInObject`。它使用正则表达式匹配 `$VAR_NAME` 和 `${VAR_NAME}` 两种格式的占位符，优先从自定义环境对象中查找值，再回退到 `process.env`。如果环境变量未定义，则保留原始占位符不替换。

对象级别的解析使用 `WeakSet` 追踪已访问对象，防止循环引用导致无限递归。

## 架构图（mermaid）

```mermaid
graph TD
    A["resolveEnvVarsInObject(obj)"] --> B{值的类型?}
    B -->|null/undefined/boolean/number| C[原样返回]
    B -->|string| D[resolveEnvVarsInString]
    B -->|Array| E{循环引用?}
    E -->|是| F[返回浅拷贝]
    E -->|否| G[标记已访问<br/>递归处理每个元素]
    B -->|object| H{循环引用?}
    H -->|是| I[返回浅拷贝]
    H -->|否| J[标记已访问<br/>递归处理每个属性]

    D --> K["正则匹配 $VAR 和 ${VAR}"]
    K --> L{customEnv 中存在?}
    L -->|是| M[使用 customEnv 值]
    L -->|否| N{process.env 中存在?}
    N -->|是| O[使用 process.env 值]
    N -->|否| P[保留原始占位符]
```

## 主要导出

| 导出名称 | 类型 | 描述 |
|---------|------|------|
| `resolveEnvVarsInString(value, customEnv?)` | 函数 | 替换字符串中的环境变量占位符 |
| `resolveEnvVarsInObject<T>(obj, customEnv?)` | 函数 | 递归替换对象中所有字符串值的环境变量占位符 |

## 核心逻辑

### resolveEnvVarsInString

使用正则 `/\$(?:(\w+)|{([^}]+)})/g` 同时匹配 `$VAR_NAME`（`\w+`）和 `${VAR_NAME}`（`{[^}]+}`）两种格式。替换顺序：自定义环境 > process.env > 保留原文。

### resolveEnvVarsInObject

内部实现 `resolveEnvVarsInObjectInternal` 使用 `WeakSet` 追踪已访问对象：
- **基本类型**（null、undefined、boolean、number）：直接返回
- **字符串**：调用 `resolveEnvVarsInString`
- **数组**：检查循环引用，标记后递归处理每个元素，处理完取消标记
- **对象**：检查循环引用，标记后递归处理每个自有属性，处理完取消标记
- **循环引用**：返回浅拷贝以打破循环

## 内部依赖

无。

## 外部依赖

无。
