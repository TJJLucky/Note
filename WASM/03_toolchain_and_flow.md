第 3 章 — 工具链、语言支持与典型编译流程

1. 常见语言与工具
- Rust：`wasm-pack`、`wasm-bindgen`、`cargo build --target wasm32-unknown-unknown`。
- C/C++：`clang --target=wasm32-unknown-unknown`、`Emscripten`（提供 POSIX/浏览器兼容层）。
- Go：支持 `GOOS=js GOARCH=wasm` 编译为 `.wasm`（运行时较重）。
- 运行时/宿主：`wasmtime`, `wasmer`, 浏览器（Chrome/Firefox/Edge/Safari）。

2. WASI（WebAssembly System Interface）
- WASI 定义了一组可移植的系统调用（文件、网络、时钟等）。使用 WASI 可让 Wasm 在服务器/边缘运行时脱离浏览器。
- 典型工具：`wasmtime`（支持 WASI 运行）、`wasm-pack`（更多用于前端打包）。

3. 典型编译流程示例（Rust -> Wasm）
- 使用 `wasm-pack`（推荐用于前端）：

```bash
# 安装 wasm-pack (若尚未安装)
cargo install wasm-pack

# 生成并打包（输出到 pkg/）
wasm-pack build --target bundler
```

- 生成的包可通过 NPM 导入到前端项目，或直接用 `WebAssembly.instantiateStreaming` 加载 `.wasm`。

4. C/C++ 示例（clang）
```bash
# 目标编译 (无运行时环境)
clang --target=wasm32-unknown-unknown -O3 -nostdlib -Wl,--no-entry -Wl,--export-all -o module.wasm module.c

# 若需要浏览器兼容层，使用 Emscripten：
emcc module.c -O3 -s WASM=1 -o module.js
```

5. 优化与工具
- `wasm-opt`（Binaryen 的一部分）用于体积与性能优化。
- `wasm-strip` 删除不必要的调试信息以减小体积。
- `wasm-bindgen` 帮助 Rust 与 JS 间的类型桥接。

6. 在浏览器中加载（示例）
```javascript
// 在支持的浏览器中
const resp = await fetch('module.wasm');
const { instance } = await WebAssembly.instantiateStreaming(resp, { /* imports */ });
console.log(instance.exports.add(1,2));
```

7. 小结与建议
- 选择工具链时以使用场景为准：前端交互优先 `wasm-pack`/`wasm-bindgen`，系统级/边缘优先 `WASI` + `wasmtime`。
- 重视构建产物的体积与跨界调用频率，使用 `wasm-opt` 进行迭代优化。