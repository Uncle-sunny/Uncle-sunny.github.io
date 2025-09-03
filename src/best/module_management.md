# crate 管理 (crate management)

## 理解

crate 可以理解为别的语言的package，它是一个独立的单元，可以被其他crate依赖和使用。

在rust中

- crate是编译时最小的单位代码，可以包含多个模块和函数
- 每个 **crate** 会包含一个 **Cargo.toml** 文件，阐述如何去构建这些 **crate**
- 一个包可以包含多个**二进制 crate** 项和一个可选的 **crate 库**，但是至少包含一个crate

## crate 分类

**crate** 分为二进制(binary)和库(library)两种类型。

### 二进制(binary) crate

binary crate 可以被编译成可执行文件

- 必须有一个 `main` 函数来定义程序被执行的时候需要做的事情
- 可以有多个二进制项
- Cargo 遵循的一个约定：`src/main.rs` 就是一个与包同名的二进制 crate 的 crate 根。

```rust
// + examples
//   - example1.rs
//   - example2.rs
// + src
//   - main.rs
//   - other_executable1.rs
//   - other_executable2.rs
//   - lib.rs
// + Cargo.toml
```

目录结构如上

在 `src/main.rs` 和 `examples/example1.rs` 中，需要有一个 **main** 函数。

在 `Cargo.toml` 中，可以添加多个 二进制项

```toml
[[bin]]
name = "executable1"
path = "src/other_executable1.rs"

[[bin]]
name = "executable2"
path = "src/other_executable2.rs"
```

运行 `cargo run --bin executable1` 或 `cargo run --bin executable2`

对于 `examples` 目录下的二进制项，可以使用 `cargo run --example example1` 或 `cargo run --example example2`

当然如果你不想在 `Cargo.toml` 中添加 `[[bin]]` 配置项，rust 也提供了另一种选择，你可以把需要的二进制项放在 `src/bin` 目录下，rust 会自动编译它们。

```rust
// ├── Cargo.lock
// ├── Cargo.toml
// └── src
//      ├── main.rs
//      └── bin
//           ├── executable1.rs
//           └── executable2.rs
```

至于第三点关于根的问题，会在下文的 `use` 中做出解释

### 库(library) crate

库中没有 `main` 函数 也不会被编译成可执行程序,他们提供一些函数之类的东西,别的项目也能使用

**大多数时间 `Rustaceans` 说的 crate 指的都是库，这与其他编程语言中 library/package 概念一致**

如果包目录中包含 `src/lib.rs`，则包带有与其同名的库 crate，且 `src/lib.rs` 是 crate 根，这点会在下文的 `use` 中做出解释

## 模块

模块可以理解为代码组织的单元，它允许我们将代码分割成更小、更可管理的部分，并且可以控制代码的可见性。

通俗的说它像一个盒子，里面可以放各种各样的成员，比如函数、结构体、枚举等等。通过模块，我们可以将代码组织得更加清晰、易于维护和复用。

## 使用

rust提供了多种模块的组织方式

内嵌模块

```rust
mod inner_module {
    fn inner_function() {
        println!("This is an inner function");
    }
}
```

模块也可以嵌套

```rust
mod outer_module {
    mod inner_module {
        fn inner_function() {
            println!("This is an inner function");
        }
    }
}
```

如果项目比较简单，这样的做法没有问题，一旦项目变大，模块的嵌套会变得复杂，难以维护。rust提供了另外的模块组织方式，可以使用文件来组织模块，每个文件都可以作为一个模块，同样每个模块都可以包含多个子模块。

```rust
// .
// ├── Cargo.lock
// ├── Cargo.toml
// └── src
//     ├── cli.rs
//     ├── lib.rs
//     └── main.rs
```

通常我们会将模块组织在 `lib.rs` 中,也可以在入口文件 `main.rs` 中, 但是我们通常不建议这么做，尽量保持`main.rs` 的简洁

```rust
// lib.rs中
mod cli;
```

在lib.rs中我们声明了一个cli模块，rust编译器会在同级目录下寻找这个 `cli` 模块, 通常我们会在同级目录中创建一个 `cli.rs` 文件

但是由于历史原因rust实际上还提供了另外一种方式，在同级目录中创建一个 `cli` 目录，在 `cli` 目录中创建一个 `mod.rs` 文件

> - src/cli.rs（推荐）
> - src/cli/mod.rs（老风格，不过仍然支持）

