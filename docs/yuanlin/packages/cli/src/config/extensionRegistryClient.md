# extensionRegistryClient.ts

> 扩展注册表客户端，负责从远程或本地注册表获取、搜索和分页浏览可用扩展。

## 概述

`extensionRegistryClient.ts` 提供了 `ExtensionRegistryClient` 类，用于与扩展注册表进行交互。注册表默认指向 `https://geminicli.com/extensions.json`，也可以配置为本地文件路径。客户端具备以下能力：

- **分页查询**：按排名或字母顺序分页获取扩展列表。
- **模糊搜索**：基于 `fzf` 算法对扩展名、描述、仓库全名进行模糊匹配。
- **单个查询**：按 ID 精确获取扩展详情。
- **请求缓存**：通过静态 `fetchPromise` 实现请求级缓存，避免重复网络请求。
- **安全校验**：拒绝私有 IP 地址，带超时的 fetch 请求（10 秒）。

## 架构图（mermaid）

```mermaid
flowchart TD
    A[ExtensionRegistryClient] --> B{registryURI 协议}
    B -->|http/https| C[fetchWithTimeout + 私有 IP 检查]
    B -->|本地路径| D[fs.readFile]
    C --> E[RegistryExtension[]]
    D --> E
    E --> F[缓存至静态 fetchPromise]

    G[getExtensions] --> E --> H[排序 + 分页]
    I[searchExtensions] --> E --> J[AsyncFzf 模糊搜索]
    K[getExtension] --> E --> L[按 ID 过滤]
```

## 主要导出

| 导出名称 | 类型 | 说明 |
|---------|------|------|
| `RegistryExtension` | `interface` | 注册表扩展条目的完整数据结构 |
| `ExtensionRegistryClient` | `class` | 扩展注册表客户端 |

### RegistryExtension 接口字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | `string` | 唯一标识 |
| `rank` | `number` | 排名 |
| `url` | `string` | 仓库 URL |
| `fullName` | `string` | 仓库全名 |
| `repoDescription` | `string` | 仓库描述 |
| `stars` | `number` | GitHub 星标数 |
| `lastUpdated` | `string` | 最后更新时间 |
| `extensionName` | `string` | 扩展名称 |
| `extensionVersion` | `string` | 扩展版本 |
| `extensionDescription` | `string` | 扩展描述 |
| `avatarUrl` | `string` | 头像 URL |
| `hasMCP` / `hasContext` / `hasHooks` / `hasSkills` / `hasCustomCommands` | `boolean` | 功能标志 |
| `isGoogleOwned` | `boolean` | 是否为 Google 官方扩展 |
| `licenseKey` | `string` | 许可证标识 |

### ExtensionRegistryClient 方法

| 方法 | 说明 |
|------|------|
| `constructor(registryURI?)` | 可选传入自定义注册表 URI，默认使用官方地址 |
| `getExtensions(page, limit, orderBy)` | 分页获取扩展，支持 `ranking` 和 `alphabetical` 排序 |
| `searchExtensions(query)` | 模糊搜索扩展 |
| `getExtension(id)` | 按 ID 获取单个扩展 |
| `resetCache()` | （内部使用）清除静态缓存 |

## 核心逻辑

### fetchAllExtensions（私有方法）

1. 检查静态缓存 `fetchPromise`，命中则直接返回。
2. HTTP URI：先检查是否为私有 IP（`isPrivateIp`），然后使用 `fetchWithTimeout` 发起 10 秒超时请求。
3. 本地路径：使用 `resolveToRealPath` 解析真实路径后 `fs.readFile` 读取。
4. 请求失败时清除缓存，确保下次可重试。

### 排序与分页

`getExtensions` 在获取全部扩展后，根据 `orderBy` 参数排序（排名数值 / 字母序），然后通过 `slice` 实现分页。

### 模糊搜索

使用 `AsyncFzf` 库，以扩展名 + 描述 + 仓库全名作为搜索字段，返回匹配结果。

## 内部依赖

无。

## 外部依赖

| 模块 | 导入内容 | 用途 |
|------|---------|------|
| `@google/gemini-cli-core` | `fetchWithTimeout`, `resolveToRealPath`, `isPrivateIp` | 网络请求、路径解析、IP 校验 |
| `fzf` | `AsyncFzf` | 模糊搜索算法 |
| `node:fs/promises` | `fs` | 异步文件读取 |
