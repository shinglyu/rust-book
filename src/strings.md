# 字串

字串是每個軟體工程師都應該精通的重要概念。 Rust 因為著重在系統程式，所以它的字串處理系統與其他的程式語言有些不同。 每當你有大小不固定的資料結構時，事情就會變得很複雜，不巧的是字串最好是一個可以改變大小的資料結構。 Rust 的字串與其他系統程式語言，例如 C，也有所不同。

讓我們深入的研究字串。 一個字串是一個編碼成 UTF-8 串流的萬國碼 (Unicode) 數值序列。所有的字串都保證是用合法的 UTF-8 編碼。 此外不同於其他的系統程式語言，字串並非由空字串結尾 (null-terminated) 而且可以含有空字串位元。

Rust 有兩種主要的字串類型: `&str` 和 `String`。 讓我們先講講 `&str`。 他們叫做字串切片 (string slices) 。一個字串切片有固定的大小，而且是不可變的。他是一個指向 UTF-8 位元序列的參照。

```rust
let greeting = "Hello there."; // greeting: &'static str
```

`"Hello there."`是一個字串常值 (string literial)，其型態是`&'static str`。一個字串常值是一個靜態分配的字串切片，也就是說它儲存在我們編譯過的程式內，而且在整個運行的過程中都存在。 `greeting` 綁定是一個知道這個靜態字串的參照。任何一個接受字串切片的函數也同樣會接受字串常值。

字串常值可以橫跨多行。 有兩種方法來表示多行的字串。第一種會包含換行符號還有每行開頭的空白：

```rust
let s = "foo
    bar";

assert_eq!("foo\n        bar", s);
```

第二種使用 `\` 符號，會把空白與換行符號移除：

```rust
let s = "foo\
    bar";

assert_eq!("foobar", s);
```

但是 Rust 不只有 `&str`。一個 `String`是分配在堆積上的一種字串。這種字串可以長大，而且也保證是 UTF-8 編碼. `String` 的產生方式常常是呼叫字串切片的 `to_string` 函式轉換而來。
```rust
let mut s = "Hello".to_string(); // mut s: String
println!("{}", s);

s.push_str(", world.");
println!("{}", s);
```

使用 `&` 符號會把 `String`  強制轉換成 `&str`：

```rust
fn takes_slice(slice: &str) {
    println!("Got: {}", slice);
}

fn main() {
    let s = "Hello".to_string();
    takes_slice(&s);
}
```
當一個函數接受 `&str` 的一種特徵 (trait) 作為參數而不是單純的 `&str` 時，這種自動轉換並不會發生。例如， [`TcpStream::connect`][connect] 有一個型態為`ToSocketAddrs` 的參數。這時傳入 `&str` 是可以的，但是 `String` 就必須要明確的使用 `&*` 轉換後才能傳入。

```rust,no_run
use std::net::TcpStream;

TcpStream::connect("192.168.0.1:3000"); // &str parameter

let addr_string = "192.168.0.1:3000".to_string();
TcpStream::connect(&*addr_string); // convert addr_string to &str
```

把一個 `String` 轉換成 `&str` 來讀取不需要什麼成本的，但是把 `&str` 轉換為一個 `String` 就必須牽涉到分配記憶體。若是沒有強烈的理由千萬不要這樣做！

## 索引 

因為字串是 UTF-8 串流，所以並不支援使用索引來存取：

```rust,ignore
let s = "hello";

println!("The first letter of s is {}", s[0]); // ERROR!!!
```

通常使用 `[]` 來存取向量是非常快的。但是因為字串是使用 UTF-8 編碼，每個字符可能有多個位元組，要找到第 N 個字符必須要從頭開始循序尋找。這是一個相對非常耗費資源的程序，所以我們不想誤導你。另外，字符 (letter) 並不是萬國碼的一個標準定義。我們可以選擇把字串看做單獨的位元組，和室單獨的編碼位置 (code point)：

```rust
let hachiko = "忠犬ハチ公";

for b in hachiko.as_bytes() {
    print!("{}, ", b);
}

println!("");

for c in hachiko.chars() {
    print!("{}, ", c);
}

println!("");
```

就會印出：

```text
229, 191, 160, 231, 138, 172, 227, 131, 143, 227, 131, 129, 229, 133, 172,
忠, 犬, ハ, チ, 公,
```

你可以看出，它所含的位元組遠多於 `char` 的數量。

你可以使用以下的方法達到類似於索引的功能：

```rust
# let hachiko = "忠犬ハチ公";
let dog = hachiko.chars().nth(1); // kinda like hachiko[1]
```
這種方法強調我們必須從頭開始循序的尋找一連串的 `chars`。

## Slicing

You can get a slice of a string with slicing syntax:

```rust
let dog = "hachiko";
let hachi = &dog[0..5];
```

But note that these are _byte_ offsets, not _character_ offsets. So
this will fail at runtime:

```rust,should_panic
let dog = "忠犬ハチ公";
let hachi = &dog[0..2];
```

with this error:

```text
thread '<main>' panicked at 'index 0 and/or 2 in `忠犬ハチ公` do not lie on
character boundary'
```

## Concatenation

If you have a `String`, you can concatenate a `&str` to the end of it:

```rust
let hello = "Hello ".to_string();
let world = "world!";

let hello_world = hello + world;
```

But if you have two `String`s, you need an `&`:

```rust
let hello = "Hello ".to_string();
let world = "world!".to_string();

let hello_world = hello + &world;
```

This is because `&String` can automatically coerce to a `&str`. This is a
feature called ‘[`Deref` coercions][dc]’.

[dc]: deref-coercions.html
[connect]: ../std/net/struct.TcpStream.html#method.connect


> *commit 310ab5e*
