# 流媒体（Streaming）

## 什么是流媒体？

流媒体是一种数据传输技术，允许你将路由拆分成更小的"块"，并在服务器准备好时逐步从服务器流向客户端。

通过流式传输，你可以防止慢速数据请求阻断整个页面。这让用户无需等待所有数据加载完毕，就能查看和交互页面的部分内容。

流式传输与 React 的组件模型配合良好，因为每个组件都可以被视为一个块。

## 为什么要用流媒体？

在动态渲染中，整个页面的渲染取决于最慢的数据取用速度。如果某个数据请求很慢，整个页面就会被 block 住。

流媒体解决的问题：**让先准备好的内容先呈现，而不是等待最慢的数据全部就绪再一次性显示。**

## Next.js 中实现流媒体的两种方式

### 1. 页面级别：`loading.tsx`

`loading.tsx` 是 Next.js 基于 React Suspense 构建的**特殊文件**，用于实现页面级流式加载。

#### 核心特性

1. **自动包装 Suspense**：Next.js 会自动将 `loading.tsx` 的内容包装在 `<Suspense>` 边界中，无需手动导入 Suspense。

2. **渐进式加载**：
   - **静态内容**（如导航栏 `<SideNav>`）：立即显示
   - **动态内容**：在后台加载时显示 fallback UI（骨架屏或 Loading 文字）

3. **可中断导航**：用户无需等待当前页面加载完成，就可以跳转到其他页面。

#### Next.js 内部原理

`loading.tsx` 实际上被 Next.js 转换为类似这样的代码：

```tsx
// Next.js 内部大致等价于
<Suspense fallback={<DashboardSkeleton />}>
  <Page />  {/* 实际页面内容 */}
</Suspense>
```

#### 文件位置与作用范围

`loading.tsx` 的位置决定了它影响哪些路由：

| 位置 | 影响的路由 |
|------|-----------|
| `/app/dashboard/loading.tsx` | `/dashboard` 及其**所有子路由**（`/invoices`、`/customers` 等）|
| `/app/dashboard/(overview)/loading.tsx` | 仅影响 `/dashboard`（在路由组内）|

如果想让 `loading.tsx` 只影响特定页面，可以使用**路由组**将页面和 loading 文件放入同一文件夹。

#### 常见用法

```tsx
// 1. 简单 Loading 文字
export default function Loading() {
  return <div>Loading...</div>;
}

// 2. 使用骨架屏组件（推荐）
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
  return <DashboardSkeleton />;
}

// 3. 更精细的骨架（可自定义）
import { MyCustomSkeleton } from '@/app/ui/skeletons';

export default function Loading() {
  return (
    <div className="space-y-4">
      <MyCustomSkeleton />
      <MyCustomSkeleton />
    </div>
  );
}
```

#### 执行顺序

当用户访问 `/dashboard` 时：

1. 服务端立即返回 `loading.tsx` 的内容（骨架屏）
2. 浏览器渲染骨架 UI，用户**立即看到反馈**，不是白屏
3. 同时，服务端继续加载 `page.tsx` 的数据
4. 数据加载完成后，`page.tsx` 的实际内容**替换**骨架 UI

这就是"流式"——**先让用户看到一些东西，而不是干等着白屏或长时间无反馈**。

### 2. 组件级别：`<Suspense>`

用于更细粒度的控制，可以单独流式传输某个组件：

```tsx
import { Suspense } from 'react';
import { RevenueChartSkeleton } from '@/app/ui/skeletons';

<Suspense fallback={<RevenueChartSkeleton />}>
  <RevenueChart />
</Suspense>
```

**Suspense** 允许你延迟渲染应用的某些部分，直到满足某些条件（如数据加载完成）。

## 加载骨架（Loading Skeleton）

加载骨架是 UI 的简化版本，作为占位符告知用户内容正在加载：

```tsx
// loading.tsx
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
  return <DashboardSkeleton />;
}
```

## 路由组（Route Groups）

用路由组可以限制 `loading.tsx` 的作用范围。

创建文件夹 `(overview)`，将 `loading.tsx` 和 `page.tsx` 放入其中：

```
/app/dashboard/(overview)/loading.tsx
/app/dashboard/(overview)/page.tsx
```

路由组的特点：
- 使用括号 `()` 命名，名称不会出现在 URL 路径中
- `/dashboard/(overview)/page.tsx` 的 URL 依然是 `/dashboard`
- 可用于按功能（营销/商店）或团队组织代码

## Suspense 边界的放置策略

放置 Suspense 边界需要考虑：

1. **用户体验**：希望页面如何逐步呈现
2. **内容优先级**：哪些内容最重要
3. **数据获取**：组件是否依赖数据请求

几种常见策略：

| 策略 | 优点 | 缺点 |
|------|------|------|
| 流式传输整个页面 | 简单 | 如果某个组件慢，整体还是会被 block |
| 流式传输每个组件 | 最细粒度 | 可能出现 UI"弹出"效果 |
| 分组流式传输 | 有节奏感，避免视觉跳动 | 需要创建包装组件 |

**最佳实践**：将数据获取下移到需要数据的组件中，然后将这些组件包装在 Suspense 中。

## 常见模式

### 1. 数据获取下移

将 `fetchRevenue()` 从页面组件移到 `<RevenueChart>` 组件内部：

```tsx
// page.tsx - 不再获取 revenue 数据
export default async function Page() {
  return (
    <Suspense fallback={<RevenueChartSkeleton />}>
      <RevenueChart /> {/* 组件自己获取数据 */}
    </Suspense>
  );
}

// revenue-chart.tsx - 组件自己负责数据获取
export default async function RevenueChart() {
  const revenue = await fetchRevenue();
  // ...
}
```

### 2. 分组加载

将多个卡片组件组合在一起同步加载，避免"弹出"效果：

```tsx
<Suspense fallback={<CardsSkeleton />}>
  <CardWrapper /> {/* 包含多个 Card 组件 */}
</Suspense>
```

## 核心优势总结

- **渐进式加载**：让用户尽快看到内容
- **可中断导航**：用户不必等待即可跳转到其他页面
- **更好的感知性能**：即使实际加载时间相同，用户感觉更快
