# if

Rust 的 `if` 沒有特別複雜，但你會發現它更像動態型別語言的 `if` 而非傳統的系統程式語言。
所以讓我們聊聊它，確定你領會了這些細微差別。

`if` 是一個更通用的概念 "分支" (branch) 的特殊形式。
這個名字來自樹的分支：一個根據選擇而有不同路徑的決策點。

在 `if` 的情況下，一個選擇會導向兩個路徑：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
}
```

如果我們改變 `x` 為其他值，就不會印出那一行。
確切的說，如果 `if` 後面的表達式是 `true`，那麼區塊內的程式碼就會被執行。
如果是 `false`，那就不執行。

如果你想在 `false` 的情況下做點什麼，就使用 `else`：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
} else {
    println!("x is not five :(");
}
```

如果有超過一種情形，使用 `else if`：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
} else if x == 6 {
    println!("x is six!");
} else {
    println!("x is not five or six :(");
}
```

這些都是標準情況。
然而你也可以這樣做：

```rust
let x = 5;

let y = if x == 5 {
    10
} else {
    15
}; // y: i32
```

我們可以（且或許應該）寫成這樣：

```rust
let x = 5;

let y = if x == 5 { 10 } else { 15 }; // y: i32
```

這是可行的，因為 `if` 是個表達式。
表達式的值就是任何一個被選擇的分支的最後一個表達式的值。
一個沒有 `else` 的 `if` 總是會回傳 `()` 值。


> *commit 024aa9a*
