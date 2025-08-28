# 集合类型

vector 允许我们一个挨着一个地储存一系列数量可变的值

字符串（string） 是字符的集合

哈希（hash map）允许我们将值与一个特定的键（key）相关联



# Vector

## 理解

vector 允许我们在一个单独的数据结构中储存多于一个的值，它在内存中彼此相邻地排列所有的值

vector 只能储存相同类型的值

## 创建 vector

```rust
let v: Vec<i32> = Vec::new();
```

或者使用宏创建,会把每一项默认推断为 `i32` 类型的

```rust
let v = Vec![1, 2, 3];
```



## 更新 vector

对于一个新建的 vector 可以使用 push 方法

```rust
let mut v = Vec::new()
v.push(1)
v.push(2)
```

果想要能够改变它的值，必须使用 `mut` 关键字使其可变



## 读取 vector 的元素

有两种方法引用 vector 中存储的值：通过索引或者是 get 方法

```rust
let v = vec![1, 2, 3, 4, 5];
let third: &i32 = &v[2];
println!("The third element is {third}");
let third: Option<&i32> = v.get(2);
match third {
    Some(third) => println!("The third element is {third}"),
    None => println!("There is no third element."),
}
```



```rust
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

第一种会直接报错

这个方法更适合当程序认为尝试访问超过 vector 结尾的元素是一个严重错误的情况，这时应该使程序崩溃。

第二种会返回一个none

索引可能来源于用户输入的数字。如果它们不慎输入了一个过大的数字那么程序就会得到 `None` 值，你可以告诉用户当前 vector 元素的数量并再请求它们输入一个有效的值。这就比因为输入错误而使程序崩溃要友好的多



>一旦程序获取了一个有效的引用，借用检查器将会执行所有权和借用规则 来确保 vector 内容的这个引用和任何其他引用保持有效

```rust
let mut v = vec![1, 2, 3, 4, 5];
let first = &v[0];
v.push(6);
println!("The first element is: {first}");
```

这样是会报错的，在对 vector 添加新元素时，在没有足够空间将所有元素依次相邻存放的情况下，可能会要求分配新内存并将老的元素拷贝到新的空间中，这时，**第一个元素的引用就指向了被释放的内存**。借用规则阻止程序陷入这种状况。



## 遍历 vector 的元素

```rust
// 遍历
let mut v = vec![1, 2, 3, 4];
for i in &v {
    println!("{i}")
}

// 修改
let mut v = vec![1, 2, 3, 4];
for i in &mut v {
    *i += 5;
}
```

`for` 循环中获取的 vector 引用阻止了同时对 vector 整体的修改。



## 使用枚举在 vector 中存储多种类型

枚举的成员都被定义为相同的枚举类型，所以当需要在 vector 中储存不同类型值时，我们可以定义并使用一个枚举

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let mut v = vec![S]
```



> Rust 在编译时必须确切知道 vector 中的类型，这样它才能确定在堆上需要为每个元素分配多少内存。我们还必须明确这个 vector 中允许的类型。如果 Rust 允许 vector 存储任意类型，那么可能会因为一个或多个类型在对 vector 元素执行操作时导致（类型相关）错误。

关于 vector 的其他 [Api](https://doc.rust-lang.org/std/vec/struct.Vec.html) 文档



# string 

字符串就是作为字节的集合外加一些方法实现的，当这些字节被解释为文本时，这些方法提供了实用的功能。

`String` 中有任何集合类型都有的操作，比如创建、更新和读取，

`String` 与其他集合不一样的地方，例如索引 `String` 是很复杂的

## 理解

字符串（`String`）类型由 Rust 标准库提供，而不是编入核心语言，它是一种可增长、可变、可拥有、**UTF-8 编码**的字符串类型

> 常常提及的 Rust 中的 “字符串”， 可能指的是 String 或者 string slice `&str` 类型，`String` 和 字符串 slices 都是 UTF-8 编码的。

## 创建 string

很多 `Vec` 可用的操作在 `String` 中同样可用，事实上 `String` 被实现为一个带有一些额外保证、限制和功能的字节 vector 的封装。

1. 通过 `new` 函数

```rust
let mut s = String::new();
```

2. 通过字面常量

通常字符串会有初始数据，因为我们希望一开始就有这个字符串。为此，可以使用 `to_string` 方法，它能用于任何实现了 `Display` trait 的类型，比如字符串字面值

```rust
let data = "initial contents";
let s = data.to_string();

// 该方法也可以直接作用于字符串字面量
let s = "initial contents".to_string();
```

## 更新 string

`String` 的大小可以增加，其内容也可以改变，就像可以放入更多数据来改变 `Vec` 的内容一样。另外，可以方便的使用 `+` 运算符或 `format!` 宏来拼接 `String` 值。

1. 使用 push_str 方法附加字符串 slice

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

2. 将字符串 slice 的内容附加到 string 后使用

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println！("s2 is {s2}");
```

3. push方法

```rust
let mut s = String::from("lo");
s.push("l");
```

4. 使用 `+` 运算符或者 `format！`宏拼接字符串

```rust
let s1 = String::from("hello");
let s2 = String::from("world");
let s3 = s1 + &s2; // 注意s1被移动了
```

这样过后 s1 的所有权被转移了 `+` 运算符使用了 `add` 函数，这个函数签名看起来像

```rust
fn add(self, s: &str) -> String {}
```



当要拼接的字符串非常多的时候，使用加号显得非常臃肿

```rust
let s1 = String::from("da");
let s2 = String::from("pa");
let s3 = String::from("ka");

let s = s1 + "-" + s2 + "-" + "s3";
```

可以使用 `format!` 宏, 宏 `format!` 生成的代码使用引用所以不会获取任何参数的所有权。

```rust
let s1 = String::from("da");
let s2 = String::from("pa");
let s3 = String::from("ka");

let s = format!("{s1}-{s2}-{s3}");
```

## 索引 string

Rust 的字符串不支持索引

### 内部表现

`String` 是一个 `Vec<u8>` 的封装

```rust
let hello = String::from("Здравствуйте");
```

这里这个字符长度是24，这是使用 `utf-8`编码需要的字节数

```rust
// u8 存储
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164, 224, 165, 135]
// unicode标量存储
['न', 'म', 'स', '्', 'त', 'े']
// 字形簇
["न", "म", "स्", "ते"]
```

Rust 提供了多种不同的方式来解释计算机储存的原始字符串数据，这样程序就可以选择它需要的表现方式，而无所谓是何种人类语言。

Rust 不允许使用索引获取 `String` 字符的原因是，索引操作预期总是需要常数时间（O(1)）。但是对于 `String` 不可能保证这样的性能，因为 Rust 必须从开头到索引位置遍历来确定有多少有效的字符。

### 字符串 slice 

```rust
let hello = "Здравствуйте";
let s = &hello[0..4];
```

会包含前四个字节，`s` 将会是 “Зд”

如果获取 `&hello[0..1] `Rust 在运行时会 panic，就跟访问 vector 中的无效索引时一样

## 遍历 string

操作字符串每一部分的最好的方法是明确表示需要字符还是字节。

获取单独的 Unicode 标量值使用 `chars` 方法。

```rust
forn c in "Зд".chars() {
	println!("{c}")
}
// З
// д
```

获取每一个原始字节使用 `bytes` 方法

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
// 208
// 151
// 208
// 180
```

