# google-auth-provider.ts

> 基于 Google Application Default Credentials (ADC) 的 MCP 认证提供者

## 概述

`GoogleCredentialProvider` 实现了 `McpAuthProvider` 接口，使用 Google ADC（应用默认凭据）为 MCP 服务端请求提供访问令牌。它专门用于连接 Google API 服务（`*.googleapis.com` 和 `*.luci.app`）。

该提供者无需用户手动登录，直接利用本地 gcloud 配置或服务账号凭据自动获取令牌，并实现了带 5 分钟缓冲区的令牌缓存机制。

## 架构图

```mermaid
graph TD
    GoogleCredentialProvider["GoogleCredentialProvider"]
    GoogleAuth["GoogleAuth (google-auth-library)"]
    ADC["Application Default Credentials"]
    MCPTransport["MCP 传输层"]

    GoogleCredentialProvider -->|获取令牌| GoogleAuth
    GoogleAuth -->|读取| ADC
    GoogleCredentialProvider -->|tokens() + getRequestHeaders()| MCPTransport
```

## 主要导出

### `GoogleCredentialProvider` (类)

实现 `McpAuthProvider` 接口。

| 方法 | 签名 | 用途 |
|------|------|------|
| `constructor` | `constructor(config?: MCPServerConfig)` | 验证 URL 白名单和 scopes 配置 |
| `tokens` | `tokens(): Promise<OAuthTokens \| undefined>` | 返回缓存的或新获取的访问令牌 |
| `getRequestHeaders` | `getRequestHeaders(): Promise<Record<string, string>>` | 返回 `X-Goog-User-Project` 等自定义头 |
| `getQuotaProjectId` | `getQuotaProjectId(): Promise<string \| undefined>` | 获取配额项目 ID |
| `clientInformation` | `clientInformation(): OAuthClientInformation \| undefined` | 返回客户端信息 |
| `saveClientInformation` | `saveClientInformation(info): void` | 保存客户端信息 |
| `saveTokens` | `saveTokens(_tokens): void` | 无操作（ADC 自管理令牌） |
| `redirectToAuthorization` | `redirectToAuthorization(_url): void` | 无操作 |
| `saveCodeVerifier` | `saveCodeVerifier(_v): void` | 无操作 |
| `codeVerifier` | `codeVerifier(): string` | 返回空字符串 |

## 核心逻辑

### 构造函数安全检查

1. **URL 白名单**: 从 `config.url` 或 `config.httpUrl` 提取主机名，必须匹配 `*.googleapis.com` 或 `*.luci.app`，否则抛出错误
2. **Scopes 必需**: `config.oauth.scopes` 必须非空

### 令牌缓存机制

`tokens()` 方法实现两级策略：
1. 检查缓存令牌是否在有效期内（当前时间 < 过期时间 - 5 分钟缓冲）
2. 缓存失效时，通过 `GoogleAuth.getClient().getAccessToken()` 获取新令牌
3. 新令牌带有 `expiry_date` 时缓存；否则不缓存（每次请求都获取新令牌）

### 自定义请求头

`getRequestHeaders()` 按优先级构建 `X-Goog-User-Project` 头：
1. 配置中已有该头（大小写不敏感匹配）-> 使用配置值
2. 否则 -> 从 ADC 客户端获取 `quotaProjectId`

## 内部依赖

| 模块 | 用途 |
|------|------|
| `auth-provider.ts` | `McpAuthProvider` 接口 |
| `oauth-utils.ts` | `FIVE_MIN_BUFFER_MS` 常量 |
| `../config/config.js` | `MCPServerConfig` 类型 |
| `../utils/events.js` | `coreEvents` 错误反馈 |

## 外部依赖

| 包 | 用途 |
|---|------|
| `google-auth-library` | `GoogleAuth`，ADC 凭据管理 |
| `@modelcontextprotocol/sdk/shared/auth.js` | OAuth 类型定义 |
