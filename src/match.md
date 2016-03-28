# Match

簡單的 [if/else][if] 往往是不夠用的，因為你可能會有兩個以上的可能性。
而且你的條件運算也可能會變得很複雜。
Rust 有個更強大的 `match` 關鍵字可以讓你取代複雜的 `if`/`else` 組合。
來看看吧：

```rust
let x = 5;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    4 => println!("four"),
    5 => println!("five"),
    _ => println!("something else"),
}
```

[if]: if.html

`match` 使用表達式，並根據它的值進行分支。
每一條執行的分支都是以 `val => expression` 的形式表示。
當值符合的時候，該分支的表達式就會被執行。
它會被稱為 `match` 是因為 `match` 是 "模式配對" (pattern matching) 的實作。
[模式][patterns] (pattern) 有一個單獨的章節會涵蓋所有可能的模式。

[patterns]: patterns.html

`match` 的許多優點之一是它的強制 "徹底檢查" (exhaustiveness checking)。
舉例來說，當我們刪除最後一條有著 `_` 的執行分支，編譯器將會給我們錯誤訊息：

```text
error: non-exhaustive patterns: `_` not covered
```

Rust 告訴我們，我們忘了一個值。
編譯器從 `x` 推斷出它可能是任何 32 位元正整數；從 1 到 2,147,483,647 都有可能。
而 `_` 就像是 "接住全部" (catch-all) _沒有_ 被特別列在 `match` 執行分支中的可能值。
在上述例子中可以看到，我們給 `match` 整數 1-5 的執行分支，當 `x` 是 6 或其他值的時候，就會被 `_` 接去執行。

`match` 也是表達式，這代表我們可以在 `let` 綁定的右邊使用它，或直接將它用在任何表達式可以使用的地方：

```rust
let x = 5;

let number = match x {
    1 => "one",
    2 => "two",
    3 => "three",
    4 => "four",
    5 => "five",
    _ => "something else",
};
```

有時候這是把東西從一個型別轉換為另一個型別的好方法；上面的例子是把整數轉換為 `String`。

## 枚舉的配對

另一個 `match` 的重要作用是處理枚舉 (enum) 的可能變體們：

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}

fn quit() { /* ... */ }
fn change_color(r: i32, g: i32, b: i32) { /* ... */ }
fn move_cursor(x: i32, y: i32) { /* ... */ }

fn process_message(msg: Message) {
    match msg {
        Message::Quit => quit(),
        Message::ChangeColor(r, g, b) => change_color(r, g, b),
        Message::Move { x: x, y: y } => move_cursor(x, y),
        Message::Write(s) => println!("{}", s),
    };
}
```

再次地，Rust 編譯器會徹底的檢查，它要求枚舉的所有變體都必須要有配對的執行分支。
如果你漏掉了一個，它會給你一個編譯期錯誤，除非你使用 `_`，或替所有可能的變體都提供執行分支。

與 `match` 之前的用法不同，你無法使用一般的 `if` 陳述式去達成一樣的功能。
但你可以使用 [if let][if-let] 陳述式，它可被視為 `match` 的簡略版。

[if-let]: if-let.html


> *commit fc4bb5f*
