# 猜數字遊戲

讓我們開始學習 Rust 吧！
作為我們的第一個專案，我們會實作一個經典的新手程式問題：猜數字 (guessing game)。
它的規則是這樣：我們的程式會產生一個 1 到 100 間的亂數。
然後提示我們輸入數字猜猜看。
當我們輸入之後，它會告訴我們太大還是太小。
當我們猜對了，它會恭喜我們。
聽起來不錯吧？

接下來，我們將會學到一些 Rust 的東西。
下一章 "語法及語意" 將會更深入探究每一部份。

## 準備

讓我們準備一個新專案。
進到你的專案目錄。
還記得我們如何替 `hello_world` 建立目錄結構和 `Cargo.toml` 嗎？
Cargo 有個能幫我們做好這些事情的指令。
讓我們試試看：

```bash
$ cd ~/projects
$ cargo new guessing_game --bin
$ cd guessing_game
```

我們傳遞專案的名字給 `cargo new`，接著是 `--bin` 的參數，因為我們要建立執行檔而不是函式庫。

看看產生的 `Cargo.toml`：

```toml
[package]

name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
```

Cargo 會從你的環境設定中取得這些資訊。
如果它不正確，就直接修改吧。

最後，Cargo 幫我們產生了一個 "Hello, world!"。
看看 `src/main.rs` 檔案：

```rust
fn main() {
    println!("Hello, world!");
}
```

讓我們試著編譯：

```{bash}
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
```

太棒了！
再次打開你的 `src/main.rs`。
我們將開始在這個檔案寫入我們所有的程式碼。

在繼續之前，讓我們告訴你另一個 Cargo 的指令 `run`。
`cargo run` 有點類似 `cargo build`，但是他同時會執行產生的執行檔。
試試看：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/debug/guessing_game`
Hello, world!
```

很好！
當我們需要快速地重複執行專案時 `run` 指令很方便。
我們的遊戲就是一個專案，我們需要在每個開發循環 (iteration) 做快速的測試之後，才進入下一個開發循環。

## 處理猜測

讓我們開始吧！
我們所要做的第一件事情是讓我們的玩家可以輸入他的猜測。
把以下內容放入你的 `src/main.rs`：