但是这两种方式不可以在一个crate中混用，在 **rust圣经** 中更推荐前者，但是在实际开发中，很多开发者比较青睐后者
这种好比在别的语言在模块中定义了一个出口文件，类似 `index.js` 在 nodejs 中，`__init__.py`在 python 中,当然啦这方面没有对和错，如果个人开发可以使用自己喜欢的方式，如果是团队开发，建议和团队保持一致性，避免给团队带来困扰

那如果我们需要在 `cli` 模块中再创建一个子模块，可以使用以下方式：

```rust
// cli.rs中
mod sub_cli;
```

在 `cli` 模块中我们声明了一个 `sub_cli` 子模块，这时候需要在同级目录下创建一个cli目录，然后在其中添加一个 `sub_cli.rs` 文件，看起来像这样

```rust
// .
// ├── Cargo.lock
// ├── Cargo.toml
// └── src
//     ├── cli
//     │   └── sub_cli.rs
//     ├── cli.rs
//     ├── lib.rs
//     └── main.rs
```

## 使用 pub 关键字控制模块成员共有或者私有

一个模块里面的代码默认对父模块私有。为了使一个模块公用，或者一个公用模块中的内部成员公用，应该在声明前使用 `pub`

```rust
pub mod animal {
    pub mod cat {
        pub const MEOW: &str = "meow";
        pub fn meow() {
            println!("meow");
        }

        pub struct Cat {
            pub name: String,
            age: u8,
        }

        pub enum CatKind {
            Domestic,
            Wild,
        }

        pub trait CatTrait {
            fn meow(&self);
        }

        pub type CatRef = &'static str;
    }
    pub mod dog
}
```

模块，常量，函数，结构体，枚举，trait，类型，这些都可以设置公有和私有，当然这里特别强调一下 `struct` 和 `enum`, enum 是 **或** 语义的结构，当在 `enum` 中使用 `pub` 时，所有成员都会被标记为公有，而 `struct` 中的字段则不会。

`struct` 是 **与** 语义的结构，如果你想要给某个字段(field)设置公有，需要在字段前加上 `pub` 关键字，例如小猫的名字，如果不想小猫的年纪被别人知道，那就不需要添加

pub 额外用法

指定 `pub` 范围，这是对公有更具细粒度的控制，例如，你可以指定一个模块中的某个成员只对当前模块的父模块可见，或者只对当前 `crate` 可见。

使用 `pub(crate)` 表示仅在当前 `crate` 公有，对使用这个 crate 外部的人保持私有

使用 `pub(super)` 表示仅在当前模块的父模块公有，父模块之外就不可用了


```rust

pub mod animal {
    pub mod cat {
        pub(super) const MEOW: &str = "meow";
        pub(crate) fn meow() {
            println!("meow");
        }
    }
    pub mod dog{}
}
```

## use 关键字引入代码

在一个作用域内，`use`关键字创建了一个成员的快捷方式，用来减少长路径的重复。

```rust
pub mod animal {
    pub mod cat {
        pub fn meow() {
            println!("The cat is meowing");
        }

        pub fn run() {
            println!("The cat is running");
        }
    }
}

use animal::cat;

fn action() {
    cat::run();
    cat::meow();
}
```

如果不这样使用的话，那么需要使用全路径来访问模块中的成员，例如：

```rust
fn action() {
    animal::cat::run();
    animal::cat::meow();
}
```

### use 路径的形式

理解 **crate 根**

```rust
// .
// ├── Cargo.lock
// ├── Cargo.toml
// └── src
//     ├── lib.rs   ---这是一个crate根
//     └── main.rs  ---这也是一个crate根
```
我们常用的组织文件中包含 `lib.rs` 、`main.rs` 这实际上是两个**crate**，两个**crate**是相互独立的，这会影响对 `use` 关键字的使用

常用的文件路径分为绝对路径和相对路径，`use` 关键字对模块的引用也是如此

绝对路径

以 crate 根（root）开头的全路径，以字面值 `crate` 开头。

```rust
// lib.rs
use crate::animal::cat;
```
这里的 crate 指的是 `lib.rs`的根，和 `main.rs` 的根是独立的

相对路径

从当前模块开始，以 `self`、`super` 或定义在当前模块中的标识符开头。

```rust
pub mod animal {
    pub mod cat {
        pub fn run() {
            println!("The cat is running");
        }
    }
    pub mod dog {
        use super::cat;
        pub fn intimidate() {
            cat::run();
        }
    }
}
```
super 可以引入父模块的内容

```rust
pub mod animal {
    pub mod cat {
        pub fn run() {
            println!("The cat is running");
        }
    }
}

use self::animal::cat;

fn run() {
    cat::run();
}
```

