---
title: Learning Rust
date: 2021-09-18 14:25:28
categories: 学习笔记
mathjax: true
tags:
  - Rust
  - Programming
references:
  - title: The Rust Programming Language
    url: https://doc.rust-lang.org/book/
---

{% noteblock quote cyan %}

本文根据[The Rust Programming Language](https://doc.rust-lang.org/book/)翻译总结而成，大多内容纯粹是对该书的翻译，但按照我个人的学习路线以及练习增删了一些内容，同时也参考了其他文档对需要深入讨论的部分进行了介绍。本文对于有其他编程语言经验的同学来说比较容易接受，容易引起歧义的英文翻译都会标注原文。由于本文完全直接参考官方教程等英文文献，可以确保内容不存在转经多人之手而出现的偏差。但受限于个人知识水平，难免出现疏漏错误，烦请告知。

{% endnoteblock %}

<!-- more -->

## 基础概念

### 变量和可变性

#### 变量

Rust 中变量有可变变量（ _mutable variable_ ）和不可变变量（ _immutable variable_ ）的区分，看如下的代码：

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

上面的错误表明，变量`x`无法二次赋值，因为它是一个 _immutable variable_ 。另外报错还说明，Rust 在编译期间就已经对程序可能出现的某些问题作了判断，如果不通过检查则编译失败。

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

第一个`string`和第二个`string`并非同一个变量，它们只是同名，不同于`mut`变量，其两个同名变量之间是没有关联的。按 Rust 官方的说法，第一个变量被第二个变量遮蔽（ _shadowed_ ）了，因此这个概念在 Rust 中叫做**Shadowing**。

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

复合类型是由多个数据类型组合成的一种数据类型，Rust 有两种原始的复合类型：元组（Tuple）和数组（Array）。

##### Tuple

Tuple 的大小固定，一旦被声明，其大小就不可修改。

创建 Tuple：

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

上面的代码创建了一个由`i32`，`f64`和`u8`三个类型组成的 Tuple `tup`，并初始化其值。

从 Tuple 取值：

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of x is: {}", x); // The value of x is: 500
    println!("The value of y is: {}", y); // The value of y is: 6.4
    println!("The value of z is: {}", z); // The value of z is: 1
}
```

类似`let (x, y, z) = tup`将一个 tuple 赋值给多个变量的用法被称为解构（ _destructuring_ ），

还可以直接根据下标取值：

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);

    println!("The value of x is: {}", tup.0); // The value of x is: 500
    println!("The value of y is: {}", tup.1); // The value of y is: 6.4
    println!("The value of z is: {}", tup.2); // The value of z is: 1
}
```

##### Array

Array 和 Tuple 的差别很大，前者只能包含相同数据类型的元素，后者则不然。不过 Rust 中的 Array 和 Tuple 一样是固定长度的，不可改变。

Array 的初始化：

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];	// 初始化
    let b: [i32; 5] = [1, 2, 3, 4, 5];	// [type; length]
}
```

Array 特殊在于其一般分配在内存空间中的栈（stack）上而不是堆（heap）上，这在后文会深入讨论。当需要固定长度的一系列元素时，用 Array 会比较合适。Rust 中还有一种可变的数组，即**vector**，但它是由标准库提供的而不是 Rust 语言本身，一般来说 vector 会更常用一些。

Array 的声明和取值：

和其他大多数编程语言相同，Rust 中取数组元素使用下标，如`a[0]`。

```rust
fn main() {
	let a: [isize; 5];	// 声明长度为 5 的空数组
    println!("{}", a[0]);
}
```

上面的代码会直接编译不通过，报`use of possibly-uninitialized`错误，这是因为 Rust 禁止使用未初始化的变量，改成下面的代码就可以了：

```rust
fn main() {
    let a: [isize; 5];
    a = [3; 5];	// 将一个长度为 5，内容全部为 3 的数组赋给变量 a
    println!("{}", a[0]); // 3
}
```

`a = [3; 5]`等同于`a = [3, 3, 3, 3, 3]`。

我们注意到一件事情：Array 变量默认是 _mutable_ 的，不需要使用`mut`修饰就可以对其重新赋值。

如果我们访问数组时越界会怎么样？在某些编程语言中越界并不会造成程序本身停止或异常，但会给内存上的数据造成破坏。在 Rust 中，编译器会自动检查数组访问的下标是否越界，如果越界则无法编译通过。当下标的值在程序运行时才会确定（如接收用户输入的下标值）时，越界会导致程序 **panic**，并随之中止。

### 函数

在大多数编程语言中，函数都是相当重要的存在，Rust 也不例外，函数的使用无处不在，比如前文中大量使用的`main`函数。

#### 命名规范

Rust 使用`fn`关键字声明函数，并使用 snake case 命名风格，所有的函数名使用小写字母并使用`_`分隔。Rust 并不在乎函数的定义位置，无论其在代码中是位于`main`函数之前还是之后。

```rust
fn sample_func() {
    println!("Sample function.");
}

