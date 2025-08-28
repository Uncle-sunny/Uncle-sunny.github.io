# 采用发布配置自定义构建

Cargo 有两个主要的配置：运行 `cargo build` 时采用的 `dev` 配置和运行 `cargo build --release` 的 `release` 配置。`dev` 配置为开发定义了良好的默认配置，`release` 配置则为发布构建定义了良好的默认配置。

当项目的 *Cargo.toml* 文件中没有显式增加任何 `[profile.*]` 部分的时候，Cargo 会对每一个配置都采用默认设置。通过增加任何希望定制的配置对应的 `[profile.*]` 部分，我们可以选择覆盖任意默认设置的子集。

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

越高的优化级别需要更多的时间编译

# 将 crate 发布到 Crate.io



## 编写有用的注释文档

Rust 也有特定的用于文档的注释类型，通常被称为 **文档注释**

它们会生成 HTML 文档。这些 HTML 展示公有 API 文档注释的内容，它们意在让对库感兴趣的程序员理解如何 **使用** 这个 crate，而不是它是如何被 **实现** 的。

文档注释使用三斜杠 `///`

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

 使用如下命令会构建当前 create 文档

```bash
cargo doc --open 
```

一般在文档注释中使用的部分包括

**Panics**: 会 `panic` 的场景

**Errors**: 如果返回 `Result` 在什么场景下返回

**Safety**: 如果这个函数使用 `unsafe` 代码



## 文档注释作为测试

`cargo test` 也会像测试那样运行文档中的示例代码

## 注释包含项的结构

文档注释风格 `//!` 为包含注释的项

## 使用 pub use 导出合适的公有 API

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

这样生成的注解文档更加的清晰，使用者不用顾虑要使用的函数在哪些层级

对于有很多嵌套模块的情况，使用 `pub use` 将类型重导出到顶级结构对于使用 crate 的人来说将会是大为不同的体验。`pub use` 的另一个常见用法是重导出当前 crate 的依赖的定义使其 crate 定义变成你 crate 公有 API 的一部分。



## 创建 Crate.io 

## 账号



登录官网，注册账号，获取 API Token， 使用cargo 登录

```bash
cargo login api_token
```

这个 api token 会存在本地切不该被共享



## 向 crate 添加元信息

发布之前，你需要在 crate 的 *Cargo.toml* 文件的 `[package]` 部分增加一些本 crate 的元信息（metadata）。

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

## 发布到 Crate.io

发布 crate 时请多加小心，因为发布是 **永久性的**（*permanent*）

对应版本不可能被覆盖，其代码也不可能被删除。

```bash
cargo publish
```



## 使用 cargo yank 从 Crate.io 弃用版本

Cargo 支持 **撤回**（*yanking*）某个版本

撤回某个版本会阻止新项目依赖此版本，不过所有现存此依赖的项目仍然能够下载和依赖这个版本

从本质上说，撤回意味着所有带有 *Cargo.lock* 的项目的依赖不会被破坏，同时任何新生成的 *Cargo.lock* 将不能使用被撤回的版本。

如果我们发布了一个名为 `guessing_game` 的 crate 的 1.0.1 版本并希望撤回它，在 `guessing_game` 项目目录运行：

```bash
cargo yank --vers 1.0.1
```

撤销撤回操作

```bash
cargo yank --vers 1.0.1 --undo
```



# Cargo 工作空间

随着项目开发的深入，库 crate 持续增大，而你希望将其进一步拆分成多个库 crate。Cargo 提供了一个叫 **工作空间**（*workspaces*）的功能，它可以帮助我们管理多个相关的协同开发的包。



## 创建工作空间

创建一个目录，在目录中创建 `Cargo.toml` 文件

创建工作空间

```toml
[workspace]

members = [
    "adder",
    "add_one"
]

```



```txt
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

`adder` 并没有自己的 *target* 目录。即使进入 *adder* 目录运行 `cargo build`，构建结果也位于 *add/target* 而不是 *add/adder/target*。

通过共享一个 *target* 目录，工作空间可以避免其他 crate 重复构建。

cargo 并不假定工作空间中的 Crates 会相互依赖，所以需要明确表明工作空间中 crate 的依赖关系

```toml
[dependencies]
add_one = { path = "../add_one" }
```

如果在 adder 中需要使用 add_one

为了在顶层 *add* 目录运行二进制 crate，可以通过 `-p` 参数和包名称来运行 `cargo run` 指定工作空间中我们希望使用的包：

```bash
cargo run -p adder
```



## 在工作空间中依赖外部包

工作空间只在根目录有一个 *Cargo.lock*，而不是在每一个 crate 目录都有 *Cargo.lock*

确保了所有的 crate 都使用完全相同版本的依赖。如果在 *Cargo.toml* 和 *add_one/Cargo.toml* 中都增加 `rand` crate，则 Cargo 会将其都解析为同一版本并记录到唯一的 *Cargo.lock* 中

顶级的 *Cargo.lock* 包含了 `add_one` 的 `rand` 依赖的信息。然而，即使 `rand` 被用于工作空间的某处，也不能在其他 crate 中使用它，除非也在它们的 *Cargo.toml* 中加入 `rand`



## 为工作空间增加测试

在顶级 *add* 目录运行 `cargo test`。在像这样的工作空间结构中运行 `cargo test` 会运行工作空间中所有 crate 的测试。



可以选择运行工作空间中特定 crate 的测试

```
cargo test -p add_one
```

如果你选择向 [crates.io ](https://crates.io/)发布工作空间中的 crate，每一个工作空间中的 crate 需要单独发布。就像 `cargo test` 一样，可以通过 `-p` 参数并指定期望发布的 crate 名来发布工作空间中的某个特定的 crate。



# 使用 cargo install 安装二进制文件

`cargo install` 命令用于在本地安装和使用二进制 crate

意在作为一个方便 Rust 开发者们安装其他人已经在 [crates.io](https://crates.io/) 上共享的工具的手段

只有拥有二进制目标文件的包能够被安装

通常 crate 的 *README* 文件中有该 crate 是库、二进制目标还是两者兼有的信息



```bash
cargo install ripgrep
```



# Cargo 自定义拓展命令

如果 `$PATH` 中有类似 `cargo-something` 的二进制文件，就可以通过 `cargo something` 来像 Cargo 子命令一样运行它。像这样的自定义命令也可以运行 `cargo --list` 来展示出来
