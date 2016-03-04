# 函式

所有的 Rust 程式都至少有一個函式，`main` 函式，又稱主函式：

```rust
fn main() {
}
```

這也許是最簡單的函式宣告了。
像我們之前說的，`fn` 表示 "這是一個函式"，後面是名稱，有個括號，沒有參數，而且有大括號去標示函式的內容。
以下是一個名叫 `foo` 的函式：

```rust
fn foo() {
}
```

那如果有參數呢？
以下是一個印出數字的函式：

```rust
fn print_number(x: i32) {
    println!("x is: {}", x);
}
```

這是一個使用 `print_number` 函式的完整程式：

```rust
fn main() {
    print_number(5);
}

fn print_number(x: i32) {
    println!("x is: {}", x);
}
```

如你所見，函式的參數作用的方式跟 `let` 宣告非常類似：
在參數名後面加上冒號，後面再加上型別。

以下是一個把兩個數字相加之後印出來的完整程式：

```rust
fn main() {
    print_sum(5, 6);
}

fn print_sum(x: i32, y: i32) {
    println!("sum is: {}", x + y);
}
```

當你呼叫函式或宣告函式時，需要用逗號分隔多個參數。

與 `let` 不同的是，你 _必須_ 宣告函式參數的型別。
以下是無法作用的程式碼：

```rust,ignore
fn print_sum(x, y) {
    println!("sum is: {}", x + y);
}
```

你會得到這些錯誤訊息：

```text
expected one of `!`, `:`, or `@`, found `)`
fn print_sum(x, y) {
```

這是個深思熟慮後的設計決定。
即使像 Haskell 這樣可以對整個程式推斷 (full-program inference) 的語言，在最佳做法中仍經常建議要明確地標註你的型別。
我們認為在宣告函式時強制宣告型別，且允許在函式內容中允許推斷，是個在全推斷 (full-inference) 與無推斷間的最佳平衡。

那要如何回傳值呢？
以下是一個把整數加一的函式：

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```

Rust 函式就只能回傳一個值，你可以在 "箭頭" 後面宣告型別，箭頭是一個破折號（`-`）加上一個大於符號（`>`）。
函式的最後一行決定了回傳什麼。
你會注意到這邊並沒有分號。
如果我們加上分號：

```rust,ignore
fn add_one(x: i32) -> i32 {
    x + 1;
}
```

我們將看到錯誤訊息：

```text
error: not all control paths return a value
fn add_one(x: i32) -> i32 {
     x + 1;
}

help: consider removing this semicolon:
     x + 1;
          ^
```

這展現出了 Rust 兩個有趣的點：它是以表達式為基礎的語言，而且分號的使用跟其他 "大括號及分號" 的語言不同。
這兩件事是息息相關的。

## 表達式 vs. 陳述式

Rust 主要是個以表達式為基礎的語言。
它只有兩種陳述式，而其他的都是表達式。

那這有什麼分別？
表達式回傳值，而陳述式則否。
這也是為什麼我們在這說 "不是所有的控制路徑都回傳值"：陳述式 `x + 1;` 不傳回值。
Rust 中有兩種陳述式："宣告陳述式" (declaration statements) 和 "表達陳述式" (expression statements)。
其他的都是表達式。
讓我們先討論宣告陳述式。

在一些語言中，變數綁定可以被寫成表達式，而非陳述式。
像 Ruby：

```ruby
x = y = 5
```

然而在 Rust 中，使用 `let` 提出綁定 _不是_ 一個表達式。
以下程式碼會產生一個編譯期錯誤：

```ignore
let x = (let y = 5); // expected identifier, found keyword `let`
```

編譯器或告訴我們這裡預期會看到表達式的開頭，而 `let` 只能用於開始一個陳述式，不是表達式。

請注意，賦值到一個已經綁定的變數（例如 `y = 5`）仍然是個表達式，雖然它的值沒有什麼用。
不像其他語言的賦值會被算為賦予的數值（在前面的例子中會是 `5`），在 Rust 中賦值的執會是一個空的多元組 (tuple) `()`，因為賦予的值[只能有一個擁有者](ownership.html)，而其他任何回傳值都會讓人意外：

```rust
let mut y = 5;

let x = (y = 6);  // x has the value `()`, not `6`
```

Rust 中的第二種陳述式是 *表達陳述式* (expression statement)。
它的作用是把表達式轉為陳述式。
實際上，Rust 的文法預期陳述式後面也是其他陳述式。
這意味著你要使用分號去分隔表達式們。
這代表 Rust 看起來跟要求在每行結果使用分號的大多數語言一樣，你幾乎將會在每行結尾看到分號。

是什麼例外讓我們說 "幾乎"？
你早就看到了，就在以下程式碼中：

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```

我們的函式聲明會回傳一個 `i32`，但是有分號時，它卻會回傳 `()`。
Rust 覺得這應該不是我們想要的，所以在上面看到的錯誤訊息中建議我們移除分號。