```rust,no_run
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

東西有點多！
讓我們一個一個來。

```rust,ignore
use std::io;
```

我們需要取得使用者的輸入，然後印出結果作為輸出。
因此，我們需要標準函式庫中的 `io` 函式庫。
Rust 預設只會替所有程式 import 很少的東西，[prelude][prelude]。
如果有東西不在其中，你就必須直接 `use` 它。
這邊還有第二種 "prelude"，叫做 [io prelude][ioprelude]，它提供類似的功能：import 它，然後它幫你 import 一些有用且跟 `io` 有關的東西。

[prelude]: https://doc.rust-lang.org/std/prelude/index.html
[ioprelude]: https://doc.rust-lang.org/std/io/prelude/index.html

```rust,ignore
fn main() {
```

如你之前所看過的，`main()` 函式是你程式的進入點。
`fn` 語法宣告一個新函式，`()` 說明這裡沒有要傳入的參數，而 `{` 後面開始函式的內容。
因為沒有提到回傳型別，所以這裡會是 `()`，一個空的[多元組][tuples] (tuple)。

[tuples]: primitive-types.html#多元組%20(Tuples)

```rust,ignore
    println!("Guess the number!");

    println!("Please input your guess.");
```

我們之前學到 `println!()` 是個將[字串][strings]印到螢幕的[巨集][macros]。

[macros]: macros.html
[strings]: strings.html

```rust,ignore
    let mut guess = String::new();
```

現在遇到有趣的東西了！
這一行有不少東西。
第一個，請注意有一個 [let 陳述式][let]，它被用來建立 "變數綁定"。
它的形式是：

```rust,ignore
let foo = bar;
```

[let]: variable-bindings.html

這將會建立一個叫做 `foo` 的新綁定，然後把它綁定在 `bar` 的值上。
在許多語言中，這叫做 "變數" (variable)，但是 Rust 的變數綁定暗藏玄機。

舉例來說，它預設是[不可變的][immutable] (immutable)。
這也是為何我們的範例使用 `mut`：它使綁定變成可變的 (mutable)，而不再是不可變的。
`let` 不會從左邊取得賦值 (assignment) 的名字，實際上他使用 "[模式][patterns]" (pattern)。
我們在後面會使用模式。
現在它已經夠讓我們簡單使用了：

```rust
let foo = 5; // immutable.
let mut bar = 5; // mutable
```

[immutable]: mutability.html
[patterns]: patterns.html

噢，還有 `//` 會開始一段註解，直到行末。
Rust 會忽略所有[註解][comments] (comment) 內的東西。

[comments]: comments.html

所以現在我們知道 `let mut guess` 會宣告一個名為 `guess` 的可變變數綁定，但我們仍必須看看 `=` 另一邊綁定的東西：`String::new()`。

`String` 是一個字串型別，由標準函式庫提供。
一個[字串][string]是一個以 UTF-8 編碼的可變長度的文字。

[string]: https://doc.rust-lang.org/std/string/struct.String.html

而 `::new()` 語法使用 `::` 是因為它是一個特定型別的 "關聯函式" (associated function)。
也就是說，它被關聯到 `String` 本身，而非特定的某個 `String` 的實體 (instance)。
一些語言稱之為 "靜態方法" (static method)。

此函式被稱為 `new()`，因為它建立一個新的、空的 `String`。
你可以在其他許多型別找到 `new()` 函式，因為它是建立某一些型別新值的通用名稱。

讓我們繼續下去：

```rust,ignore
    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");
```

稍微有點多了！
一步一步來。
第一行有兩個部分。
第一部分是：

```rust,ignore
io::stdin()
```

還記得在程式第一行我們怎麼 `use` 那個 `std::io` 的嗎？
我們現在正在呼叫它的關聯函式。
如果我們沒有 `use std::io`，我們必須把這行改成 `std::io::stdin()`。

這個特殊的函式會回傳一個你的終端機標準輸入的控制代碼 (handle)。
更具體的內容，可以參考 [std::io::Stdin][iostdin]。

[iostdin]: https://doc.rust-lang.org/std/io/struct.Stdin.html

下一部份將會使用這個控制代碼 (handle) 來取得使用者的輸入：

```rust,ignore
.read_line(&mut guess)
```

此處，我們呼叫控制代碼 (handle) 的 [read_line()][read_line] 方法。
[方法][method] (method) 跟關聯函式很類似，但是只能在特定型別的實體 (instance) 中取用，而不是從型別本身取用。
我們也會傳遞一個參數給 `read_line()`：`&mut guess`。

[read_line]: https://doc.rust-lang.org/std/io/struct.Stdin.html#method.read_line
[method]: method-syntax.html

還記得前面我們如何綁定 `guess` 嗎？
我們提到它是可變的。
然而，`read_line` 不接受把 `String` 當作參數：它只接受 `&mut String`。
Rust 有一個叫做[參照][references] (references) 的功能，它允許你有多個參照指向同一塊資料，這樣可以降低複製的動作。
參照是個複雜的功能，Rust 的主要賣點就是能安全、簡單的使用參照。
現在我們不需要知道太多細節。
我們只需知道參照與 `let` 綁定類似，它預設是不可變的。
因此，我們必須寫成 `&mut guess` 而不是 `&guess`。

為何 `read_line()` 需要字串的可變參照？
因為它的工作是從標準輸入取得使用者的輸入，然後放進字串中。
所以它將字串當作參數，用來放入輸入值，所以它也必須是可變的。

[references]: references-and-borrowing.html

但這行程式碼還沒結束。
雖然它只有一行，但它只是程式碼的單一邏輯的第一部份。

```rust,ignore
        .expect("Failed to read line");
```

當我們使用 `.foo()` 語法呼叫方法時，你能以空白或是開新的一行當開頭。
這可以幫助你把太長的一行程式碼切斷。
我們當然 _可以_ 這樣做：

```rust,ignore
    io::stdin().read_line(&mut guess).expect("failed to read line");
```

但是這樣很難閱讀。
所以我們把它根據兩個方法的呼叫來切成兩行。
我們已經說過 `read_line()` 了，但 `expect()` 呢？
恩，我們已經提過 `read_line()` 會把使用者的輸入放入 `&mut String` 參數中。
但它也會回傳一個值：在這裡是一個 [io::Result][ioresult]。
Rust 的標準函式庫中有許多叫做 `Result` 的型別：一般的 [Result][result] 與子函式庫的特別版本，像 `io::Result`。

[ioresult]: https://doc.rust-lang.org/std/io/type.Result.html
[result]: https://doc.rust-lang.org/std/result/enum.Result.html

這種 `Result` 型別的目的是把錯誤處理訊息編碼。
`Result` 型別的值與其他任何型別一樣，有定義它自己的方法。
此處的 `io::Result` 有 [expect() 方法][expect]，可以用來取得呼叫它的結果值，如果呼叫它的結果不是成功的值，就會 [panic!][panic] 且帶著你傳給它的訊息。
這樣的 `panic!` 會讓你的程式當機，並顯示出你傳給它的訊息。

[expect]: https://doc.rust-lang.org/std/result/enum.Result.html#method.expect
[panic]: error-handling.html

如果我們去掉這個方法的呼叫，我們的程式仍可以編譯成功，但是我們會看到警告訊息：

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
src/main.rs:10:5: 10:39 warning: unused result which must be used,
#[warn(unused_must_use)] on by default
src/main.rs:10     io::stdin().read_line(&mut guess);
                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Rust 警告我們沒有使用 `Result` 的值。
這個警告來自 `io::Result` 的特殊註釋 (annotation)。
Rust 會試圖告訴你你沒有處理可能的錯誤。
處理錯誤的正確方式應該是撰寫錯誤處理的程式碼。
幸運的是，如果我們只想要一有問題就當機，我們可以直接使用這兩個方法。
而如果我們想從錯誤中恢復正常，我們必須做一些其他的事，但我們留在之後的專案再說。

這個範例現在只剩下一行：

```rust,ignore
    println!("You guessed: {}", guess);
}
```

這行印出我們所存的輸入值。
`{}` 是個 placeholder，因此我們傳遞 `guess` 作為參數。
如果我們有多個 `{}`，我們需要傳遞多個參數：

```rust
let x = 5;
let y = 10;

println!("x and y: {} and {}", x, y);
```

簡單吧。

總之，這就像個導覽。
我們可以用 `cargo run` 執行我們的專案：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

好吧！我們的第一部份完成了：我們可以從鍵盤取得輸入值，然後印出來。

## 產生祕密數字

接著，我們需要產生一個祕密數字。
Rust 的標準函式庫還沒有包含亂數功能。
但是 Rust 團隊提供了 [rand crate][randcrate]。
這個 "crate" 是一包 Rust 程式碼。
我們會建置可以執行的 "binary crate"。
`rand` 是個 "library crate"，包含了可以被其他程式使用的程式碼。

[randcrate]: https://crates.io/crates/rand

使用外部的 crates 是 Cargo 的亮點。
在我們使用 `rand` 撰寫程式碼前，我們要修改 `Cargo.toml`。
打開它，然後加入以下幾行：

```toml
[dependencies]

rand="0.3.0"
```

`Cargo.toml` 的 `[dependencies]` 一節跟 `[package]` 類似：它後面的所有東西都是它的一部份，直到下一節的開始為止。
Cargo 使用 dependencies 來瞭解你需要的外部 crates 以及版本。
在這個例子中，我們特別指定版本 `0.3.0`，所以 Cargo 瞭解知道任何發行版都要跟這個版本相容。
Cargo 能瞭解 [Semantic Versioning][semver] 的版本編碼方式，這是撰寫版本號碼的一種標準。
上面的版號實際上可以寫成 `^0.3.0`，代表 "所有跟 0.3.0 相容的版本"。
如果我們想要確實的只用 `0.3.0`，我們可以寫成 `rand="=0.3.0"`（請注意兩個等號）。
而當我們想要使用最新版，我們可以使用 `*`。
我們也可以使用一個範圍內的版本。
[Cargo 的說明文件][cargodoc] 有更多細節。

[semver]: http://semver.org
[cargodoc]: http://doc.crates.io/crates-io.html

現在，不修改任何我們的程式碼，讓我們建置我們的專案：

```bash
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.8
 Downloading libc v0.1.6
   Compiling libc v0.1.6
   Compiling rand v0.3.8
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
```

（當然，你可能會看到不同版本。）

有很多新的輸出資訊！
現在我們有了外部的 dependency，Cargo 會從註冊表取得最新版，這個註冊表是 [Crates.io][cratesio] 的複本。
Crates.io 是 Rust 生態系中，讓人們發表他們的 Rust 開放原始碼專案給其他人使用的地方。

[cratesio]: https://crates.io

更新了註冊表之後，Cargo 會檢查我們的 `[dependencies]` 然後下載任何我們還沒有的東西。
在這邊的例子，雖然說我們只說想要相依 `rand`，但我們仍需要取得 `libc` 的複本。
這是因為 `rand` 相依於 `libc` 才能運作。
下載它們之後，Cargo 會編譯他們，然後編譯我們的專案。

如果我們再次執行 `cargo build`，我們將會看到不同的輸出結果：

```bash
$ cargo build
```

沒錯，沒有輸出！
Cargo 知道我們的專案已經建置好了，而且所有的 dependencies 也建置好了，所以他沒有任何理由再做一遍。
因為沒事情做，它就簡單地結束了。
如果我們再次開啟 `src/main.rs`，做一些簡單的修改，然後存檔再試一次，我們只會看到一行：

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
```

所以，我們告訴 Cargo 想要任何 `rand` 的 `0.3.x` 版，所以它抓了當下的最新版 `v0.3.8`。
但如果下週，一個帶有重要問題修正的 `v0.3.9` 版出現之後會發生什麼事情？
雖然取得問題的修正很重要，但是如果 `0.3.9` 包含了會弄壞我們程式碼的 regression 呢？

這個問題的答案是在專案中能找到的 `Cargo.lock` 檔。
當你第一次建置專案時，Cargo 會檢查所有符合你的條件的版本，然後寫入 `Cargo.lock` 檔。
未來當你再次建置你的專案時，Cargo 會視 `Cargo.lock` 檔案是否已經存在，然後使用裡面的特定版本，而非再次尋找所有版本。
這讓你有一個可以重複建置的環境。
也就是說，我們會停在 `0.3.8` 直到我們確定要升級，這也對任何我們與之分享程式碼的人有效，感謝 lock 檔。

當我們 _真的_ 想要用 `0.3.9` 時怎麼辦？
Cargo 有另一個指令 `update`，代表著 "忽略 lock 檔，尋找所有符合我們指定條件的最新版。如果運行成功，把這些版本寫入 lock 檔"。
但是，預設 Cargo 只會尋找大於 `0.3.0` 及小於 `0.4.0` 的版本。
如果我們想要換到 `0.4.x`，我們必須直接更新 `Cargo.toml`。
當我們這樣做，下次 `cargo build` 時，Cargo 將會更新索引並重新評估 `rand` 的需求。

關於 [Cargo][doccargo] 和其[生態系][doccratesio]還有很多可以說，但是現在，這就是我們所需知道的全部了。
Cargo 讓重新使用函式庫變得很簡單，讓 Rustaceans 易於寫出由許多子套件 (sub-packages) 組成的小專案。

[doccargo]: http://doc.crates.io
[doccratesio]: http://doc.crates.io/crates-io.html

接著讓我們真正的 _用用看_ `rand` 吧。
這是我們的下一步：

```rust,ignore
extern crate rand;

use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("failed to read line");

    println!("You guessed: {}", guess);
}
```

第一步是修改第一行。
現在他是 `extern crate rand`。
因為在 `[dependencies]` 中我們宣告了 `rand`，我們可以使用 `extern crate` 讓 Rust 知道我們要使用它。
這也等同於 `use rand;`，所以我們可以透過 `rand::` 前綴詞來使用任何 `rand` crate 內的東西。

接著，我們加入另一行 `use`：`use rand::Rng`。
我們即將使用一個方法 (method)，這個方法需要 `Rng` 在有效範圍 (scope) 中才能運作。
基本的概念是：方法被定義在某些被稱為 "traits" 的東西上，trait 必須要在有效範圍中，方法才能運作。
更多細節可以閱讀 [traits][traits] 一節。

[traits]: traits.html

我們還加入了其他兩行在中間：

```rust,ignore
    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);
