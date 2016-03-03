# 詞彙表

並非每位 Rustacean 都有系統程式設計或電腦科學的背景，所以我們加上了一些可能不易理解的名詞解釋。

## 抽象語法樹 (Abstract Syntax Tree)

當編譯器編譯程式時，它會做很多不同的事。
其中之一就是轉換你的程式文字內容變成 "抽象語法樹"，或稱為 "AST"。
這個樹展現了你的程式結構。
例如， `2 + 3` 會轉換成這樣的樹：

```text
  +
 / \
2   3
```

而 `2 + (3 * 4)` 則看起來像這樣：

```text
  +
 / \
2   *
   / \
  3   4
```

## 元數 (Arity)

元數是指函式或操作所需要的參數數量。

```rust
let x = (2, 3);
let y = (4, 6);
let z = (8, 2, 6);
```

在範例中，`x` 和 `y` 的元數是 2。
`z` 的元數是 3。

## 界線 (Bounds)

界線是型別或 [trait][traits] 的限制。
例如，如果界線在函式的參數，那麼傳遞給函式的參數型別必須遵守這樣的限制。

[traits]: traits.html

## 動態大小型別 (DST - Dynamically Sized Type)

靜態大小或 alignment 未知的型別。（[更多資訊][link]）

[link]: https://doc.rust-lang.org/nomicon/exotic-sizes.html#dynamically-sized-types-dsts

## 表達式 (Expression)

程式語言中，表達式是一組數值、常數、變數、運算子、及函式的組合，可以估出一個數值。
例如 `2 + (3 * 4)` 是一個回傳數值 14 的表達式。
值得注意的是，表達式可以有副作用。
例如，一個包含表達式的函式可能會執行一些動作，而不只是單純的回傳值。

## 表達式導向語言 (Expression-Oriented Language)

早期的程式語言中，[表達式][expression] and [陳述式][statement] 是兩種不同的語法類型：表達式有數值，而陳述式做事。
然而，之後的語言模糊了界線，允許表達是做事，而陳述式有數值。
在表達式導向語言中，（幾乎）所以陳述式都是表達式，且回傳值。
因此，這些表達陳述式自身可以組成更大的表達式。

[expression]: glossary.html#expression
[statement]: glossary.html#statement

## 陳述式 (Statement)

在程式語言中，陳述式是程式語言中最小的獨立元素，用來命令電腦執行動作。

> 譯註：關於表達式及陳述式，可參考維基百科的 [Expression][wiki:expression] 及 [Statement][wiki:statement] 條目。

[wiki:expression]: https://en.wikipedia.org/wiki/Expression_(computer_science)
[wiki:statement]: https://en.wikipedia.org/wiki/Statement_(computer_science)


> *commit 024aa9a*
