# tips.ts

> 定义 CLI 加载界面显示的信息类提示文本列表

## 概述

`tips.ts` 导出一个包含 160+ 条用户提示的字符串数组 `INFORMATIVE_TIPS`，这些提示按类别组织：设置提示、键盘快捷键提示和命令提示。在 CLI 加载等待时随机展示，帮助用户发现新功能和快捷操作。

## 架构图（mermaid）

```mermaid
graph TD
    A[INFORMATIVE_TIPS] --> B[设置提示 ~82条]
    A --> C[键盘快捷键提示 ~37条]
    A --> D[命令提示 ~42条]

    B --> B1[编辑器/Vim/更新/主题设置]
    B --> B2[UI显示控制设置]
    B --> B3[上下文/内存管理设置]
    B --> B4[安全/沙箱/工具设置]
    B --> B5[遥测/调试设置]

    C --> C1[基础操作: Esc/Ctrl+C/Ctrl+D]
    C --> C2[导航: 方向键/Ctrl+P/N]
    C --> C3[编辑: Ctrl+W/U/K]
    C --> C4[功能切换: Ctrl+Y/Alt+M]

    D --> D1[会话管理: /resume]
    D --> D2[工具管理: /tools/mcp]
    D --> D3[上下文管理: /memory/directory]
    D --> D4[配置: /settings/model/theme]
```

## 主要导出

| 名称 | 类型 | 说明 |
|------|------|------|
| `INFORMATIVE_TIPS` | `string[]` | 信息提示文本数组，每条以省略号结尾 |

## 核心逻辑

纯数据定义文件。提示内容涵盖：
- **设置提示**：指导用户通过 `/settings` 或 `settings.json` 进行各项配置
- **键盘快捷键提示**：介绍常用的快捷键操作
- **命令提示**：介绍各种斜杠命令的用法

## 内部依赖

无

## 外部依赖

无
