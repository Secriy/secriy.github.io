---
title: Rust Basic Tutorial
date: 2021-09-18 14:25:28
categories: 学习笔记
tags:
  - Rust
  - Programming
---

## 基础概念

### 变量和可变性

#### 变量

Rust 中变量有可变变量（mutable variable）和不可变变量（immutable variable）的区分，看如下的代码：

```rust
fn main() {
    let x = 5;
    println!("{}", x);
    x = 6;
    println!("{}", x);
}
```

使用 `cargo run`运行：

```shell
cargo run
   Compiling hello-rust v0.1.0 (H:\Coding\Rust\hello-rust)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src\main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("{}", x); // 5
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

error: aborting due to previous error

For more information about this error, try `rustc --explain E0384`.
error: could not compile `hello-rust`

To learn more, run the command again with --verbose.
```

上面的错误表明，变量`x`无法二次赋值，因为它是一个 _immutable variable_。另外报错还说明，Rust 在编译期间就已经对程序可能出现的某些问题作了判断，如果不通过检查则编译失败。

```rust
fn main() {
    let mut x = 5;
    println!("{}", x);	// 5
    x = 6;
    println!("{}", x);	// 6
}
```

通过`let mut x = 5`的方式将`x`初始化为可变变量。

#### 常量

常量的声明使用如下形式的语句：

```rust
// const [NAME]: [DATA_TYPE] = [DATA];
const PI: f32 = 3.14159;
```

#### 差别

常量和变量有什么差别？

1.  常量无法使用`mut`修饰，它始终是不可变的；
2.  常量使用`const`声明，变量使用`let`；
3.  常量必须始终注明其数据类型；
4.  常量可以再任何范围声明，如全局常量、局部常量；
5.  常量是通过常量表达式确定值的，在编译器其值就会被确定，而变量的值可以在运行时赋予。

#### Shadowing

Rust 支持使用`let`关键字进行同名变量的重复初始化：

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);	// The value of x is: 12
}
```

甚至可以赋值给一个不同类型的同名变量：

```rust
fn main() {
    let string = "abc";

    let string = string.len();

    println!("The length of string is: {}", string)	// The length of string is: 3
}
```

第一个`string`和第二个`string`并非同一个变量，它们只是同名，不同于`mut`变量，其两个同名变量之间是没有关联的。按 Rust 官方的说法，第一个变量被第二个变量遮蔽（_shadowed_）了，因此这个概念在 Rust 中叫做**Shadowing**。

如果对`mut`变量赋新值：

```rust
fn main() {
    let mut string = "abc";

	string = string.len();

    println!("The length of string is: {}", string)
}
```

上面的代码在编译时就无法通过，因为其类型不匹配。

看下面一段代码，对`mut`变量进行了重新赋值：

```rust
fn main() {
    let mut string = "abc";

    string = "xyz";

    println!("The string is: {}", string)	// The string is: xyz
}
```

这段代码能够通过编译，但是编译期间会输出`warning`，即警告信息。这是因为`"abc"`字符串并没有使用就被`"xyz"`覆盖了，所以出现警告，但这并不影响程序的执行。

### 数据类型

数据类型是静态类型相当重要的部分，Rust 作为一种静态类型语言，其每一个值都有指定的类型，所有变量的类型在编译期间就已经确定了。

Rust 中数据类型分为两大类：标量类型（scalar）以及复合类型（compound）。

#### 标量类型

标量类型是简单类型，代表了单个值。Rust 中有四种标量类型：

- 整数
- 浮点数
- 布尔值
- 字符

##### 整数

Rust 中的整数类型如下表：

| 长度（Length） | 有符号（Signed） | 无符号（Unsigned） |
| :------------: | :--------------: | :----------------: |
|     8-bit      |        i8        |         u8         |
|     16-bit     |       i16        |        u16         |
|     32-bit     |       i32        |        u32         |
|     64-bit     |       i64        |        u64         |
|    128-bit     |       i128       |        u128        |
|      arch      |      isize       |       usize        |

`arch` 即 architecture，表示 CPU 架构的位数，如 64-bit 机器上就是 64 bits，32-bit 机器上就是 32 bits，这在其他编程语言里也很常见，如 C 中的`int`类型。

整数类型的字面量表示如下表：

| 字面量（Number literals） |    示例     |
| :-----------------------: | :---------: |
|      十六进制（Hex）      |    0xff     |
|     十进制（Decimal）     |   98_222    |
|      八进制（Octal）      |    0o77     |
|     二进制（Binary）      | 0b1111_0000 |
| 字节（Byte，仅支持`u8`）  |    b'A'     |

Rust 默认使用`i32`初始化整数。

##### 浮点数

Rust 中有`f32`和`f64`两种浮点数类型，分别占用 32 bits 和 64 bits 的大小。Rust 使用`f64`作为浮点数的默认类型，这是因为在现代 CPU 中，64 位浮点数的运算速度和 32 位基本相同，但 64 位浮点数精度更高。

示例：

```rust
let x = 1.0;		// f64
let y: f32 = 3.0;	// f32
```

按照 IEEE-754 标准，`f32`为单精度浮点数，`f64`为双精度浮点数。

##### 数值运算

示例：

```rust
fn main() {
    // 加法 addition
    let sum = 5 + 10;

    // 减法 subtraction
    let difference = 95.5 - 4.3;

    // 乘法 multiplication
    let product = 4 * 30;

    // 除法 division
    let quotient = 56.7 / 32.2;

    // 取余 remainder
    let remainder = 43 % 5;
}
```

##### 布尔值

布尔值的大小为 1 byte，有`true`和`false`这两个可能的值。

示例：

```rust
let t = true;
let f: bool = false; // 显式指定类型
```

##### 字符类型

Rust 中字符类型（`char`）使用单引号字面量，用于存储单个字符。Rust 的`char`类型在编程语言中比较特殊，它的大小为 4 bytes，表示一个 Unicode 标量值。

示例：

```rust
let c = 'z';
let z = 'ℤ';
let 汉 = '好';
let heart_eyed_cat = '😻';
```

可以注意到，Rust 是支持非英文变量名的。

#### 复合类型
