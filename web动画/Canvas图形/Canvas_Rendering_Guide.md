# Canvas 渲染机制与核心知识指南

Canvas 是 HTML5 提供的一个用于展示绘图内容的元素，它提供了一套基于 JavaScript 的绘图 API。

## 1. 核心渲染原理：黑板模式 (Immediate Mode)

Canvas 与 DOM 的根本区别在于渲染模式。

### DOM (Retained Mode / 保留模式)
*   **机制**：浏览器维护一个“场景图”（DOM 树）。
*   **特点**：你告诉浏览器“这里有一个红色方块”。浏览器负责记录它、绘制它、并在你修改属性时自动管理重绘。
*   **交互**：浏览器自动进行碰撞检测，支持 `onclick`、`:hover` 等原生事件。

### Canvas (Immediate Mode / 立即模式)
*   **机制**：就像一块黑板。
*   **特点**：你发出指令：“在左上角画个圈”。浏览器执行指令，修改像素位图。画完后，浏览器就**忘**了这个圈，只记得这里有一堆颜色的像素。**它不记录任何对象信息**。
*   **交互**：一旦绘制完成，就无法通过 API 直接修改那个圆，也无法直接监听它的点击事件。必须擦除重画。

---

## 2. Canvas 与浏览器渲染管线

Canvas 在标准的 `JS -> Style -> Layout -> Paint -> Composite` 流程中表现非常特殊：

1.  **隔离性 (Isolation)**:
    *   Canvas 元素本身只是 DOM 树上的一个节点（类似 `<img>`）。
    *   **Skipping Layout**: 只要你不改变 `<canvas>` 标签本身的宽高或 CSS 布局属性，Canvas **内部**的任何绘图操作都不会触发页面的 Reflow (重排/回流)。

2.  **纯位图操作**:
    *   Canvas API 的本质是 JS 驱动 CPU/GPU 修改内存中的一块像素数组。
    *   **流程缩短**: `JS调用` -> `更新Bitmap` -> `Composite(合成)`.
    *   **优势**: 极其适合每帧都在变化的密集型动画（如粒子系统、游戏），因为没有 DOM 操作的开销。

---

## 3. 交互难点：手动碰撞检测

因为 Canvas 不知道“圆”的存在，你不能给圆加 `onclick`。必须手动实现：

**实现 Canvas 元素点击/Hover 的步骤**：
1.  **监听**：在 `<canvas>` 元素上监听 `mousemove` 或 `click`。
2.  **计算**：获取鼠标相对于 Canvas 左上角的坐标 `(monitorX, monitorY)`。
3.  **Hit Detection (碰撞检测)**：
    *   遍历你的数据对象列表（例如存了所有圆的数组）。
    *   使用数学公式判断鼠标是否在形状内。
    *   *圆形*：`Math.sqrt((x-cx)^2 + (y-cy)^2) < radius`
    *   *矩形*：`x > rect.x && x < rect.x + w && ...`
4.  **响应**：如果检测到命中，修改数据状态，**清空画布**，根据新状态重新绘制。

---

## 4. Canvas 基础知识补充

### 4.1 环境搭建
```html
<canvas id="stage" width="800" height="600"></canvas>
<script>
    const canvas = document.getElementById('stage');
    // 获取 2D 绘图上下文 (画笔对象)
    const ctx = canvas.getContext('2d');
</script>
```

### 4.2 坐标系系统
*   **原点 (0, 0)**：画布的左上角。
*   **X 轴**：向右为正。
*   **Y 轴**：向下为正。

### 4.3 核心绘制 API
*   **路径 (Path)** - 一切的基础:
    ```javascript
    ctx.beginPath();       // 1. 开始新路径
    ctx.moveTo(100, 100);  // 2. 将笔触移动到指定点
    ctx.lineTo(200, 200);  // 3. 规划一条线
    ctx.arc(150, 150, 50, 0, Math.PI * 2); // 规划圆弧
    ctx.closePath();       // 4. (可选) 闭合形状
    
    ctx.fillStyle = 'red'; // 设置填充色
    ctx.fill();            // 5. 真正的填充操作
    
    ctx.lineWidth = 2;
    ctx.stroke();          // 6. 真正的描边操作
    ```
*   **矩形快捷方式**: `ctx.fillRect(x, y, w, h)`, `ctx.clearRect(x, y, w, h)` (橡皮擦)。
*   **状态管理**:
    *   `ctx.save()`: 把当前的颜色、线宽、变形状态压入栈中保存。
    *   `ctx.restore()`: 弹栈恢复。**在做局部变换或复杂绘制时必用，防止污染后续绘制**。

### 4.4 变换 (Transforms)
Canvas 的变换并不是针对“当前画的物体”，而是针对**整个坐标系网格**。
*   `ctx.translate(x, y)`: 移动原点位置。
*   `ctx.rotate(rad)`: 旋转坐标纸。
*   `ctx.scale(x, y)`: 缩放坐标纸。
*   *技巧*：通常配合 `save()` 和 `restore()` 使用，画完一个歪的物体后立刻恢复坐标系。

---

## 5. 进阶性能优化技巧

### 5.1 分层渲染 (Layering)
**场景**：游戏背景不动，人物一直在动。
**优化**：不要在一个 Canvas 里每帧重画背景。
*   **层1 (DOM z-index: 0)**: `<canvas id="bg">` -> 只画一次背景。
*   **层2 (DOM z-index: 1)**: `<canvas id="player">` -> 每帧 `clearRect` 并重画人物。
*   **原理**：大幅减少每帧需要处理的像素量。

### 5.2 离屏渲染 (Offscreen Canvas)
**场景**：需要频繁绘制一个非常复杂的静态图形（例如几千条线组成的徽章）。
**优化**：
1.  创建一个内存里的 Canvas (`document.createElement('canvas')`)。
2.  先把复杂的图形画在这个隐形的 Canvas 上（缓存）。
3.  在渲染循环中，直接用 `ctx.drawImage(offScreenCanvas, x, y)` 把图贴过来。
*   **原理**：以内存换速度，把“几千次绘制指令”变成“一次图片拷贝”。

### 5.3 避免浮点数坐标
Canvas 支持亚像素渲染，但如果坐标是 `10.5`，为了抗锯齿，浏览器会做额外的插值计算导致模糊。在高性能场景下，尽量使用 `Math.floor()` 或位运算 `(x | 0)` 取整坐标。

---

## 6. 选型总结：DOM vs Canvas

| 维度 | DOM / SVG | Canvas |
| :--- | :--- | :--- |
| **内存占用** | 与元素数量成正比 (DOM 节点重) | 与画布面积成正比 (元素再多也不怕) |
| **事件交互** | 原生支持，开发效率极高 | 困难，需手写算法 |
| **文本能力** | 完美 (排版、换行、无障碍) | 极弱 (简单的文本绘制) |
| **适用场景** | 管理后台、表单、文档、简单动效 | 游戏、数据可视化(ECharts)、图片编辑、粒子特效 |
