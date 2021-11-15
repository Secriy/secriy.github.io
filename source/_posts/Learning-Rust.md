---
title: "Learning Rust"
date: 2021-09-18 14:25:28
urlname: learning-rust
categories: 学习笔记
tags:
  - Rust
  - PL
references:
  - title: The Rust Programming Language
    url: https://doc.rust-lang.org/book/
mathjax: true
---

{% noteblock quote cyan %}

本文主要根据 [The Rust Programming Language](https://doc.rust-lang.org/book/) 翻译总结而成，大多内容纯粹是对该书的翻译，但按照我个人的学习路线以及练习增删了一些内容，同时也参考了其他文档对需要深入讨论或是难以理解的部分进行了详细介绍，**因此对于本文内容请不要认为其等同于原文**。本文对于有其他编程语言经验的同学来说比较容易接受，容易引起歧义的英文翻译都会标注原文。由于本文完全直接参考官方教程等英文文献，可以确保内容不存在由于转经多人之手而出现的偏差。但受限于个人知识水平，难免出现疏漏错误，烦请告知。

{% endnoteblock %}

<!-- more -->

## 基础概念

> 本章没有什么难度，主要是对 Rust 基础概念和用法的学习和适应，无需强行记忆，需要时参考即可，很快能够熟练掌握。

### 变量和可变性

#### 变量

Rust 中变量有可变变量（*mutable variable*）和不可变变量（*immutable variable*）的区分，看如下的代码：

```rust
fn main() {
    let x = 5;
    println!("{}", x);
    x = 6;
    println!("{}", x);
}
```

使用 `cargo run` 运行：

```shell
$ cargo run
   Compiling hello-rust v0.1.0
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

上面的错误表明，变量 `x` 无法二次赋值，因为它是一个 *immutable variable*。另外报错还说明，Rust 在编译期间就已经对程序可能出现的某些问题作了判断，如果不通过检查则编译失败。

```rust
fn main() {
    let mut x = 5;
    println!("{}", x); // 5
    x = 6;
    println!("{}", x); // 6
}
```

通过 `let mut x = 5` 的方式将 `x` 初始化为可变变量。

#### 常量

常量的声明使用如下形式的语句：

```rust
// const [NAME]: [DATA_TYPE] = [DATA];
const PI: f32 = 3.14159;
```

#### 差别

常量和变量有什么差别？

1. 常量无法使用 `mut` 修饰，它始终是不可变的；
2. 常量使用 `const` 声明，变量使用 `let`；
3. 常量必须始终注明其数据类型；
4. 常量可以在任何范围声明，如全局常量、局部常量；
5. 常量是通过常量表达式确定值的，在编译期其值就会被确定，而变量的值可以在运行时赋予。

#### Shadowing

Rust 支持使用 `let` 关键字进行同名变量的重复初始化：

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x); // The value of x is: 12
}
```

甚至可以赋值给一个不同类型的同名变量：

```rust
fn main() {
    let string = "abc";

    let string = string.len();

    println!("The length of string is: {}", string) // The length of string is: 3
}
```

第一个 `string` 和第二个 `string` 并非同一个变量，它们只是同名，不同于 `mut` 变量，其两个同名变量之间是没有关联的。按 Rust 官方的说法，第一个变量被第二个变量遮蔽（*shadowed*）了，因此这个概念在 Rust 中叫做 *Shadowing*。

如果对 `mut` 变量赋新值：

```rust
fn main() {
    let mut string = "abc";

    string = string.len();

    println!("The length of string is: {}", string)
}
```

上面的代码在编译时就无法通过，因为其类型不匹配。

看下面一段代码，对 `mut` 变量进行了重新赋值：

```rust
fn main() {
    let mut string = "abc";

    string = "xyz";

    println!("The string is: {}", string) // The string is: xyz
}
```

这段代码能够通过编译，但是编译期间会输出 `warning`，即警告信息。这是因为 `"abc"` 字符串并没有使用就被 `"xyz"` 覆盖了，所以出现警告，但这并不影响程序的运行。

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
| 字节（Byte，仅支持 `u8`） |    b'A'     |

Rust 默认使用 `i32` 初始化整数。

##### 浮点数

Rust 中有 `f32` 和 `f64` 两种浮点数类型，分别占用 32 bits 和 64 bits 的大小。Rust 使用 `f64` 作为浮点数的默认类型，这是因为在现代 CPU 中，64 位浮点数的运算速度和 32 位基本相同，但 64 位浮点数精度更高。

示例：

```rust
let x = 1.0;  // f64
let y: f32 = 3.0; // f32
```

按照 IEEE-754 标准，`f32` 为单精度浮点数，`f64` 为双精度浮点数。

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

布尔值的大小为 1 byte，有 `true` 和 `false` 这两个可能的值。

示例：

```rust
let t = true;
let f: bool = false; // 显式指定类型
```

##### 字符类型

Rust 中字符类型（`char`）使用单引号字面量，用于存储单个字符。Rust 的 `char` 类型在编程语言中比较特殊，它的大小为 4 bytes，表示一个 Unicode 标量值。

示例：

```rust
let c = 'z';
let z = 'ℤ';
let 汉 = '好';
let heart_eyed_cat = '😻';
```

可以注意到，Rust 是支持非英文变量名的。

#### 复合类型

复合类型是由多个数据类型组合成的一种数据类型，Rust 有两种原始的复合类型：Tuple（元组）和 Array（数组）。

##### Tuple

Tuple 的大小固定，一旦被声明，其大小就不可修改。

创建 Tuple：

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

上面的代码创建了一个由 `i32`，`f64` 和 `u8` 三个类型组成的 Tuple `tup`，并初始化其值。

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

类似 `let (x, y, z) = tup` 将一个 tuple 赋值给多个变量的用法被称为解构（*destructuring*），

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
    let a = [1, 2, 3, 4, 5]; // 初始化
    let b: [i32; 5] = [1, 2, 3, 4, 5]; // [type; length]
}
```

Array 特殊在于其一般分配在内存空间中的栈（stack）上而不是堆（heap）上，这在后文会深入讨论。当需要固定长度的一系列元素时，用 Array 会比较合适。Rust 中还有一种可变的数组，即 **vector**，但它是由标准库提供的而不是 Rust 语言本身，一般来说 vector 会更常用一些。

Array 的声明和取值：

和其他大多数编程语言相同，Rust 中取数组元素使用下标，如 `a[0]`。

```rust
fn main() {
 let a: [isize; 5]; // 声明长度为 5 的空数组
    println!("{}", a[0]);
}
```

上面的代码会直接编译不通过，报 ``use of possibly-uninitialized `a` `` 错误，这是因为 Rust 禁止使用未初始化的变量，改成下面的代码就可以了：

```rust
fn main() {
    let a: [isize; 5];
    a = [3; 5]; // 将一个长度为 5，内容全部为 3 的数组赋给变量 a
    println!("{}", a[0]); // 3
}
```

`a = [3; 5]` 等同于 `a = [3, 3, 3, 3, 3]`。

我们注意到一件事情：Array 变量默认是 *mutable* 的，不需要使用 `mut` 修饰就可以对其重新赋值。

如果我们访问数组时越界会怎么样？在某些编程语言中越界并不会造成程序本身停止或异常，但会给内存上的数据造成破坏。在 Rust 中，编译器会自动检查数组访问的下标是否越界，如果越界则无法编译通过。当下标的值在程序运行时才会确定（如接收用户输入的下标值）时，越界会导致程序 **panic**，并随之中止。

### 函数

在大多数编程语言中，函数都是相当重要的存在，Rust 也不例外，函数的使用无处不在，比如前文中大量使用的 `main` 函数。

#### 命名规范

Rust 使用 `fn` 关键字声明函数，并使用 snake case 命名风格，所有的函数名使用小写字母并使用 `_` 分隔。Rust 并不在乎函数的定义位置，无论其在代码中是位于 `main` 函数之前还是之后。

```rust
fn sample_func() {
    println!("Sample function.");
}

