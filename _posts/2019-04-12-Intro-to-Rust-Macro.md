---
layout: post
title:  "Introduction to Rust Macro"
date: 2019-04-12 19:06:26 +0800
categories: Rust
---

Rust 通过反射和 AST 语法扩展这种宏方式来支持元编程。宏（Macro）分为两种：文本替换和语法扩展，Rust 使用后一种。宏在 Rust 的标准库和第三方库中广泛使用，如 std 里的 `println!`、`vec!` 和 `thread_local!` 等，第三方库包括 rust-prometheus，fail-rs，lazy_static 等。

<!--description-->

## 宏与函数的区别
宏是一种元编程，即使用代码来生成代码。例如，`derive` 属性可以为自定义类型生成各种 trait 的代码。

函数必须声明参数的类型和数量，而宏没有限制。另外，宏是在编译时进行代码展开的，所以宏可以为类型生成 trait 的代码，而函数只能在运行时调用。

使用宏比函数更为复杂，更难阅读、理解和维护。在一个文件里使用宏时，需要在文件中定义宏，或者 "bring macros into scope"。

## 两种类型的宏
- 声明宏：Declarative macros `macro_rules!`
- 过程宏：Procedural macros 
  - derive 宏：Custom `#[derive]` macros
  - attribute 宏：Attribute-like macros
  - function 宏：Function-like macros

### 声明宏 macro_rules!
- 使用最多的一种宏，也称为 “示例宏”。类似于 Rust 的 `match` 表达式，形如 “匹配 => 替换”。将来会被关键字 `macro` 替代。
- 以 "vec!" 为例，通常使用是 `let v: Vec<u32> = vec![1, 2, 3]`。其定义如下（简化版本）：
```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```
- 标注 `#[macro_export]` 会 "bring macro into scope"。
- 以 `macro_rules!` 加宏名开始定义，宏名后面不需要接`!`。
- 花括号里面是宏定义体。类似 `match` 表达式，模式 `( $( $x:expr ),* )` 在 `=>` 后面跟着代码块。如果模式匹配上了，就会触发后面的代码块。这里的宏模式是用于匹配 Rust 的代码结构的。括号里面是 "$(),*"，其中 "$()" 是来捕获匹配上其里面的 "$x:expr"，然后将 "$x" 用于代码块里的代码替换。
- "$()" 后面的逗号表示","可能会出现匹配的代码后面。"*" 表示模式匹配会进行 0 或多次。
- 当使用 `vec![1, 2, 3]`，"$x" 模式会匹配三次，"$x" 依次是 `1`，`2` 和 `3`。
- "=>" 后的代码块里，`temp_vec.push($x)` 位于 "$()*" 的括号里，会根据 "=>" 前面的 "$()" 匹配情况进行相应的代码生成，即每匹配一次就生成一次。其 "$x" 也会被依次替换。 `vec![1, 2, 3]` 进行宏展开后的代码如下：
```rust
let mut temp_vec = Vec::new();
temp_vec.push(1);
temp_vec.push(2);
temp_vec.push(3);
temp_vec
```
- 更详细的介绍，参见 [The Little Book of Rust Macros](https://danielkeep.github.io/tlborm/book/index.html)

### 过程宏
- 更像函数，输入 Rust 代码，然后对代码进行修改，最后输出 Rust 代码。
- 分为三种，但工作方式很类似。derive 宏 (`#[proc_macro_derive]`) 用于为类型（如 struct、enum）生成 trait 的代码，即 derive 属性；attribute 宏 (`#[proc_macro_attribute]`) 不同于 derive 宏，它用创建新的属性，更加灵活，除了 struct、enum 之外，还能为 function 定义属性。 function 宏 (`#[proc_macro]`)，可以将函数一样被调用。

#### derive 宏
- 讲解代码见 https://github.com/fredchenbj/RustLearning/tree/master/macro
- 背景 hello_macro 包定义了 HelloMacro trait，有函数 hello_macro 来将 struct XX 按规定格式打印出来。一般情况下，只需要为 XX 实现如下定义就可以了：
```rust
Impl HelloMacro for XX {
    pub fn hello_macro() {
        ...
    }
}
```
- 但这种方式的话，每种类型都需要 Impl HelloMacro。使用 derive 宏，可以在 struct XX 之前添加 "#[derive(HelloMacro)]" 来自动生成相关代码。
- 在根包 hello_macro 里定义 `trait HelloMacro`，子包 hello_macro_derive 里定义 `#[proc_macro_derive(HelloMacro)]`，其 Cargo.toml 里包含以下内容：

```toml
[lib]
proc-macro = true

[dependencies]
syn = "0.14.4"
quote = "0.6.3"
```
- 看一下 hello_macro_derive 里的定义，如下：

```rust
extern crate proc_macro;

use crate::proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    let ast = syn::parse(input).unwrap();
    impl_hello_macro(&ast)
}

fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}", stringify!(#name));
            }
        }
    };
    gen.into()
}
```
- 当将 `#[derive(HelloMacro)]` 放在 struct XX 之前时，就会调用 `hello_macro_derive`。这是因为 `#[derive(HelloMacro)]` 和 `#[proc_macro_derive(HelloMacro)]` 对应上了。
- 获取 TokenStream 之后，需要将其转换为我们可以理解和操作的结构。sync::parse 会解析 Rust 代码生成一个 DeriveInput 的 AST 结构。这个结构的 `indent` 字段就是 `struct XX` 的名字 `XX`。详细结构内容见 [syn doc](https://docs.rs/syn/0.14.4/syn/struct.DeriveInput.html)。
- 返回的也是 TokenStream，所以需要使用 `quote!` + `into` 将生成的代码块转换为 TokenStream。

#### attribute 宏
- 如一个 rest api 的 route 服务，为每个访问接口定义 route 属性，如下：
```rust
#[route(GET, "/")]
fn get_something() {}
```
- route 的 attribute 宏定义的函数如下：
```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream { }
```
- 这里的输入是两个 TokenStream，第一个是 route attibute 的内容 `GET, "/")`，第二个是属性作用的内容，即 `fn get_something() {}`
- 其他都与 derive 宏类似。

#### function 宏
- 访问和定义，如下所示。其他与 derive 宏类似。

```rust
// call 
let sql = sql!(SELECT * FROM posts WHERE id=1);

// define
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream { }
```

## 实战应用
- `thread_local!`: https://doc.rust-lang.org/std/macro.thread_local.html
- `lazy_static!`: https://docs.rs/lazy_static/1.3.0/lazy_static/
