# 枚舉

Rust 中的枚舉 `enum` 是可以表現多種可能變體之一的資料的型別。
每個 `enum` 中的變體都可以選擇是否要與資料有關聯：

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}
```

> 譯註：`Quit` 是沒有資料的變體，`ChangeColor(i32, i32, i32)` 與 `Write(String)` 是有著沒名稱的資料的變體，`Move { x: i32, y: i32 }` 是著有名稱資料的變體。

定義變體的語法跟定義結構體 (struct) 的語法類似：你可以定義沒有資料的變體（與類單元結構體類似）、有名稱的資料的變體、或沒有名稱的資料的變體（與多元組結構體類似）。
然而，與各別結構體的定義不同，`enum` 是單一的型別。
一個枚舉的值可以是任何可能的變量之一。
也因為這個原因，枚舉有時候會被稱為 "集合型別" (sum type)：枚舉可能值的集合，是所有變體可能值的集合的總和。

我們使用 `::` 語法去使用各個變體的名稱：變體的有效範圍存在於 `enum` 自身的命名中。
因此以下程式碼可以運作：

```rust
# enum Message {
#     Move { x: i32, y: i32 },
# }
let x: Message = Message::Move { x: 3, y: 4 };

enum BoardGameTurn {
    Move { squares: i32 },
    Pass,
}

let y: BoardGameTurn = BoardGameTurn::Move { squares: 1 };
```

兩個變體都名為 `Move`，但因為它們的有效範圍都各自存在於枚舉的命名中，因此它們可以毫無衝突的使用。

`enum` 型別的值包含了它是哪個變體的資訊，以及任何與變體有關的資料。
有時候會被當成 "tagged union"，因為資料中包含了 "tag" 去標明它的型別。
編譯器會使用這些資訊去確保你能安全的存取枚舉中的資料。
例如，你不能像解構一個值一樣簡單的解構枚舉中可能變體之一的值：

> 譯註：因為枚舉中可能的變體不一定恰好是你所想像中的變體。

```rust,ignore
fn process_color_change(msg: Message) {
    let Message::ChangeColor(r, g, b) = msg; // compile-time error
}
```

不支援這些操作可能看起來像是限制，但我們可以克服這些限制。
有兩種方式：透過自己實作等式 (equality)、或藉由下一節會學到的 [match][match] 表達式去配對模式 (pattern) 與變體。
我們對 Rust 還所知不多，所以不知道如何實作等式，但我們會在 [traits][traits] 一節學到。

[match]: match.html
[traits]: traits.html

## 建構子當作函式

`enum` 的建構子 (constructor) 也可以像函式一樣使用。
例如：

```rust
# enum Message {
# Write(String),
# }
let m = Message::Write("Hello, world".to_string());
```

這與以下程式碼一樣：

```rust
# enum Message {
# Write(String),
# }
fn foo(x: String) -> Message {
    Message::Write(x)
}

let x = foo("Hello, world".to_string());
```

這現在對我們來說沒什麼用，但當我們談到 [closures][closures] 時，我們將會談到將函式作為參數傳遞到其他函式中。
舉例來說，使用[疊代器][iterators] (iterators)，我們可以像這樣把 `String` 的向量 (vector) 轉換為 `Message::Write` 的向量：

```rust
# enum Message {
# Write(String),
# }

let v = vec!["Hello".to_string(), "World".to_string()];

let v1: Vec<Message> = v.into_iter().map(Message::Write).collect();
```

[closures]: closures.html
[iterators]: iterators.html


> *commit 31e39cd*
