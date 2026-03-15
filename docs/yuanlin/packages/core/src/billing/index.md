# index.ts (billing)

> 计费模块的桶文件，重新导出 billing.ts 的所有公共 API。

## 概述

`billing/index.ts` 是计费模块的入口文件，采用桶文件模式将 `billing.ts` 的全部导出重新暴露。这使得外部消费者可以通过 `from './billing/index.js'` 或简写 `from './billing'` 来访问所有计费相关功能。

## 架构图

```mermaid
graph LR
    CONSUMER[外部消费者] --> INDEX[billing/index.ts]
    INDEX --> |export *| BILLING[billing/billing.ts]
```

## 主要导出

重新导出 `billing.ts` 的所有内容，包括：
- 类型：`OverageStrategy`
- 常量：`G1_CREDIT_TYPE`, `OVERAGE_ELIGIBLE_MODELS`, `G1_UTM_CAMPAIGNS`, `MIN_CREDIT_BALANCE`
- 函数：`isOverageEligibleModel`, `wrapInAccountChooser`, `buildG1Url`, `getG1CreditBalance`, `shouldAutoUseCredits`, `shouldShowOverageMenu`, `shouldShowEmptyWalletMenu`

## 核心逻辑

无业务逻辑，纯导出聚合。

## 内部依赖

| 模块 | 导入项 | 用途 |
|------|--------|------|
| `./billing.js` | `*`（全部） | 计费核心逻辑 |

## 外部依赖

无。
