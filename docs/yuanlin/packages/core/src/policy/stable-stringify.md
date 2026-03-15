# stable-stringify.ts

> 确定性 JSON 序列化：为策略参数匹配提供键排序的稳定字符串输出

## 概述

`stable-stringify.ts` 提供了一个关键的安全基础设施函数 `stableStringify`。标准 `JSON.stringify` 的输出取决于属性插入顺序，这在不同 JavaScript 引擎或运行时条件下可能不一致。该函数通过排序对象键并处理特殊情况，确保相同的对象始终产生相同的字符串表示，从而使策略规则的 `argsPattern` 正则匹配可靠且可预测。

此外，该函数还在顶层属性上注入 `\0`（null 字节）边界标记，配合 `buildParamArgsPattern` 实现精确的顶层属性匹配，防止参数注入攻击。

## 架构图

```mermaid
graph TD
    A[stableStringify] --> B[递归 stringify]

    B --> C{类型判断}
    C -->|原始类型/null| D[JSON.stringify 直接序列化]
    C -->|undefined/function| E[返回 'null']
    C -->|object| F{循环引用检测}

    F -->|是循环引用| G[返回 '"[Circular]"']
    F -->|否| H{检查 toJSON 方法}

    H -->|有 toJSON| I[调用 toJSON 并递归]
    H -->|无 toJSON| J{数组 or 对象?}

    J -->|数组| K[逐元素递归]
    J -->|对象| L[排序键 -> 逐键递归]

    L -->|顶层| M[添加 \0 边界标记]
    L -->|非顶层| N[标准 key:value 对]

    subgraph 安全特性
        G
        M
    end
```

## 主要导出

### `stableStringify(obj: unknown): string`

产生稳定的、确定性的 JSON 字符串表示。

**关键行为：**

1. **键排序**：对象属性总是按字母序排列
2. **循环引用保护**：使用祖先链跟踪检测真正的循环引用，替换为 `"[Circular]"`
3. **JSON 规范兼容**：
   - `undefined` 在对象中省略，在数组中转为 `null`
   - 函数在对象中省略，在数组中转为 `null`
   - 尊重 `toJSON` 方法
4. **顶层边界标记**：顶层属性键值对被 `\0` 包围，用于防止参数注入

**示例：**

```typescript
stableStringify({b: 2, a: 1})
// 返回: '{\0"a":1\0,\0"b":2\0}'

// 循环引用安全处理
const obj = {a: 1}; obj.self = obj;
stableStringify(obj)
// 返回: '{\0"a":1\0,\0"self":"[Circular]"\0}'
```

## 核心逻辑

### 循环引用检测

使用 `Set<unknown>` 跟踪当前遍历的祖先对象链。进入对象时 `add`，离开时 `delete`（在 `finally` 中确保）。这与简单的 `visited` 集合不同，它能正确处理同一对象在非循环路径上被多次引用的情况。

### 顶层 `\0` 标记

顶层属性对会被包装为 `\0"key":value\0`，这使得 `buildParamArgsPattern` 生成的正则可以精确匹配顶层属性而不会误匹配嵌套属性。`\0` 字符是安全的分隔符，因为用户数据中的任何 `\0` 字符在经过 `JSON.stringify` 后会变为 `\u0000`，不会与结构性标记冲突。

## 内部依赖

无内部依赖。

## 外部依赖

无外部依赖。这是一个纯函数实现。
