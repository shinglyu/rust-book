# 所有權

本指南是當前 Rust 的三個所有權系統之一。
這是 Rust 最獨特且引人注目的功能之一，作為 Rust 的開發者應該對此要有相當的了解。
所有權是 Rust 用來達成其最大的目標，記憶體安全，的方法。
它有一些不同的概念，各自有各自的章節：

* 所有權 (ownership)，你正在閱讀的章節
* [借用][borrowing] (borrowing)，及其相關功能 "參照" (references)
* [生命週期][lifetimes] (lifetime)，借用的進階概念

這三章依序相關。
你需要了解全部三章來完整了解所有權系統。

[borrowing]: references-and-borrowing.html
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

記住這些之後，讓我們開始學習所有權。

## 所有權 (Ownership)

[變數綁定][bindings]在 Rust 中個屬性：它們有所綁定的值的 "所有權"。
這代表當綁定離開有效範圍，Rust 就會釋放綁定的資源。
例如：

```rust
fn foo() {
    let v = vec![1, 2, 3];
}
```

當 `v` 進入有效範圍時，一個新的[向量][vectors] (vector) 會在[堆疊][stack] (stack) 中被建立，且在[堆積][heap] (heap) 中替其元素配置空間。
當 `v` 在 `foo()` 結尾離開有效範圍時，Rust 會清除任何與這個向量有關的東西，甚至是堆積內配置的記憶體。
這在有效範圍結束後必定發生。

我們會在後面章節詳細說明[向量][vectors]；在此我們只用它來作為一個在執行期配置堆積中空間的例子。
它的行為跟[陣列][arrays]類似，除了它的大小可以透過 `push()` 加入元素而改變這點。

向量有個[泛型][generics] `Vec<T>`，在本範例 `v` 是 `Vec<i32>` 型別。
我們將會在本章後面提及泛型。

[arrays]: primitive-types.html#arrays
[vectors]: vectors.html
[heap]: the-stack-and-the-heap.html
[stack]: the-stack-and-the-heap.html#the-stack
[bindings]: variable-bindings.html
[generics]: generics.html

## 移動語意 (Move semantics)

此處是更加精妙的部分：Rust 確保所有的資源都 _只有一個_ 對應的綁定。
例如，如果我們有個向量，我們可以賦值給另一個綁定：

```rust
let v = vec![1, 2, 3];

let v2 = v;
```

但是，當我們在之後試著使用 `v` 時，會得到錯誤訊息：

```rust,ignore
let v = vec![1, 2, 3];

let v2 = v;

println!("v[0] is: {}", v[0]);
```

看起來像這樣：

```text
error: use of moved value: `v`
println!("v[0] is: {}", v[0]);
                        ^
```

當我們定義一個取得所有權的函式，且試著在傳遞參數之後使用同個參數時，會發生類似的事：

```rust,ignore
fn take(v: Vec<i32>) {
    // what happens here isn’t important.
}

let v = vec![1, 2, 3];

take(v);

println!("v[0] is: {}", v[0]);
```

同樣的錯誤： "use of moved value"。
當我們轉移所有權給其他綁定後，我們稱此為 "移動" 了所引用的值。
這是 Rust 的預設行為，你不需特別註記什麼。

### 細節

在移動綁定後我們無法使用它的原因很精妙，也很重要。

當我們寫了以下程式碼：

```rust
let x = 10;
```

Rust 替 [i32] 配置記憶體在[堆疊][sh]上，複製代表 10 的值的位元到配置的記憶體中，且綁定變數名稱 `x` 到此區記憶體以便未來使用。

現在細想以下程式碼片段：

```rust
let v = vec![1, 2, 3];

let mut v2 = v;
```

第一行就如同上述的 `x` 一樣，它替向量物件 `v` 配置記憶體到堆疊上。
但它還配置一些[堆積][sh]上的記憶體放實際的資料（`[1, 2, 3]`）。
Rust 複製堆積配置的記憶體位址給內部指標，這個內部指標是向量物件存在堆疊上的部份（讓我們稱它資料指標 data pointer）。

值得說明的是，向量物件和它的資料分別放在不同的記憶體區塊，而不在單一連續的記憶體配置中（因為一些理由我們不會細說）。
向量的這兩部分（一部份在堆疊、一部份在堆積）必須在任何時候都與對方一致，像長度、容量等。

