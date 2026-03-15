# hookRegistry.ts

> 从多个配置源加载、验证和管理 Hook 定义的注册表。

## 概述

`HookRegistry` 负责收集来自不同来源（运行时注册、项目配置、扩展）的 Hook 定义，验证其合法性，并按事件名称和优先级提供查询接口。它还集成了 `TrustedHooksManager` 来处理项目级 Hook 的信任检查。

**设计动机：** Hook 可以来自多个源头——项目 `.gemini/settings.json`、用户全局配置、扩展、运行时代码注册。注册表提供统一的验证、存储和检索机制，并通过优先级排序确保高优先级源的 Hook 先执行。

**在模块中的角色：** 作为 Hook 数据的权威存储，被 `HookPlanner` 查询以获取匹配的 Hook。

## 架构图

```mermaid
flowchart TD
    subgraph 配置源
        RT[运行时注册<br/>priority: 0]
        PJ[项目配置<br/>priority: 1]
        US[用户配置<br/>priority: 2]
        SY[系统配置<br/>priority: 3]
        EX[扩展配置<br/>priority: 4]
    end

    subgraph HookRegistry
        V[验证 validateHookConfig]
        E[entries: HookRegistryEntry[]]
        T[TrustedHooksManager]
    end

    RT --> V
    PJ --> V
    EX --> V
    V --> E
    PJ --> T

    HP[HookPlanner] --> |getHooksForEvent| E
```

## 主要导出

### `interface HookRegistryEntry`

| 字段 | 类型 | 说明 |
|------|------|------|
| `config` | `HookConfig` | Hook 配置（command 或 runtime） |
| `source` | `ConfigSource` | 配置来源 |
| `eventName` | `HookEventName` | 绑定的事件名 |
| `matcher` | `string?` | 匹配模式 |
| `sequential` | `boolean?` | 是否要求顺序执行 |
| `enabled` | `boolean` | 是否启用 |

### `class HookRegistry`

#### 构造函数

```typescript
constructor(config: Config)
```

#### 公开方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `initialize()` | `(): Promise<void>` | 初始化注册表：保留运行时 Hook，重新从配置加载 |
| `registerHook` | `(config, eventName, options?)` | 编程式注册 Hook |
| `getHooksForEvent` | `(eventName): HookRegistryEntry[]` | 获取指定事件的所有启用 Hook（按优先级排序） |
| `getAllHooks` | `(): HookRegistryEntry[]` | 获取所有注册的 Hook |
| `setHookEnabled` | `(hookName, enabled)` | 启用/禁用指定名称的 Hook |

## 核心逻辑

### 初始化流程（`initialize`）

1. **保留运行时 Hook**：过滤出 `ConfigSource.Runtime` 的条目
2. **处理项目配置**：
   - 检查文件夹是否可信（`isTrustedFolder()`）
   - 调用 `TrustedHooksManager` 检查未信任的项目 Hook 并发出警告
   - 自动信任已警告的 Hook
3. **处理扩展**：遍历活跃扩展的 `hooks` 配置

### 配置处理（`processHooksConfiguration`）

1. 遍历 hooks 对象的键（事件名称）
2. 跳过非事件名字段（`enabled`、`disabled`、`notifications`）
3. 验证事件名称合法性
4. 对每个 `HookDefinition`，遍历其 `hooks` 数组
5. 验证每个 `HookConfig`，检查禁用列表
6. 标记配置来源到 `hookConfig.source`

### 验证规则（`validateHookConfig`）

- `type` 必须是 `'command'`、`'plugin'` 或 `'runtime'`
- Command Hook 必须有 `command` 字段
- Runtime Hook 必须有 `name` 字段

### 安全机制

- **不可信文件夹保护**：项目级 Hook 在不可信文件夹中被完全跳过
- **二级安全检查**：即使通过 Registry 验证，`HookRunner` 还会在执行前再次检查信任状态

### 优先级排序

| 来源 | 优先级 | 说明 |
|------|--------|------|
| Runtime | 0 | 最高，代码注册 |
| Project | 1 | 项目配置 |
| User | 2 | 用户全局配置 |
| System | 3 | 系统级配置 |
| Extensions | 4 | 扩展提供 |

## 内部依赖

| 模块 | 说明 |
|------|------|
| `../config/config.js` | Config 类型 |
| `./types.js` | HookEventName、ConfigSource、HookConfig 等 |
| `./trustedHooks.js` | TrustedHooksManager |
| `../utils/debugLogger.js` | 调试日志 |
| `../utils/events.js` | 核心事件发射（警告反馈） |

## 外部依赖

无。
