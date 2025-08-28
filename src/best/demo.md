# 包

Cargo 的一个功能，允许你构建、测试和分享crate

**一个包会包含一个 *Cargo.toml* 文件，阐述如何去构建这些 crate**

一个包可以包含多个二进制 crate 项和<u>一个可选的 crate 库</u>,但是至少包含一个crate

# Crate

一个树形结构

crate 是 Rust 在编译时最小的代码单位

## 形式

crate 有两种形式：二进制项和库

### 二进制项

可以被编译为可执行程序，比如一个命令行程序或者一个 web server

必须有一个 `main` 函数来定义程序被执行的时候需要做的事情

Cargo 遵循的一个约定：`src/main.rs` 就是一个与包同名的二进制 crate 的 crate 根。

### 库

库中没有 `main` 函数 也不会被编译成可执行程序,他们提供一些函数之类的东西,别的项目也能使用

**大多数时间 `Rustaceans` 说的 crate 指的都是库，这与其他编程语言中 library 概念一致**

如果包目录中包含 `src/lib.rs`，则包带有与其同名的库 crate，且 `src/lib.rs` 是 crate 根

# 模块

## 模块查找规则

### 从 crate 根节点开始

当编译一个 crate, 编译器首先在 crate 根文件（通常，对于一个库 crate 而言是*src/lib.rs*，对于一个二进制 crate 而言是*src/main.rs*）中寻找需要被编译的代码

### 声明模块

在crate根文件中，声明一个新模块，比如用 `mod garden;`声明了一个叫做 `garden` 的模块编译器查找规则

1. 内联，在大括号中, 当 `mod garden` 后方不是一个分号而是一个大括号

```rust
mod garden {
    // ****
}
```

2. 在文件 `src/garden.rs`
3. 在文件 `src/garden/mod.rs`



### 声明子模块

在除了 crate 根节点以外的其他文件中，可以定义子模块，编译器会在以父模块命名的目录中寻找子模块代码

比如在 `src/garden.rs` 中定义了 `mod vegetables`

1. 内联，在大括号中，当 `mod vegetables` 后方不是一个分号而是一个大括号

```rust
mod garden {
    mod vegetables {
        // *****
    }
}
```

### 模块中的代码路径

一旦一个模块是你 crate 的一部分，你可以在隐私规则允许的前提下，从同一个 crate 内的任意地方，通过代码路径引用该模块的代码

例如 garden vegetables 模块下的 `Asparagus` 类型可以在`crate::garden::vegetables::Asparagus`被找到。

### 私有和公有

一个模块里面的代码默认对父模块私有。为了使一个模块公用，或者一个公用模块中的内部成员公用，应该在声明前使用 `pub`

```rust
pub mod garden{
    pub fn add() {}
}
```

### use 关键字

在一个作用域内，`use`关键字创建了一个成员的快捷方式，用来减少长路径的重复。

在任何可以引用`crate::garden::vegetables::Asparagus`的作用域

可以通过 `use crate::garden::vegetables::Asparagus;`创建一个快捷方式

然后可以在作用域中只写`Asparagus`来使用该类型。



# 引用模块项目路径

## 路径形式

绝对路径和相对路径都后跟一个或多个由双冒号（`::`）分割的标识符。

1. 绝对路径

以 crate 根（root）开头的全路径；对于外部 crate 的代码，是以 crate 名开头的绝对路径，对于当前 crate 的代码，则以字面值 `crate` 开头。

2. 相对路径

从当前模块开始，以 `self`、`super` 或定义在当前模块中的标识符开头。



> 建议使用绝对路径，因为把代码定义和项调用各自独立地移动是更常见的。（实际取决于独立移动还是单独移动）

父模块中的项不能使用子模块中的私有项，子模块中的项可以使用它们父模块中的项,兄弟模块可以相互使用



`supper`开始的相对路径 从父模块开始构建相对路径，而不是从当前模块或者 crate 根开始。这类似以 `..` 语法开始一个文件系统路径



# 创建公有的结构体和枚举

在结构体和枚举上使用 `pub` 有一些额外的细节需要注意

在一个结构体定义的前面使用了 `pub` ，这个结构体会变成公有的，但是这个结构体的字段仍然是私有的

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // 在夏天订购一个黑麦土司作为早餐
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // 改变主意更换想要面包的类型
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // 如果取消下一行的注释代码不能编译；
    // 不允许查看或修改早餐附带的季节水果
    // meal.seasonal_fruit = String::from("blueberries");
}
```





如果我们将枚举设为公有，则它的所有成员都将变为公有

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

# 使用 use 关键字把路径引入作用域

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

use关键字可以用来创建简短的路径，类似于在文件系统中创建软连接（符号连接，symbolic link）

`use` 只能创建 `use` 所在的特定作用域内的短路径

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}
```

使用 as 关键字可以提供新的名称（避免冲突问题）

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

使用 pub use 重导出名称

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```



嵌套路径消除大量的use行

```rust
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--



// --snip--
use std::{cmp::Ordering, io};
// --snip--

```

使用 glob 运算符将所有的公有定义引入作用域

```rust
use std::collections::*;
```

这样会有风险

# 组织代码目录和文件

> - src/front_of_house/hosting.rs（推荐）
> - src/front_of_house/hosting/mod.rs（老风格，不过仍然支持）

目录中使用 `mod.rs` 是一种老的风格仍然支持，但是如果在同一模块中使用两种风格会编译错误