當我們移動 `v` 到 `v2` 時，Rust 實際上把向量物件 `v` 照著位元複製一份到堆疊配置中的 `v2`。
這份複本並沒有建立堆積配置中實際資料的複本。
這代表會同時有指向向量內容的兩個指標，兩者都指向堆積中的同一塊記憶體配置。
如果同時可以存取 `v` 跟 `v2` 將會有資料競爭 (data race)，這會違背 Rust 的安全保證。

> 譯註：當 `let mut v2 = v` 時，只是把 `v` 中的資料指標複製一份給 `v2`，所以 `v` 與 `v2` 同時指向堆積中的同一份資料。
> 當兩者同時存取時就會出現資料競爭的可能性。

舉例來說，如果我們從 `v2` 截去向量中的兩個元素：

```rust
# let v = vec![1, 2, 3];
# let mut v2 = v;
v2.truncate(2);
```

而 `v` 仍可存取，因此我們最終將得到一個非法的向量，因為 `v` 不知道堆積資料被截去了。
現在，`v` 向量在堆疊上的部份與堆積上的部分並不一致。
`v` 仍認為有三個元素在向量中，且仍非常歡迎我們存取已不存在的元素 `v[2]`，但就如你早已知道的，這將導致後患無窮。
特別是這可能還會引起記憶體區段錯誤 (segmentation fault)，甚至更糟的情況會允許未經認證的使用者從記憶體中讀取不應存取的東西。

> 譯註：此處原文為 `v1`，修正為 `v`。

這也是為何 Rust 禁止在我們移動綁定之後使用 `v`。

[i32]: primitive-types.html#數字型別
[sh]: the-stack-and-the-heap.html

另外也很重要的是，依照情況，最佳化可能會移除堆疊上位元的實際複本。
所以它可能不像一開始看起來的那麼沒效率。

### `Copy` 型別

我們已經知道當所有權轉移到另一個綁定後，你不能再使用原來的綁定。
然而，有一個 [trait][traits] 可以改變這個行為，它稱為 `Copy`。
我們還沒說到 traits，但現在，你可以把它當成替特定型別加上額外行為的一種註釋。
例如：

```rust
let v = 1;

let v2 = v;

println!("v is: {}", v);
```

在此情況下，`v` 是個實作 `Copy` trait 的 `i32`。
這代表，跟移動一樣，當我們把 `v` 賦值給 `v2` 時，一個資料的複本會被建立。
但跟移動不同的是，在之後我們仍可使用 `v`。
這是因為 `i32` 並沒有指標指向其他資料，複製時就是個完整的複本。

所有的基本型別都實作了 `Copy` trait，也因此它們的所有權並不會像 "所有權規則" 所假設的那樣被移動。
以下兩個程式碼片段的範例都可以編譯，因為 `i32` 與 `bool` 型別都實作了 `Copy` trait。

```rust
fn main() {
    let a = 5;

    let _y = double(a);
    println!("{}", a);
}

fn double(x: i32) -> i32 {
    x * 2
}
```

```rust
fn main() {
    let a = true;

    let _y = change_truth(a);
    println!("{}", a);
}

fn change_truth(x: bool) -> bool {
    !x
}
```

如果我們使用沒有實作 `Copy` trait 的型別，我們將會得到編譯錯誤，因為我們試圖使用一個被移動的值。

```text
error: use of moved value: `a`
println!("{}", a);
               ^
```

我們將在 [traits][traits] 章節討論如何撰寫你自己型別的 `Copy`。

[traits]: traits.html

## 所有權以外 (More than ownership)

當然，如果我們必須在每個函式都交還所有權：

```rust
fn foo(v: Vec<i32>) -> Vec<i32> {
    // do stuff with v

    // hand back ownership
    v
}
```

這將會非常煩人。
當我們想處理更多所有權的時候會變得更糟：

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

呃！
回傳型別、回傳的那行程式碼、和呼叫函式都變得更複雜了。

幸運的是，Rust 提供一個功能，借用 (borrowing)，可以幫助我們解決這個問題。
這就是下一節的主題！


> *commit 145190b*
