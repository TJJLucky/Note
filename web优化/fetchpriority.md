# fetchpriority — 资源加载优先级优化

## 一、核心定义

**`fetchpriority`** 是 HTML 属性，用于**明确指示浏览器资源加载的优先级**（指导浏览器调度器如何分配网络和处理资源）。它直接影响**关键资源的加载顺序和完成时间**，从而优化核心Web性能指标（LCP、FCP等）。

### 价值主张

在复杂页面中，浏览器加载资源时往往存在**优先级冲突**：
- 哪个 script 应该先加载？
- 哪张图片应该优先下载？
- 字体 vs 样式表，谁更紧急？

**`fetchpriority`** 让开发者**显式告诉浏览器**这些优先级决策，避免浏览器启发式算法的误判，从而**加快关键路径资源的交付**。

---

## 二、核心特性与使用场景

### 1. 核心特性

| 特性 | 说明 |
|------|------|
| **三层优先级** | high（高）、low（低）、auto（默认） |
| **适用资源类型** | script、img、link（字体/样式表）、iframe |
| **浏览器原生支持** | 不需要 JavaScript，HTML 属性即可生效 |
| **无网络成本** | 仅改变调度顺序，不产生额外请求 |
| **可覆盖启发式算法** | 显式指定时优先级 > 浏览器自动判断 |

### 2. 典型使用场景

#### ✅ **high（高优先级）**
- **LCP 关键图片** — 首屏视觉内容（hero image、banner）
- **关键字体** — 防止 FOUT（Flash of Unstyled Text）
- **核心 CSS** — 页面布局必需的样式表
- **优先级JavaScript** — 初始化逻辑、用户交互必需脚本

#### ✅ **low（低优先级）**
- **非关键图片** — 下方折叠内容、轮播图备选项
- **分析脚本** — GA、埋点代码等第三方跟踪
- **可选 JavaScript** — 增强功能特性、非核心交互
- **非必需资源** — 装饰性内容、可延后加载的资源

#### ✅ **auto（默认行为）**
- 让浏览器按其默认启发式算法决定
- 适用于 **优先级不确定** 的资源
- 等价于不设置 `fetchpriority` 属性

---

## 三、基本语法与属性值

### 1. 基础结构

```html
<!-- 通用语法 -->
<标签 fetchpriority="high|low|auto" ...其他属性...>

<!-- 具体示例 -->
<script fetchpriority="high" src="critical.js"></script>
<img fetchpriority="high" src="hero.jpg" alt="Hero">
<link fetchpriority="low" rel="stylesheet" href="optional.css">
```

### 2. 在不同资源类型上的用法

| 资源类型 | 语法示例 | 常见优先级 |
|---------|---------|---------|
| **Script** | `<script fetchpriority="high" src="..."></script>` | high / low |
| **Image** | `<img fetchpriority="high" src="..." alt="...">` | high / low / auto |
| **Link (字体)** | `<link fetchpriority="high" rel="preload" href="..." as="font">` | high |
| **Link (样式)** | `<link fetchpriority="high" rel="stylesheet" href="...">` | high / auto |
| **IFrame** | `<iframe fetchpriority="low" src="..."></iframe>` | low |

### 3. 属性值详解

| 值 | 浏览器行为 | 适用场景 |
|----|---------|---------|
| **`high`** | 提升资源加载优先级，与关键资源并行加载 | 首屏关键资源、LCP候选项、初始化脚本 |
| **`low`** | 降低加载优先级，在关键资源加载完后加载 | 次要资源、分析脚本、装饰内容 |
| **`auto`** | 浏览器自动判断（默认行为） | 优先级决策不明确时 |

---

## 四、与其他优化方案的对比

### 对比表：fetchpriority vs preload vs prefetch

