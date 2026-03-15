# inputBlocker.ts

> 浏览器输入拦截器，在自动化期间屏蔽用户输入并显示状态提示

## 概述

`inputBlocker.ts` 实现了浏览器自动化期间的用户输入屏蔽功能。它通过注入一个透明的全屏覆盖层来拦截所有用户输入事件（点击、键盘、触摸、滚轮等），同时在页面底部显示一个胶囊状浮动提示条（"Gemini CLI is controlling this browser"）。

与 `automationOverlay`（蓝色边框，仅视觉指示）不同，`inputBlocker` 是功能性的——它实际阻止用户与页面交互，防止用户操作干扰 AI 自动化流程。

关键创新设计：覆盖层是**持久性的**（整个会话期间不移除）。为了让 CDP 工具调用能正常操作页面元素，采用 `pointer-events` CSS 属性切换（suspend/resume）而非 DOM 增删，实现零闪烁的输入控制。

## 架构图

```mermaid
stateDiagram-v2
    [*] --> 未注入

    未注入 --> 活跃: injectInputBlocker()
    活跃 --> 挂起: suspendInputBlocker()
    挂起 --> 活跃: resumeInputBlocker()
    活跃 --> 已移除: removeInputBlocker()
    挂起 --> 已移除: removeInputBlocker()

    state 活跃 {
        note right of 活跃: pointer-events: auto\n用户输入被拦截
    }
    state 挂起 {
        note right of 挂起: pointer-events: none\nCDP工具可交互\n覆盖层不可见于hit-testing
    }
```

## 主要导出

### `injectInputBlocker(browserManager: BrowserManager): Promise<void>`

注入输入拦截覆盖层。幂等操作——若已存在则仅重置 `pointer-events: auto`，跳过完整的 DOM 创建流程。

### `removeInputBlocker(browserManager: BrowserManager): Promise<void>`

完全移除输入拦截覆盖层和关联的 `<style>` 元素。仅在最终清理阶段调用。

### `suspendInputBlocker(browserManager: BrowserManager): Promise<void>`

临时挂起输入拦截——将覆盖层的 `pointer-events` 设为 `none`，使其对 hit-testing 不可见。CDP 工具调用执行前调用此函数。

### `resumeInputBlocker(browserManager: BrowserManager): Promise<void>`

恢复输入拦截——将 `pointer-events` 恢复为 `auto`。CDP 工具调用完成后调用。

## 核心逻辑

### 注入脚本（INPUT_BLOCKER_FUNCTION）

注入的 JavaScript 创建以下 DOM 结构：

```
<div id="__gemini_input_blocker">  ← 全屏透明覆盖层
  <div>                            ← 胶囊状浮动提示条
    <span>                         ← 脉冲红点（CSS 动画）
    <span>Gemini CLI is controlling this browser</span>
    <span>|</span>                 ← 分隔线
    <span>Input disabled during automation</span>
  </div>
</div>
<style id="__gemini_input_blocker_style">  ← 脉冲动画 @keyframes
```

事件拦截列表（15 种事件，全部使用 `capture: true`）：
- 鼠标：`click`, `mousedown`, `mouseup`, `dblclick`, `contextmenu`
- 指针：`pointerdown`, `pointerup`, `pointermove`
- 键盘：`keydown`, `keyup`, `keypress`
- 触摸：`touchstart`, `touchend`, `touchmove`
- 其他：`wheel`

每个事件处理器调用 `preventDefault()` + `stopPropagation()` + `stopImmediatePropagation()` 三重阻止。

### 覆盖层样式

- `position: fixed; inset: 0` 覆盖整个视口
- `z-index: 2147483646`（比自动化蓝色边框低 1）
- `cursor: not-allowed` 鼠标样式提示
- `background: transparent` 不遮挡页面视觉

### 提示条 UI 设计

- 底部居中浮动，胶囊形状（`border-radius: 999px`）
- 深色半透明背景 + 毛玻璃效果（`backdrop-filter: blur(16px)`）
- 入场动画：从下方滑入 + 淡入（CSS transition）
- 脉冲红点：`@keyframes __gemini_pulse` 2 秒循环
- 提示条本身 `pointer-events: none` 不影响覆盖层的事件捕获

### 幂等性优化

重新注入时（如页面导航后），若覆盖层已存在，仅执行 `existing.style.pointerEvents = 'auto'` 一行操作，避免完整的 DOM 重建开销。

### 错误容忍

所有四个导出函数都采用 try-catch 包装，失败仅记录日志。输入拦截是 UX 增强功能，不应阻断自动化核心流程。

## 内部依赖

| 模块 | 导入内容 | 用途 |
|------|---------|------|
| `./browserManager.js` | `BrowserManager` (type) | 通过 `callTool('evaluate_script', ...)` 执行页面脚本 |
| `../../utils/debugLogger.js` | `debugLogger` | 日志输出 |

## 外部依赖

无。