fn hello_world() {
    println!("Hello world.");
}
```

调用函数也很简单，例如 `sample_func()`：

```rust
fn main() {
    hello_world(); // Hello world.
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
    println!("The value of x is: {}", x); // The value of x is: 5
}
```

多个参数：

```rust
fn main() {
    another_function(5, 6);
}

fn another_function(x: i32, y: i32) {
    println!("The value of x is: {}", x); // The value of x is: 5
    println!("The value of y is: {}", y); // The value of y is: 6
}
```

函数的每个参数都必须指明其类型。

#### 函数返回值

函数可以向调用它们的代码返回值。我们不命名返回值，但在箭头（`->`）后面声明它们的类型。在 Rust 中，函数的返回值与函数体块中最终表达式的值同义。通过使用 return 关键字并指定值，可以提前从函数返回，但大多数函数隐式返回最后一个表达式。下面是一个返回值的函数示例：

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

    println!("The value of x is: {}", x); // The value of x is: 6
}
```

注意，作为函数返回值的语句，其后是没有 `;` 的，如上面的 `5` 和 `x + 1`，加上分号就变成了普通语句，而不是返回值。

`plus_one` 函数还可以改成如下的形式：

```rust
fn plus_one(mut x: i32) -> i32 {
    x = x + 1;
    x
}
```

