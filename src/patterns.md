# 模式

模式 (pattern) 在 Rust 中很常見。
我們在[變數綁定][bindings]、[match 陳述式][match]、及其他地方都會用到。
讓我們開始快速地了解模式可以做到些什麼！

[bindings]: variable-bindings.html
[match]: match.html

快速複習一下：你可以直接配對變數，而 `_` 將會配對到 "任何" 沒列出來的其他情況：

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

這會印出 `one`。

模式有個陷阱：它會產生遮蔽 (shadowing)，與其他會產生新的綁定的任何東西一樣。
舉例來說：

```rust
let x = 1;
let c = 'c';

match c {
    x => println!("x: {} c: {}", x, c),
}

println!("x: {}", x)
```

這會印出：

```text
x: c c: c
x: 1
```

換句話說，`x =>` 符合模式，而且產生了新的名為 `x` 的綁定。
新綁定的有效範圍在該 match 的執行分支中，且它的值被賦予為 `c`。
請注意，在 match 有效範圍之外的 `x` 的值不會影響到範圍內的 `x`。
因為我們本來已經有了 `x` 綁定，而新的 `x` 遮蔽了它。

## 多重模式 (Multiple patterns)

你可以使用 `|` 來配對多重模式：

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

這會印出 `one or two`。

## 解構 (Destructuring)

如果你有個複合的資料型別，像是[結構體][struct] (struct)，你可以在模式中解構它：

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x, y } => println!("({},{})", x, y),
}
```

[struct]: structs.html

我們可以使用 `:` 給予值新的不同命名。

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x: x1, y: y1 } => println!("({},{})", x1, y1),
}
```

如果我們只在意某些值，我們不需要給它所有名稱：

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

這會印出 `x is 0`。

你可以對任何成員進行這樣的配對，而不僅限於第一個：

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { y, .. } => println!("y is {}", y),
}
```

這會印出 `y is 0`。

這種 "解構" 行為在任何複合資料型別上都可以運作，像[多元組][tuples] (tuple) 或[枚舉][enums] (enum) 都可以。

[tuples]: primitive-types.html#多元組%20(Tuples)
[enums]: enums.html

## 忽略綁定

你可以在模式中使用 `_` 忽略型別跟值。
舉例來說，這是一個 `Result<T, E>` 的 `match` 例子：

```rust
# let some_value: Result<i32, &'static str> = Err("There was an error");
match some_value {
    Ok(value) => println!("got a value: {}", value),
    Err(_) => println!("an error occurred"),
}
```

在第一個執行分支中，我們把 `OK` 內的值綁定到 `value` 上。
但是在 `Err` 分支，我們使用 `_` 來忽略特定的錯誤，印出通用的錯誤訊息。

`_` 在任何建立綁定的模式中都有效。
這在大型結構體中要忽略某些部份時也非常有用：

```rust
fn coordinate() -> (i32, i32, i32) {
    // generate and return some sort of triple tuple
# (1, 2, 3)
}

let (x, _, z) = coordinate();
```

在這邊，我們把多元組中的第一個跟最後一個元素綁定到 `x` 跟 `z`，但是忽略了中間的元素。

值得注意的是，`_` 並不會綁定變數，這代表值不會移動所有權：

> 譯註：但如果給了變數名稱，沒有實作 Copy 的元素就會把所有權移到該變數上，因此使用原來的綁定時會出現 `error: use of moved value` 錯誤。
> 在以下例子中，`tuple.0` 因為是 `u32` 有實作 Copy trait，所以仍可以使用，只有 `tuple.1` 是 `String`，因為給了命名，所以所有權被轉移了（也就是移動了值）。

```rust
let tuple: (u32, String) = (5, String::from("five"));

// Here, tuple is moved, because the String moved:
let (x, _s) = tuple;

// The next line would give "error: use of partially moved value: `tuple`"
// println!("Tuple is: {:?}", tuple);

// However,

let tuple = (5, String::from("five"));

// Here, tuple is _not_ moved, as the String was never moved, and u32 is Copy:
let (x, _) = tuple;

