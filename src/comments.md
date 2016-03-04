# 註解

現在我們有了一些函式，開始學習註解是個好點子。
註解是讓你留給其他程式設計師的備註，可以幫忙解釋你的程式碼。
編譯器幾乎會直接忽略它們。

Rust 有兩種你需要注意的註解方式：*行內註解* (line comments) 與 *文件註解* (doc comments)。

> 譯註：以下是行內註解。

```rust
// Line comments are anything after ‘//’ and extend to the end of the line.

let x = 5; // this is also a line comment.

// If you have a long explanation for something, you can put line comments next
// to each other. Put a space between the // and your comment so that it’s
// more readable.
```

另一種註解是文件註解。
文件註解使用 `///` 而不是 `//`，而且支援 Markdown 標記：

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, add_one(5));
/// # fn add_one(x: i32) -> i32 {
/// #     x + 1
/// # }
/// ```
fn add_one(x: i32) -> i32 {
    x + 1
}
```

還有另一種文件註解 `//!`，用來註解包含它的項目（例如：crates、模組、或函式），而不是在其後的東西。
一般用在 crates 的 root 檔（lib.rs）或模組的 root（mod.rs）：

```
//! # The Rust Standard Library
//!
//! The Rust Standard Library provides the essential runtime
//! functionality for building portable Rust software.
```

當撰寫文件註解的時候，提供一些使用範例是非常非常有用的。
你會注意到我們用了一個新的巨集：`assert_eq!`。
它會比較兩個值，然後如果它們彼此不相等就會 `panic!`。
這在文件中非常有用。
還有一個巨集 `assert!`，如果傳給它 `false` 時會 `panic!`。

你可以使用 [rustdoc](documentation.html) 工具從文件註解去產生 HTML 文件，也可以執行程式碼範例當作測試。


> *commit 024aa9a*
