# commands 架构

> 斜杠命令（Slash Commands）定义和实现，提供 CLI 内所有交互式命令

## 概述

`commands` 目录定义了 Gemini CLI 中所有可用的斜杠命令（以 `/` 开头的命令）。每个命令文件导出一个 `SlashCommand` 对象，包含命令名、描述、子命令和执行动作。命令系统支持子命令、参数补全、并发安全执行等特性。这些命令在 `slashCommandProcessor` Hook 中被注册和调度。

## 架构图

```mermaid
graph TD
    SlashCommandProcessor[slashCommandProcessor] --> CommandRegistry[命令注册表]
    CommandRegistry --> BuiltInCommands[内置命令]

    BuiltInCommands --> aboutCommand[/about]
    BuiltInCommands --> authCommand[/auth]
    BuiltInCommands --> chatCommand[/chat & /resume]
    BuiltInCommands --> clearCommand[/clear]
    BuiltInCommands --> compressCommand[/compress]
    BuiltInCommands --> copyCommand[/copy]
    BuiltInCommands --> docsCommand[/docs]
    BuiltInCommands --> extensionsCommand[/extensions]
    BuiltInCommands --> helpCommand[/help]
    BuiltInCommands --> memoryCommand[/memory]
    BuiltInCommands --> modelCommand[/model]
    BuiltInCommands --> quitCommand[/quit]
    BuiltInCommands --> settingsCommand[/settings]
    BuiltInCommands --> themeCommand[/theme]
    BuiltInCommands --> toolsCommand[/tools]

    CommandContext[CommandContext] --> Services[services]
    CommandContext --> UI[ui]
    CommandContext --> Session[session]
```

## 目录结构

```
commands/
├── types.ts                # 命令类型定义（CommandContext, SlashCommand 等）
├── aboutCommand.ts         # /about - 显示版本信息
├── agentsCommand.ts        # /agents - 管理 AI 代理
├── authCommand.ts          # /auth - 认证管理
├── bugCommand.ts           # /bug - 提交 Bug 报告
├── chatCommand.ts          # /chat (resume) - 会话管理
├── clearCommand.ts         # /clear - 清屏
├── commandsCommand.ts      # /commands - 列出所有命令
├── compressCommand.ts      # /compress - 压缩上下文
├── copyCommand.ts          # /copy - 复制到剪贴板
├── corgiCommand.ts         # /corgi - 彩蛋模式
├── directoryCommand.tsx    # /directory - 工作区目录管理
├── docsCommand.ts          # /docs - 打开文档
├── editorCommand.ts        # /editor - 编辑器设置
├── extensionsCommand.ts    # /extensions - 扩展管理
├── footerCommand.tsx       # /footer - 页脚配置
├── helpCommand.ts          # /help - 帮助信息
├── hooksCommand.ts         # /hooks - Hook 管理
├── ideCommand.ts           # /ide - IDE 集成
├── initCommand.ts          # /init - 初始化 GEMINI.md
├── mcpCommand.ts           # /mcp - MCP 服务器管理
├── memoryCommand.ts        # /memory - 内存/上下文管理
├── modelCommand.ts         # /model - 模型选择
├── oncallCommand.tsx       # /oncall - 值班相关（triage）
├── permissionsCommand.ts   # /permissions - 权限管理
├── planCommand.ts          # /plan - 计划模式
├── policiesCommand.ts      # /policies - 策略管理
├── privacyCommand.ts       # /privacy - 隐私设置
├── profileCommand.ts       # /profile - 配置文件
├── quitCommand.ts          # /quit - 退出
├── restoreCommand.ts       # /restore - 恢复文件
├── resumeCommand.ts        # /resume - 恢复会话
├── rewindCommand.tsx       # /rewind - 回退操作
├── settingsCommand.ts      # /settings - 设置
├── setupGithubCommand.ts   # /setup-github - GitHub Actions 配置
├── shellsCommand.ts        # /shells - 后台 Shell 管理
├── shortcutsCommand.ts     # /shortcuts - 快捷键帮助
├── skillsCommand.ts        # /skills - 技能列表
├── statsCommand.ts         # /stats - 统计信息
├── terminalSetupCommand.ts # /terminal-setup - 终端配置
├── themeCommand.ts         # /theme - 主题切换
├── toolsCommand.ts         # /tools - 工具列表
├── upgradeCommand.ts       # /upgrade - 升级 CLI
└── vimCommand.ts           # /vim - Vim 模式切换
```

## 关键文件

| 文件 | 功能 |
|------|------|
| `types.ts` | 定义 CommandContext、SlashCommand 接口、CommandKind 枚举以及各种 ActionReturn 类型 |
| `chatCommand.ts` | 会话管理命令（list/save/resume/delete/share），支持参数补全 |
| `memoryCommand.ts` | 内存管理命令（show/add/reload/list），管理 GEMINI.md 上下文 |
| `extensionsCommand.ts` | 扩展管理命令（list/update/enable/disable/add/remove） |
| `mcpCommand.ts` | MCP 服务器管理命令（list/auth/reload/enable/disable） |
| `settingsCommand.ts` | 打开设置对话框 |

## 内部依赖

- `../types` - HistoryItem、ConfirmationRequest 等类型
- `../hooks/useHistoryManager` - 历史管理器返回类型
- `../state/extensions` - 扩展更新状态和 Action 类型
- `../contexts/SessionContext` - 会话统计状态

## 外部依赖

| 包名 | 用途 |
|------|------|
| `@google/gemini-cli-core` | Config、GitService、Logger、CommandActionReturn、AgentDefinition |
| `react` | ReactNode 类型 |
