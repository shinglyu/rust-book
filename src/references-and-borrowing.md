# 參考與借用

本指南是當前 Rust 的三個所有權系統之一。
這是 Rust 最獨特且引人注目的功能之一，作為 Rust 的開發者應該對此要有相當的了解。
所有權是 Rust 用來達成其最大的目標，記憶體安全，的方法。
它有一些不同的概念，各自有各自的章節：

* [所有權][ownership] (ownership)，關鍵的概念
* 借用 (borrowing)，你正在閱讀的章節
* [生命週期][lifetimes] (lifetime)，借用的進階概念

這三章依序相關。
你需要了解全部三章來完整了解所有權系統。

[ownership]: ownership.html
[lifetimes]: lifetimes.html

## Meta

在我們開始細述前，有兩個所有權系統的重點。

Rust 注重安全和速度。
它透過許多 "零成本抽象化" 的方式去實現目標，也就是 Rust 將盡可能縮小抽象化成本去達成目標。
所有權系統是零成本抽象化的一個最佳範例。
我們在本指南中談到的所有分析，都是在 _編譯期完成的_。
這些功能不需要花費你任何執行期的成本。

然而，這套系統仍有某些成本：學習曲線。
許多 Rust 的新使用者會經歷我們所說的 "與借用檢查器 (borrow checker) 戰鬥" 的經驗，也就是 Rust 編譯器無法編譯一個作者認為合理的程式。
在程式設計師內心的所有權運作模型與實際上 Rust 實作不相符的時候，這會常常發生。
一開始你可能也會經歷類似的事情。
然而有個好消息：許多有經驗的 Rust 開發者回報，當他們適應所有權系統的規則一陣子之後，他們跟借用檢查器的戰鬥就越來越少了。

記住這些之後，讓我們開始學習借用。

## 借用 (Borrowing)

在[所有權][ownership]一節的最後，我們提到一個看起來頗糟的函式：

```rust
fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // do stuff with v1 and v2

    // hand back ownership, and the result of our function
    (v1, v2, 42)
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let (v1, v2, answer) = foo(v1, v2);
```

這不符合 Rust 的語言習慣，因為它沒有利用到借用的優點。
以下是第一步：

```rust
fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    // do stuff with v1 and v2

    // return the answer
    42
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let answer = foo(&v1, &v2);

// we can use v1 and v2 here!
```

與其把 `Vec<i32>` 作為我們的參數，不如使用參考 (reference)：`&Vec<i32>`。
而且不要直接傳遞 `v1` 與 `v2`，我們傳遞 `&v1` 與 `&v2`。
我們把 `&T` 型別稱為 "參考" (reference)，它借用了所有權，而非掌握所有權。
一個借用東西的綁定不會在離開有效範圍時釋放資源。
這代表在呼叫 `foo()` 完之後，我們仍可以再度使用我們原始的綁定。

跟綁定一樣，參考是不可變的 (immutable)。
這意味著在 `foo()` 內，向量完全不能更改：

```rust,ignore
fn foo(v: &Vec<i32>) {
     v.push(5);
}

let v = vec![];

foo(&v);
```

出現以下錯誤：

```text
error: cannot borrow immutable borrowed content `*v` as mutable
v.push(5);
^
```

因為放入一個值會改變向量，所以我們不允許這樣做。

## &mut 參考

還有第二類的參考：`&mut T`。
一個 "可變的參考" 允許你改變你所借用的資源。
例如：

```rust
let mut x = 5;
{
    let y = &mut x;
    *y += 1;
}
println!("{}", x);
```

這將會印出 `6`。
我們將 `y` 作為一個指到 `x` 的可變參考，然後對 `y` 指向的東西加一。
你將注意到 `x` 也必須被標註為 `mut`。
如果不是的話，我們將無法建立一個可變得借用到一個不可變的值。

> 譯註：如果 `x` 不是 `mut`，會得到 `error: cannot borrow immutable local variable `x` as mutable` 錯誤訊息。

你同時也會發現我們在 `y` 前面加上星號（`*`）成為 `*y`，這是因為 `y` 是一個 `&mut` 參考，
你需要使用它們去存取參考的內容。

此外，`&mut` 參考如同一般的參考。它們兩者和它們的互動方式 _有著_ 巨大的區別。
你可以說以上的範例有點不可靠，因為我們需要額外的 `{` 和 `}` 定義有效區域。
如果移除它們，我們會得到錯誤訊息：

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
note: previous borrow of `x` occurs here; the mutable borrow prevents
subsequent moves, borrows, or modification of `x` until the borrow ends
        let y = &mut x;
                     ^