这和原来的 `plus_one` 函数是等价的，只不过使用了 `mut` 修饰符让 `x` 可变。

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

Rust 中 `if` 语句不需要使用括号：

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("number < 5"); // number < 5
    } else {
        println!("number >= 5");
    }
}
```

在 Rust 中，`if` 表达式只能用布尔值作为条件，即只能判断布尔变量以及条件表达式。

还可以使用 `else if` 表达式来增加分支，只有一个分支会被执行，一旦某个分支为 `true`，就不会判断下一个分支：

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3"); // number is divisible by 3
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

使用 `if` 语句，可以实现下面的条件表达式，类似于某些语言中的三元运算符：

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {}", number); // 5
}
```

其中，根据前面有关函数返回值的学习，我们知道 `{}` 中没有分号结尾的作为返回值，由此我们可以写出下面的语句，同样是正确的：

```rust
fn main() {
    let condition = true;
    let number = if condition {let x = 2; x+2} else {let x = 1; x+1};

    println!("The value of number is: {}", number); // 4
}
```

然而 `if` 和 `else` 语句块的值不能为不同类型，这是无法编译通过的，这是因为 Rust 需要在编译期确定所有变量的类型，如果 `if` 和 `else` 能够得到不同类型的结果，那么也就无法知道变量 `number` 的类型。不过，这并不代表这样的实现是不可能的，如果编译器必须跟踪任何变量的多个假设类型，那么编译器将更加复杂，对代码的保证也会更少。

#### 循环

Rust 有三种循环语句，`loop`、`while` 和 `for`。

##### loop

简而言之，`loop` 语句就是无限循环，它没有条件语句，也就不能在循环上让其退出。但是 `loop` 可以通过循环体内的语句退出：

```rust
fn main() {
    let mut count = 0; // 统计循环次数
    loop {
        if count == 5 {break;} // 当 count 等于 5 则退出循环
        println!("hello");
        count += 1; // 次数加一
    }
}
```

毋庸置疑，上面的代码会输出五行 `hello`。

关于 `loop` 语句还有一点比较特别的是，它可以返回值：

```rust
fn main() {
    let mut count = 0;
    let result = loop {
        if count == 5 {break count + 1;}
        println!("hello");
        count += 1;
    };
    println!("The value of result is: {}", result); // The value of result is: 6
}
```

可以看到，`break` 关键字后面跟返回值，就能赋值给 `result`，当执行到 `break` 语句后 `loop` 立即返回了 `count + 1`。

结合前面的内容，可以知道下面的代码也是正确的：

```rust
fn main() {
    let mut count = 0;
    let result = loop {
        if count == 5 {break {let x = 12; count + x};}
        println!("hello");
        count += 1;
    };
    println!("The value of result is: {}", result); // The value of result is: 17
}
```

`{let x = 12; count + x}` 的值为 `count + x` 即 17，通过 `break` 返回给 `result`。

##### While

循环语句中写循环条件可以让循环处理更简单，Rust 提供了 `while` 循环支持指定循环条件：

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number -= 1;
    }

    println!("LIFTOFF!!!"); // LIFTOFF!!!
}
```

这和其他编程语言没什么大的区别，不再赘述。

##### for

`for` 循环用于迭代元素集合，如数组（Array），这使用其他两种循环也可以做到：

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

但使用 `for` 循环可以将上面的语句简化为下面的：

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

`1..4` 很明显，是用来生成范围 $[1,4)$ 的三个数字，接着使用 `for` 循环遍历这三个数字并输出。

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

### 练习

1. 用目前学习的内容写一个计算斐波那契数列的函数：

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
        fib(1);  // 1
        fib(2);  // 1
        fib(3);  // 2
        fib(10); // 55
    }
    ```

## 所有权

所有权（Ownership）是 Rust 最独特的特性，它使 Rust 能够在不需要垃圾收集器的情况下保证内存安全。因此，了解所有权在 Rust 中的作用非常重要。在本章中，我们将讨论所有权以及几个相关特性：借用（borrowing）、切片（slices）以及 Rust 如何在内存中布局数据。

> 注：Ownership 应当是 Rust 较复杂的一部分，但对于理解和掌握 Rust 编程相当重要，因此本文遵循 The Rust Programming Language 的章节安排，先介绍该部分。

### 什么是所有权