self 可以引入当前模块的内容, 指代自己当下的模块

### 额外的一些使用

使用嵌套路径消除大量的行

```rust
use std::io;
use std::collections;
use std::fs;
```
如果需要引入多个标准库里面的路径，我们会发现前面的std会大量重复，我们可以使用嵌套路径来消除大量的行

```rust
use std::{io, collections, fs};
```

使用glob通配符*引入模块中所有的成员,通常不建议这么做，因为这可能会导致命名冲突和难以追踪的依赖关系。

```rust
pub mod animal {
    pub mod cat {
        pub fn run() {
            println!("The cat is running");
        }
        pub fn eat() {
            println!("The cat is eating");
        }
    }
}

use animal::cat::*;

fn run() {
    run();
    eat();
}
```

使用 self和嵌套路径，对自身以及子模块的部分成员进行引入,适用于仅部分使用子成员，又想保留自身

```rust
pub mod animal {
    pub mod cat {
        pub fn run() {
            println!("The cat is running");
        }
        pub fn eat() {
            println!("The cat is eating");
        }
    }
}

use animal::cat::{self, eat};

fn run() {
    cat::run();
    eat();
}
```

使用 as 关键字别名引入，可以防止重名，在一些trait和函数方法同名的场景中可以使用 `as _`

```rust
use animal::cat::{self as cat_module, eat as cat_eat};

fn run() {
    cat_module::run();
    cat_eat();
}
```

```rust
// 解决命名冲突问题
use tracing_subscriber::{Layer as _, fmt::Layer};
```

使用 pub use 重新导出, 有些库的模块嵌套很深，如果引入的话非常麻烦，可以使用 `pub use` 来重新导出模块中的成员，这样就可以在外部直接使用这些成员，而不需要引入整个模块。在写库的时候非常有用

```rust
pub mod animal {
    pub mod cat {
        pub fn run() {
            println!("The cat is running");
        }
        pub fn eat() {
            println!("The cat is eating");
        }
    }

    pub use cat::*;
}

use animal::{eat, run};

fn action() {
    run();
    eat();
}

```

> 前面我们说到 `lib.rs` 和 `main.rs` 是两个单独的 crate，它们各自拥有独立的根模块，因此它们的模块结构和命名空间是相互独立的。
> 在 `lib.rs` 中定义的模块和函数，无法在 `main.rs` 通过 `crate::*` 引入
> 如果想在 `main.rs` 中使用 `lib.rs` 中的模块和函数，可以通过 `cargo.toml` 里面的包名引入，例如：
> ```rust
> use your_package_name_in_cargo_toml::animal::cat::{run, eat};
> ```
> 记得要在需要使用的 `mod` 前面加上 `pub`


## 使用 workspace 组织代码

随着项目工程的扩大，可以使用 workspace 来组织代码，workspace 是一个包含多个 crate 的项目，可以方便地管理多个 crate 的依赖和构建。

在顶级目录下的 `Cargo.toml` 中定义 workspace, 这里的话需要指定 `resolver` 详细解析可以查看[cargo官网](https://doc.rust-lang.org/cargo/reference/workspaces.html)

```toml
[workspace]
members = [ "migration","util"]
resolver = "2"
[workspace.dependencies]
anyhow = "1.0.0"
tokio = { version = "1.47.1", features = ["rt"] }
```
通过 `cargo new menber` 添加新的 crate

顶级路径下的 **dependencies**， 可以被各个子 crate 使用

现在文件的 **tree** 如下
```rust
// .
// ├── Cargo.lock
// ├── Cargo.toml
// ├── migration
// │   ├── Cargo.toml
// │   └── src
// │       └── main.rs
// └── util
//     ├── Cargo.toml
//     └── src
//         └── main.rs
```

如果想在 `util` 目录下面使用 workspace 的 **dependencies**，需要在 `util` 目录下 `Cargo.toml` 中添加对 `workspace.dependencies` 的依赖

```toml
[package]
name = "util"
version = "0.1.0"
edition = "2024"

[dependencies]
anyhow = { workspace = true }
tokio = { workspace = true, features = ["rt-multi-thread", "macros"] }
```

当然每个子 crate 都可以添加自己的依赖，或者使用别的子成员

```toml
[package]
name = "migration"
version = "0.1.0"
edition = "2024"

[dependencies]
anyhow = { workspace = true }
tokio = { workspace = true, features = ["rt-multi-thread", "macros"] }
serde = "1.0.130"
migration = { path = "../migration" }
```

如果有什么错误或者遗漏的地方欢迎批评指正
