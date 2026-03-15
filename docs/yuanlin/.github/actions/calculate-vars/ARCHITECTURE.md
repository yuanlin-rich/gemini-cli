# calculate-vars 架构

> 计算发布流程中 dry-run 标志变量的轻量级 Composite Action

## 概述

`calculate-vars` 是一个简单的 GitHub Composite Action，用于在发布流程中标准化 `dry_run` 输入参数。它将来自不同工作流的 `dry_run` 输入（可能为空字符串、`"false"` 或 `"true"`）统一转换为一致的布尔字符串输出。该 Action 位于发布工作流链的起始位置，为后续步骤提供可靠的条件判断依据。

## 架构图

```mermaid
graph LR
    INPUT["inputs.dry_run<br/>(空字符串 / 'false' / 'true')"] --> CALC["calculate-vars<br/>标准化处理"]
    CALC --> OUTPUT["outputs.is_dry_run<br/>('true' / 'false')"]
    OUTPUT --> RELEASE["后续发布步骤<br/>条件判断"]
```

## 目录结构

```
calculate-vars/
└── action.yml    # Action 定义文件
```

## 关键文件

| 文件 | 功能 |
|------|------|
| `action.yml` | 定义 Composite Action：接收 `dry_run` 输入，输出标准化的 `is_dry_run` 布尔字符串。当输入为空或 `"false"` 时输出 `"false"`，否则输出 `"true"` |

## 内部依赖

无。该 Action 是独立的工具 Action，不依赖其他自定义 Action。

## 外部依赖

无。仅使用 Bash shell 进行字符串处理。
