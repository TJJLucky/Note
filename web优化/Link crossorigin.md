### 一、`crossorigin` 核心定义

`crossorigin` 是 HTML `<link>`（以及 `<script>`/`<img>` 等标签）的跨域属性，用于**声明浏览器加载跨域资源时是否携带跨域凭证（如** **Cookie****、HTTP 认证信息）**，同时触发浏览器按照 CORS（跨源资源共享）规则处理资源请求——这是实现跨域资源合法加载、缓存复用的关键。

  

简单来说：当资源的「请求域名」和「当前页面域名」不一致时（比如 Shopify 店铺加载 `fonts.shopifycdn.com` 上的字体），必须通过 `crossorigin` 声明跨域策略，否则浏览器可能拒绝加载资源，或导致预加载的资源无法复用（重复请求）。

  

  

### 二、`crossorigin` 的取值与区别

|   |   |   |
|---|---|---|
|取值|含义|适用场景|
|`anonymous`|（最常用）跨域请求**不携带**任何凭证（Cookie、HTTP 认证等），服务器仅返回公开资源|字体、CDN 托管的 CSS/JS、公共图片等|
|`use-credentials`|跨域请求**携带**凭证（如用户登录态 Cookie）|需验证用户身份的跨域资源（极少用）|
|不设置该属性|浏览器以「无 CORS 模式」加载资源，不发送 `Origin` 请求头，资源无法被脚本访问|仅同域资源，跨域资源会触发报错/缓存失效|

  

> ❗ 核心结论：Shopify 场景下（加载 CDN 字体、跨域 CSS 等），99% 场景用 `crossorigin="anonym``ous"` 即可。

  

  

### 三、为什么 `<link rel="preload">` 加载跨域资源必须加 `crossorigin`？

以 Shopify 字体预加载为例（你代码中的场景）：

```HTML
<!-- 预加载 Shopify CDN 字体（跨域） -->
<link 
  rel="preload" 
  href="https://fonts.shopifycdn.com/my-font.woff2" 
  as="font" 
  crossorigin="anonymous"  <!-- 必须加！ -->
>
```

如果省略 `crossorigin`，会触发两个关键问题：

1. **资源加载失败/缓存无法复用**：
    

浏览器对字体文件有「CORS 强制校验」——即使字体加载成功，也会标记为「非 CORS 资源」，后续 `@font-face` 使用该字体时，浏览器会重新发起请求（预加载的缓存作废），导致重复加载、页面闪烁（FOUT）。

2. **控制台报错**：
    

会抛出类似 `Font from origin 'https://fonts.shopifycdn.com' has been blocked by CORS policy` 的错误，字体无法正常渲染。

#### 补充：同域资源是否需要加？

如果资源是「当前店铺域名下的资源」（比如 Shopify 主题 Assets 里的 `critical.css`），理论上可以省略 `crossorigin`；但为了兼容性（比如部分 CDN 配置导致的隐性跨域），加 `anonymous` 也不会有副作用。

  

  

### 四、不同资源类型对 `crossorigin` 的要求

|   |   |   |
|---|---|---|
|资源类型|是否需要 `crossorigin`（跨域场景）|原因|
|字体文件（woff/woff2）|必须加|浏览器强制要求字体文件通过 CORS 加载，否则拒绝渲染|
|CSS 文件|建议加|避免预加载的 CSS 缓存无法复用，尤其 CDN 托管的 CSS|
|图片/视频|可选（除非脚本要读取资源内容）|仅当通过 JS（如 `canvas`）操作跨域图片时需要，纯展示无需|
|JavaScript 文件|必须加（若脚本要跨域请求数据）|避免跨域脚本执行时的 CORS 报错|

  

  

### 五、Shopify 场景下的典型正确用法

#### 1. 预加载跨域字体（核心场景）

```Plain
{% unless settings.type_primary_font.system? %}
  <link rel="preconnect" href="https://fonts.shopifycdn.com" crossorigin>
  <!-- 字体预加载必须加 crossorigin="anonymous" -->
  {{ settings.type_primary_font | font_url | preload_tag: as: 'font', crossorigin: 'anonymous' }}
{% endunless %}
```

  

#### 2. 预加载跨域 CSS

```HTML
<!-- 预加载 CDN 托管的 critical.css -->
<link 
  rel="preload" 
  href="https://cdn.shopify.com/.../critical.css" 
  as="style" 
  crossorigin="anonymous"
>
<link rel="stylesheet" href="https://cdn.shopify.com/.../critical.css">
```

  

#### 3. 同域资源（可选加，兼容写法）

```HTML
<!-- 加载主题 Assets 里的 CSS（同域），加 anonymous 无副作用 -->
<link 
  rel="preload" 
  href="{{ 'critical.css' | asset_url }}" 
  as="style" 
  crossorigin="anonymous"
>
```

  

  

### 六、常见误区

3. ❌ 认为「加了 `crossorigin` 就一定会跨域」：
    

`crossorigin` 只是**声明跨域策略**，即使资源同域，加 `anonymous` 也不会触发跨域请求，仅会让浏览器按 CORS 规则处理（无负面影响）。

4. ❌ 混用 `anonymous` 和 `use-credentials`：
    

Shopify 公共资源（字体、CDN 样式）无需携带凭证，用 `use-credentials` 会导致请求头冗余，甚至被服务器拒绝。

5. ❌ 预加载跨域资源时省略 `crossorigin`：
    

最常见错误，直接导致预加载失效、资源重复请求或渲染失败。

### 总结

在 Shopify 主题开发中，`<link>` 标签的 `crossorigin` 核心作用是**解决跨域资源（尤其是字体）的合法加载与缓存复用**，记住两个关键点：

6. 加载 `fonts.shopifycdn.com` 字体时，必须加 `crossorigin="anonymous"`；
    
7. 预加载跨域 CSS/JS 时，建议加 `crossorigin="anonymous"`；
    
8. 同域资源加不加都可，但加了更兼容。