fn hello_world() {
    println!("Hello world.");
}
```

调用函数也很简单，例如`sample_func()`：

```rust
fn main() {
    hello_world();	// Hello world.
}

fn hello_world() {
    println!("Hello world.");
}
```

#### 函数参数

函数参数用法：

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);	// The value of x is: 5
}
```

多个参数：

```rust
fn main() {
    another_function(5, 6);
}

fn another_function(x: i32, y: i32) {
    println!("The value of x is: {}", x);	// The value of x is: 5
    println!("The value of y is: {}", y);	// The value of y is: 6
}
```

函数的每个参数都必须指明其类型。

#### 函数返回值

函数可以向调用它们的代码返回值。我们不命名返回值，但在箭头（->）后面声明它们的类型。在 Rust 中，函数的返回值与函数体块中最终表达式的值同义。通过使用 return 关键字并指定值，可以提前从函数返回，但大多数函数隐式返回最后一个表达式。下面是一个返回值的函数示例：

Rust 中函数返回值是无名的，只需要指定其类型：

```rust
fn five() -> i32 {
    5
}

fn plus_one(x: i32) -> i32 {
    x + 1
}

fn main() {
    let mut x = five();
    x = plus_one(x);

    println!("The value of x is: {}", x);	// The value of x is: 6
}
```

注意，作为函数返回值的语句，其后是没有`;`的，如上面的`5`和`x + 1`，加上分号就变成了普通语句，而不是返回值。

`plus_one`函数还可以改成如下的形式：

```rust
fn plus_one(mut x: i32) -> i32 {
    x = x + 1;
    x
}
```

这和原来的`plus_one`函数是等价的，只不过使用了`mut`修饰符让`x`可变。

### 注释

注释很简单，例如单行注释：

```rust
// This is a comment.
```

多行注释：

```rust
// Multiple lines
// comments.
```

```rust
fn main() {
    println!("Hello World!"); // Hello World!
}
```

Rust 中还有一种文档注释，但暂时用不到，在之后会介绍。

### 控制流

控制流是很基本的概念，Rust 同样包含了多种基本的控制流。

#### 条件

Rust 中`if`语句不需要使用括号：

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("number < 5");	// number < 5
    } else {
        println!("number >= 5");
    }
}
```

在 Rust 中，`if`表达式只能用布尔值作为条件，即只能判断布尔变量以及条件表达式。

还可以使用`else if`表达式来增加分支，只有一个分支会被执行，一旦某个分支为`true`，就不会判断下一个分支：

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");	// number is divisible by 3
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

使用`if`语句，可以实现下面的条件表达式，类似于某些语言中的三元运算符：

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {}", number);	// 5
}
```

其中，根据前面有关函数返回值的学习，我们知道`{}`中没有分号结尾的作为返回值，由此我们可以写出下面的语句，同样是正确的：

