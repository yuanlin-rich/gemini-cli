# fsErrorMessages.ts

> 将 Node.js 文件系统错误码转换为用户友好的提示消息

## 概述
`fsErrorMessages.ts` 封装了常见 Node.js 文件系统错误码（EACCES、ENOENT、ENOSPC 等）到友好错误消息的映射逻辑。其设计动机是让文件操作工具在出错时能向用户（和 LLM）提供清晰、可操作的错误提示，而非晦涩的系统错误码。该文件在模块中作为文件系统错误的统一"翻译层"。

## 架构图
```mermaid
flowchart TD
    A[未知错误] --> B[getFsErrorMessage]
    B --> C{是 NodeError?}
    C -->|是| D{已知错误码?}
    D -->|是| E[生成友好消息]
    D -->|否| F{有 error.code?}
    F -->|是| G["baseMessage (code)"]
    F -->|否| H[getErrorMessage fallback]
    C -->|否| H
```

## 主要导出

### 函数
- **`getFsErrorMessage(error: unknown, defaultMessage?: string): string`** — 将文件系统错误转换为用户友好消息

## 核心逻辑
1. **错误码映射表**：`errorMessageGenerators` 是一个 `Record<string, (path?) => string>` 映射，覆盖 EACCES（权限拒绝）、ENOENT（文件未找到）、ENOSPC（磁盘空间不足）、EISDIR（路径是目录）、EROFS（只读文件系统）、EPERM（操作不允许）、EEXIST（已存在）、EBUSY（资源繁忙）、EMFILE/ENFILE（打开文件过多）。
2. **路径感知**：每个错误消息生成器接受可选的 `path` 参数，有路径时消息更具体。
3. **降级策略**：未知错误码附加 `(code)` 后缀；非 Node 错误调用通用 `getErrorMessage`。

## 内部依赖
- `./errors.js` — `isNodeError`、`getErrorMessage`

## 外部依赖
无