note: previous borrow ends here
fn main() {

}
^
```

事實證明如此，所以以下是一些規則。

## 規則

以下是 Rust 中關於借用 (borrowing) 的規則：

首先，任何借用的有效範圍都必須比擁有者的有效範圍還要小。
其次，你可以使用以下兩種借用，但是不能同時使用兩者：

* 一到多個對資源的參考（`&T`）
* 唯一一個可變參考（`&mut T`）

你可能有注意到有點類似、但不完全相同於資料競爭的定義：

> 當兩個或多個指標同時存取相同的記憶體，其中至少有一個正在寫入，且操作沒有同步時，會出現 "資料競爭" (date race)。

使用參考時，你可以想要多少參考就用多少參考，因為它們沒有人在寫入。
然而，一次只能擁有一個 `&mut`，這才不會產生資料競爭。
這就是 Rust 如何在編譯期預防資料競爭：當我們違反規則時，會得到錯誤訊息。

記住這些之後，讓我們再次想想我們的範例。

### 對有效範圍的深思 (Thinking in scopes)

以下是程式碼：

```rust,ignore
let mut x = 5;
let y = &mut x;

*y += 1;

println!("{}", x);
```

這段程式碼會有這些錯誤訊息：

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
```

> 譯註：此處 `println!()` 試圖借用 `x`。

這是因為我們違反了規則：我們有了一個指向 `x` 的 `&mut T`，所以我們不允許建立任何其他 `&T`。
要在兩者間做出選擇。
錯誤中的註解提示我們該如何思考這個問題：

```text
note: previous borrow ends here
fn main() {

}
^
```

換句話說，可變借用 (mutable borrow) 在我們的範例中一直存在。
所以我們希望可變借用能在我們呼叫 `println!` 並建立不可變借用 _之前_ 能結束掉。
在 Rust 中，借用綁定在借用有效的範圍中。
我們的有效範圍看起來會像這樣：

```rust,ignore
let mut x = 5;

let y = &mut x;    // -+ &mut borrow of x starts here
                   //  |
*y += 1;           //  |
                   //  |
println!("{}", x); // -+ - try to borrow x here
                   // -+ &mut borrow of x ends here
```

這些有效範圍有衝突：我們不能在 `y` 仍在有效範圍時建立 `&x`。

而當我們增加大括號之後：

```rust
let mut x = 5;

{
    let y = &mut x; // -+ &mut borrow starts here
    *y += 1;        //  |
}                   // -+ ... and ends here

println!("{}", x);  // <- try to borrow x here
```

這樣就沒有問題了。
我們的可變借用會在我們建立不可變借用前離開有效範圍。
有效範圍是個看清借用持續多久的關鍵。

### 借用所預防的問題 (Issues borrowing prevents)

為何我們需要這些限制規則？
好吧，如同我們所說的，這些規則避免資料競爭。
資料競爭會引發哪些問題？
以下是一些例子。

#### 疊代器失效

一個例子是 "疊代器失效" (Iterator invalidation)，當你試圖改變一個正在疊代的集合 (collection) 時會發生。
Rust 的借用檢查器會預防這件事發生：

```rust
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
}
```

這會印出一到三。
當我們疊代這個向量時，我們只會用到元素的參考。
而且 `v` 是一個不可變的借用，所以我們在疊代時不能更改它：

```rust,ignore
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
    v.push(34);
}
```

以下是錯誤訊息：

```text
error: cannot borrow `v` as mutable because it is also borrowed as immutable
    v.push(34);
    ^
note: previous borrow of `v` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `v` until the borrow ends
for i in &v {
          ^
note: previous borrow ends here
for i in &v {
    println!(“{}”, i);
    v.push(34);
}
^
```

我們不能修改 `v`，因為它已經借用給迴圈了。

#### 在釋放之後使用 (use after free)

參考不能存活得比所參考的資源還久。
Rust 會檢查你的參考的有效範圍來確保符合這個條件。

如果 Rust 沒有檢查這個屬性，我們可能會意外地用到一個無效的參考。
例如：

```rust,ignore
let y: &i32;
{
    let x = 5;
    y = &x;
}

println!("{}", y);
```

會得到以下錯誤：

```text
error: `x` does not live long enough
    y = &x;
         ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
let y: &i32;
{
    let x = 5;
    y = &x;
}

note: ...but borrowed value is only valid for the block suffix following
statement 0 at 4:18
    let x = 5;
    y = &x;
}
```

換句話說，`y` 只在 `x` 存在的有效範圍內有效。
當 `x` 消失，它就成為無效的參考。
這個錯誤訊息說，這個借用 "存在得不夠久" 就是因為它在應該存在的時候已經失效了。

當一個參考在它所參考的變數 _之前_ 宣告，也會發生一樣的問題。
這是因為資源在同樣的有效範圍內，它們被釋放的順序是跟它們宣告順序相反：

> 譯註：也就是 `宣告 y; 宣告 x;` 的話，離開有效範圍時會 `釋放 x; 釋放 y`。所以 `y` 比 `x` 存活的久。

```rust,ignore
let y: &i32;
let x = 5;
y = &x;

println!("{}", y);
```

得到以下錯誤訊息：

```text
error: `x` does not live long enough
y = &x;
     ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
    let y: &i32;
    let x = 5;
    y = &x;

    println!("{}", y);
}

note: ...but borrowed value is only valid for the block suffix following
statement 1 at 3:14
    let x = 5;
    y = &x;

    println!("{}", y);
}
```

在以上範例，`y` 在 `x` 之前宣告，代表 `y` 存活的比 `x` 還長，這不被允許。


> *commit 6ba9520*

