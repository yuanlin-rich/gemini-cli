# keyMatchers.ts

> 创建命令到按键匹配函数的映射，支持默认和自定义键绑定

## 概述

`keyMatchers.ts` 是键绑定系统的消费端适配层。它将 `KeyBindingConfig`（命令→键绑定列表的映射）转化为 `KeyMatchers`（命令→匹配函数的映射），使得 UI 组件可以用简单的 `matchers.QUIT(key)` 调用来判断按键是否匹配某个命令，而无需直接操作键绑定配置。

## 架构图（mermaid）

```mermaid
graph TD
    A[KeyBindingConfig] -->|createKeyMatchers| B[KeyMatchers]
    B --> C["matchers[Command.QUIT] = (key) => boolean"]
    B --> D["matchers[Command.UNDO] = (key) => boolean"]
    B --> E["...每个 Command 一个匹配函数"]

    F[loadKeyMatchers] -->|加载自定义配置| G[loadCustomKeybindings]
    G -->|创建匹配器| A
    G -->|返回错误| H[errors: string[]]
```

## 主要导出

| 名称 | 类型 | 说明 |
|------|------|------|
| `KeyMatchers` | `type` | `{ readonly [C in Command]: (key: Key) => boolean }` |
| `createKeyMatchers` | `function` | 从 `KeyBindingConfig` 创建 `KeyMatchers` |
| `defaultKeyMatchers` | `KeyMatchers` | 使用默认配置创建的匹配器实例 |
| `loadKeyMatchers` | `async function` | 加载自定义键绑定并创建匹配器，同时返回加载错误 |
| `Command` | re-export | 从 `keyBindings.js` 重新导出 |

## 核心逻辑

1. `matchCommand` 内部函数：遍历命令的所有绑定，检查是否有任何一个匹配传入的 `Key` 事件
2. `createKeyMatchers`：遍历所有 `Command` 枚举值，为每个命令创建一个闭包匹配函数
3. `loadKeyMatchers`：异步加载用户自定义键绑定，创建合并后的匹配器

## 内部依赖

| 模块 | 用途 |
|------|------|
| `../hooks/useKeypress.js` → `Key` | 按键事件类型 |
| `./keyBindings.js` | `Command`, `defaultKeyBindingConfig`, `loadCustomKeybindings`, `KeyBindingConfig` |

## 外部依赖

无
