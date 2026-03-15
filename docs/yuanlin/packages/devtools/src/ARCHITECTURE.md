# devtools/src 架构

> DevTools 服务端核心，实现 HTTP/WebSocket 服务器、会话管理和日志收集。

## 概述

`src` 目录包含 DevTools 的服务端逻辑。核心 `DevTools` 类（单例模式）内嵌一个 HTTP 服务器和 WebSocket 服务器。HTTP 服务器提供静态资源（HTML/JS）和 SSE 事件流端点。WebSocket 服务器接收来自 CLI 会话的注册消息、网络日志和控制台日志。日志数据存储在内存中（网络日志最多 2000 条，控制台日志最多 5000 条），通过 EventEmitter 事件驱动推送给所有 SSE 客户端。支持端口自动递增重试（默认 25417，最多重试 10 次）。

## 架构图

```mermaid
graph TD
    DevTools[DevTools 单例] --> HTTPServer[HTTP Server]
    DevTools --> WSS[WebSocketServer]
    DevTools --> Logs[内存日志存储]
    DevTools --> Sessions[会话 Map]

    HTTPServer --> ServeHTML[提供 HTML/JS]
    HTTPServer --> SSE[/events SSE 端点]
    SSE --> Snapshot[发送完整快照]
    SSE --> Incremental[增量更新推送]

    WSS --> Register[会话注册]
    WSS --> NetworkMsg[网络日志消息]
    WSS --> ConsoleMsg[控制台日志消息]
    WSS --> Heartbeat[心跳检测]

    Register --> Sessions
    NetworkMsg --> Logs
    ConsoleMsg --> Logs
    Logs --> |EventEmitter| SSE
```

## 关键文件

| 文件 | 功能 |
|------|------|
| `index.ts` | `DevTools` 类：单例模式（`getInstance()`）；`start()` 启动 HTTP+WS 服务器；`stop()` 优雅关闭。HTTP 端点：`/` 提供 HTML、`/assets/main.js` 提供客户端 JS、`/events` 提供 SSE 流。WebSocket 处理：会话注册/注销、网络和控制台日志接收、心跳（30s 超时自动断开）。安全：仅允许同源请求，禁止跨域 |
| `types.ts` | `NetworkLog`：网络请求日志（URL、method、headers、body、response、chunks 等）。`ConsoleLogPayload`：控制台日志载荷（type + content）。`InspectorConsoleLog`：扩展 ConsoleLogPayload 添加 id、sessionId、timestamp |
| `_client-assets.ts` | 自动生成文件，包含 INDEX_HTML 和 CLIENT_JS 字符串常量，使服务器无需运行时文件读取 |

## 内部依赖

- `types.ts` - NetworkLog、ConsoleLogPayload、InspectorConsoleLog
- `_client-assets.ts` - 内嵌的 HTML 和 JS 资源

## 外部依赖

| 包名 | 用途 |
|------|------|
| `ws` | WebSocket 服务器实现 |
| `node:http` | HTTP 服务器 |
| `node:crypto` | randomUUID 生成日志 ID |
| `node:events` | EventEmitter 基类 |
