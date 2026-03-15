# extensions.ts

> 列出当前配置中注册的所有扩展。

## 概述

`extensions.ts` 是命令模块中最简单的文件，提供一个函数用于从配置对象中获取已注册的扩展列表。它作为 `/extensions` 斜杠命令的实现层，将配置数据直接传递给 CLI 界面展示。

## 架构图

```mermaid
graph LR
    CMD[/extensions 命令] --> LE[listExtensions]
    LE --> CONFIG[config.getExtensions]
    CONFIG --> EXTS[扩展列表]
```

## 主要导出

### 函数

| 函数 | 签名 | 说明 |
|------|------|------|
| `listExtensions` | `(config: Config) => GeminiCLIExtension[]` | 返回配置中注册的所有扩展 |

## 核心逻辑

直接委托给 `config.getExtensions()` 方法，无额外处理。

## 内部依赖

| 模块 | 导入项 | 用途 |
|------|--------|------|
| `../config/config.js` | `Config` (type) | 全局配置类型 |

## 外部依赖

无。
