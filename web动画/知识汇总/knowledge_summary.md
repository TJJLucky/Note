# Web 前端高性能渲染与动画核心知识体系

本文档汇总了本项目涉及的核心知识点，重点梳理了浏览器渲染管线、Event Loop 机制及极端情况下的调度策略。

---

## 1. 渲染管线 (The Rendering Pipeline)

为了保证 60FPS 的流畅度，浏览器必须在 **16.6ms** 内完成以下流程：

1.  **JavaScript**: 处理业务逻辑，修改 DOM 或 CSS。
2.  **Style**: 计算每个元素的最终计算样式 (Computed Styles)。
3.  **Layout (回流)**: 计算元素在屏幕上的几何位置 (x, y, width, height)。**成本最高**。
4.  **Paint (重绘)**: 填充像素 (文本、颜色、阴影)。成本次之。
5.  **Composite (合成)**: 将图层 (Layers) 合并并上传 GPU 显示。**成本最低**。

> **优化原则**: 优先使用 `transform` / `opacity` 做动画，只触发 Composite 阶段。

---

## 2. 动画触发机制与 Event Loop

### 2.1 CSS Transition 原理
Transition 本质是**状态插值**。它依赖浏览器捕捉到两个不同的状态“快照”。
*   **Frame N (Start)**: 元素 `display: block`, `opacity: 0`。
*   **Frame N+1 (End)**: 元素 `display: block`, `opacity: 1`。
只有当 Frame N 的样式被**提交 (Commit)** 后，Frame N+1 的样式变化才会产生 Diff，从而触发动画。

### 2.2 `requestAnimationFrame (rAF)`
*   **时机**: 在**渲染阶段之前**执行。
*   **核心作用**: 确保回调函数与屏幕刷新率 (VSync) 同步。

### 2.3 双层 rAF 及其必要性
当我们需要在“下一帧”执行逻辑时（例如先 `display:block`，再 `opacity:1`）：

```javascript
// Frame N (当前)
this.el.style.display = 'block';

requestAnimationFrame(() => {
    // 依然在 Frame N (或 Frame N 的渲染前夕)
    // 注册下一帧的回调
    requestAnimationFrame(() => {
        // Frame N+1
        this.el.classList.add('active'); // 此时 Frame N 已绘制，Transition 生效
    });
});
```

---

## 3. 阻塞与帧合并 (Blocking & Frame Coalescing)

**核心现象**: 当主线程被长时间阻塞 (Long Task > 16ms) 时，浏览器会错过 VSync 信号。

### 3.1 “追赶”机制 (Catch-up Behavior)
*   **积压**: rAF 回调在队列中积压。
*   **合并**: 线程释放后，浏览器可能在**一次 Loop** 中连续执行多个积压的 rAF 回调。
*   **后果**: `t1` (rAF1) 和 `t2` (rAF2) 获得极其接近的时间戳。此时 Start State 和 End State 在同一帧提交，中间无 Diff，导致**动画失效**。

### 3.2 幸存者偏差：为什么有时阻塞后动画依然生效？
如果你观察到阻塞 1s 后动画依然生效，通常是因为：
1.  **Layout Thrashing (布局抖动)**: 代码隐式触发了同步 Layout，强制浏览器更新了 Start State。
2.  **浏览器调度差异**: 某些引擎在极长任务后，可能会强制推迟渲染，反而意外地把 rAF 分配到了正确的“下一帧”。
> **注意**: 这不可靠！严禁依赖此行为。

---

## 4. Canvas 图形渲染 (Canvas Graphics)

### 3.1 渲染模式
*   **DOM**: 保留模式 (Retained Mode)，内存随节点增加，适合文档。
*   **Canvas**: 立即模式 (Immediate Mode)，绘制即遗忘，内存仅与分辨率有关，适合高性能图形/游戏。

### 3.2 优化技巧
*   **离屏渲染 (Offscreen)**: 预渲染复杂图形。
*   **分层 (Layering)**: 动静分离 (背景层 vs 角色层)。

---

## 5. JavaScript 国际化 (Intl)
*   使用 `Intl.NumberFormat` 替代复杂的正则进行货币格式化。

---

## 6. 最佳实践总结
1.  **严禁阻塞**: 主线程任务勿超过 50ms。
2.  **标准模板**: 涉及 DOM 状态切换的动画，始终使用 **Nested rAF**。
3.  **验证工具**: 使用 `transitionstart` 事件验证动画是否真实触发。
