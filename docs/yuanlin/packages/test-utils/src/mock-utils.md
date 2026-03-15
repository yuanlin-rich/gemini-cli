# mock-utils.ts

> 提供测试用的 Mock 工厂函数，用于快速构造 `SandboxConfig` 等配置对象。

## 概述

`mock-utils.ts` 是 `test-utils` 包中的 Mock 工具模块。目前仅包含一个工厂函数 `createMockSandboxConfig`，用于在测试中快速创建带有合理默认值的 `SandboxConfig` 对象，并支持通过 `Partial<SandboxConfig>` 进行局部覆盖。

设计动机：在编写涉及沙箱配置的单元测试时，测试代码需要频繁构造 `SandboxConfig` 对象。将默认值集中管理在一个工厂函数中，可以：
- 减少测试代码的冗余
- 当 `SandboxConfig` 接口新增必填字段时，只需修改工厂函数，所有测试自动适配
- 测试用例只需指定与默认值不同的字段，提高可读性

在模块中的角色：这是一个轻量级的辅助模块，为其他测试文件提供类型安全的 Mock 对象生成能力。

## 架构图

```mermaid
graph LR
    subgraph "mock-utils.ts"
        CMSC["createMockSandboxConfig()"]
    end

    CORE["@google/gemini-cli-core"] -->|"SandboxConfig 类型"| CMSC
    CMSC -->|"返回"| SC["SandboxConfig 实例"]
```

## 主要导出

### `createMockSandboxConfig(overrides?: Partial<SandboxConfig>): SandboxConfig`

创建一个 `SandboxConfig` Mock 对象。

**默认值：**

| 字段 | 默认值 | 说明 |
|---|---|---|
| `enabled` | `true` | 沙箱功能开启 |
| `allowedPaths` | `[]` | 无额外允许的路径 |
| `networkAccess` | `false` | 禁止网络访问 |

**参数：**
- `overrides`（可选）：`Partial<SandboxConfig>` 类型，传入的字段将覆盖对应默认值。使用展开运算符 `...overrides` 实现浅合并。

**使用示例：**

```typescript
// 使用全部默认值
const config1 = createMockSandboxConfig();
// { enabled: true, allowedPaths: [], networkAccess: false }

// 覆盖部分字段
const config2 = createMockSandboxConfig({ networkAccess: true, allowedPaths: ['/tmp'] });
// { enabled: true, allowedPaths: ['/tmp'], networkAccess: true }
```

## 核心逻辑

实现非常简洁，采用经典的"默认值 + 展开覆盖"模式：

```typescript
export function createMockSandboxConfig(
  overrides?: Partial<SandboxConfig>,
): SandboxConfig {
  return {
    enabled: true,
    allowedPaths: [],
    networkAccess: false,
    ...overrides,
  };
}
```

该模式确保：
1. 不传参数时返回完整的默认配置
2. 传入部分字段时只覆盖指定字段，其余保持默认值
3. 返回值始终满足 `SandboxConfig` 类型约束

## 内部依赖

无（此模块不依赖包内其他模块）。

## 外部依赖

| npm 包 | 用途 |
|---|---|
| `@google/gemini-cli-core` | 导入 `SandboxConfig` 类型定义（仅类型导入，`import type`） |
