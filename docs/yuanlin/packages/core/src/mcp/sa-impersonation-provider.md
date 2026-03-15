# sa-impersonation-provider.ts

> 基于 Google 服务账号模拟的 MCP 认证提供者，通过 IAM API 获取 ID Token

## 概述

`ServiceAccountImpersonationProvider` 实现了 `McpAuthProvider` 接口，通过 Google IAM Credentials API 模拟目标服务账号来生成 OIDC ID Token。它适用于需要以特定服务账号身份访问 MCP 服务端的场景。

与 `GoogleCredentialProvider` 不同，本提供者不直接使用 ADC 的访问令牌，而是用 ADC 凭据调用 IAM `generateIdToken` API，获取面向特定受众（audience）的 ID Token，并将其放入 `access_token` 字段以构造 `Authorization: Bearer` 请求头。

## 架构图

```mermaid
graph TD
    SAProvider["ServiceAccountImpersonationProvider"]
    GoogleAuth["GoogleAuth (ADC)"]
    IAMAPI["IAM Credentials API"]
    IDToken["OIDC ID Token"]
    MCPServer["MCP 服务端"]

    SAProvider -->|获取 ADC 客户端| GoogleAuth
    GoogleAuth -->|调用| IAMAPI
    IAMAPI -->|返回| IDToken
    SAProvider -->|Authorization: Bearer {IDToken}| MCPServer
```

## 主要导出

### `ServiceAccountImpersonationProvider` (类)

实现 `McpAuthProvider` 接口。

| 方法 | 签名 | 用途 |
|------|------|------|
| `constructor` | `constructor(config: MCPServerConfig)` | 验证必需配置：url/httpUrl、targetAudience、targetServiceAccount |
| `tokens` | `tokens(): Promise<OAuthTokens \| undefined>` | 返回缓存的或新获取的 ID Token |
| `clientInformation` | `clientInformation(): OAuthClientInformation \| undefined` | 返回客户端信息 |
| `saveClientInformation` | `saveClientInformation(info): void` | 保存客户端信息 |
| `saveTokens` | `saveTokens(_tokens): void` | 无操作 |
| `redirectToAuthorization` | `redirectToAuthorization(_url): void` | 无操作 |
| `saveCodeVerifier` | `saveCodeVerifier(_v): void` | 无操作 |
| `codeVerifier` | `codeVerifier(): string` | 返回空字符串 |

## 核心逻辑

### 构造函数验证

要求配置中提供三个必需参数：
- `url` 或 `httpUrl` -- MCP 服务端地址
- `targetAudience` -- OAuth 客户端 ID（作为 ID Token 的 audience）
- `targetServiceAccount` -- 要模拟的服务账号邮箱

### 令牌获取流程 (`tokens`)

1. **缓存检查**: 如果缓存令牌在有效期内（当前时间 < 过期时间 - 5 分钟），直接返回
2. **清除缓存**: 过期或无缓存时清空
3. **调用 IAM API**: 通过 ADC 客户端向 `https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/{sa}:generateIdToken` 发送 POST 请求，参数包含 `audience` 和 `includeEmail: true`
4. **解析过期时间**: 使用 `OAuthUtils.parseTokenExpiry()` 从 JWT 中提取 `exp` 字段
5. **构造令牌对象**: 将 ID Token 放入 `access_token` 字段（这是故意的设计，因为 MCP 传输层使用该字段构建 Bearer 头）
6. **缓存新令牌**: 仅在成功解析过期时间时缓存

### 辅助函数

```typescript
function createIamApiUrl(targetSA: string): string
```

构建 IAM generateIdToken API 的 URL，对服务账号邮箱进行 URI 编码。

## 内部依赖

| 模块 | 用途 |
|------|------|
| `auth-provider.ts` | `McpAuthProvider` 接口 |
| `oauth-utils.ts` | `OAuthUtils.parseTokenExpiry()`, `FIVE_MIN_BUFFER_MS` |
| `../config/config.js` | `MCPServerConfig` 类型 |
| `../utils/events.js` | `coreEvents` 错误反馈 |

## 外部依赖

| 包 | 用途 |
|---|------|
| `google-auth-library` | `GoogleAuth`，ADC 凭据获取 |
| `@modelcontextprotocol/sdk/shared/auth.js` | OAuth 类型定义 |
