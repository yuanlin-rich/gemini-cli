# value-resolver.ts

> 认证值动态解析器：支持环境变量、Shell 命令和字面量三种来源

## 概述

`value-resolver.ts` 提供了一套统一的认证值解析机制，允许用户在配置文件中使用 `$ENV_VAR`、`!command` 或直接字面量来指定敏感认证信息（如 API Key、Token、密码等）。它是 `ApiKeyAuthProvider` 和 `HttpAuthProvider` 等 Provider 获取动态凭据的底层基础设施。

设计动机：将凭据获取方式从具体 Provider 实现中解耦，提供灵活的"晚绑定"能力——用户可以在运行时通过环境变量或命令行工具（如 `gcloud auth print-access-token`）动态获取凭据，而无需硬编码。

## 架构图

```mermaid
flowchart TD
    A[输入值] --> B{前缀判断}
    B -->|"$$" 或 "!!"| C[转义：去掉第一个前缀字符，返回字面量]
    B -->|"$"| D[读取 process.env 对应变量]
    B -->|"!"| E[通过 Shell 执行命令]
    B -->|其他| F[直接返回字面量]

    D -->|变量存在且非空| G[返回值]
    D -->|变量不存在或为空| H[抛出 Error]

    E --> I[spawnAsync 执行命令]
    I -->|stdout 非空| J[返回 trimmed stdout]
    I -->|stdout 为空| K[抛出 Error]
    I -->|超时 60s| L[抛出 AbortError]

    style H fill:#f66
    style K fill:#f66
    style L fill:#f66
```

## 主要导出

### `resolveAuthValue(value: string): Promise<string>`

核心解析函数。根据输入值的前缀判断解析策略：

| 前缀 | 行为 | 示例 |
|------|------|------|
| `$$` 或 `!!` | 转义：去掉一个前缀字符，返回字面量 | `$$FOO` -> `$FOO` |
| `$` | 读取环境变量 | `$API_KEY` -> `process.env.API_KEY` |
| `!` | 执行 Shell 命令（60 秒超时） | `!gcloud auth print-access-token` |
| 其他 | 返回原始字面量 | `my-secret-key` -> `my-secret-key` |

### `needsResolution(value: string): boolean`

快速判断值是否需要动态解析（以 `$` 或 `!` 开头）。用于 Provider 初始化时优化：字面量值无需调用 `resolveAuthValue`。

### `maskSensitiveValue(value: string): string`

将敏感值脱敏为日志安全的格式。短于等于 12 字符的值显示为 `****`，较长的值显示首尾各 2 字符（如 `ab****yz`）。

## 核心逻辑

### Shell 命令执行流程

1. 通过 `getShellConfiguration()` 获取当前平台的 Shell 配置（如 `/bin/zsh -c` 或 `cmd.exe /c`）
2. 使用 `spawnAsync` 创建子进程执行命令
3. 设置 `AbortSignal.timeout(60_000)` 作为超时保护
4. 对 stdout 进行 `trim()` 处理，空输出视为错误
5. `windowsHide: true` 防止 Windows 上弹出命令行窗口

### 转义机制

双前缀（`$$`、`!!`）用于转义，只剥离第一个字符。这允许用户配置以 `$` 或 `!` 开头的字面量值。

## 内部依赖

| 模块 | 导入内容 | 用途 |
|------|---------|------|
| `../../utils/debugLogger.js` | `debugLogger` | 调试日志输出 |
| `../../utils/shell-utils.js` | `getShellConfiguration`, `spawnAsync` | 获取 Shell 配置并执行命令 |

## 外部依赖

无直接的 npm 包依赖（间接使用 Node.js 内置的 `process.env` 和 `AbortSignal`）。
