# tokio 使用

## 为你的项目添加 **tokio**

在 Cargo.toml 文件中添加以下依赖：

通常在学习中会添加 tokio 的所有 feature
```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

在实际生产中一般我们会根据需要添加所需的 feature，例如：

```toml
[dependencies]
tokio = { version = "1.47.1", features = [
    "rt",
    "rt-multi-thread",
    "net",
    "fs",
    "macros",
] }
```

## 创建 tokio 运行时

### 使用宏创建 tokio 运行时
使用宏创建 tokio 运行时，需要有 macros 这个feature

```rust
#[tokio::main]
async fn main() {
    println!("Hello, world!");
}
```

使用宏的时候要注意，宏会自动引入 tokio 的预定义宏，因此在使用宏时需要避免与 tokio 的预定义宏冲突。要记得添加 **async** 关键字

ex: 我们可以使用 `cargo expand` 展开宏,

安装 `cargo install cargo-expand`

展开后的代码如下

```rust
#![feature(prelude_import)]
#[prelude_import]
use std::prelude::rust_2024::*;
#[macro_use]
extern crate std;
fn main() {
    let body = async {
        {
            ::std::io::_print(format_args!("Hello, world!\n"));
        }
    };
    #[allow(
        clippy::expect_used,
        clippy::diverging_sub_expression,
        clippy::needless_return
    )]
    {
        return tokio::runtime::Builder::new_multi_thread()
            .enable_all()
            .build()
            .expect("Failed building the Runtime")
            .block_on(body);
    }
}
```

### 使用函数创建 tokio 运行时

使用函数创建 tokio 运行时，需要有 rt 这个feature

```rust
use tokio::runtime::Runtime;

fn main() {
    let rt = Runtime::new().unwrap();
    rt.block_on(async {
        println!("Hello, world!");
    });
}
```