```

我們使用 `rand::thread_rng()` 函式去取得隨機數字產生器的複本，這個複本位於我們正在執行的特定[執行緒][concurrency] (thread) 之上。
因為我們前面已經 `use rand::Rng`，所以 `gen_rang()` 方法可以使用。
這個方法需要兩個參數，然後產生一個在兩個參數之間的數字。
包含了下限值，但不包含上限值，所以我們需要 `1` 和 `101` 去取得 1 到 100 範圍內的數字。

[concurrency]: concurrency.html

第二行印出祕密數字。
當我們正在開發程式時這很有用，讓我們可以簡單的測試一下。
不過我們在最後會刪掉它。
沒什麼遊戲會在一開始就印出答案的！

試著執行我們的新程式幾次：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4
$ cargo run
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

太棒了！
接著：就要開始比較我們的猜測值和祕密數字。

## 比較猜測值

現在，我們拿到了使用者的輸入值，讓我們來比較猜測值和祕密數字。
以下是我們的下一步，雖然這還沒編譯過：

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("failed to read line");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less    => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal   => println!("You win!"),
    }
}
```

這裡有些新東西。
首先是另一個 `use`。
我們用了稱為 `std::cmp::Ordering` 的型別到有效範圍中。
然後底下有五行程式碼用到它：

