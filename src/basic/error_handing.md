# 错误处理

Rust 要求你承认错误的可能性，并在你的代码编译前采取一些行动。这一要求使你的程序更加健壮，因为它可以确保你在将代码部署到生产环境之前就能发现错误并进行适当的处理。

# 分类

## **可恢复的**（recoverable）错误

对于一个可恢复的错误，比如文件未找到的错误，我们很可能只想向用户报告问题并重试操作。

有 `Result<T, E>` 类型，用于处理可恢复的错误

## **不可恢复的**（unrecoverable）错误

不可恢复的错误总是 bug 出现的征兆，比如试图访问一个超过数组末端的位置，因此我们要立即停止程序。

有 `panic!` 宏，在程序遇到不可恢复的错误时停止执行。



# panic！宏

## 触发

1. 执行代码会触发 panic（比如访问超过数组结尾的内容）
2. 显示调用 `panic!` 宏

> 通常情况下这些 panic 会打印一个错误信息，展开并清理栈数据，然后退出

```rust
fn main() {
	panic!("crash and burn")
}
```



## 内存清理

出现怕panic时候程序默认会展开，这意味着Rust 会回溯栈并清理它遇到的每一个函数的数据，不过这个回溯并清理的过程有很多工作。

另一种选择是直接**终止**（abort），这会不清理数据就退出程序，内存由操作系统释放，这种方法需要修改Cargo.toml配置文件

```toml
[profile.release]
panic = 'abort'
```

## 回溯（backtrace）

```rust
fn main() {
	let v = vec![1, 3, 5];
	v[100];
}
```

C 语言中，尝试读取数据结构之后的值是未定义行为（undefined behavior）。你会得到任何对应数据结构中这个元素的内存位置的值，甚至是这些内存并不属于这个数据结构的情况。这被称为 **缓冲区溢出**（buffer overread），并可能会导致安全漏洞，比如攻击者可以像这样操作索引来读取储存在数据结构之后不被允许的数据。

读 backtrace 的关键是从头开始读直到发现你编写的文件

为了获取带有这些信息的 backtrace，必须启用 debug 标识。当不使用 `--release` 参数运行 cargo build 或 cargo run 时 debug 标识会**默认启用**

# Result 处理可恢复的错误

```rust
enum Result<T, K> {
	Ok(T),
	Error(E),
}
```

`T` 代表成功时返回的 `Ok` 成员中的数据的类型，而 `E` 代表失败时返回的 `Err` 成员中的错误的类型。

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("test.txt");
    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```

注意与 `Option` 枚举一样，`Result` 枚举和其成员也被导入到了 prelude 中，所以就不需要在 `match` 分支中的 `Ok` 和 `Err` 之前指定 `Result::`。

## 匹配不同的错误

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");
    match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Create file defailed {e:?}"),
            },
            other_error => {
                panic!("Problem in opening the file: {other_error}")
            }
        },
    };
}
```



File::open返回的 `Err` 成员中的值类型 `io::Error` ,它是一个标准库中提供的结构体,这个结构体有一个返回 `io::ErrorKind` 值的 `kind` 方法可供调用

> 这里使用了很多 match 语句， 在处理代码中 Result<T, E> 值时使用闭包会更加简洁

## 失败时 panic 的简写：unwrap 和 expect

```rust
use std::fs::File

fn main() {
	let greeting_file = File::open("hello.txt").unwrap();
}
```

`Result<T, E>` 类型定义了很多辅助方法来处理各种情况, 其中之一叫做 `unwrap`

`Result` 值是成员 `Ok`，`unwrap` 会返回 `Ok` 中的值。如果 `Result` 是成员 `Err`，`unwrap` 会为我们调用 `panic!` 



还有一种是 `expect` 

使用 `expect` 而不是 `unwrap` 并提供一个好的错误信息可以表明你的意图并更易于追踪 panic 的根源

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

> unwrap 是默认的错误信息， expect 是我们传递的错误信息

**尽量使用 expect**



## 传播错误

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");
    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    let mut username = String::new();
    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
fn main() {
    let result = read_username_from_file();
    print!("{result:?}\n");
}
```

这样无论是成功调用还是出现错误，结果都在这个函数的返回值中，是否决定panic取决于函数的调用者

## 传播错误的简写形式 ？ 运算符

```rust
fn main() {
    fn read_username_from_file() -> Result<String, io::Error> {
        let mut username_file = File::open("hello.txt")?;
        let mut username = String::new();
        username_file.read_to_string(&mut username)?;
        Ok(username)
    }
    let result = read_username_from_file();
    println!("{result:?}");
}
```

如果 `Result` 的值是 `Ok`，这个表达式将会返回 `Ok` 中的值而程序将继续执行。如果值是 `Err`，`Err` 将作为整个函数的返回值，就好像使用了 `return` 关键字一样，这样错误值就被传播给了调用者。

`?` 运算符消除了大量样板代码并使得函数的实现更简单。我们甚至可以在 `?` 之后直接使用链式方法调用来进一步缩短代码

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();
    File::open("hello.txt")?.read_to_string(&mut username)?;
    Ok(username)
}
```



actually, 有更简短的写法

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

将文件读取到一个字符串是相当常见的操作，所以 Rust 提供了名为 `fs::read_to_string` 的函数，它会打开文件、新建一个 `String`、读取文件的内容，并将内容放入 `String`，接着返回它。当然，这样做就没有展示所有这些错误处理的机会了，所以我们最初就选择了艰苦的道路。

## 哪里可以使用

`?` 运算符被定义为从函数中提早返回一个值，与 `match` 表达式有着完全相同的工作方式



错误信息也提到 `?` 也可用于 `Option<T>` 值。如同对 `Result` 使用 `?` 一样，只能在返回 `Option` 的函数中对 `Option` 使用 `?`



目前为止，我们所使用的所有 `main` 函数都返回 `()`。`main` 函数是特殊的因为它是可执行程序的入口点和退出点，为了使程序能正常工作，其可以返回的类型是有限制的。

幸运的是 `main` 函数也可以返回 `Result<(), E>`，不过要修改 `main` 的返回值为 `Result<(), Box<dyn Error>>` 并在结尾增加了一个 `Ok(())` 作为返回值

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}；
```

可以将 `Box<dyn Error>` 理解为 “任何类型的错误”

# 要不要 panic

把选择权交给函数的调用者

返回 `Result` 是定义可能会失败的函数的一个好的默认选择。

在一些类似示例、原型代码（prototype code）和测试中，panic 比返回 `Result` 更为合适

当错误预期会出现时，返回 `Result` 仍要比调用 `panic!`

函数通常都遵循 **契约**（*contracts*）：它们的行为只有在输入满足特定条件时才能得到保证。当违反契约时 panic 是有道理的

。函数的契约，尤其是当违反它会造成 panic 的契约，应该在函数的 API 文档中得到解释。