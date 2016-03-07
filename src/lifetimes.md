# 生命週期

本指南是當前 Rust 的三個所有權系統之一。
這是 Rust 最獨特且引人注目的功能之一，作為 Rust 的開發者應該對此要有相當的了解。
所有權是 Rust 用來達成其最大的目標，記憶體安全，的方法。
它有一些不同的概念，各自有各自的章節：

* [所有權][ownership] (ownership)，關鍵的概念
* [借用][borrowing] (borrowing)，及其相關功能 "參考" (references)
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

## Lifetimes

Lending out a reference to a resource that someone else owns can be
complicated. For example, imagine this set of operations:

1. I acquire a handle to some kind of resource.
2. I lend you a reference to the resource.
3. I decide I’m done with the resource, and deallocate it, while you still have
  your reference.
4. You decide to use the resource.

Uh oh! Your reference is pointing to an invalid resource. This is called a
dangling pointer or ‘use after free’, when the resource is memory.

To fix this, we have to make sure that step four never happens after step
three. The ownership system in Rust does this through a concept called
lifetimes, which describe the scope that a reference is valid for.

When we have a function that takes a reference by argument, we can be implicit
or explicit about the lifetime of the reference:

```rust
// implicit
fn foo(x: &i32) {
}

// explicit
fn bar<'a>(x: &'a i32) {
}
```

The `'a` reads ‘the lifetime a’. Technically, every reference has some lifetime
associated with it, but the compiler lets you elide (i.e. omit, see
["Lifetime Elision"][lifetime-elision] below) them in common cases.
Before we get to that, though, let’s break the explicit example down:

[lifetime-elision]: #lifetime-elision

```rust,ignore
fn bar<'a>(...)
```

We previously talked a little about [function syntax][functions], but we didn’t
discuss the `<>`s after a function’s name. A function can have ‘generic
parameters’ between the `<>`s, of which lifetimes are one kind. We’ll discuss
other kinds of generics [later in the book][generics], but for now, let’s
focus on the lifetimes aspect.

[functions]: functions.html
[generics]: generics.html

We use `<>` to declare our lifetimes. This says that `bar` has one lifetime,
`'a`. If we had two reference parameters, it would look like this:


```rust,ignore
fn bar<'a, 'b>(...)
```

Then in our parameter list, we use the lifetimes we’ve named:

```rust,ignore
...(x: &'a i32)
```

If we wanted a `&mut` reference, we’d do this:

```rust,ignore
...(x: &'a mut i32)
```

If you compare `&mut i32` to `&'a mut i32`, they’re the same, it’s that
the lifetime `'a` has snuck in between the `&` and the `mut i32`. We read `&mut
i32` as ‘a mutable reference to an `i32`’ and `&'a mut i32` as ‘a mutable
reference to an `i32` with the lifetime `'a`’.

## In `struct`s

You’ll also need explicit lifetimes when working with [`struct`][structs]s that
contain references:

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

As you can see, `struct`s can also have lifetimes. In a similar way to functions,

```rust
struct Foo<'a> {
# x: &'a i32,
# }
```

declares a lifetime, and

```rust
# struct Foo<'a> {
x: &'a i32,
# }
```

uses it. So why do we need a lifetime here? We need to ensure that any reference
to a `Foo` cannot outlive the reference to an `i32` it contains.

### `impl` blocks

Let’s implement a method on `Foo`:

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

As you can see, we need to declare a lifetime for `Foo` in the `impl` line. We repeat
`'a` twice, like on functions: `impl<'a>` defines a lifetime `'a`, and `Foo<'a>`
uses it.

### Multiple lifetimes

If you have multiple references, you can use the same lifetime multiple times:

```rust
fn x_or_y<'a>(x: &'a str, y: &'a str) -> &'a str {
#    x
# }
```

This says that `x` and `y` both are alive for the same scope, and that the
return value is also alive for that scope. If you wanted `x` and `y` to have
different lifetimes, you can use multiple lifetime parameters:

```rust
fn x_or_y<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {
#    x
# }
```

In this example, `x` and `y` have different valid scopes, but the return value
has the same lifetime as `x`.

### Thinking in scopes

A way to think about lifetimes is to visualize the scope that a reference is
valid for. For example:

```rust
fn main() {
    let y = &5;     // -+ y goes into scope
                    //  |
    // stuff        //  |
                    //  |
}                   // -+ y goes out of scope
```

Adding in our `Foo`:

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

Our `f` lives within the scope of `y`, so everything works. What if it didn’t?
This code won’t work:

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

Whew! As you can see here, the scopes of `f` and `y` are smaller than the scope
of `x`. But when we do `x = &f.x`, we make `x` a reference to something that’s
about to go out of scope.

Named lifetimes are a way of giving these scopes a name. Giving something a
name is the first step towards being able to talk about it.

### 'static

The lifetime named ‘static’ is a special lifetime. It signals that something
has the lifetime of the entire program. Most Rust programmers first come across
`'static` when dealing with strings:

```rust
let x: &'static str = "Hello, world.";
```

String literals have the type `&'static str` because the reference is always
alive: they are baked into the data segment of the final binary. Another
example are globals:

```rust
static FOO: i32 = 5;
let x: &'static i32 = &FOO;
```

This adds an `i32` to the data segment of the binary, and `x` is a reference
to it.

### Lifetime Elision

Rust supports powerful local type inference in function bodies, but it’s
forbidden in item signatures to allow reasoning about the types based on
the item signature alone. However, for ergonomic reasons a very restricted
secondary inference algorithm called “lifetime elision” applies in function
signatures. It infers only based on the signature components themselves and not
based on the body of the function, only infers lifetime parameters, and does
this with only three easily memorizable and unambiguous rules. This makes
lifetime elision a shorthand for writing an item signature, while not hiding
away the actual types involved as full local inference would if applied to it.

When talking about lifetime elision, we use the term *input lifetime* and
*output lifetime*. An *input lifetime* is a lifetime associated with a parameter
of a function, and an *output lifetime* is a lifetime associated with the return
value of a function. For example, this function has an input lifetime:

```rust,ignore
fn foo<'a>(bar: &'a str)
```

This one has an output lifetime:

```rust,ignore
fn foo<'a>() -> &'a str
```

This one has a lifetime in both positions:

```rust,ignore
fn foo<'a>(bar: &'a str) -> &'a str
```

Here are the three rules:

* Each elided lifetime in a function’s arguments becomes a distinct lifetime
  parameter.

* If there is exactly one input lifetime, elided or not, that lifetime is
  assigned to all elided lifetimes in the return values of that function.

* If there are multiple input lifetimes, but one of them is `&self` or `&mut
  self`, the lifetime of `self` is assigned to all elided output lifetimes.

Otherwise, it is an error to elide an output lifetime.

#### Examples

Here are some examples of functions with elided lifetimes.  We’ve paired each
example of an elided lifetime with its expanded form.

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
