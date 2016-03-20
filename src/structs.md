# 結構體

結構體 `struct` 是一種建立更複雜資料型別的方法。
舉例來說，如果我們要進行 2D 空間中的座標計算，我們需要 `x` 和 `y` 兩者：

```rust
let origin_x = 0;
let origin_y = 0;
```

而 `struct` 讓我們可以將兩者合而為一，成為一個有 `x` 及 `y` 欄位 (field) 的統一資料型態：

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let origin = Point { x: 0, y: 0 }; // origin: Point

    println!("The origin is at ({}, {})", origin.x, origin.y);
}
```

這邊有許多東西，讓我們分開來解說。
我們使用 `struct` 關鍵字及名字來宣告結構體。
照慣例，`struct` 的命名用大寫開始，且使用駝峰式大小寫 (camel case) 規則：`PointInSpace`，而非 `Point_In_Space`。

我們可以透過 `let` 宣告我們 `struct` 的實體，但我們會使用 `key: value` 風格的語法去設定每一個欄位。
順序不需與原始宣告中的排序相同。

最後，因為欄位都有名字，我們可以透過點記號存取它們：`origin.x`。

跟 Rust 中的其他綁定一樣，`struct` 中的值預設是不可變的。
可以使用 `mut` 讓它成為可變的：

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let mut point = Point { x: 0, y: 0 };

    point.x = 5;

    println!("The point is at ({}, {})", point.x, point.y);
}
```

這將會印出 `The point is at (5, 0)`。

Rust 不支援欄位的可變性，所以你不能寫成這樣：

```rust,ignore
struct Point {
    mut x: i32,
    y: i32,
}
```

可變性是綁定的一種屬性，而不是結構體本身。
如果你習慣使用欄位等級的可變性，以下的方式可能一開始看起來會有點奇怪，但它明顯的簡化了問題。
它甚至讓你可以只暫時的可變一下：

> 譯註：也就是使用 `let mut` 宣告成可變綁定，更動其中的欄位。不想更動時，使用 `let foo = foo;` 使之成為不可變的。

```rust,ignore
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let mut point = Point { x: 0, y: 0 };

    point.x = 5;

    let point = point; // now immutable

    point.y = 6; // this causes an error
}
```

你的結構體仍然可以包含 `&mut` 指標，這讓你可以有某些可變性：

```rust
struct Point {
    x: i32,
    y: i32,
}

struct PointRef<'a> {
    x: &'a mut i32,
    y: &'a mut i32,
}

fn main() {
    let mut point = Point { x: 0, y: 0 };

    {
        let r = PointRef { x: &mut point.x, y: &mut point.y };

        *r.x = 5;
        *r.y = 6;
    }

    assert_eq!(5, point.x);
    assert_eq!(6, point.y);
}
```

## 更新語法

`struct` 可以使用 `..` 來表示你想使用其他 `struct` 中某些值的複本。
舉例來說：

```rust
struct Point3d {
    x: i32,
    y: i32,
    z: i32,
}

let mut point = Point3d { x: 0, y: 0, z: 0 };
point = Point3d { y: 1, .. point };
```

這邊給了 `point` 一個新的 `y` 值，但是會保留原來舊的 `x` 和 `z` 值。
這不一定要在同一個 `struct` 操作，當你要建立新的一個結構體時，同樣可以使用這個語法，而且它將會把你沒有指明的值複製過來：

```rust
# struct Point3d {
#     x: i32,
#     y: i32,
#     z: i32,
# }
let origin = Point3d { x: 0, y: 0, z: 0 };
let point = Point3d { z: 1, x: 2, .. origin };
```

## 多元組結構體 (tuple struct)

Rust 還有其他類似[多元組][tuple] (tuple) 與 `struct` 混合體的資料型別，叫做 "多元組結構體" (tuple struct)。
多元組結構體本身有命名，但是它的欄位沒有。
它也使用 `struct` 關鍵字宣告，在其後加上名字和多元組：

[tuple]: primitive-types.html#多元組%20(Tuples)

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

在這邊，`black` 與 `origin` 即使有著相同的值，但它們並不相等。

使用結構體大多時候都比使用多元組結構體要來得好。
我們可以把 `Color` 及 `Point` 重寫成這樣：

```rust
struct Color {
    red: i32,
    blue: i32,
    green: i32,
}

struct Point {
    x: i32,
    y: i32,
    z: i32,
}
```

好的命名是很重要的，雖然多元組結構體 (tuple struct) 也能使用點記號去參考它的值，但結構體 (struct) 使用的是真實的命名而不是位置。

> 譯註：多元組的存取是使用索引 `tuple.0`，而結構體可以使用欄位的名稱，比較具有意義，在程式碼的可讀性上會比較強。

但有 _一種情況_ 下多元組結構體非常有用，就是當他只有一個元素時。
我們稱它為 "新型別" (newtype) 模式，因為它允許你建立與內含的值有所區別的新型別，同時又表達出我們所想表達的語意：

```rust
struct Inches(i32);

let length = Inches(10);

let Inches(integer_length) = length;
println!("length is {} inches", integer_length);
```

如你所見，你能透過 `let` 解構內部的整數型態，跟一般的多元組一樣。
在上述情況，`let Inches(integer_length)` 會把 `10` 賦值給 `integer_length`。

## 類單元結構體 (Unit-like structs)

你可以定義一個沒有任何成員的 `struct`：

```rust
struct Electron;

let x = Electron;
```

這樣的結構體被稱為 "類單元"，因為它類似空的多元組 `()`，有時候被稱為單元 (unit)。
跟多元組結構體一樣，它會定義新的型別。

它很難得會有用處（雖然有時候它能用來當作標記型別 (marker type) ），但與其他功能組合時，它可以變得有用。
例如，一個函式庫可能要你建立一個結構體去實作某個 [trait][trait] 來處理事件 (events)。
如果你並不需要再結構體中存入任何資料，你可以建立一個類單元結構體。

[trait]: traits.html


> *commit 6ba9520*