所有程序在运行的时候都必须管理它们使用计算机内存的方式。有些编程语言使用垃圾回收机制来不断地寻找不再使用的内存（如 Java、Go）；还有些编程语言中，开发者必须手动分配和释放内存（如 C、C++）。Rust 则另辟蹊径：内存通过一个所有权系统来管理，它具有一组编译器在编译时检查的规则。当程序运行时，所有权的任何功能都不会影响程序的效率。

Ownership 对大多数学习 Rust 的人来说是一个全新的概念，因此需要花一些时间来学习和适应。

> **栈（Stack）和堆（Heap）**
>
> 在许多编程语言中，开发者不必经常考虑栈和堆。但是在像 Rust 这样的系统级编程语言中，值（value）是在栈上还是在堆上对**语言的行为**以及**为什么必须做出某些决定**有很大的影响。下面作出简要说明。
>
> 栈和堆都是可供程序在运行时使用的内存的一部分，但它们的结构不同。类似于数据结构中栈的概念，内存中的栈空间同样满足 LIFO 的规则，不再赘述。栈上存储的所有数据的大小必须在编译期已知且固定，如果编译期无法确定一个数据的大小或者其大小可能发生改变，那它将被分配到堆上。堆的组织性较差，当数据放在堆上时，需要请求一定量的空间，然后由内存分配器在堆上找到一个足够大的空间，将该空间标记为正在使用，接着返回一个指针，即该地址的位置。这个过程被称为堆分配（*allocating on the heap*），有时简称为分配（*allocating*）。将值放到栈上不视为分配。由于指针是已知的固定大小，所以可以将指针存储在栈上，但当需要实际数据时，必须找寻指针的指向地址。
>
> 显然，在栈上分配内存更快，因为只需要从栈顶入栈就行了，而堆中由于空闲空间并不一定连续，需要花时间寻找满足的堆空间，并记录变化。
>
> 访问堆中的数据同样比访问栈上的数据慢，因为必须通过指针才能找到目标位置。现代处理器在内存中的跳跃越少，则速度越快。继续类推，考虑一家餐馆的服务员从许多表格中获取订单的例子。在去到下一张桌子之前，在一张桌子上拿到所有订单是最高效的。从表 A 获取订单，然后从表 B 获取订单，然后再从 A 获取订单，然后再从 B 获取订单将是一个慢得多的过程。同样，如果处理器处理的数据与其他数据相近（如栈上的数据），而不是距离较远（如堆上的数据），则处理器可以更好地完成其工作。在堆上分配大量空间也需要时间。
>
> 当代码调用函数时，传递到函数中的值（这里指的是函数参数，可能包括指向堆上数据的指针）和函数的局部变量被压入栈中。当函数结束（返回）时，这些值会从栈中弹出。
>
> 跟踪代码的哪些部分正在使用堆上的哪些数据，最小化堆上的重复数据量，清理堆上未使用的数据以避免耗尽空间，这些都是所有权所要解决的问题。一旦了解了所有权，就不需要经常考虑栈和堆，但是明白**管理堆数据**是所有权存在的原因有助于理解它为什么以这种方式工作。

#### Ownership 规则

首先牢记以下规则：

- Rust 中的每个值（value）都有一个称为其所有者（owner）的变量。
- 一次只能有一个所有者。
- 当所有者超出作用域范围（即程序不能访问它）时，该值将被删除。

#### 变量作用域

变量作用域（Variable Scope）指的是某个变量的作用范围，即它对其他代码可见的范围。

在下面的代码中，`x` 的作用域就是整个 `main` 函数，在 `main` 函数内 `x` 可以被任何语法访问。

```rust
fn main() {
    let x = 1;
    println!("{}", x); // 1
}
```

在 Rust 中，可以使用 `{}` 指定一块作用域：

```rust
fn main() {
    let x = 5;
    {
        let y = 6;
    }
    println!("{}", x);
    println!("{}", y);
}
```

上面的代码不能通过编译，会出现 `` cannot find value `y` in this scope `` 的报错，这是因为`y`在一个独立的作用域里，外部无法访问到它。

作用域能够访问到其外部的作用域，因此下面的代码中，`x` 和 `y` 都能被打印出来：

```rust
fn main() {
    let x = 5;
    {
        let y = 6;
        println!("{}", x); // 5
        println!("{}", y); // 6
    }
    // println!("{}", x);
    // println!("{}", y);
}
```

现在来分析一下作用域的生命周期：

```rust
fn main() {
    {                      // 变量 s 不可用, 因为它还没有被声明
        let s = "hello";   // 变量 s 现在可用

        // 处理 s 相关的代码
    }                      // 作用域结束，s 不再可用
}
```