## 提早回傳 (Early returns)

那麼提早回傳又該怎樣做？
Rust 的確有一個關鍵字 `return` 可以提早回傳：

```rust
fn foo(x: i32) -> i32 {
    return x;

    // we never run this code!
    x + 1
}
```

使用 `return` 在函式的最後一行是可行的，但是這被認為是個不好的程式風格：

```rust
fn foo(x: i32) -> i32 {
    return x + 1;
}
```

如果你過去沒有寫過以表達式為基礎的語言，那麼在之前沒有 `return` 的定義可能讓你覺得看起來有些奇怪。
不過隨著時間過去，它會變得很直觀。

## 發散函式 (Diverging functions)

Rust 也有一些特別的語法叫做 "發散函式"，這種函式不回傳值：

```rust
fn diverges() -> ! {
    panic!("This function never returns!");
}
```

`panic!` 是一個巨集，跟我們看過的 `println!()` 類似。
不像 `println!()`，`panic!()` 會讓當前的執行緒帶著指定的訊息當機 (crash)。
因為這個函式將會導致當機，所以它不會回傳值，也因此它的型別是 "`!`"，它叫做 "發散" (diverges)。

如果你加入一個主函式去呼叫 `diverges()` 並執行它，你將會得到像以下的輸出：

```text
thread ‘<main>’ panicked at ‘This function never returns!’, hello.rs:2
```

如果你想看到更多資訊，你可以透過設定 `RUST_BACKTRACE` 環境變數去取得 backtrace：

```text
$ RUST_BACKTRACE=1 ./diverges
thread '<main>' panicked at 'This function never returns!', hello.rs:2
stack backtrace:
   1:     0x7f402773a829 - sys::backtrace::write::h0942de78b6c02817K8r
   2:     0x7f402773d7fc - panicking::on_panic::h3f23f9d0b5f4c91bu9w
   3:     0x7f402773960e - rt::unwind::begin_unwind_inner::h2844b8c5e81e79558Bw
   4:     0x7f4027738893 - rt::unwind::begin_unwind::h4375279447423903650
   5:     0x7f4027738809 - diverges::h2266b4c4b850236beaa
   6:     0x7f40277389e5 - main::h19bb1149c2f00ecfBaa
   7:     0x7f402773f514 - rt::unwind::try::try_fn::h13186883479104382231
   8:     0x7f402773d1d8 - __rust_try
   9:     0x7f402773f201 - rt::lang_start::ha172a3ce74bb453aK5w
  10:     0x7f4027738a19 - main
  11:     0x7f402694ab44 - __libc_start_main
  12:     0x7f40277386c8 - <unknown>
  13:                0x0 - <unknown>
```

`RUST_BACKTRACE` 也能用在 Cargo 的 `run` 指令上：

```text
$ RUST_BACKTRACE=1 cargo run
     Running `target/debug/diverges`
thread '<main>' panicked at 'This function never returns!', hello.rs:2
stack backtrace:
   1:     0x7f402773a829 - sys::backtrace::write::h0942de78b6c02817K8r
   2:     0x7f402773d7fc - panicking::on_panic::h3f23f9d0b5f4c91bu9w
   3:     0x7f402773960e - rt::unwind::begin_unwind_inner::h2844b8c5e81e79558Bw
   4:     0x7f4027738893 - rt::unwind::begin_unwind::h4375279447423903650
   5:     0x7f4027738809 - diverges::h2266b4c4b850236beaa
   6:     0x7f40277389e5 - main::h19bb1149c2f00ecfBaa
   7:     0x7f402773f514 - rt::unwind::try::try_fn::h13186883479104382231
   8:     0x7f402773d1d8 - __rust_try
   9:     0x7f402773f201 - rt::lang_start::ha172a3ce74bb453aK5w
  10:     0x7f4027738a19 - main
  11:     0x7f402694ab44 - __libc_start_main
  12:     0x7f40277386c8 - <unknown>
  13:                0x0 - <unknown>
```

一個發散函式可以被用在任何型別上：

```should_panic
# fn diverges() -> ! {
#    panic!("This function never returns!");
# }
let x: i32 = diverges();
let x: String = diverges();
```

## 函式指標 (Function pointers)

我們也可以建立指向函式的變數綁定：

```rust
let f: fn(i32) -> i32;
```

`f` 是一個指向以 `i32` 作為參數且回傳 `i32` 的函式的變數綁定。
例如：

```rust
fn plus_one(i: i32) -> i32 {
    i + 1
}

// without type inference
let f: fn(i32) -> i32 = plus_one;

// with type inference
let f = plus_one;
```

接著，我們可以使用 `f` 去呼叫函式：

```rust
# fn plus_one(i: i32) -> i32 { i + 1 }
# let f = plus_one;
let six = f(5);
```


> *commit 3eebec6*