有效的 Unicode 标量值可能会由不止一个字节组成。



# Hash Map

## 理解

`HashMap<K, V>` 类型储存了一个键类型 `K` 对应一个值类型 `V` 的映射。它通过一个 **哈希函数**（*hashing function*）来实现映射，决定如何将键和值放入内存中。

哈希 map 可以用于需要任何类型作为键来寻找数据的情况

## 创建 HashMap

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("blue"), 10);
scores.insert(String::from("yellow"), 50);
```

首先 `use` 标准库中集合部分的 `HashMap`。在这三个常用集合中，`HashMap` 是最不常用的，所以并没有被 prelude 自动引用。标准库中对 `HashMap` 的支持也相对较少, 并没有内建的构建宏。

像 vector 一样，哈希 map 将它们的数据储存在堆上，这个 `HashMap` 的键类型是 `String` 而值类型是 `i32`。类似于 vector，哈希 map 是同质的：所有的键必须是相同类型，值也必须都是相同类型。

## 访问 HashMap 中的值

```rust
use std::collections::HashMap;
let mut scores = HashMap::new();
scores.insert(String::from("yellow"), 20);
scores.insert(String::from("blue"), 30);
let team_name = String::from("green");
let score = scores.get(&team_name).copied().unwrap_or(0);
```

`get` 方法返回 `Option<&V>`如果某个键在哈希 map 中没有对应的值，`get` 会返回 `None`

`copied` 方法来获取一个 `Option<i32>` 而不是 `Option<&i32>`

`unwrap_or` 在 `scores` 中没有该键所对应的项时将其设置为零

## 遍历 HashMap 中的键值对

可以使用与 vector 类似的方式来遍历哈希 map 中的每一个键值对,**打印顺序是任意的，不是固定的**

```rust
use std::collections::HashMap;
fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("yellow"), 10);
    scores.insert(String::from("blue"), 20);
    for (k, v) in scores {
        println!("{k} -- {v}")
    }
}
```



## HashMap和所有权

对于像 `i32` 这样的实现了 `Copy` trait 的类型，其值可以拷贝进哈希 map。对于像 `String` 这样拥有所有权的值，其值将被移动而哈希 map 会成为这些值的所有者，

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");
let mut map = HashMap::new();
map.insert(field_name, field_value);
// 这里 field_name 和 field_value 不再有效
```

## 更新 HashMap

### 覆盖一个值

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{scores:?}");
// 键 Blue 对应的值被覆盖了 最终是25
```

### 在键没有对应值时插入键值对

需要检查特定的键是否已经存在于HashMap中

为此HashMap中有一个特有的 API-- `entry` ,`entry` 函数的返回值是一个枚举，`Entry`，它代表了可能存在也可能不存在的值。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{scores:?}");
```



### 根据旧值更新一个值

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{map:?}");
```

`split_whitespace` 方法返回一个由空格分隔 `text` 值子 slice 的迭代器。

`or_insert` 方法返回这个键的值的一个可变引用（`&mut V`），这个可变引用储存在 `count` 变量中，所以为了赋值必须首先使用星号（`*`）解引用 `count`。

这个可变引用在 `for` 循环的结尾离开作用域，这样所有这些改变都是安全的并符合借用规则。



### 哈希函数

`HashMap` 默认使用一种叫做 SipHash 的哈希函数，它可以抵御涉及哈希表（hash table）[1](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#siphash) 的拒绝服务（Denial of Service, DoS）攻击。然而这并不是可用的最快的算法，不过为了更高的安全性值得付出一些性能的代价。如果性能监测显示此哈希函数非常慢，以致于你无法接受，你可以指定一个不同的 *hasher* 来切换为其它函数。hasher 是一个实现了 `BuildHasher` trait 的类型。