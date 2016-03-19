# 可變性

可變性 (mutability)，就是改變事物的能力，它在 Rust 中的運作方式跟其他語言不太一樣。
第一個方面就是 Rust 的可變性並不是預設狀態：

```rust,ignore
let x = 5;
x = 6; // error!
```

我們可以透過 `mut` 關鍵字來使用可變性：

```rust
let mut x = 5;

x = 6; // no problem!
```

這是一個可變的 (mutable) [變數綁定][vb]。
當一個綁定是可變的，代表你被允許改變它所指向的內容。
所以在上述範例中，`x` 的值並沒有太多改變，只是從一個 `i32` 換到另一個而已。

[vb]: variable-bindings.html

如果你想改變綁定指向的東西，你需要[可變參照][mr] (mutable reference)：

```rust
let mut x = 5;
let y = &mut x;
```

[mr]: references-and-borrowing.html

`y` 是個指向一個可變參照的不可變 (immutable) 綁定，這代表你不能把 `y` 綁定到其他東西上（例如 `y = &mut z`），但你可以改變綁定到 `y` 上的變數的值（`*y = 5`）。
這有一些微妙的不同。

當然，如果你兩種都想要可變：

```rust
let mut x = 5;
let mut y = &mut x;
```

現在 `y` 可以綁到其他變數上，且它所參照的值也可以被改變。

很重要的是 `mut` 也是[模式][pattern] (pattern) 的一部份，所以你可以像這樣做：

```rust
let (mut x, y) = (5, 6);

fn foo(mut x: i32) {
# }
```

[pattern]: patterns.html

## 內部及外部可變性 (Interior vs. Exterior Mutability)

然而，當我們談及 Rust 中有些東西是 "不可變" 的同時，這不代表它不能真的被改變：我們說它有著 "外部可變性" (exterior mutability)。
以 [Arc&lt;T&gt;][arc] 為例：

```rust
use std::sync::Arc;

let x = Arc::new(5);
let y = x.clone();
```

[arc]: https://doc.rust-lang.org/std/sync/struct.Arc.html

當我們呼叫 `clone()` 時，`Arc<T>` 需要更新參照計數器。
沒錯，我們在此處沒有使用任何 `mut`，`x` 是個不可變綁定，而且我們沒有使用 `&mut 5` 或其他東西。
那麼是為什麼呢？

要明白這些，我們要回去看 Rust 的指導哲學的核心，記憶體的安全性，以及 Rust 用以保證其安全的機制，[所有權][ownership]系統，和更特別的[借用][borrowing]：

> 你可能有以下兩種借用的其中一種，但是不能同時使用兩者：
>
> * 一到多個對資源的參照（`&T`）
> * 唯一一個可變參照（`&mut T`）

[ownership]: ownership.html
[borrowing]: references-and-borrowing.html#borrowing

所以，這才是 "不可變性" (immutability) 的真正定義：當兩個參照指向同一個資源時安全嗎？
在 `Arc<T>` 的例子中，是的：改變完全被包含在結構自身當中。
與使用者無關。
因為這個原因，它會在 `clone()` 時提供 `&T`。
如果它提供 `&mut T`，那就會是個問題。

其他型別，像是 [`std::cell`][stdcell] 模組中的這個，有相反的屬性：內部可變性 (interior mutability)。
例如：

```rust
use std::cell::RefCell;

let x = RefCell::new(42);

let y = x.borrow_mut();
```

[stdcell]: https://doc.rust-lang.org/std/cell/index.html

RefCell 替其內部的東西提供 `&mut` 參照給 `borrow_mut()` 方法。
這難道不危險嗎？
如果我們這樣做：

```rust,ignore
use std::cell::RefCell;

let x = RefCell::new(42);

let y = x.borrow_mut();
let z = x.borrow_mut();
# (y, z);
```

實際上這會在執行時引發 panic。
這是 `RefCell` 做的事情：它在執行期強制使用 Rust 得借用規則，且在有違反規則時 `panic!`。
這使我們可以繞開 Rust 的可變性規則的另一面。
讓我們來說明它。

### 欄位等級的可變性 (Field-level mutability)

可變性，它不是借用（`&mut`）就是綁定（`let mut`）的一個屬性。
這代表你不可能有某些欄位可變、而某些欄位不可變的[結構體][struct] (struct)。

```rust,ignore
struct Point {
    x: i32,
    mut y: i32, // nope
}
```

結構體的可變性在於它的綁定：

```rust,ignore
struct Point {
    x: i32,
    y: i32,
}

let mut a = Point { x: 5, y: 6 };

a.x = 10;

let b = Point { x: 5, y: 6};

b.x = 10; // error: cannot assign to immutable field `b.x`
```

[struct]: structs.html

但是，透過使用 [Cell&lt;T&gt;][cell]，你可以模擬欄位等級的可變性：

```rust
use std::cell::Cell;

struct Point {
    x: i32,
    y: Cell<i32>,
}

let point = Point { x: 5, y: Cell::new(6) };

point.y.set(7);

println!("y: {:?}", point.y);
```

[cell]: https://doc.rust-lang.org/std/cell/struct.Cell.html

這會印出 `y: Cell { value: 7 }`。
我們成功的更新了 `y`。


> *commit 024aa9a*