也就是说 `s` 的生命周期是从其被声明直到遇到 `}` 其作用域结束的这个区间。

就目前来看，Rust 中变量作用域机制和其他很多编程语言中的变量作用域机制很近似。现在我们引入字符串类型再进行探讨。

#### 字符串类型

前面介绍的数据类型都存储在栈上，在它们所在的作用域结束（如函数返回）后就会被弹出栈空间。我们现在看看存储在堆上的数据，并探究 Rust 如何知道何时清理这些数据。

这里我们使用 `String`（字符串）类型作为示例，重点介绍 `String` 中与所有权相关的部分。这些部分也同样适用于其他复杂数据类型（无论它们是由标准库提供的还是由程序员自行创建的）。

我们早已经见过字符串字面量（string literals），如 `"hello world!"`，这是写在代码中的一段字符串。使用字符串字面量很直观也很方便，但可能并不适用于所有需要进行文本处理的情况。其中一个原因是字符串是不可变（*immutable*）的，还有一个原因是字符串内容很可能是在运行时才能够被确定的（比如是由用户输入的字符串）。出于这些情况，Rust 提供了不同于字符串字面量的 `String` 类型。这个类型是在堆上分配的（而字符串字面量在栈上），因此能够存储大量在编译时未知的文本。

`String` 的用法如下：

```rust
let s = String::from("hello"); // 使用字符串字面量创建一个 String
```

其中，`::` 是一个运算符，在这里用于调用 `String` 类型命名空间下的 `from` 函数。

现在可以使用可变的字符串了：

```rust
fn main() {
    let mut s = String::from("hello");

    s.push_str(", world!"); // push_str() 将一个字符串字面量放入一个 String 中

    println!("{}", s); // hello, world!
}
```

从字符串的例子中我们知道，字符串字面量不可变，但 `String` 可变，这是因为这两种类型处理内存的方式不同。

#### 内存和内存分配

由于字符串字面量是在编译时就确定的，因此它可以被直接硬编码到最终的可执行文件中。这也是字符串字面量快速并高效的原因。但其高效的前提是字符串字面量是不可变的。我们无法将**在编译时大小未知**或是**在程序运行时大小可能会改变**的文本放入二进制文件中。

对于 `String` 类型，为了支持可变的、可增长的文本，我们需要在堆上分配一定量的内存（编译时未知）来保存内容。这意味着：

- 必须在运行时（runtime）从内存分配器（memory allocator）请求内存。

- 我们需要一种在处理完 `String` 后将内存返回给分配器的方法（即回收内存）。

第一点由编程人员完成，通过调用如 `String::from` 的函数来实现请求所需的内存。这在编程语言中非常普遍。

然而，第二点就有一些不同了。在使用垃圾回收（*garbage collector, GC*）机制的语言中，GC 跟踪并清理不再使用的内存，编程人员无需考虑这些。如果没有 GC，编程人员就需要判断内存何时不再被使用，并调用代码显式返回（return）它，就像我们请求（request）它一样。然而完全正确地执行此操作历来是一个相当困难的编程问题。如果忘记了回收内存，轻则造成内存浪费，重则出现大量内存泄漏，导致内存占用越来越大；然而如果回收的过早，会导致某些仍需要使用的变量失效，出现悬挂指针（*dangling pointer*）问题；如果重复进行了回收，这同样会导致程序 bug。我们的任务是将 `allocate`（分配）和 `free`（回收）操作准确地配对。

Rust 采用了不同的路径：**一旦占用内存的变量超出作用域，内存就会自动返回。**

看下面一段代码，使用了 `String` 类型声明变量 `s`：

```rust
fn main() {
    {
        let s = String::from("hello"); // 变量 s 被初始化

        // 处理 s 相关的代码
    } // 作用域结束，s 不再可用

}
```

注意作用域结束的地方 `}`，当这个作用域结束，`s` 即不再可用，也就可以回收其内存了。Rust 在这时会调用一个特殊的函数 `drop`，`String` 的作者（author，这个作者指的是 `String` 标准库的开发者，并非使用者）可以在这里放置代码以返回内存。

> 注：在 C++ 中，在项目的生命周期结束时释放资源的这种模式有时称为 *Resource Acquisition Is Initialization*（*RAII*）。如果你使用过 RAII 模式，Rust 中的 drop 函数对你来说会很熟悉。

> 【截至写下本文，我都还不会 C++，我的下半生估计是废了 o(T ヘ To)】

