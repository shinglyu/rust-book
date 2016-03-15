# 生命週期

本指南是當前 Rust 的三個所有權系統之一。
這是 Rust 最獨特且引人注目的功能之一，作為 Rust 的開發者應該對此要有相當的了解。
所有權是 Rust 用來達成其最大的目標，記憶體安全，的方法。
它有一些不同的概念，各自有各自的章節：

* [所有權][ownership] (ownership)，關鍵的概念
* [借用][borrowing] (borrowing)，及其相關功能 "參照" (references)
* 生命週期 (lifetimes)，你正在閱讀的章節

這三章依序相關。
你需要了解全部三章來完整了解所有權系統。

[ownership]: ownership.html
[borrowing]: references-and-borrowing.html

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

記住這些之後，讓我們開始學習生命週期。

## 生命週期 (Lifetimes)

出借一個指向他人擁有的資源的參照是很複雜的。
舉例來說，想像一下以下的操作行為：

1. 我獲得一個某種資源的控制代碼 (handle)。
2. 我把指向這個資源的參照借給你。
3. 我決定不再使用這個資源，釋放它，但你仍還有你那邊的參照。
4. 你決定要使用此資源。

噢！你的參照指向了一個無效的資源。
如果資源是記憶體時，我們叫它迷途指標 (dangling pointer) 或 "釋放後的使用"。

要修正這個問題，我們必須確保上述第四步不會在第三步之後發生。
Rust 的所有權系統透過一個叫做生命週期 (lifetimes) 的概念達成這點，它會描述參照的有效範圍。

當我們有一個函式透過參數取用了一個參照，我們可以不言明 (implicit) 或言明 (explicit) 參照的生命週期：

```rust
// implicit
fn foo(x: &i32) {
}

// explicit
fn bar<'a>(x: &'a i32) {
}
```

`'a` 唸作 "生命週期 a"。
技術上，所有參照都有關聯的生命週期，但是編譯器通常讓你可以省略它（參考後述的 ["生命週期省略"][lifetime-elision]）。
在我們講到省略之前，先讓我們拆解言明生命週期的範例：

[lifetime-elision]: #生命週期省略%20(Lifetime%20Elision)

```rust,ignore
fn bar<'a>(...)
```

我們之前談到了一些[函式語法][functions]，但我們沒有討論到函式後面的 `<>`。
一個函式可以在 `<>` 間放入 "泛型參數" (generic parameters)，生命週期也是其中一種。
我們[在之後][generics]會討論到其他的泛型，但現在讓我們專注在生命週期方面。

[functions]: functions.html
[generics]: generics.html

我們使用 `<>` 去宣告我們的生命週期。
這代表 `bar` 有一個生命週期 `'a`。
如果我們有兩個參照參數，那麼將會看起來像：


```rust,ignore
fn bar<'a, 'b>(...)
```

然後在我們的參數列表中，我們使用我們所命名的生命週期：

```rust,ignore
...(x: &'a i32)
```

如果我們想要一個 `&mut` 參照，這樣做：
I
```rust,ignore
...(x: &'a mut i32)
```

如果你比較 `&mut i32` 和 `&'a mut i32` 兩者，除了在 `&` 和 `mut i32` 間多出生命週期 `'a`，你會發現他們是一樣的。
我們把 `&mut i32` 叫做 "一個 `i32` 可變參照"，而 `&'a mut i32` 叫做 "一個生命週期為 `'a` 的 `i32` 可變參照"

## 在 `struct` 中

當處理含有參照的[結構體][structs] (struct) 食，你也同樣需要言明生命週期：

```rust
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let y = &5; // this is the same as `let _y = 5; let y = &_y;`
    let f = Foo { x: y };

    println!("{}", f.x);
}
```

[structs]: structs.html

如你所見，`struct` 也有生命週期。
它的表示方式跟函式很像：

```rust
struct Foo<'a> {
# x: &'a i32,
# }
```

宣告生命週期，接著：

```rust
# struct Foo<'a> {
x: &'a i32,
# }
```

使用生命週期。
那為何我們在此處需要生命週期呢？
因為我們必須確保任何參考到 `Foo` 的參照不能存活的比 `Foo` 所內含的 `i32` 參照還久。

> 譯註：簡單說，如果 `Foo` 內的 `i32` 參照失效後，如果仍有指向 `Foo` 的參照，取用此參照內的 `i32` 參照就會出問題。

### `impl` 區塊

讓我們替 `Foo` 實作一個方法：

```rust
struct Foo<'a> {
    x: &'a i32,
}

impl<'a> Foo<'a> {
    fn x(&self) -> &'a i32 { self.x }
}

fn main() {
    let y = &5; // this is the same as `let _y = 5; let y = &_y;`
    let f = Foo { x: y };

    println!("x is: {}", f.x());
}
```

如你所見，我們需要在 `impl` 那行宣告 `Foo` 的生命週期。
我們重複了 `'a` 兩次，跟函式一樣：`impl<'a>` 定義了生命週期 `'a`，而 `Foo<'a>` 則使用它。

### 多重生命週期

如果你有多個參照，你可以重複使用相同的生命週期：

```rust
fn x_or_y<'a>(x: &'a str, y: &'a str) -> &'a str {
#    x
# }
```

這代表 `x` 和 `y` 都存活在同樣的有效範圍內，且回傳值也存活在同樣的有效範圍。
如果你想要讓 `x` 和 `y` 有不同的生命週期，你可以使用多個生命週期參數：

