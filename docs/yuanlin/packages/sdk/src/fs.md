# fs.ts

> 提供受权限控制的文件系统读写能力，实现 `AgentFilesystem` 接口。

## 概述

`SdkAgentFilesystem` 是 SDK 中供工具（Tool）在执行上下文中安全访问文件系统的适配层。它实现了 `AgentFilesystem` 接口，在执行实际的 `fs.readFile` / `fs.writeFile` 操作之前，先通过 `Config.validatePathAccess()` 进行路径权限校验，确保工具只能在允许的目录范围内进行文件操作。

设计动机：
- 将文件系统访问与权限校验解耦，工具代码无需关心安全策略细节。
- 读取失败时返回 `null` 而非抛出异常，方便调用方做"文件不存在"的优雅降级。
- 写入时权限不足则直接抛出异常，防止静默失败。

## 架构图

```mermaid
classDiagram
    class AgentFilesystem {
        <<interface>>
        +readFile(path: string): Promise~string | null~
        +writeFile(path: string, content: string): Promise~void~
    }

    class SdkAgentFilesystem {
        -config: CoreConfig
        +constructor(config: CoreConfig)
        +readFile(path: string): Promise~string | null~
        +writeFile(path: string, content: string): Promise~void~
    }

    class CoreConfig {
        +validatePathAccess(path, mode): string | null
    }

    SdkAgentFilesystem ..|> AgentFilesystem : implements
    SdkAgentFilesystem --> CoreConfig : 权限校验
    SdkAgentFilesystem ..> fs : node:fs/promises
```

## 主要导出

### `class SdkAgentFilesystem`

实现 `AgentFilesystem` 接口。

| 成员 | 签名 | 说明 |
|------|------|------|
| 构造函数 | `constructor(config: CoreConfig)` | 注入核心配置对象，用于路径权限校验 |
| `readFile` | `readFile(path: string): Promise<string \| null>` | 读取文件内容（UTF-8）。权限不足或文件不存在时返回 `null` |
| `writeFile` | `writeFile(path: string, content: string): Promise<void>` | 写入文件内容（UTF-8）。权限不足时抛出 `Error` |

## 核心逻辑

### `readFile(path)`

1. 调用 `config.validatePathAccess(path, 'read')` 校验读取权限。
2. 若校验返回错误信息，直接返回 `null`（视为不可读）。
3. 调用 `fs.readFile(path, 'utf-8')` 读取文件。
4. 若 `fs.readFile` 抛出异常（如文件不存在），捕获后返回 `null`。

### `writeFile(path, content)`

1. 调用 `config.validatePathAccess(path, 'write')` 校验写入权限。
2. 若校验返回错误信息，抛出 `Error(error)` 中断操作。
3. 调用 `fs.writeFile(path, content, 'utf-8')` 写入文件。

## 内部依赖

| 模块 | 导入项 | 说明 |
|------|--------|------|
| `./types.js` | `AgentFilesystem`（类型） | 文件系统接口定义 |

## 外部依赖

| 包 | 导入项 | 说明 |
|----|--------|------|
| `node:fs/promises` | `fs`（默认导入） | Node.js 异步文件系统 API |
| `@google/gemini-cli-core` | `Config`（类型，别名 `CoreConfig`） | 核心配置类，提供 `validatePathAccess` 方法 |
