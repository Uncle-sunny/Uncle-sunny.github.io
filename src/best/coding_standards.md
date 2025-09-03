# 代码规范
## 命名规范
关联常量：常量名全部大写(过长时采用蛇形命名法)

临时变量名、全局变量名： 蛇形命名法（snake_case）

函数： 蛇形命名法（snake_case）

```rust
const MY_CONSTANT: u32 = 42;
fn func_name() {}
let mut my_variable = 42;
```

结构体：大驼峰命名法（UpperCamelCase）

枚举：大驼峰命名法（UpperCamelCase）

类型：大驼峰命名法（UpperCamelCase）

trait: 大驼峰命名法（UpperCamelCase）

```rust
struct FirstName {
    name: String
}

enum Gender {
    Male,
    Female,
}

type UserId = u32;

trait User {
    fn new(name: String, age: u8) -> Self;
}
```
