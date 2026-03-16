# CSS Transition 核心机制与实战陷阱

CSS Transition（过渡）是前端最基础的动画技术，但它经常因为浏览器底层的渲染优化机制而“失效”。本文总结了 Transition 生效的核心原理及常见陷阱解决。

---

## 1. 核心原理：补间动画 (Tweening)

Transition 的本质是计算两个明确状态之间的**插值**。

$$ \text{动画} = \text{State A (起点)} \xrightarrow{\text{Duration}} \text{State B (终点)} $$

为了让过渡生效，浏览器必须在渲染流水线中明确捕获到这两个状态的时间差。

### 关键三要素
1.  **明确的起始值**：元素必须已经被渲染到 Render Tree 上，且拥有计算好的样式（Computed Style）。
2.  **明确的结束值**：必须是具体的数值（`100px`, `1`），不能是像 `auto` 这种依赖布局计算的值。
3.  **时间差 (跨帧 Diff)**：Transition 的本质是浏览器在 **Style (计算样式)** 阶段，对 **“前一帧的样式快照”** 与 **“当前帧的新样式”** 进行 Diff (比对) 的结果。只有检测到值发生了变化，才会生成动画插值。

---

## 2. 经典陷阱：为什么动画会失效？

### 场景 A：从 `display: none` 出现
当你把元素从 `display: none` 改为 `display: block` 时，Transition 往往不生效。
*   **原因**：`display: none` 的元素不在 Render Tree 中，它**没有初始状态 (State A)**。浏览器第一次画它时，它就是终点状态。

### 场景 B：JS 连续修改 (Batch Optimization)
```javascript
div.style.backgroundColor = 'red';  // State A
div.style.backgroundColor = 'blue'; // State B
```
*   **原因**：浏览器为了性能，会把 JS 执行栈里的所有 DOM 写操作**合并**。它看到的不是 A -> B，而是一个直接的结果 B。
*   **结果**：瞬间变蓝，无动画。

---

## 3. 解决方案：制造“时间差”

我们要强迫浏览器先把 State A 画出来（或者至少算出来），再设置 State B。

### 方案一：强制强制重绘 (Forced Reflow)
读取任何布局属性，迫使浏览器中断合并优化，立即计算当前样式。

```javascript
element.style.display = 'block'; // set State A
element.style.opacity = '0';

// 🛑 核心 Hack：强迫浏览器确认 A 状态
void element.offsetWidth; 

element.style.opacity = '1';     // set State B
```
*   **优点**：同步执行，代码量少。
*   **缺点**：甚至微小的性能损耗（触发了一次 Layout），但在单个元素动画中可忽略。

### 方案二：Double requestAnimationFrame
利用 `requestAnimationFrame` 将 State B 的设置推迟到下一帧（准确说是下下一帧的开始）。

```javascript
element.style.display = 'block'; // State A

requestAnimationFrame(() => {
    // 第一层：此时 State A 已经提交给浏览器，但还没画
    requestAnimationFrame(() => {
        // 第二层：此时 State A 已经画好了
        element.style.opacity = '1'; // State B
    });
});
```
*   **优点**：性能最好，符合浏览器渲染周期。
*   **原理**：
    *   rAF 1: 确保代码在下一帧开始前执行。
    *   rAF 2: 确保浏览器完成了一次 Style/Layout/Paint 循环。

---

## 4. 特殊难题：`height: auto` 动画

CSS Transition 不支持从 `0px` 过渡到 `auto`，因为 `auto` 不是一个数字。

**解决策略：JS 计算真实高度**
1.  设为 `style.height = 'auto'`。
2.  读取 `scrollHeight` 或 `offsetHeight` 获取真实高度值。
3.  立即设回 `0px`（此时利用了浏览器的批量优化，用户看不见刚才那个瞬间的 auto）。
4.  强制重绘（让浏览器确认 0px）。
5.  设为 `targetHeight + 'px'`。

---

## 5. 总结

| 问题 | 原因 | 解决 |
| :--- | :--- | :--- |
| **JS 改了属性没动画** | 浏览器合并了读写操作 | `void el.offsetWidth` 或 `Double rAF` |
| **display:none 出现无动画** | 元素不在渲染树，无初始态 | 先设 block，强制重绘，再加 active 类 |
| **height:auto 无动画** | auto 无法插值计算 | JS 算出具体像素值赋给 height |