上述模式对 Rust 代码的编写方式有着深远的影响。现在看起来可能很简单，但在更复杂的情况下，当我们想要让多个变量使用我们在堆上分配的数据时，代码的行为可能会出人意料。接下来让我们来探讨其中的一些情况。

#### 变量和数据交互的方式：移动

在 Rust 中，多个变量可以以不同的方式与相同的数据交互。比如下面这个例子：

```rust
let x = 5;
let y = x;
```

我们大概可以猜到这是在做什么：“将值 5 绑定到 `x`，然后复制 `x` 中的值并将其绑定到 `y`。”我们现在有两个变量，`x` 和 `y`，它们都等于 5。这确实是实际的执行过程，因为整数是具有已知固定大小的简单值，这两个 5 会被压入栈。

现在我们来看看 `String` 的版本：

```rust
let s1 = String::from("hello");
let s2 = s1;
```

这和固定大小的 `i32` 整数有很大的区别，它并不是从 `s1` 直接复制值并赋给 `s2` 的。我们先看看 `String` 背后的结构：

![String in memory](Learning-Rust/trpl04-01.svg)

> 注：非单独注明，图片均来自 [The Rust Programming Language](https://doc.rust-lang.org/book/title-page.html)。

这和 Go 对字符串类型的实现基本相同。

其中，左边的部分是位于栈上的，右边的部分在堆上。长度（len）表示当前字符串已经占用的空间大小（以字节为单位），容量（capacity）表示可供使用的空间大小，即底层数组的实际大小（以字节为单位）。当复制 `s1` 时，仅仅是复制了栈中的部分，因此是下图的结果：

![s1 and s2 pointing to the same value](Learning-Rust/trpl04-02.svg)

由此可以看出，`s1` 和 `s2` 实际的内容是相同的。假设 Rust 同时也会复制堆数据会怎样？如下图的情况：

![s1 and s2 to two places](Learning-Rust/trpl04-03.svg)

可以预见，当字符串占用大量空间时，对其进行复制会耗费相当的时间和空间。下面我们丢掉这个假设，看实际的情况。

前面我们提到了，因为所有权机制，变量超出作用域会立即释放堆内存空间，那么对于具有相同内存空间来说的 `s1` 和 `s2`，会发生什么？显然，会调用两次 `drop()` 来释放同一块内存，这被称为**双重释放**（*double free*）错误，是我们之前提到的内存安全错误之一。

为了确保内存安全，在 `s2 = s1` 这条语句后，Rust 就会认为 `s1` 不再有效了，因此无需再释放 `s1`，作用域结束时释放 `s2` 即可。这直接导致了下面的现象：

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    println!("{}", s1);
    println!("{}", s2);
}
```

```shell
$ cargo run
   Compiling ownership v0.1.0
error[E0382]: borrow of moved value: `s1`
  --> src\main.rs:15:20
   |
12 |     let s1 = String::from("hello");
   |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
13 | 
14 |     let s2 = s1;
   |              -- value moved here
15 |     println!("{}", s1);
   |                    ^^ value borrowed here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `ownership` due to previous error
```

可以看到，编译直接出错了，其原因是 `s1` 已经失效了。尝试修改为下面的代码，再次运行：

```rust
fn main() {
    let s1 = String::from("hello");
    println!("{}", s1);  // hello
    let s2 = s1;
    println!("{}", s2);  // hello
}
```

这一次就正常输出了意料之中的结果。

如果你在使用其他语言时听说过**浅拷贝**（*shallow copy*）和**深拷贝**（*deep copy*）这两个术语，那么复制指针、长度和容量而不复制堆中数据的概念可能听起来像进行浅拷贝。但是由于 Rust 会让第一个变量无效，因此这个概念没有被叫做浅拷贝，它被称为**移动**（*move*）。在上述的例子中，我们会说 `s1` 被移动到了 `s2` 中，即把 `s1` 放到 `s2` 中并删除 `s1`。

现在，我们可以说解决了双重释放的问题。从这个设计中我们可以看出，Rust 永远不会自动创建一个“深拷贝”，因此，任何自动化的复制行为都是性能很优良的。

#### 变量和数据交互的方式：克隆

如果想要深拷贝 `String` 的堆中数据，可以使用一种叫做 `clone` 的通用方法。

> 方法（method）是一种很常见的编程语言组成部分，在后文中才会对方法进行介绍。

下面是一个示例：

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();
    println!("s1 = {}, s2 = {}", s1, s2);
}
```

上面这段代码显式地进行了 `s1` 的深拷贝。通过 `clone` 方法进行操作，能够提醒开发者注意这样的操作是可能影响性能的，要三思而后行。

