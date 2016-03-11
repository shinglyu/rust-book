# 變數綁定


事實上所有非 "Hello World" 的 Rust 程式都會用到 *變數綁定*。
他們將數值綁定到一個名字上，以便在之後使用它。
`let` 用來聲明一個綁定，就像：

```rust
fn main() {
    let x = 5;
}
```

在所有範例中放入 `fn main() {` 有點冗餘，所以我們在後面都會省略它。
如果你一路看下去，請記得寫上 `main()` 韓式，不要忘記了。
否則你會得到錯誤訊息。

## 模式 (Patterns)

在很多語言，變數綁定被叫做 *變數* (variable)，但是 Rust 的變數綁定藏有秘招。
例如 `let` 表達式左邊是一個[模式][pattern]，而不是變數名稱。
這代表我們可以做一些如下的事：

```rust
let (x, y) = (1, 2);
```

在這個表達式計算後，`x` 會是 1，而 `y` 會是 2。
模式非常強大，而且在本書中有[自己的章節][pattern]。
現在我們還不需要這些功能，所以我們只要記住有這個東西，然後繼續就可以了。

[pattern]: patterns.html

## 型別註釋 (Type annotations)

Rust 是靜態型別語言，這代表我們要先具體指定我們所需的型別，然後它們會在編譯期被檢查。
那為什麼第一個例子可以編譯過呢？
恩，因為 Rust 有一個 "型別推斷" (type inference) 的功能。
如果它能判斷某個東西的型別，你就不需要確切地指出來。

如果我們想要，也可以加上型別。
型別寫在冒號 (`:`) 後面：

```rust
let x: i32 = 5;
```

如果我要求你大聲唸給班上同學聽，你應該唸成『`x` 被綁定為 `i32` 型別，而且數值是 `5`。』

這個例子中我們宣稱 `x` 為一個 32 位元的帶號整數 (signed integer)。
Rust 有許多不同的基本整數型別。
以 `i` 開頭的是帶號整數 (signed integers)，而以 `u` 開頭的是非帶號整數 (unsigned integers)。
整數可能的大小有 8、16、32、及 64 位元。

之後的範例，我們會把型別註釋在註解中。
範例會像這樣：

```rust
fn main() {
    let x = 5; // x: i32
}
```

注意，註釋和 `let` 語法很類似。
Rust 習慣上不會有這些註解，但是我們偶爾會加上它們來幫助你理解 Rust 推斷的是什麼型別。

## 可變性 (Mutability)

綁定的預設是 *不可變的* (immutable)。
下面的原始碼無法編譯：

```rust,ignore
let x = 5;
x = 10;
```

他會給你以下錯誤訊息：

```text
error: re-assignment of immutable variable `x`
     x = 10;
     ^~~~~~~
```

如果你要綁定成為可變的 (mutable)，你可以用 `mut`：

```rust
let mut x = 5; // mut x: i32
x = 10;
```

有不只一個理由要讓綁定的預設是不可變的，但是我們可以藉由一個 Rust 的一個主要目標來想想看：安全。
如果你忘記宣告 `mut`，編譯器會抓到它，然後讓你知道你可能改了某些並非真的想要改的東西。
如果綁定預設是可變的，編譯器就無法告訴你了。
如果你 _真的_ 打算要改變，解決方式很簡單：加上 `mut`。

盡可能的避免改變狀態有其他好的理由，但是他們不在本指南的範圍內。
一般來說，你會避免直接了當的使用可變數 (mutation)，這也是 Rust 希望的。
雖然如此，有時候，你還是需要可變數，所以他沒有被禁止使用。

## 初始化綁定 (Initializing bindings)

Rust 的變數綁定有跟其他語言不同的一個方面：綁定需要再使用之前初始化一個數值。

讓我們試試看。
把你的 `src/main.rs` 改成以下這樣：

```rust
fn main() {
    let x: i32;

    println!("Hello world!");
}
```

你可以輸入 `cargo build` 去建構它。
你將會得到警告訊息，但是它仍會印出 "Hello, world!"：

```text
   Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
src/main.rs:2:9: 2:10 warning: unused variable: `x`, #[warn(unused_variable)]
   on by default
src/main.rs:2     let x: i32;
                      ^
```

Rust 警告我們沒有使用這個變數綁定，但是的確我們沒有使用它，沒關係。
但如果我們真的使用 `x`，事情就不同了。
讓我們試試看。
把你的程式改成這樣：

