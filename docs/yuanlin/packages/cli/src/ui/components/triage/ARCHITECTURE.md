# triage 架构

> 问题分诊组件，用于 GitHub Issue 的自动化分析和去重

## 概述

`triage` 目录包含用于 GitHub Issue 分诊的 UI 组件。这些组件集成了 Gemini LLM 的能力，支持自动分析 Issue 是否应关闭，以及检测重复 Issue。这是面向项目维护者的内部工具，通过 `/oncall` 命令触发。

## 架构图

```mermaid
graph TD
    OncallCommand[/oncall 命令] --> TriageIssues[TriageIssues]
    OncallCommand --> TriageDuplicates[TriageDuplicates]

    TriageIssues --> GH[gh CLI]
    TriageIssues --> LLM[Gemini LLM 分析]
    TriageIssues --> IssueResult[关闭/保留建议]

    TriageDuplicates --> GH
    TriageDuplicates --> LLM
    TriageDuplicates --> DuplicateResult[重复候选排名]
```

## 目录结构

```
triage/
├── TriageIssues.tsx      # Issue 分诊分析组件
└── TriageDuplicates.tsx  # 重复 Issue 检测组件
```

## 关键文件

| 文件 | 功能 |
|------|------|
| `TriageIssues.tsx` | 使用 `gh` CLI 获取 Issue 列表，通过 Gemini 分析每个 Issue，给出关闭/保留建议和建议评论 |
| `TriageDuplicates.tsx` | 检测可能重复的 Issue，使用 LLM 对候选 Issue 进行打分和排名 |

## 内部依赖

- `../../hooks/useKeypress` - 键盘导航
- `../../hooks/useKeyMatchers` - 键绑定匹配
- `../../key/keyMatchers` - Command 枚举
- `../shared/TextInput` - 文本输入
- `../shared/text-buffer` - 文本缓冲区

## 外部依赖

| 包名 | 用途 |
|------|------|
| `ink` | Box、Text 组件 |
| `ink-spinner` | 加载动画 |
| `react` | useState、useEffect、useCallback、useRef |
| `@google/gemini-cli-core` | debugLogger、spawnAsync（执行 gh CLI）、LlmRole、Config |