// That means this works:
println!("Tuple is: {:?}", tuple);
```

這也代表任何暫時的變數會在陳述式結束之後被丟棄：

```rust
// Here, the String created will be dropped immediately, as it’s not bound:

let _ = String::from("  hello  ").trim();
```

你也可以在模式中使用 `..` 去忽略多個值：

```rust
enum OptionalTuple {
    Value(i32, i32, i32),
    Missing,
}

let x = OptionalTuple::Value(5, -2, 3);

match x {
    OptionalTuple::Value(..) => println!("Got a tuple!"),
    OptionalTuple::Missing => println!("No such luck."),
}
```

這會印出 `Got a tuple!`。

## ref 與 ref mut

如果你想要獲得一個[參照][ref] (reference)，可以使用 `ref` 關鍵字：

```rust
let x = 5;

match x {
    ref r => println!("Got a reference to {}", r),
}
```

這會印出 `Got a reference to 5`。

[ref]: references-and-borrowing.html

這邊 `match` 中的 `r` 是一個 `&i32` 型別。
換句話說，`ref` 關鍵字 _建立_ 了一個可在模式當中使用的參照。
如果你需要一個可變的 (mutable) 參照，可以使用 `ref mut`：

```rust
let mut x = 5;

match x {
    ref mut mr => println!("Got a mutable reference to {}", mr),
}
```

## 範圍

你可以用 `...` 配對一個範圍的值：

```rust
let x = 1;

match x {
    1 ... 5 => println!("one through five"),
    _ => println!("anything"),
}
```

這會印出 `one through five`。

範圍通常用在整數和 `char` 上：

```rust
let x = '💅';

match x {
    'a' ... 'j' => println!("early letter"),
    'k' ... 'z' => println!("late letter"),
    _ => println!("something else"),
}
```

這會印出 `something else`。

## 綁定

你可以使用 `@` 把值綁定在命名上：

```rust
let x = 1;

match x {
    e @ 1 ... 5 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

這會印出 `got a range element 1`。
當你想對資料結構中的部分進行複雜的配對時，這十分有用：

```rust
#[derive(Debug)]
struct Person {
    name: Option<String>,
}

let name = "Steve".to_string();
let x: Option<Person> = Some(Person { name: Some(name) });
match x {
    Some(Person { name: ref a @ Some(_), .. }) => println!("{:?}", a),
    _ => {}
}
```

這會印出 `Some("Steve")`：因為我們把內部的 `name` 綁定到 `a` 了。

如果你將 `@` 與 `|` 一起使用，你需要確保在模式的每一部份都有綁定命名：

```rust
let x = 5;

match x {
    e @ 1 ... 5 | e @ 8 ... 10 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

## 守衛 (Guards)

你可以使用 `if` 來產生 "配對守衛" (match guards)：

```rust
enum OptionalInt {
    Value(i32),
    Missing,
}

let x = OptionalInt::Value(5);

match x {
    OptionalInt::Value(i) if i > 5 => println!("Got an int bigger than five!"),
    OptionalInt::Value(..) => println!("Got an int!"),
    OptionalInt::Missing => println!("No such luck."),
}
```

這會印出 `Got an int!`。

如果你在多重模式中使用 `if`，那 `if` 將會套用在多重模式的所有模式上：

```rust
let x = 4;
let y = false;

match x {
    4 | 5 if y => println!("yes"),
    _ => println!("no"),
}
```

這會印出 `no`，因為 `if` 套用在整個 `4 | 5` 之上，而不只是 `5`。
換句話說，`if` 行為的優先權就像是：

```text
(4 | 5) if y => ...
```

而不是：

```text
4 | (5 if y) => ...
```

## 混合與配對 (Mix and Match)


恩！有非常多種方法可以進行配對，而且它們可以根據你想做的事情而組合使用：

```rust,ignore
match x {
    Foo { x: Some(ref name), y: None } => ...
}
```

模式非常強大。
好好善用它們。


> *commit b49ce1a*