#### 仅存在栈上的数据：复制

需要注意的是，那些只存储在栈上的数据是没有深拷贝和浅拷贝的区分的，拷贝它们不会导致各种问题。比如下面的代码：

```rust
fn main() {
    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);
}
```

大小确定的类型并不需要 `clone` 这样的方法来进行复制，因为它们在栈上，而不是堆上。对这样大小确定的栈数据的复制是很高效的。

Rust 有一个特殊的注解（annotation）叫做 `Copy` trait，我们可以把它放在像整数（integer）这样存储在栈中的类型上。如果一个类型实现了 `Copy` trait，那么这个类型的一个旧的变量在赋给其他变量后仍然可用（就像上面的那段代码中的 `x`，赋值给 `y` 后仍然可用）。如果一个类型或者这个类型的任何部分实现了 `Drop` trait，Rust 就会不允许使用 `Copy` trait 注解该类型。

如果某个类型的变量离开作用域时需要进行某些特殊处理，我们对该变量使用 `Copy` 注解会发生编译错误。

> 关于 Trait，这是一种 Rust 中的概念，将在后文中进行介绍，暂时可以忽略。需要了解如何将 `Copy` 注解添加到类型中以实现 trait，请参阅 [Derivable  Traits](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html)。

那么什么类型实现了 `Copy` trait？可以查看给定类型的文档来确定，但作为一个通用的规则，任何一组简单的**标量值**都可以实现 `Copy`，并且任何不需要内存分配（allocation，指前文所提到的在堆中分配）和某些形式资源的类型都可以实现 `Copy`。以下是一些实现 `Copy` 的类型：

- 所有整数类型，例如 `u32`；
- 布尔类型 `bool`；
- 所有浮点类型，例如 `f64`；
- 字符类型，`char`；
- Tuple，如果只包含同样实现 `Copy` 的类型。例如，`(i32, i32)` 实现了 `Copy`，但 `(i32, String)` 没有。

#### 所有权和函数

将值传递给函数类似于为变量赋值。和变量赋值一样，将变量传递给函数会出现移动（move）或复制（copy）。看下面的一段代码：

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值被移动（move）到了 takes_ownership 函数中，
                                    // 因此它不再可用了

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 被传入 makes_copy 函数,
                                    // 但是因为 i32 类型实现了 Copy,
                                    // 因此在接下来还能够被使用

} // x 离开作用域, 接着 s 离开作用域. 但是由于 s 的所有权被拿走了，因此不会有什么特别的操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // some_string 离开作用域，接着 drop() 被调用以回收内存

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // some_integer 离开作用域，没有什么特别的操作
```

如果调用 `takes_ownership()` 之后尝试使用 `s`，Rust 会抛出一个编译错误。这些静态检查能够使我们免于出错。

#### 返回值和作用域

返回值也可以转移所有权：

```rust
fn main() {
    let s1 = gives_ownership();         // s1 获得了 gives_ownership() 返回值的所有权

    let s2 = String::from("hello");

    let s3 = takes_and_gives_back(s2);  // s2 的所有权交给了 takes_and_gives_back() 的参数，
                                        // 但又通过这个函数的返回值传给了 s3
} // s3、s1 经由 drop() 回收

fn gives_ownership() -> String {
    let some_string = String::from("yours");

    some_string // some_string 堆中部分的所有权将被交给调用它的变量
}

fn takes_and_gives_back(a_string: String) -> String { 
    a_string
}
```

所有变量的所有权都遵循相同的程式：把值直接赋给另一个变量会产生移动（move）。当包含了堆数据的变量离开作用域时，该值将通过 `drop()` 清除，除非该数据的所有权已经归另一个变量所有。

测试下面的一段代码：

```rust
fn main() {
    let s = String::from("hello");
    
    takes_ownership(s);
    
    println!("{}", s.len())
}

