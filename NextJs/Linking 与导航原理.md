# Next.js Linking 与导航原理

---

## 一、渲染策略概述

Next.js 的页面默认在**服务器端渲染**（Server Rendering），但通过一系列优化技术，让导航体验接近客户端渲染的速度。

### 渲染类型对比

| 类型 | 生成时机 | 是否缓存 | 首屏速度 | 交互时机 |
|------|---------|---------|---------|---------|
| **SSG**（静态生成） | 构建时 | ✅ 完全静态 | 最快 | 需等 Hydration |
| **SSR**（服务端渲染） | 请求时 | ❌ 动态 | 较快 | 需等 Hydration |
| **CSR**（客户端渲染） | 浏览器 | ❌ 不缓存 | 慢 | 需等 JS 下载执行 |
| **ISR**（增量静态再生） | 构建时 + 按需 | ✅ 混合 | 快 | 需等 Hydration |

---

## 二、Prefetching（预取）

### 触发时机

> "Next.js automatically prefetches routes linked with the `<Link>` component **when they enter the user's viewport**."

- **生产环境**：每当 `<Link>` 组件出现在浏览器视口时，Next.js 会自动在后台预取链接路由的代码
- 开发环境：预取行为可能有所不同（通常被禁用或延迟）

### 预取内容

根据路由类型不同，预取策略也不同：

| 路由类型 | 预取策略 |
|---------|---------|
| **静态路由** | 完整路由都会被预取（Server Component Payload + JS bundles + CSS） |
| **动态路由** | 默认跳过预取；若存在 `loading.tsx`，则部分预取（共享布局 + 骨架屏） |

### 为什么这样设计？

- **节省带宽** — 用户可能不会滚动到页面底部，提前预取所有链接浪费资源
- **按需预取** — 只预取用户实际可能访问的路由
- **避免不必要的工作** — 动态路由参数不确定，提前渲染可能是无用功

### 预取方式对比

```tsx
// 方式1：进入视口时预取（默认行为）
<Link href="/dashboard">Dashboard</Link>

// 方式2：鼠标悬停时预取（更早触发）
<Link href="/dashboard" onMouseEnter={() => router.prefetch('/dashboard')}>
  Dashboard
</Link>

// 方式3：禁用预取
<Link href="/dashboard" prefetch={false}>
  Dashboard
</Link>
```

---

## 三、Streaming（流式传输）

### 核心思想

服务器分块发送内容，**先到先展示**，不必等待全部数据准备好。

### 实现方式

创建 `loading.tsx` 文件：

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <LoadingSkeleton />
}
```

### 效果

- **立即导航** — 用户点击链接后立即跳转，不等待
- **骨架屏先行** — `loading.tsx` 的 UI 先显示
- **内容替换** — 数据就绪后，骨架屏自动替换为真实内容

### 优势

- 改善 TTFB、FCP、TTI 等 Core Web Vitals 指标
- 共享布局保持交互，导航可中断

---

## 四、Client-side Transitions（客户端过渡）

### 传统页面导航的问题

点击链接时，传统 SSR 会：
- 触发**整页刷新**（full page reload）
- 清空所有客户端状态（表单输入、滚动位置等）
- 阻塞交互，直到服务器响应完成

### Next.js 的解决方案

使用 `<Link>` 组件实现**无刷新导航**：

```tsx
import Link from 'next/link'

// 点击这个链接不会触发整页刷新
<Link href="/dashboard">
  Go to Dashboard
</Link>
```

### 底层原理

```
1. 拦截点击事件 → <Link> 拦截原生点击，阻止浏览器默认行为
2. 检查预取缓存 → 查看目标路由数据是否已预取
3. 更新 URL → 使用 window.history.pushState 更新地址栏
4. 客户端渲染 → 使用缓存数据在客户端渲染新页面
5. 保留共享布局 → Layout 组件不重新挂载，只替换 Page 内容
```

### 保留的状态

- 共享 Layout 组件状态
- React 组件树（未变化的部分）
- 滚动位置（可配置 `scroll={false}`）
- 表单输入（如果不在导航区域）

---

## 五、如何实现"只替换 Page，保留 Layout"

### React Server Components 架构

```
Root Layout (app/layout.tsx)
  ├── <html> <body>
  ├── 共享导航组件 (Sidenav)
  │
  └── <Suspense>
      └── Page (app/dashboard/page.tsx)  ← 路由切换时这里被替换
```

### 替换机制

**1. Server Component Payload（RSC Payload）**

服务器返回的不是完整 HTML，而是一个特殊的 JSON 格式：

```json
{
  "type": "page",
  "props": { "params": {}, "searchParams": {} },
  "cache": {},
  "prerender": false
}
```

**2. 客户端路由切换流程**

```
点击 Link → 发送 RSC 请求 → 获取新 Page 的 RSC Payload
                              ↓
                    客户端对比新旧 payload
                              ↓
              只更新 Page 对应的 React 组件树
                        ↓
        Layout 的 DOM 节点和状态完全保留
```

**3. 为什么 Layout 不会重新挂载**

- Layout 是**父组件**，React diff 算法只比较 children 部分
- RSC Payload 包含边界信息，告诉客户端"Layout 不变，只需更新 Page"
- Next.js Router 管理组件树，而不是完全卸载/重挂

**核心理解**：Layout 一直是同一个 React 组件实例，只是 props（children）发生了变化。

---

## 六、常见错误

### Error: Event handlers cannot be passed to Client Component props

**错误信息**：
```
Event handlers cannot be passed to Client Component props.
<... href=... onNavigate={function onNavigate} children=...>
If you need interactivity, consider converting part of this to a Client Component.
```

**原因**：
- Server Component 不能将事件处理器传递给 Client Component
- 函数、事件处理器无法被序列化，无法作为 props 传递

**解决方案**：

在需要交互的文件顶部添加 `'use client'`：

```tsx
// sidenav.tsx 或 nav-links.tsx
'use client'

import Link from 'next/link'
// ...
```

---

## 七、总结

> **服务端组件负责"展示内容"，客户端组件负责"用户交互"**。

Next.js 通过三层优化实现"快如 CSR 的 SSR"：

| 优化机制 | 作用 |
|---------|------|
| **Prefetching** | 提前加载路由数据，点击即用 |
| **Streaming** | 分块发送，先展示骨架屏 |
| **Client-side Transitions** | 无刷新导航，保留页面状态 |

合理利用这些机制，可构建**高性能、低开销、SEO 友好**的现代 React 应用。
