# 理解

trait 定义了某个特定类型可能与其他类型共享的功能，和函数签名很类似，是一种抽象

> trait 类似于其他语言中的常被称为 接口（interfaces）的功能，虽然不同

# 定义

```rust
pub trait Summary {
	fn summarize(&self) -> String;
}
```

trait 体中可以有多个方法：一行一个方法签名且都以分号结尾。

# 实现trait

```rust
pub struct NewsArticle {
	pub headline: String,
	pub location: String,
  pub author: String,
  pub content: String,
}

impl Summary for NewsArticle {
	fn summarize(&self) -> String {
		format!("{self.headline}-{self.location}-{self.author}")
	}
}
```

在于 `impl` 关键字之后，我们提供需要实现 trait 的名称, 接着是 `for` 和需要实现 trait 的类型的名称

trait 必须和类型一起引入作用域以便使用额外的 trait 方法

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

只有在 trait 或类型至少有一个属于当前 crate 时，我们才能对类型实现该 trait

这个限制是被称为 **相干性**（_coherence_）的程序属性的一部分，或者更具体的说是 **孤儿规则**（_orphan rule_），其得名于不存在父类型

## 定义 trait 时默认实现

```rust
pub trait Summary {
	fn summarize(&self) -> String {
		String::from("This is default")
	}
}
```

## trait 作为参数

```rust
pub fn notify(item: &impl Summary) {
	println!("Breaking news! {}", item.summarize());
}
```

这里就是传入了一个实现 Summary trait的类型

## Trait Bound 语法

`impl Trait` 语法更直观，但它实际上是更长形式的 _trait bound_ 语法的语法糖。

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

`impl Trait` 很方便，适用于短小的例子。trait bound 则适用于更复杂的场景

```rust
pub fn notify1(item1: &impl Summary, item2: &impl Summary) {}

pub fn notify2<T: Summary>(item1: &T, item2: &T) {}
```

相比较1， 2的写法更加简单

### 通过 `+` 可以制定多个 trait bound

```rust
pub fn notify(item: &(impl Summary + Display)) {}
pub fn notify<T: Summary + Display>(item: &T) {}
```

### 通过 where 简化 trait bound

```rust
fn some_function1<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}

fn some_function2<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{}
```

## 返回实现了 trait 的类型

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

通过使用 `impl Summary` 作为返回值类型，我们指定了 `returns_summarizable` 函数返回某个实现了 `Summary` trait 的类型，但是不确定其具体的类型。

## 使用 trait bound 有条件地实现方法

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

这样仅仅给实现了 `Display + PartialOrd` trait 的 Pair 实现了 `cmp_display` 方法

也可以对任何实现了特定 trait 的类型有条件地实现 trait。对任何满足特定 trait bound 的类型实现 trait 被称为 _blanket implementations_，它们被广泛的用于 Rust 标准库中。

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```
