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
使用宏创建 tokio 运行时，需要有 macros 这个 feature

```rust
#[tokio::main]
async fn main() {
    println!("Hello, world!");
}
```

使用宏的时候要注意，宏会自动引入 tokio 的预定义宏，因此在使用宏时需要避免与 tokio 的预定义宏冲突。

要记得添加 **async** 关键字

> ex:  我们可以使用 `cargo expand` 展开宏,

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

使用函数创建 tokio 运行时，需要有 rt 这个 feature

```rust
use tokio::runtime::Runtime;

fn main() {
    let rt = Runtime::new().unwrap();
    rt.block_on(async {
        println!("Hello, world!");
    });
}
```

这样创建的运行时是默认参数，它使用了 tokio 的默认配置，包括线程池的大小、调度器的类型等。

我们可以自定义运行时的配置，使用 `tokio::runtime::Builder` 来创建运行时。

```rust
use tokio::runtime::Builder;

fn main() {
    let rt = Builder::new_multi_thread()
        .worker_threads(4)
        .thread_name("my-worker")
        .max_blocking_threads(2)
        .enable_all() // Enables both I/O and time drivers
        .build()
        .unwrap();
    rt.block_on(async {
        println!("Hello, world!");
    });

    let crt = Builder::new_current_thread()
        .enable_all()
        .build()
        .unwrap();
    crt.block_on(async {
        println!("Hello, world!");
    });
}
```

当然，[官网的文档](https://docs.rs/tokio/latest/tokio/runtime/struct.Builder.html)中也提供了一些别的配置项

### 运行时的使用

### 从同步进入到异步运行时
好了我们已经学会如何创建 tokio 运行时，这个运行时包含了I/O、任务调度器、计时器和阻塞池。

通常我们会使用 Runtime 提供的两种方法将需要编排的 future（暂时可以把这里的 future 理解为一系列任务总的集合，像是 todolist） 交给运行时处理


如果需要更多方法可以查看[官网](https://docs.rs/tokio/latest/tokio/runtime/struct.Runtime.html#)

block_on 方法

```rust
use tokio::runtime::Runtime;

// Create the runtime
let rt  = Runtime::new().unwrap();

// Execute the future, blocking the current thread until completion
rt.block_on(async {
    println!("hello");
});
```

block_on 方法里面的 async 块（future）是由 tokio 的运行时来执行的。它是同步的，整个 async 块是个同步上下文，一般作为程序的入口点

spawn 方法

```rust
use tokio::runtime::Runtime;

// Create the runtime
let rt  = Runtime::new().unwrap();

// Spawn a future onto the runtime
rt.spawn(async {
    println!("hello");
});
```

spawn 方法里面的 async 块（future）是由 tokio 的运行时来执行的。它是异步的，整个 async 块是个异步上下文

spawn 方法返回一个 JoinHandle，可以用来等待子任务的完成

### 在异步运行时中创建任务

我们通常使用 task 在异步运行时中创建任务

对于 task 的理解

任务是一种轻量级、非阻塞的执行单元，任务类似于操作系统线程，但是它不是由操作系统调度的，而是由 tokio 的运行时的任务调度器来调度的， 在 tokio 官网中被称为 green-thread， 和 go 的 goroutine、Kotlin 的 coroutines 很类似，都是用户态的线程，可以并发执行，但是不会阻塞操作系统线程，因此可以实现高并发低资源消耗。

更多信息可以查看 [官方文档](https://docs.rs/tokio/latest/tokio/task/index.html)

```rust
use tokio::task;

#[tokio::main]
async fn main() {
    let join = task::spawn(async {
        "hello world!"
    });

    let result = join.await?;
    assert_eq!(result, "hello world!");
}

```

`task::spawn` 中传入的是一个 future（这里的 future 可以理解为单个任务，当然它也可有很多子任务）

join 本身也是一个 future  可以使用 await 关键字等待这个 future 的完成

任务通常是非阻塞的，通常不应该执行可能阻塞的系统的调用，因为这会阻止同一个线程其他任务的执行，当然了，task 也提供了一些阻塞的 api 应对一些需要阻塞的场景

如果有什么错误或者遗漏的地方欢迎批评指正