```rust
fn x_or_y<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {
#    x
# }
```

在此範例中，`x` 與 `y` 有不同的有效範圍，但回傳值的生命週期與 `x` 相同。

### 對有效範圍的深思 (Thinking in scopes)

瞭解生命週期的一個方式是將參照的有效範圍視覺化。
例如：

```rust
fn main() {
    let y = &5;     // -+ y goes into scope
                    //  |
    // stuff        //  |
                    //  |
}                   // -+ y goes out of scope
```

加入 `Foo`：

```rust
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let y = &5;           // -+ y goes into scope
    let f = Foo { x: y }; // -+ f goes into scope
    // stuff              //  |
                          //  |
}                         // -+ f and y go out of scope
```

`f` 存活在 `y` 的有效範圍內，所以一切運作正常。
但如果不是（在有效範圍內）呢？
以下程式碼無法運作：

```rust,ignore
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let x;                    // -+ x goes into scope
                              //  |
    {                         //  |
        let y = &5;           // ---+ y goes into scope
        let f = Foo { x: y }; // ---+ f goes into scope
        x = &f.x;             //  | | error here
    }                         // ---+ f and y go out of scope
                              //  |
    println!("{}", x);        //  |
}                             // -+ x goes out of scope
```

唷！如你所見，`f` 和 `y` 的有效範圍比 `x` 的有效範圍小。
但是當我們執行 `x = &f.x` 時，我們會使 `x` 參考到已經離開有效範圍的東西。

命名生命週期是給予有效範圍名稱的方式。
賦予東西一個名稱，是開始討論它的第一步。

### 'static

名稱叫做 `static` 的生命週期是特殊的生命週期。
它代表了某樣東西具有整個程式的生命週期。
大多數 Rust 程式設計師會在處理字串時第一次遇到 `'static`：

```rust
let x: &'static str = "Hello, world.";
```

字串是 `&'static str` 型別，因為參照一直存活著：他們被放在最後的執行檔內的資料區段。
另一個例子是全域變數：

```rust
static FOO: i32 = 5;
let x: &'static i32 = &FOO;
```

這會加入一個 `i32` 到執行檔的資料區段，`x` 則是指向它的參照。

### 生命週期省略 (Lifetime Elision)

Rust 在函式中支援強大的局部型別推斷 (local type inference)，但在 item signatures 中則被禁止，才能讓它根據簽署去推論其型別。
然而，為了人機工程的理由，第二種受限的推斷方式 "生命週期省略" 被允許用在函式的簽署。
它只根據簽署元件本身推斷，而不基於函式內容，而且只根據三個簡單易記、不易混淆的規則推斷生命週期參數。
這使得撰寫 item signatures 時可以速寫生命週期省略，但不能隱藏它的實際型別，因為局部型別推斷時可能會需要它。

> 譯註：`item signatures` 應是指宣告函式時的輸入參數們，以及回傳值的宣告。

當我們談到生命週期省略，我們使用 *輸入生命週期* (input lifetime) 與 *輸出生命週期* (output lifetime) 的字眼。
*輸入生命週期* 是指函式的參數的生命週期，而 *輸出生命週期* 是指函式回傳值的生命週期。
例如，以下函式有一個輸入生命週期：

```rust,ignore
fn foo<'a>(bar: &'a str)
```

此函式則有一個輸出生命週期：

```rust,ignore
fn foo<'a>() -> &'a str
```

此一函式兩者皆有：

```rust,ignore
fn foo<'a>(bar: &'a str) -> &'a str
```

以下是三條規則：

* 每個函式的參數所省略的生命週期都成為個別不同的生命週期參數。
* 如果恰好只有一個輸入生命週期，無論省略與否，其生命週期會被賦予所有回傳值中所省略的生命週期。
* 如果有多個輸入生命週期，但其中有 `&self` 或 `&mut self`，那 `self` 的生命週期會被賦予所有省略的生命週期。

否則，省略輸入生命週期會出現錯誤訊息。

#### 範例

以下有一些省略生命週期的函式範例。
我們把省略的範例與其展開生命週期的樣子，各別放在一起比對。

```rust,ignore
fn print(s: &str); // elided
fn print<'a>(s: &'a str); // expanded

fn debug(lvl: u32, s: &str); // elided
fn debug<'a>(lvl: u32, s: &'a str); // expanded

// In the preceding example, `lvl` doesn’t need a lifetime because it’s not a
// reference (`&`). Only things relating to references (such as a `struct`
// which contains a reference) need lifetimes.

fn substr(s: &str, until: u32) -> &str; // elided
fn substr<'a>(s: &'a str, until: u32) -> &'a str; // expanded

fn get_str() -> &str; // ILLEGAL, no inputs

fn frob(s: &str, t: &str) -> &str; // ILLEGAL, two inputs
fn frob<'a, 'b>(s: &'a str, t: &'b str) -> &str; // Expanded: Output lifetime is ambiguous

fn get_mut(&mut self) -> &mut T; // elided
fn get_mut<'a>(&'a mut self) -> &'a mut T; // expanded

fn args<T: ToCStr>(&mut self, args: &[T]) -> &mut Command; // elided
fn args<'a, 'b, T: ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command; // expanded

fn new(buf: &mut [u8]) -> BufWriter; // elided
fn new<'a>(buf: &'a mut [u8]) -> BufWriter<'a>; // expanded
```


> *commit f4fac9b*
