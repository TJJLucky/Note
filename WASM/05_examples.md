第 5 章 — 示例与实战

1. Rust 最小示例（使用 `wasm-pack` + `wasm-bindgen`）

项目结构（示例）：
- my-wasm/
  - Cargo.toml
  - src/lib.rs

Cargo.toml（重要部分示例）：

```toml
[package]
name = "my_wasm"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
```

src/lib.rs：

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

构建并发布：

```bash
wasm-pack build --target bundler
```

在前端项目中使用：

```javascript
import init, { add } from './pkg/my_wasm.js';
await init();
console.log(add(1,2));
```

2. C 示例（clang 直接编译）

简单 C 函数：

```c
// add.c
int add(int a, int b) { return a + b; }
```

编译：

```bash
clang --target=wasm32-unknown-unknown -O3 -nostdlib -Wl,--no-entry -Wl,--export-all -o add.wasm add.c
```

3. 浏览器加载示例（通用）

```javascript
const resp = await fetch('add.wasm');
const { instance } = await WebAssembly.instantiateStreaming(resp, {});
console.log(instance.exports.add(2,3));
```

4. 在服务器/边缘运行（WASI + wasmtime）

- 构建目标为 `wasm32-wasi`（Rust/C 可用）。
- 运行：

```bash
wasmtime run module.wasm -- <args>
```

5. 调试与验证
- 将 `.wasm` 转为 `.wat` 用于阅读：

```bash
wasm2wat module.wasm -o module.wat
```

- 使用 `wasm-opt` 做优化或调试变换。

6. 小结
- 示例示范了从源代码到 `.wasm` 的最小可运行环节。实际项目中你需要加入内存管理（分配/释放）、错误处理、以及高效的数据传输策略。