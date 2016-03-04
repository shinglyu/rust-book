# 迴圈

Rust 目前提供三種方式去執行一些疊代行為。
它們是 `loop`、`while`、及 `for`。
它們各有各自的用途。

> 譯註：iterative activity 這邊參考[維基百科](https://zh.wikipedia.org/wiki/%E8%BF%AD%E4%BB%A3)，故使用疊代來翻譯。

## loop

無限 `loop` 是 Rust 中最簡單的 loop 形式。
使用 `loop` 關鍵字，Rust 會提供給你一個方法，會無限循環直到某些終結陳述達成。
Rust 的無限 `loop` 看起來像這樣：

```rust,ignore
loop {
    println!("Loop forever!");
}
```

## while

Rust 也有 `while` 迴圈。
看起來像這樣：

```rust
let mut x = 5; // mut x: i32
let mut done = false; // mut done: bool

while !done {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 {
        done = true;
    }
}
```

當你不確定你需要循環幾次時，`while` 迴圈是你的正確選擇。

如果你需要無限迴圈，你可能會想這樣寫：

```rust,ignore
while true {
```

然而 `loop` 更適合去處理這種情況：

```rust,ignore
loop {
```

Rust 的控制流程分析器會差別對待這個結構與 `while true`，因為我們知道它會一直循環。
一般來說，你給編譯器越多資訊，越能讓它更安全且產生更好的程式碼，所以當你計畫要無限循環時，你應該使用 `loop`。

## for

`for` 迴圈用來循環特定次數。
然而，Rust 的 `for` 迴圈與其他系統程式語言有些不同。
Rust 的 `for` 迴圈看起來不像 "C 語言風格" 的 `for` 迴圈：

```c
for (x = 0; x < 10; x++) {
    printf( "%d\n", x );
}
```

反之，它看起來像這樣：

```rust
for x in 0..10 {
    println!("{}", x); // x: i32
}
```

更抽象一點：

```ignore
for var in expression {
    code
}
```

這邊的表達式是一個能以 [IntoIterator] 轉成[疊代器][iterator] (iterator) 的東西。
疊代器會傳回一系列的元素。
每個元素是迴圈中的一次循環。
在這次迴圈的有效範圍內，元素的值會跟 `var` 綁定。
當迴圈結束，下一個值會從疊代器中取出，然後重複另一次。
當疊代器中沒有值了，`for` 迴圈就結束。

[iterator]: iterators.html
[IntoIterator]: https://doc.rust-lang.org/std/iter/trait.IntoIterator.html

在我們的範例中，`0..10` 表達式會根據開始和結束的值，給予一個含有兩者之間數字的疊代器。
上限值不包含在其中，所以我們的迴圈會印出 `0` 到 `9`，沒有 `10`。

Rust 特別不使用 "C 語言風格" 的 `for` 迴圈。
即使對有經驗的 C 語言程式設計師來說，手動控制迴圈中的每個元素仍是複雜且容易出錯的。

### 枚舉 (Enumerate)

當你需要追蹤你已經循環了幾次，你可以使用 `.enumerate()` 函式。

#### 對範圍

```rust
for (i,j) in (5..10).enumerate() {
    println!("i = {} and j = {}", i, j);
}
```

輸出：

```text
i = 0 and j = 5
i = 1 and j = 6
i = 2 and j = 7
i = 3 and j = 8
i = 4 and j = 9
```

別忘了在範圍加上括號。

> 譯註：`for (i,j) in (5..10).enumerate()` 中的 `i` 就像是 index，可以用來計數，`j` 則是原來的值。

#### 對疊代器

```rust
let lines = "hello\nworld".lines();

for (linenumber, line) in lines.enumerate() {
    println!("{}: {}", linenumber, line);
}
```

輸出：

```text
0: hello
1: world
```

## 提早結束疊代

讓我們看看前面提到的 `while` 迴圈：

```rust
let mut x = 5;
let mut done = false;

while !done {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 {
        done = true;
    }
}
```

我們必須保持一個專用的 `mut` 布林變數綁定 `done`，用來知道何時我們應該離開迴圈。
Rust 有兩個關鍵字可以幫助我們修改這個疊代：`break` 和 `continue`。

在這個情況下，我們可以用 `break` 把迴圈寫得更好：

```rust
let mut x = 5;

loop {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 { break; }
}
```

我們現在使用 `loop` 無限循環，並使用 `break` 去提早結束。
使用 `return` 陳述式也可以提早結束迴圈。

`continue` 也很類似，但是並非結束迴圈，而是跳到下一次疊代。
這是一個只印出奇數的程式碼：

```rust
for x in 0..10 {
    if x % 2 == 0 { continue; }

    println!("{}", x);
}
```

## 迴圈標籤 (Loop labels)

你可能也會遇到一些情況，例如當你有槽狀迴圈，且希望你的 `break` 或 `continue` 能指定做用到哪一個的時候。
跟其他多數語言一樣，預設的 `break` 或 `continue` 會作用在最內層的迴圈。
當你希望你的 `break` 或 `continue` 要作用在外層的某個迴圈時，你可以使用標籤去指定 `break` 或 `continue` 要作用在哪層迴圈。
下面程式碼只會在 `x` 和 `y` 都是奇數的時候印出東西：

```rust
'outer: for x in 0..10 {
    'inner: for y in 0..10 {
        if x % 2 == 0 { continue 'outer; } // continues the loop over x
        if y % 2 == 0 { continue 'inner; } // continues the loop over y
        println!("x: {}, y: {}", x, y);
    }
}
```


> *commit 7df3bf1*