fn takes_ownership(s: String) {
    println!("{}", s.len());
}
```

意料之中地发生了编译错误，那么如何重新取得一个变量的所有权呢？可以通过下面的方式解决：

```rust
fn main() {
    let s1 = String::from("hello");

    let (s1, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();

    (s, length)
}
```

上面的代码借 Tuple 实现了多返回值，并使用了 Rust 的 *shadowing* 机制重新得到了所有权并交给了同名变量 `s1`，可以感觉到这个过程还是太麻烦了。幸运的是，Rust 提供了**引用**（*references*）机制来解决这个问题。

#### 总结

本节对于 Rust 的所有权机制进行了一个初步的介绍，Rust 用所有权机制让栈中的变量与其在堆中分配的数据一对一地关联起来（个人感觉这就像一条牵狗绳），在变量赋值的过程中，原变量会失去堆中数据的所有权，转交给被赋值的变量。通过这种方式，Rust 硬性地解决了常见的各种内存分配问题。由于所有权的唯一化，在变量作用域结束时就可以将其自动回收，不会出现忘记回收以及重复回收的问题。

### 引用和借用

上一节末尾提到的**引用**（*references*）机制，可以在不移动所有权的前提下访问一个变量的堆中数据。下面是由上一节末尾处代码改写的一个示例：

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

注意这段代码中的新语法 `&`（`&s1` 和 `&String`），使用这个操作符能够引用一个变量，从而在没有所有权的情况下操作这个变量的堆中数据。其底层的情况如下图所示：

![&String s pointing at String s1](Learning-Rust/trpl04-05.svg)

> 注意：使用 `&` 进行引用的相反操作是取消引用，可以通过取消引用操作符 `*` 来完成，这些内容暂时不进行讨论。

上述代码中，`calculate_length` 函数中的 `s` 是一个 `&String` 类型，即一个 `String` 的引用类型，这表明它不具有某个值的所有权，在离开作用域时不会对其调用 `drop()` 操作，并且这个函数也不需要归还任何值的所有权。

我们把创建引用的行为称为**借用**（*borrowing*），这个名字包含了“有借有还”的原则。

看到这里很自然地会有一个问题：我们能不能对“借”来的值进行写操作呢？尝试编译运行下面的代码：

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

```shell
$ cargo run
   Compiling ownership v0.1.0
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
  --> src\main.rs:42:5
   |
41 | fn change(some_string: &String) {
   |                        ------- help: consider changing this to be a mutable reference: `&mut String`
42 |     some_string.push_str(", world");
   |     ^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `ownership` due to previous error
```

答案是不能，从编译错误的信息中我们能够看到，由 `&` 借用来的 `String` 引用，默认是不可变的。

#### 可变引用

我们可以用下面的方式解决引用不可变的问题：

```rust
fn main() {
    let mut s = String::from("hello"); // mut

    change(&mut s); // mut
}

fn change(some_string: &mut String) { // mut
    some_string.push_str(", world");
}
```

上面的代码成功编译并运行了。需要注意的是所有权的拥有者（`s`）同样也要被改为可变的。

然而有个很大的问题是只能借用一次可变，下面的代码就会编译出错：

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{}, {}", r1, r2);
}
```

不能多次借用 `s` 作为可变参数。可能有人会想（比如我就这么想）：既然不能进行多次可变借用，那多次不可变借用行不行？

```rust
fn main() {
    let mut s = String::from("hello"); // 由于 mut 属性没有被用到，编译时会警告但不会报错

    let r1 = &s;
    let r2 = &s;

    println!("{}, {}", r1, r2);
}
```

这么写就可以了，但要是一个可变借用一个不可变借用呢？尝试下面的情况：

```rust
let r1 = &mut s;
let r2 = &s;
```

这次编译报错，提示 ``cannot borrow `s` as immutable because it is also borrowed as mutable``，意思是已经以可变的方式借用过了，不能再以不可变的方式借用了。再给它颠倒过来试一下：

```rust
let r1 = &s;
let r2 = &mut s;
```

这次仍然报错，提示 ``cannot borrow `s` as mutable because it is also borrowed as immutable``，同样不行，报错信息也反过来了。

第一个可变借用在 `r1` 中并且必须持续到它在 `println!` 中使用为止，要想再借用就必须在这之后（无论是可变还是不可变），所以下面的代码是有效的：

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    println!("{}", r1);
    
    let r2 = &mut s;
    println!("{}", r2);
}
```

综上所述，防止对同一数据进行多个可变引用的限制很严格，从而非常安全可控。不过虽然可以通过上面的代码来实现值的变动，但这实在是有些麻烦。

> 这也是很多新的 Rust 人士（Rustaceans，Rust 开发者的自称）想要努力解决的问题，因为大多数语言都可以让开发者随时进行变动。

有这个限制的好处是 Rust 可以在编译时防止出现**数据竞争**（*data races*）。数据竞争类似于竞争条件（*race condition*），当出现以下三种行为时就会发生：

- 两个或多个指针同时访问相同的数据；
- 至少有一个指针用于写入数据；
- 没有用于同步数据访问的机制。

数据竞争会导致未定义的行为，并且当你尝试在运行时追踪它们，可能难以诊断和修复。Rust 防止了这个问题的发生，因为带有数据竞争的代码编译直接不能通过。
