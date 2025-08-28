# 理解

Rust 包含了编写自动化软件测试的功能支持。

# 如何编写测试

测试函数体通常执行如下三种操作

>1. 设置任何所需的数据或者状态
>2. 运行需要测试的代码
>3. 断言其结果是我们所期望的

**Rust 中的测试就是一个带有 `test` 属性注解的函数**

属性（attribute）是关于 Rust 代码片段的元数据；

```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```

`#[test]`：这个属性表明这是一个测试函数，这样测试执行者就知道将其作为测试处理。

`assert_eq!` 宏来断言 2 加 2 等于 4

每一个测试都在一个新线程中运行，当主线程发现测试线程异常了，就将对应测试标记为失败。

## 使用 assert！宏来检查结果

需要向 `assert!` 宏提供一个求值为布尔值的参数。如果值是 `true`，`assert!` 什么也不做，同时测试会通过。如果值为 `false`，`assert!` 调用 `panic!` 宏，这会导致测试失败

`assert_eq!` 和 `assert_ne!` 宏在底层分别使用了 `==` 和 `!=`

这意味着被比较的值必须实现了 `PartialEq` 和 `Debug` trait

**自定义信息**

任何在 `assert!` 的一个必需参数和 `assert_eq!` 和 `assert_ne!` 的两个必需参数之后指定的参数都会传递给 `format!` 宏

# 使用should_panic 检查 panic

should_panic 也是一个属性注解

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    // 属性注解在函数上面
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

以给 `should_panic` 属性增加一个可选的 `expected` 参数。测试工具会确保错误信息中包含其提供的文本。

`#[should_panic(expected = "less than or equal to 100")]`

# 将Result<T, E>用于测试

```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    // ANCHOR: here
    #[test]
    fn it_works() -> Result<(), String> {
        let result = add(2, 2);

        if result == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
    // ANCHOR_END: here
}
```

这样编写测试来返回 `Result<T, E>` 就可以在函数体中使用问号运算符，如此可以方便的编写任何运算符会返回 `Err` 成员的测试。

> 测试通过时返回 `Ok(())`，在测试失败时返回带有 `String` 的 `Err`。
>
> 不能对这些使用 `Result<T, E>` 的测试使用 `#[should_panic]` 注解。
>
> 为了断言一个操作返回 `Err` 成员，**不要**使用对 `Result<T, E>` 值使用问号表达式（`?`）。而是使用 `assert!(value.is_err())`

# 并行或者连续的运行测试

当运行多个测试时，Rust 默认使用线程来并行运行。这意味着测试会更快地运行完毕，所以你可以更快的得到代码能否工作的反馈。因为测试是在同时运行的，你**应该确保测试不能相互依赖，或依赖任何共享的状态，包括依赖共享的环境，比如当前工作目录或者环境变量**。

可以传递 `--test-threads` 参数和希望使用线程的数量给测试二进制文件。例如：

```bash
cargo test --test-threads
```

# 显示函数输出

测试通过时，Rust 的测试库会**截获**打印到标准输出的所有内容

只会看到说明测试通过的提示行。如果测试失败了，则会看到所有标准输出和其他错误信息。

可以传递 `--show-output` 参数告诉 Rust 显示成功测试的输出

# 筛选测试

## 单个测试

可以向 `cargo test` 传递任意测试的名称来只运行这个测试

```bash
cargo test one_hundred
```

## 过滤运行多个测试

我们可以指定部分测试的名称，任何名称匹配这个名称的测试会被运行

```bash
cargo test add
```

这运行了所有名字中带有 `add` 的测试

**实践中注意以名字分类测试**



## 忽略某个测试

使用 `ignore` 属性注解

```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

// ANCHOR: here
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }

    #[test]
    #[ignore]
    fn expensive_test() {
        // 需要运行一个小时的代码
    }
}
// ANCHOR_END: here
```

如果我们只希望运行被忽略的测试,可以运行

```bash
cargo test -- --ignored
```

管是否忽略都要运行全部测试，可以运行

```bash
cargo test -- --include-ignored
```

# 测试的组织结构

Rust 社区倾向于根据测试的两个主要分类来考虑问题：**单元测试**（*unit tests*）与 **集成测试**（*integration tests*）

## 单元测试

单元测试与它们要测试的代码共同存放在位于 *src* 目录下相同的文件中。规范是在每个文件中创建包含测试函数的 `tests` 模块，并使用 `cfg(test)` 标注模块。

测试模块的 `#[cfg(test)]` 注解告诉 Rust 只在执行 `cargo test` 时才编译和运行测试代码

集成测试因为位于另一个文件夹，所以它们并不需要 `#[cfg(test)]` 注解

```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```

`cfg` 属性代表*配置*（*configuration*） ，它告诉 Rust，接下来的项，只有在给定特定配置选项时，才会被包含。在这种情况下，配置选项是 `test`

## 集成测试

为了编写集成测试，需要在项目根目录创建一个 *tests* 目录，与 *src* 同级。Cargo 知道如何去寻找这个目录中的集成测试文件。接着可以随意在这个目录中创建任意多的测试文件，Cargo 会将每一个文件当作单独的 crate 来编译

```txt
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

文件名：tests/integration_test.rs

```rust
use adder::add_two;

#[test]
fn it_adds_two() {
    let result = add_two(2);
    assert_eq!(result, 4);
}
```

`tests` 文件夹在 Cargo 中是一个特殊的文件夹，Cargo 只会在运行 `cargo test` 时编译这个目录中的文件

可以使用 `cargo test` 的 `--test` 后跟文件的名称来运行某个特定集成测试文件中的所有测试：

```bash
cargo test --test integration_test
```

为了不让一些工具函数出现在测试中

可以创建 *tests/common/mod.rs* ，而不是创建 *tests/common.rs* 

```txt
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

这就是许多 Rust 二进制项目使用一个简单的 *src/main.rs* 调用 *src/lib.rs* 中的逻辑的原因之一。因为通过这种结构，集成测试 **就可以** 通过 `extern crate` 测试库 crate 中的主要功能了