```rust
fn main() {
    let condition = true;
    let number = if condition {let x = 2; x+2} else {let x = 1; x+1};

    println!("The value of number is: {}", number);	// 4
}
```

然而`if`和`else`语句块的值不能为不同类型，这是无法编译通过的，这是因为 Rust 需要在编译期确定所有变量的类型，如果`if`和`else`能够得到不同类型的结果，那么也就无法知道变量`number`的类型。不过，这并不代表这样的实现是不可能的，如果编译器必须跟踪任何变量的多个假设类型，那么编译器将更加复杂，对代码的保证也会更少。

#### 循环

Rust 有三种循环语句，`loop`、`while`和`for`。

##### loop

简而言之，`loop`语句就是无限循环，它没有条件语句，也就不能在循环上让其退出。但是`loop`可以通过循环体内的语句退出：

```rust
fn main() {
    let mut count = 0;	// 统计循环次数
    loop {
        if count == 5 {break;}	// 当 count 等于 5 则退出循环
        println!("hello");
        count += 1;	// 次数加一
    }
}
```

毋庸置疑，上面的代码会输出五行`hello`。

关于`loop`语句还有一点比较特别的是，它可以返回值：

```rust
fn main() {
    let mut count = 0;
    let result = loop {
        if count == 5 {break count + 1;}
        println!("hello");
        count += 1;
    };
    println!("The value of result is: {}", result);	// The value of result is: 6
}
```

可以看到，`break`关键字后面跟返回值，就能赋值给`result`，当执行到`break`语句后`loop`立即返回了`count + 1`。

结合前面的内容，可以知道下面的代码也是正确的：

```rust
fn main() {
    let mut count = 0;
    let result = loop {
        if count == 5 {break {let x = 12; count + x};}
        println!("hello");
        count += 1;
    };
    println!("The value of result is: {}", result);	// The value of result is: 17
}
```

`{let x = 12; count + x}`的值为`count + x`即 17，通过`break`返回给`result`。

##### While

循环语句中写循环条件可以让循环处理更简单，Rust 提供了`while`循环支持指定循环条件：

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number -= 1;
    }

    println!("LIFTOFF!!!");	// LIFTOFF!!!
}
```

这和其他编程语言没什么大的区别，不再赘述。

##### for

`for`循环用于迭代元素集合，如数组（Array），这使用其他两种循环也可以做到：

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index += 1;
    }
}

// 10
// 20
// 30
// 40
// 50
```

但使用`for`循环可以将上面的语句简化为下面的：

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}

// 10
// 20
// 30
// 40
// 50
```

这样的代码会更安全，因为不需要指定数组下标之类，不存在越界的问题。

下面的语句可以用于生成一个数字范围，类似 Python 中的写法：

```rust
fn main() {
	for number in 1..4 {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}

// 1!
// 2!
// 3!
// LIFTOFF!!!
```

`1..4`很明显，是用来生成范围$[1,4)$的三个数字，接着使用`for`循环遍历这三个数字并输出。

我们可以写出如下的代码来指定下标从数组中取值：

```rust
fn main() {
	let arr = [10, 20, 30, 40, 50];
    for number in 3..5 {
        println!("{}", arr[number]);
    }
}

// 40
// 50
```

##### 练习

用目前学习的内容写一个计算斐波那契数列的函数：

```rust
fn fib(mut n: i32) {
    let mut pre = 1;
    let mut post = 1;

    while n > 2 {
        let tmp = post;
        post += pre;
        pre = tmp;
        n-=1;
    }

    println!("{}", post);
}

fn main() {
    fib(1);		// 1
    fib(2);		// 1
    fib(3);		// 2
    fib(10);	// 55
}
```

## 所有权

所有权（Ownership）是 Rust 最独特的特性，它使 Rust 能够在不需要垃圾收集器的情况下保证内存安全。因此，了解所有权在 Rust 中的作用非常重要。在本章中，我们将讨论所有权以及几个相关特性：借用（borrowing）、切片（slices）以及 Rust 如何在内存中布局数据。
