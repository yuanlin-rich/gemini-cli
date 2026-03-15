# envExpansion.ts

> 在字符串中展开环境变量，支持 POSIX（$VAR、${VAR}）和 Windows（%VAR%）语法

## 概述
该文件提供了环境变量展开功能，使用 `dotenv-expand` 库处理 POSIX/Bash 语法的变量引用，并在 Windows 平台上额外处理 `%VAR%` 语法。用于配置文件中的环境变量替换场景。

## 架构图
```mermaid
graph TD
    A[expandEnvVars] --> B{空字符串?}
    B -->|是| C[返回原值]
    B -->|否| D{Windows 平台?}
    D -->|是| E[预处理 %VAR% 语法]
    D -->|否| F[跳过预处理]
    E --> G[dotenv-expand 展开<br/>$VAR / ${VAR} 语法]
    F --> G
    G --> H[返回展开后的字符串]

    subgraph dotenv-expand 包装
        G --> I[创建临时键 __GCLI_EXPAND_TARGET__]
        I --> J[expand 处理包含变量的字符串]
        J --> K[提取结果]
    end
```

## 主要导出

### `expandEnvVars(str: string, env: Record<string, string | undefined>): string`
在字符串中展开环境变量。

- **参数**:
  - `str` - 包含环境变量占位符的字符串
  - `env` - 环境变量名值映射
- **返回值**: 展开后的字符串，未定义的变量解析为空字符串
- **支持语法**: `$VAR`、`${VAR}`（所有平台）；`%VAR%`（仅 Windows）

## 核心逻辑
- **Windows 预处理**: 仅在 `win32` 平台上将 `%VAR%` 替换为对应值，避免在其他系统中误处理 URL 或 shell 命令中的 `%`
- **dotenv-expand 包装**: 将单个字符串包装为 `{ parsed: { __GCLI_EXPAND_TARGET__: str } }` 格式传给 `expand()`
- **undefined 过滤**: 从 env 中过滤掉 `undefined` 值以满足 `Record<string, string>` 类型要求

## 内部依赖
无

## 外部依赖
| 依赖 | 说明 |
|------|------|
| `dotenv-expand` | 环境变量展开库（POSIX 语法） |