| 方案 | 语法 | 作用 | 加载时机 | 网络影响 | 典型场景 |
|-----|------|------|---------|---------|---------|
| **fetchpriority** | `<link fetchpriority="high">` | 改变**优先级排序** | 立即获取，优先级可控 | ✅ 占用宽带（关键路径） | 首屏LCP资源、关键字体 |
| **preload** | `<link rel="preload" as="...">` | **强制前置加载**，不执行 | 立即获取 | ✅ 占用宽带 | 预加载字体、CSS、关键JS |
| **prefetch** | `<link rel="prefetch">` | **低优先级预加载**，后续使用 | 浏览器空闲时（低优先级） | 🟡 可选占用（非关键） | 下一页资源、SPA路由切换 |
| **preconnect** | `<link rel="preconnect">` | 建立**TCP连接**，不加载资源 | 立即建立连接 | 🟢 无宽带占用 | 跨域API、CDN连接 |

### 核心区别解析

```html
<!-- fetchpriority：调整优先级 -->
<img fetchpriority="high" src="hero.jpg" alt="Hero">
<!-- 行为：立即加载，高优先级，会阻塞低优先级资源 -->

<!-- preload：强制前置加载 -->
<link rel="preload" as="image" href="hero.jpg">
<img src="hero.jpg" alt="Hero">
<!-- 行为：无论HTML中何时引用，都提前加载；但仍需在HTML/CSS中引用才能使用 -->

<!-- prefetch：低优先级预加载 -->
<link rel="prefetch" as="image" href="next-page-hero.jpg">
<!-- 行为：浏览器空闲时去加载；不影响当前页关键资源 -->
```

❗ **关键洞察**：`fetchpriority` 是**动态调整现有资源的加载顺序**，而 `preload` 是**显式声明资源的前置加载**。两者可搭配使用。

---

## 五、实战示例

### 场景 1：优化 LCP 关键图片加载

```html
<!-- ❌ 未优化：图片与非关键脚本竞争网络连接 -->
<img src="hero.jpg" alt="Hero Banner">
<script src="analytics.js"></script>
<script src="ads.js"></script>

<!-- ✅ 已优化：确保关键图片高优先级 -->
<img fetchpriority="high" src="hero.jpg" alt="Hero Banner" loading="eager">

<!-- 同时降低非关键资源 -->
<script fetchpriority="low" src="analytics.js"></script>
<script fetchpriority="low" src="ads.js"></script>
```

### 场景 2：关键字体加载优化

```html
<!-- 在 <head> 中预加载字体，高优先级 -->
<link 
  rel="preload" 
  as="font" 
  href="/fonts/system-ui-bold.woff2"
  type="font/woff2"
  crossorigin
  fetchpriority="high"
>

<!-- 在样式表中引用 -->
<link rel="stylesheet" href="styles.css">

<!-- 在 CSS 中定义 -->
<style>
  body {
    font-family: 'System UI', system-ui, sans-serif;
    font-weight: bold;
  }
  
  @font-face {
    font-family: 'System UI';
    src: url('/fonts/system-ui-bold.woff2') format('woff2');
    font-weight: bold;
  }
</style>
```

### 场景 3：SPA 路由优化（预取 vs 优先级）

```html
<!-- 当前页：高优先级关键资源 -->
<script fetchpriority="high" src="/js/app-current.js"></script>
<img fetchpriority="high" src="current-page-hero.jpg" alt="Current">

<!-- 降低当前页的非关键资源 -->
<link fetchpriority="low" rel="stylesheet" href="advertising.css">
<script fetchpriority="low" src="third-party-widget.js"></script>

<!-- 预取下一个页面的资源（低优先级，利用空闲带宽） -->
<link rel="prefetch" as="script" href="/js/app-next.js">
<link rel="prefetch" as="image" href="next-page-hero.jpg">
```

### 场景 4：响应式图片的优先级

```html
<!-- LCP 关键图片，确保高优先级 -->
<picture>
  <source 
    fetchpriority="high"
    media="(max-width: 640px)" 
    srcset="hero-mobile.jpg"
  >
  <source 
    fetchpriority="high"
    media="(min-width: 641px)" 
    srcset="hero-desktop.jpg"
  >
  <img 
    fetchpriority="high"
    src="hero-desktop.jpg" 
    alt="Hero Banner"
  >
</picture>

<!-- 下方折叠内容，低优先级 -->
<img 
  loading="lazy"
  fetchpriority="low"
  src="fold-content.jpg" 
  alt="Below Fold"
>
```

---

## 六、浏览器兼容性