```rust,ignore
match guess.cmp(&secret_number) {
    Ordering::Less    => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal   => println!("You win!"),
}
```

`cmp()` 方法可以被任何能用來比較的東西呼叫，且它會要求傳入你想比較的東西的參照。
它回傳我們前面 `use` 的 `Ordering` 型別。
我們使用 [match][match] 陳述去判定它實際上是哪種 `Ordering`。
`Ordering` 是個 [enum][enum]，枚舉 (enumeration) 的簡寫，看起來會有點像這樣：

```rust
enum Foo {
    Bar,
    Baz,
}
```

[match]: match.html
[enum]: enums.html

這裡定義了任何 `Foo` 不是 `Foo::Bar` 就是 `Foo::Baz`。
我們使用 `::` 來表示特定 `enum` 變數的命名空間 (namespace)。

[Ordering][ordering] `enum` 有三個可能的變數：`Less`、`Equal`、及 `Greater`。
`match` 陳述式取得型別的值，讓你能為每個可能的值建立一條執行的分支。
因為 `Ordering` 有三種型別，我們就有三個分支：

```rust,ignore
match guess.cmp(&secret_number) {
    Ordering::Less    => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal   => println!("You win!"),
}
```

[ordering]: https://doc.rust-lang.org/std/cmp/enum.Ordering.html