```rust,ignore
fn main() {
    let x: i32;

    println!("The value of x is: {}", x);
}
```

然後試著建構它。
你會得到錯誤訊息：

```bash
$ cargo build
   Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
src/main.rs:4:39: 4:40 error: use of possibly uninitialized variable: `x`
src/main.rs:4     println!("The value of x is: {}", x);
                                                    ^
note: in expansion of format_args!
<std macros>:2:23: 2:77 note: expansion site
<std macros>:1:1: 3:2 note: in expansion of println!
src/main.rs:4:5: 4:42 note: expansion site
error: aborting due to previous error
Could not compile `hello_world`.
```

Rust 不讓我們使用沒有被初始化的數值。
接著，讓我們討論我們加在 `println!` 的東西。

如果你在要印出的字串中加入大括號（`{}` 有些人稱作 moustaches ...），Rust 會理解為這是一個插入數值的要求。
*字串插值* (string interpolation) 是個電腦科學術語，代表 "插入到字串中"。
我們加上一個逗號和 `x`，來表示我們想要用 `x` 當成插入的值。
逗號是當我們傳遞多個參數時，用來分隔我們傳遞給函式和巨集的參數用的。

當你使用大括號時，Rust 會試著檢查型別，用有意義的方式顯示數值。
如果你想用更詳細的方式去指定格式，這邊有[很多方式可供選擇][format]。
現在，我們使用預設方式插入：印出整數並不難。

[format]: ../std/fmt/index.html

## 有效範圍及遮蔽 (Scope and shadowing)

讓我們回到綁定。
變數綁定有有效範圍 - 它們被限制在它們被定義的區塊中存在。
一個區塊 (block) 是一個被用 `{` 和 `}` 包起來的陳述式的集合。
函式的定義也是一個區塊！
在以下範例我們定義兩個變數綁定，`x` 和 `y`，他們存在於不同區塊中。
`x` 可以在 `fn main() {}` 內存取，而 `y` 只能在內部區塊 (inner block) 存取。

```rust,ignore
fn main() {
    let x: i32 = 17;
    {
        let y: i32 = 3;
        println!("The value of x is {} and value of y is {}", x, y);
    }
    println!("The value of x is {} and value of y is {}", x, y); // This won't work
}
```

第一個 `println!` 會印出 "The value of x is 17 and the value of y is 3"，但是本範例無法成功編譯，因為第二個 `println!` 不能存取 `y` 的值，因為它已經不在有效範圍內了。
我們會得到以下錯誤訊息：

```bash
$ cargo build
   Compiling hello v0.1.0 (file:///home/you/projects/hello_world)
main.rs:7:62: 7:63 error: unresolved name `y`. Did you mean `x`? [E0425]
main.rs:7     println!("The value of x is {} and value of y is {}", x, y); // This won't work
                                                                       ^
note: in expansion of format_args!
<std macros>:2:25: 2:56 note: expansion site
<std macros>:1:1: 2:62 note: in expansion of print!
<std macros>:3:1: 3:54 note: expansion site
<std macros>:1:1: 3:58 note: in expansion of println!
main.rs:7:5: 7:65 note: expansion site
main.rs:7:62: 7:63 help: run `rustc --explain E0425` to see a detailed explanation
error: aborting due to previous error
Could not compile `hello`.

To learn more, run the command again with --verbose.
```

另外，變數綁定可以被遮蔽 (shadowed)。
這代表與其他變數綁定有同樣名稱的後一個變數綁定，在有效範圍內，將會覆蓋前一個綁定。

```rust
let x: i32 = 8;
{
    println!("{}", x); // Prints "8"
    let x = 12;
    println!("{}", x); // Prints "12"
}
println!("{}", x); // Prints "8"
let x =  42;
println!("{}", x); // Prints "42"
```

遮蔽 (shadowing) 和可變綁定 (mutable bindings) 也許像同一枚硬幣的兩面一樣，但他們是兩個不同的概念，所以不一定總是可以直接替換。
作為其中之一，遮蔽可以讓我們重新綁定名稱到不同型別的變數。
它也可以改變綁定的可變性。

```rust
let mut x: i32 = 1;
x = 7;
let x = x; // x is now immutable and is bound to 7

let y = 4;
let y = "I can also be bound to text!"; // y is now of a different type
```


> *commit 6ba9520*