| 浏览器 | 最小版本 | 支持状态 | 备注 |
|------|---------|--------|------|
| **Chrome/Edge** | 101+ | ✅ 完全支持 | 2022年4月起支持 |
| **Firefox** | 121+ | ✅ 完全支持 | 2024年1月起支持 |
| **Safari** | 17+ | ✅ 完全支持 | 2024年WWDC支持 |
| **Opera** | 87+ | ✅ 完全支持 | 同Chrome版本支持 |
| **IE 11** | — | ❌ 不支持 | 属性被忽略，无降级问题 |
| **旧版浏览器** | <2022 | ❌ 不支持 | 属性被忽略，按默认优先级加载 |

### 兼容性处理

```html
<!-- 渐进增强：根据浏览器支持情况 -->
<!-- 现代浏览器：识别 fetchpriority，按指定优先级加载 -->
<!-- 旧版浏览器：忽略属性，按默认启发式算法加载（仍可正常工作） -->

<img fetchpriority="high" src="hero.jpg" alt="Hero">

<!-- ✅ 无需条件渲染或降级方案，属性被陈旧浏览器安全忽略 -->
```

---

## 七、注意事项（避坑）

### ❗ 误区 1：过度使用 high 优先级

```html
<!-- ❌ 错误：把所有资源都标记为 high -->
<script fetchpriority="high" src="script1.js"></script>
<script fetchpriority="high" src="script2.js"></script>
<script fetchpriority="high" src="script3.js"></script>

<!-- ✅ 正确：只标记真正关键的资源 -->
<script fetchpriority="high" src="initialization.js"></script>
<script fetchpriority="low" src="feature-a.js"></script>
<script fetchpriority="low" src="feature-b.js"></script>
```

**影响**：所有资源都标记为 high 时，属性失效；浏览器无法辨别真正的优先级需求。

### ❗ 误区 2：混淆 fetchpriority 与 preload

```html
<!-- ❌ 容易混淆的做法 -->
<link rel="preload" as="image" href="hero.jpg">
<img fetchpriority="high" src="hero.jpg" alt="Hero">
<!-- 两个属性各有用途，不是重复设置 -->

<!-- ✅ 正确理解 -->
<!-- preload 的目的：声明这个资源很重要，浏览器需要主动获取 -->
<!-- fetchpriority 的目的：在众多资源中，这个的加载顺序优先 -->

<!-- 常见组合：关键字体通常两个都用 -->
<link 
  rel="preload"
  as="font"
  href="app-font-bold.woff2"
  type="font/woff2"
  crossorigin
  fetchpriority="high"
>
```

### ❗ 误区 3：忽视 loading="lazy" 与 fetchpriority 的交互

```html
<!-- ❌ 矛盾设置 -->
<img 
  loading="lazy"
  fetchpriority="high"
  src="hero.jpg"
  alt="Hero"
>
<!-- loading="lazy" 意味着延后加载，与 fetchpriority="high" 矛盾 -->

<!-- ✅ 正确搭配 -->
<!-- 关键图片：high 优先级 + eager 加载（默认） -->
<img 
  fetchpriority="high"
  src="hero.jpg" 
  alt="Hero"
>

<!-- 非关键图片：low 优先级 + lazy 加载 -->
<img 
  loading="lazy"
  fetchpriority="low"
  src="below-fold.jpg" 
  alt="Below Fold"
>
```

### ❗ 误区 4：第三方资源优先级设置不当

```html
<!-- ❌ 错误：第三方广告脚本用 high 优先级 -->
<script fetchpriority="high" src="https://ads-network.com/script.js"></script>
<!-- 会阻塞自有关键资源的加载 -->

<!-- ✅ 正确：第三方资源通常用 low -->
<script fetchpriority="low" src="https://ads-network.com/script.js"></script>
<script fetchpriority="low" src="https://analytics.com/tracker.js"></script>

<!-- 例外：第三方支付/认证必需时，才用 high -->
<script fetchpriority="high" src="https://checkout-provider.com/checkout.js"></script>
```

### ❗ 误区 5：不考虑实际的 Core Web Vitals 指标

