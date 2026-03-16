在没有使用模块化的情况下，我们虽然可以将应用中的逻辑拆分到了**不同的 js 文件中**，但实际上在每个 js 文件中的代码运行时的顶层作用域都是**同一个全局作用域（此处是 window ）**
```javascript
<body>
    <script src="./a.js"></script>
    <script src="./b.js"></script>
    <script src="./c.js"></script>
    ...
</body>
```
1. 全局变量污染
   同一个全局作用域
2. 依赖管理混乱
   通过 `<script></script>`标签引入了3个 js 文件,会按照从上到下的顺序依次加载并执行

现在主流的两种模块化规范是`CommonJS`和`ES Module`


**CommonJS** 在 Node.js 中被原生支持，而浏览器原生是不支持 CommonJS 的，因为 CommonJS 适用于加载本地模块，是一个**同步加载**的过程，比如 Node.js 中加载模块其实是一个**读取本地文件并执行的同步过程**，而在浏览器中要获取资源通常是需要**异步请求**获取的，所以以前才会出现用于浏览器端的模块加载规范—— AMD（Asynchronous Module Definition 异步模块定义）

作者：一知_ScoutYin  
链接：https://juejin.cn/post/7108410887052427301  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

# AMD - Asynchronous Module Definition 异步模块定义
- 规范代表库：Require.js
[RequireJS API --- RequireJS API](https://requirejs.org/docs/api.html)
# CJS - CommonJS
Node.js


[CommonJS Spec Wiki](https://wiki.commonjs.org/wiki/CommonJS)