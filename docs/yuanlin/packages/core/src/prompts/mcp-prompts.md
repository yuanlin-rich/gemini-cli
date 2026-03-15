# mcp-prompts.ts

> 获取指定 MCP 服务器注册的提示词列表

## 概述

`mcp-prompts.ts` 是一个轻量级辅助模块，提供从 Config 对象中检索特定 MCP 服务器提示词的便捷函数。它封装了 `Config -> PromptRegistry -> getPromptsByServer` 的访问链路，为调用方提供简洁的 API。

该文件作为 MCP 提示词系统与配置层之间的桥梁，隔离了内部访问细节。

## 架构图

```mermaid
graph LR
    A[getMCPServerPrompts] --> B[config.getPromptRegistry]
    B --> C[PromptRegistry]
    C --> D[getPromptsByServer]
    D --> E[DiscoveredMCPPrompt 数组]
```

## 主要导出

### `getMCPServerPrompts(config: Config, serverName: string): DiscoveredMCPPrompt[]`

获取指定 MCP 服务器注册的所有提示词。

**参数：**
- `config`: 全局配置对象
- `serverName`: MCP 服务器名称

**返回：**
- `DiscoveredMCPPrompt[]` - 该服务器注册的提示词列表
- 如果 PromptRegistry 不存在，返回空数组

## 核心逻辑

函数内部先获取 PromptRegistry 实例，若不存在则安全返回空数组。存在时委托给 `getPromptsByServer` 执行实际查询。

## 内部依赖

| 模块 | 用途 |
|------|------|
| `../config/config.js` | Config 类型 |
| `../tools/mcp-client.js` | DiscoveredMCPPrompt 类型 |

## 外部依赖

无外部依赖。