```html
<!-- ❌ 盲目优化而不测试 -->
<!-- 未经测量，直接调整所有资源优先级 -->

<!-- ✅ 数据驱动的优化流程 -->
<!-- 1. 用 Lighthouse / WebPageTest 识别 LCP 元素 -->
<!-- 2. 确认这个元素由哪些资源驱动（CSS、字体、图片、JS） -->
<!-- 3. 对那些驱动资源应用 fetchpriority="high" -->
<!-- 4. 对非关键资源应用 fetchpriority="low" -->
<!-- 5. 重新测量，验证 LCP 改善 -->
```

---

## 八、总结决策树

### 快速选择指南

```
我有一个资源 (script / img / font / link)
        ↓
这个资源在 "首屏加载完成" 前必需吗？
        ├─ ✅ 是（LCP、初始化、关键交互必需）
        │   └─→ 使用 fetchpriority="high" ✓
        │       └─ 进一步检查：这是性能关键路径的必需资源吗？
        │           ├─ ✅ 是 → 考虑配合 rel="preload"
        │           └─ ❌ 否 → 仅用 fetchpriority="high"
        │
        ├─ ❌ 否（非首屏必需）
        │   └─ 这个资源会在用户交互时需要吗？
        │       ├─ ✅ 立即需要（下方折叠、轮播） 
        │       │   └─→ fetchpriority="low" ✓
        │       └─ ❌ 可能后续才需要（路由跳转、用户交互）
        │           └─→ 考虑 rel="prefetch" 而非 fetchpriority
        │
        └─ ❓ 不确定
            └─→ 使用 fetchpriority="auto" 或省略属性
```

### 常见资源的推荐配置

| 资源类型 | 推荐设置 | 理由 |
|---------|--------|------|
| **英雄图片 (Hero Image)** | `<img fetchpriority="high">` | LCP 候选项，首屏必需 |
| **关键字体** | `<link rel="preload" ... fetchpriority="high">` | 防止 FOUT，双保险 |
| **核心 CSS** | `<link rel="stylesheet" ... fetchpriority="high">` | 页面布局基础，或省略用默认 |
| **初始化 JS** | `<script fetchpriority="high" src="app-init.js">` | 初始化逻辑必需 |
| **可选 JS 库** | `<script fetchpriority="low" src="feature.js">` | 增强特性，延后加载 |
| **分析脚本** | `<script fetchpriority="low" src="analytics.js">` | 降低对关键资源的影响 |
| **广告脚本** | `<script fetchpriority="low" src="ads.js">` | 非关键，应被延后 |
| **下方折叠内容** | `<img loading="lazy" fetchpriority="low">` | 双管齐下：延后加载 + 低优先级 |
| **跨域 CDN 资源** | 优先配合 `rel="preconnect"` | 建立连接优先，再用 fetchpriority |

---

## 总结

**`fetchpriority`** 是一个**简单却强大的性能优化属性**，通过显式控制资源加载优先级来改善关键性能指标。

### 核心要点

✅ **何时使用**：当浏览器的启发式优先级排序不符合你的业务需求时  
✅ **怎么用**：在关键资源上加 `fetchpriority="high"`，在非关键资源上加 `fetchpriority="low"`  
✅ **与其他方案的关系**：  
   - `fetchpriority` 改变优先级排序（调度）  
   - `preload` 强制前置加载（发现）  
   - `prefetch` 低优先级预加载（推测）  
✅ **最佳实践**：  
   - 数据驱动（测量LCP候选项）  
   - 少即是多（只标记真正关键的资源）  
   - 避免过度（不要把所有资源都标为 high）

### 立即行动清单

- [ ] 用 Lighthouse 识别你页面的 LCP 元素
- [ ] 追踪驱动该 LCP 元素的资源（CSS、字体、JS、图片）
- [ ] 对这些驱动资源添加 `fetchpriority="high"`
- [ ] 对明确的非关键资源添加 `fetchpriority="low"`
- [ ] 在真实用户设备上验证（考虑 3G / 4G 网络）
- [ ] 监测 Core Web Vitals 的改善是否显著

---

**相关资源**
- [Web.dev: Optimize your Core Web Vitals](https://web.dev/vitals)
- [MDN: fetchpriority](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/fetchpriority)
- [preload 与 prefetch 详解](./Link%20preload.md)
