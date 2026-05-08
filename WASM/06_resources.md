第 6 章 — 进阶资源与学习路线

官方与基础文档：
- WebAssembly 官方站点: https://webassembly.org/
- WASI 文档: https://wasi.dev/

书籍与教程：
- 《Programming WebAssembly with Rust》
- wasm-by-example（交互式示例集）

工具与项目：
- wasm-bindgen（Rust ↔ JS 互操作）
- wasm-pack（Rust 打包工具）
- wasmtime / wasmer（运行时）
- Binaryen（包括 `wasm-opt`）

学习路线建议：
1. 理解核心概念（模块、线性内存、导入/导出）
2. 用 Rust/C 写最小示例并在浏览器运行
3. 学习 WASI 并在 `wasmtime` 中运行服务端示例
4. 深入组件模型、Interface Types 与 GC proposal 以支持更复杂语言运行时

我将持续把进一步的教学、练习与代码样例追加到本目录。