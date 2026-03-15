# formatters.ts

> 提供字节单位格式化工具函数

## 概述
`formatters.ts` 是一个极其精简的工具文件，提供字节数到人类可读格式的转换函数。它在模块中作为通用的数据量展示工具，被需要显示文件大小或数据量的组件调用。

## 架构图
```mermaid
flowchart LR
    A[字节数] --> B{< 1MB?}
    B -->|是| C["KB 格式 (1位小数)"]
    B -->|否| D{< 1GB?}
    D -->|是| E["MB 格式 (1位小数)"]
    D -->|否| F["GB 格式 (2位小数)"]
```

## 主要导出

### 函数
- **`bytesToMB(bytes: number): number`** — 字节转兆字节（纯数值）
- **`formatBytes(bytes: number): string`** — 字节转人类可读字符串，自动选择 KB/MB/GB 单位

## 核心逻辑
- `formatBytes` 采用简单的阈值判断：< 1MB 显示 KB（1 位小数）、< 1GB 显示 MB（1 位小数）、>= 1GB 显示 GB（2 位小数）。

## 内部依赖
无

## 外部依赖
无
