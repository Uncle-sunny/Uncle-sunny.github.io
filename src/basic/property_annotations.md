# 属性注解

## 概念

属性注解就是元数据, 用于为代码项添加额外信息

可以作用于**结构体 函数 模块**

能够影响编译器的行为,或者被某些工具用来生成代码,文档

属性注解分为 内建属性和自定义属性



## 常用内建属性

### #[derive]

derive(派生的意思)

` #[derive]` 常常用于自动为结构体或者枚举实现一些常用的 trait 例如:

```rust
#[derive(Debug, Clone, Copy)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let pos = Point { x: 30, y: 30 };
    println!("{:?}", pos)
}
```



### #[cfg]

` #[cfg]`常常用于条件编译,根据指定的配置来决定是否编译某段代码

```rust
#[cfg(target_os = "windows")]
fn system_info() {
    println!("this is windows");
}

#[cfg(target_os = "linux")]
fn system_info() {}

#[cfg(target_os = "macos")]
fn system_info() {
    println!("this is macos");
}

fn main() {
    system_info();
}
```

### #[allow]

`#[allow]` 用于抑制特定的编译警告

```rust
#[allow(unused_variables)]
fn main() {
    let a = 10;
    // 变量a未使用，但不会产生警告
}
```

## 自定义属性

自定义属性可以用来为代码添加特定领域的元数据

通过宏来处理自定义属性