如果它是 `Less`，我們印出 `Too small!`，如果是 `Greater` 則印出 `Too big!`，如果是 `Equal` 就印出 `You win!`。
`match` 非常有用，而且在 Rust 中常會用到。

雖然我確實提到過我們還不能編譯成功。
讓我們還是試試看：

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
src/main.rs:28:21: 28:35 error: mismatched types:
 expected `&collections::string::String`,
    found `&_`
(expected struct `collections::string::String`,
    found integral variable) [E0308]
src/main.rs:28     match guess.cmp(&secret_number) {
                                   ^~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `guessing_game`.
```

噢！這裡有個大問題。
它的核心問題是我們有 "無法匹配的型別" (mismatched types)。
Rust 有個強大的靜態型別系統 (static type system)。
但是，它也有型別推斷 (type inference) 系統。
當我們寫到 `let guess = String::new()` 時，Rust 會推斷 `guess` 應該是 `String`，因此不需要我們特別寫出型別。
而我們的 `secret_number` 則有多種可能的型別都可以存 1 到 100 的值：32 位元整數 `i32`、非帶號 (unsigned) 32 位元整數 `u32`、64 位元整數 `i64` 或其他等等。
目前這些都不重要，所以 Rust 預設為 `i32`。
然而，Rust 不知道怎麼去比對 `guess` 與 `secret_number`。
它們必須是相同型別。
最終，為了可以比較，我們想要把輸入的 `String` 轉換為一個真正的數字型別。 
我們可以用額外的兩行做到。
以下是我們的新程式：

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("failed to read line");

    let guess: u32 = guess.trim().parse()
        .expect("Please type a number!");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less    => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal   => println!("You win!"),
    }
}
```

新加的兩行：

```rust,ignore
    let guess: u32 = guess.trim().parse()
        .expect("Please type a number!");
```

等等，我想我們已經有 `guess` 了吧？
沒錯，但是 Rust 允許我們用新的 `guess` 去 "遮蔽" (shadow) 前一個。
一開始 `guess` 是 `String`，但我們希望能轉換為 `u32`，像是這樣的情況下我們很常會用到。
遮蔽讓我們可以重複利用 `guess` 的命名，而非強迫我們去想出兩個獨特的命名，像是 `guess_str` 和 `guess`，或其他的名稱。

我們把 `guess` 綁定在我們前面所寫的表達式上：

```rust,ignore
guess.trim().parse()
```

這邊的 `guess` 參考到存著我們輸入值的舊的 `guess`。
而 `String` 中的 `trim()` 方法則會去除任何字串開頭結尾的空白。
這很重要，因為我們必須按下 "Return" 按鍵去符合 `read_line()` 的輸入條件。
也就是說如果我們輸入 `5` 然後按下 Return，那 `guess` 看起來就會像是：`5\n`。
`\n` 代表 "新的一行" (newline)、enter 鍵。
`trim()` 會去除這些，只留下我們的字串 `5`。
而[字串的 parse() 方法][parse]則會把字串分析為數字。
因為它可以被分析為很多種數字型別，我們必須給 Rust 我們確切想要的數字型別的提示。
因次我們給它 `let guess: u32`。
`guess` 後面的分號（`:`）告訴 Rust 我們要註釋它的型別。
`u32` 是沒有帶正負號的 32 位元整數。
Rust 有[許多內建的數字型別][number]，我們選了 `u32`。
對於不大的正整數來說，它是個好選擇。

[parse]: https://doc.rust-lang.org/std/primitive.str.html#method.parse
[number]: primitive-types.html#數字型別

跟 `read_line()` 一樣，我們呼叫 `parse()` 時可能會發生錯誤。
如果我們的字串包含了 `A👍%` 怎麼辦？
我們可無法把它轉換為數字。
為此我們做了與之前替 `read_line()` 所做的事一樣的事：使用 `expect()` 方法，讓它出錯時當機。

讓我們試試我們的程式！

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

不錯！
你可以看到我在猜測值前面加入空白，它仍可以認出我猜的是 76。
多執行程式幾次，然後驗證看看猜中正確的數字，和猜一個比較小的數字。

現在我們的遊戲大致上能運作了，但是我們只能猜一次。
讓我們加入迴圈 (loops) 來修改它。

## 迴圈

關鍵字 `loop` 會給我們一個無限迴圈。
讓我們加上它：

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("failed to read line");

        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => println!("You win!"),
        }
    }
}
```

然後試試看。
等等，我們不是加了個無限迴圈嗎？
沒錯，還記得我們討論過的 `parse()` 嗎？
如果我們輸入一個不是數字的答案，我們就會 `panic!` 然後結束。
看著：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread '<main>' panicked at 'Please type a number!'
```

