# googleQuotaErrors.ts

> 将 Google API 错误分类为可重试配额错误、终端配额错误、验证要求错误或模型未找到错误

## 概述
`googleQuotaErrors.ts` 是 Google API 错误分类链的核心环节，在 `googleErrors.ts` 解析出结构化错误后，进一步将其分类为具有业务语义的错误类型。其设计动机是让重试引擎和用户提示系统能根据错误类型做出正确决策——可重试错误触发退避重试，终端错误显示配额耗尽提示，验证错误引导用户完成验证。该文件约 418 行，涵盖了 429/499/403/404/503 等 HTTP 状态码的完整分类逻辑。

## 架构图
```mermaid
flowchart TD
    A[classifyGoogleError] --> B[parseGoogleApiError + getErrorStatus]
    B --> C{status === 404?}
    C -->|是| D[ModelNotFoundError]
    C -->|否| E{status === 403?}
    E -->|是| F[classifyValidationRequiredError]
    F --> G{VALIDATION_REQUIRED + CloudCode域?}
    G -->|是| H[ValidationRequiredError]
    G -->|否| I[继续]
    E -->|否| J{status === 503?}
    J -->|是| K[RetryableQuotaError]
    J -->|否| L{429/499 + details?}
    L -->|有details| M[检查 QuotaFailure]
    M --> N{PerDay/Daily?}
    N -->|是| O[TerminalQuotaError]
    N -->|否| P[检查 ErrorInfo]
    P --> Q{INSUFFICIENT_G1_CREDITS?}
    Q -->|是| O
    Q -->|否| R{CloudCode RATE_LIMIT_EXCEEDED?}
    R -->|是| S{delay > 5min?}
    S -->|是| O
    S -->|否| T[RetryableQuotaError]
    R -->|否| U{QUOTA_EXHAUSTED?}
    U -->|是| O
    U -->|否| V[检查 RetryInfo delay]
    L -->|无details| W[尝试解析 retry message]
    W --> X{匹配 "Please retry in Xs"?}
    X -->|是| Y{delay > 5min?}
    Y -->|是| O
    Y -->|否| T
    X -->|否| Z[返回原错误]
```

## 主要导出

### 类
- **`TerminalQuotaError extends Error`** — 不可重试的配额错误（如日限额耗尽、信用余额不足），含 `retryDelayMs`、`reason`、`isInsufficientCredits` getter
- **`RetryableQuotaError extends Error`** — 可重试的配额错误（如每分钟限速），含 `retryDelayMs`
- **`ValidationRequiredError extends Error`** — 需要用户验证的错误，含 `validationLink`、`validationDescription`、`learnMoreUrl`、`userHandled`

### 函数
- **`classifyGoogleError(error: unknown): unknown`** — 核心分类函数，将未知错误分类为上述特定类型或原样返回

## 核心逻辑
1. **分类优先级**：404 → ModelNotFoundError，403 + VALIDATION_REQUIRED → ValidationRequiredError，503 → RetryableQuotaError，429/499 → 详细分类。
2. **CloudCode 域校验**：`isCloudCodeDomain` 对域名进行字符清理后比对白名单（cloudcode-pa.googleapis.com 及其 staging/autopush 变体）。
3. **重试延迟阈值**：`MAX_RETRYABLE_DELAY_SECONDS = 300`（5 分钟），超过此阈值的"可重试"错误升级为终端错误。
4. **多层回退**：优先从 `QuotaFailure.violations` 判断日/分钟限额 → 从 `ErrorInfo.reason` 判断 CloudCode 特定原因 → 从 `RetryInfo.retryDelay` 提取延迟 → 从错误消息正则匹配 `"Please retry in Xs"`。
5. **持续时间解析**：`parseDurationInSeconds` 支持秒（`"60s"`）和毫秒（`"900ms"`）两种格式。

## 内部依赖
- `./googleErrors.js` — `parseGoogleApiError`、`ErrorInfo`、`GoogleApiError`、`Help`、`QuotaFailure`、`RetryInfo`
- `./httpErrors.js` — `getErrorStatus`、`ModelNotFoundError`

## 外部依赖
无