哈！
`quit` 真的退出了。
就跟任何其他不是數字的輸入一樣。
好吧，這至少是個還可以的做法。
接著，讓我們能在贏了遊戲的時候真的退出：

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("failed to read line");

        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

透過在 `You win!` 之後加入 `break`，我們可以在贏了之後離開迴圈。
因為它同時是 `main()` 的最後一部份，所以離開迴圈也代表著退出程式。
我們剩下最後一點需要修改：當輸入一個不是數字的輸入，我們不想退出程式，我們要忽略它。
我們可以像以下這樣做：

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

這是我們所修改的部分：

```rust,ignore
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

把 `expect()` 改為 `match` 陳述的方式，大致上就是如何把 "錯誤時當機" 改為 "實際處理錯誤" 的方法。
`parse()` 回傳的 `Result` 是個跟 `Ordering` 類似的 `enum`，但是這裡的變數跟資料有關：`Ok` 代表成功，`Err` 則是錯誤。
它們個別包含更多的資訊：成功的分析出整數，或是一個錯誤型別。
在本例中，當我們 `match` 到 `Ok(num)` 時，會把 `Ok` 內的值設給 `num` 這個名稱，然後在右邊回傳它。
在 `Err` 的情況，我們不在意發生了什麼錯誤，所以我們使用 `_` 沒有取名。
這樣會忽略錯誤，接著 `continue` 讓我們可以繼續 `loop` 的下一次疊代 (iteration)。

現在應該弄好了！
讓我們試試看：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

太棒了！
剩下最後一個小修改，我們就完成了猜數字遊戲。
你能猜到是什麼嗎？
沒錯，我們不希望印出祕密數字。
在測試時它很好，但是它會毀掉遊戲。
這是我們最終的程式碼：

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

## 完成！

此時此刻，你成功的建置了猜謎遊戲！
恭喜你！

這第一個專案告訴你不少東西：`let`、`match`、方法 (methods)、關聯函式 (associated functions)、使用外部的 crates、等等。
我們的下個專案會告訴你更多東西。

> 譯註：實作 `dining-philosophers` 專案的章節已經被移除了，所以後續會接到語法的部分。

> *commit edd5f33*
