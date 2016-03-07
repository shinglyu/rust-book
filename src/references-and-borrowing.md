# 參考與借用

本指南是當前 Rust 的三個所有權系統之一。
這是 Rust 最獨特且引人注目的功能之一，作為 Rust 的開發者應該對此要有相當的了解。
所有權是 Rust 用來達成其最大的目標，記憶體安全，的方法。
它有一些不同的概念，各自有各自的章節：

* [所有權][ownership] (ownership)，關鍵的概念
* 借用 (borrowing)，你正在閱讀的章節
* [生命週期][lifetimes] (lifetime)，借用的進階概念

這三章依序相關。
你需要了解全部三章來完整了解所有權系統。

[ownership]: ownership.html
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

記住這些之後，讓我們開始學習借用。

## Borrowing

At the end of the [ownership][ownership] section, we had a nasty function that looked
like this:

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

This is not idiomatic Rust, however, as it doesn’t take advantage of borrowing. Here’s
the first step:

```rust
fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    // do stuff with v1 and v2

    // return the answer
    42
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let answer = foo(&v1, &v2);

// we can use v1 and v2 here!
```

Instead of taking `Vec<i32>`s as our arguments, we take a reference:
`&Vec<i32>`. And instead of passing `v1` and `v2` directly, we pass `&v1` and
`&v2`. We call the `&T` type a ‘reference’, and rather than owning the resource,
it borrows ownership. A binding that borrows something does not deallocate the
resource when it goes out of scope. This means that after the call to `foo()`,
we can use our original bindings again.

References are immutable, like bindings. This means that inside of `foo()`,
the vectors can’t be changed at all:

```rust,ignore
fn foo(v: &Vec<i32>) {
     v.push(5);
}

let v = vec![];

foo(&v);
```

errors with:

```text
error: cannot borrow immutable borrowed content `*v` as mutable
v.push(5);
^
```

Pushing a value mutates the vector, and so we aren’t allowed to do it.

## &mut references

There’s a second kind of reference: `&mut T`. A ‘mutable reference’ allows you
to mutate the resource you’re borrowing. For example:

```rust
let mut x = 5;
{
    let y = &mut x;
    *y += 1;
}
println!("{}", x);
```

This will print `6`. We make `y` a mutable reference to `x`, then add one to
the thing `y` points at. You’ll notice that `x` had to be marked `mut` as well.
If it wasn’t, we couldn’t take a mutable borrow to an immutable value.

You'll also notice we added an asterisk (`*`) in front of `y`, making it `*y`,
this is because `y` is a `&mut` reference. You'll also need to use them for
accessing the contents of a reference as well.

Otherwise, `&mut` references are like references. There _is_ a large
difference between the two, and how they interact, though. You can tell
something is fishy in the above example, because we need that extra scope, with
the `{` and `}`. If we remove them, we get an error:

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
note: previous borrow of `x` occurs here; the mutable borrow prevents
subsequent moves, borrows, or modification of `x` until the borrow ends
        let y = &mut x;
                     ^
note: previous borrow ends here
fn main() {

}
^
```

As it turns out, there are rules.

## The Rules

Here’s the rules about borrowing in Rust:

First, any borrow must last for a scope no greater than that of the owner.
Second, you may have one or the other of these two kinds of borrows, but not
both at the same time:

* one or more references (`&T`) to a resource,
* exactly one mutable reference (`&mut T`).


You may notice that this is very similar, though not exactly the same as,
to the definition of a data race:

> There is a ‘data race’ when two or more pointers access the same memory
> location at the same time, where at least one of them is writing, and the
> operations are not synchronized.

With references, you may have as many as you’d like, since none of them are
writing. However, as we can only have one `&mut` at a time, it is impossible to
have a data race. This is how Rust prevents data races at compile time: we’ll
get errors if we break the rules.

With this in mind, let’s consider our example again.

### Thinking in scopes

Here’s the code:

```rust,ignore
let mut x = 5;
let y = &mut x;

*y += 1;

println!("{}", x);
```

This code gives us this error:

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
```

This is because we’ve violated the rules: we have a `&mut T` pointing to `x`,
and so we aren’t allowed to create any `&T`s. One or the other. The note
hints at how to think about this problem:

```text
note: previous borrow ends here
fn main() {

}
^
```

In other words, the mutable borrow is held through the rest of our example. What
we want is for the mutable borrow to end _before_ we try to call `println!` and
make an immutable borrow. In Rust, borrowing is tied to the scope that the
borrow is valid for. And our scopes look like this:

```rust,ignore
let mut x = 5;

let y = &mut x;    // -+ &mut borrow of x starts here
                   //  |
*y += 1;           //  |
                   //  |
println!("{}", x); // -+ - try to borrow x here
                   // -+ &mut borrow of x ends here
```

The scopes conflict: we can’t make an `&x` while `y` is in scope.

So when we add the curly braces:

```rust
let mut x = 5;

{
    let y = &mut x; // -+ &mut borrow starts here
    *y += 1;        //  |
}                   // -+ ... and ends here

println!("{}", x);  // <- try to borrow x here
```

There’s no problem. Our mutable borrow goes out of scope before we create an
immutable one. But scope is the key to seeing how long a borrow lasts for.

### Issues borrowing prevents

Why have these restrictive rules? Well, as we noted, these rules prevent data
races. What kinds of issues do data races cause? Here’s a few.

#### Iterator invalidation

One example is ‘iterator invalidation’, which happens when you try to mutate a
collection that you’re iterating over. Rust’s borrow checker prevents this from
happening:

```rust
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
}
```

This prints out one through three. As we iterate through the vector, we’re
only given references to the elements. And `v` is itself borrowed as immutable,
which means we can’t change it while we’re iterating:

```rust,ignore
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
    v.push(34);
}
```

Here’s the error:

```text
error: cannot borrow `v` as mutable because it is also borrowed as immutable
    v.push(34);
    ^
note: previous borrow of `v` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `v` until the borrow ends
for i in &v {
          ^
note: previous borrow ends here
for i in &v {
    println!(“{}”, i);
    v.push(34);
}
^
```

We can’t modify `v` because it’s borrowed by the loop.

#### use after free

References must not live longer than the resource they refer to. Rust will
check the scopes of your references to ensure that this is true.

If Rust didn’t check this property, we could accidentally use a reference
which was invalid. For example:

```rust,ignore
let y: &i32;
{
    let x = 5;
    y = &x;
}

println!("{}", y);
```

We get this error:

```text
error: `x` does not live long enough
    y = &x;
         ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
let y: &i32;
{
    let x = 5;
    y = &x;
}

note: ...but borrowed value is only valid for the block suffix following
statement 0 at 4:18
    let x = 5;
    y = &x;
}
```

In other words, `y` is only valid for the scope where `x` exists. As soon as
`x` goes away, it becomes invalid to refer to it. As such, the error says that
the borrow ‘doesn’t live long enough’ because it’s not valid for the right
amount of time.

The same problem occurs when the reference is declared _before_ the variable it
refers to. This is because resources within the same scope are freed in the
opposite order they were declared:

```rust,ignore
let y: &i32;
let x = 5;
y = &x;

println!("{}", y);
```

We get this error:

```text
error: `x` does not live long enough
y = &x;
     ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
    let y: &i32;
    let x = 5;
    y = &x;

    println!("{}", y);
}

note: ...but borrowed value is only valid for the block suffix following
statement 1 at 3:14
    let x = 5;
    y = &x;

    println!("{}", y);
}
```

In the above example, `y` is declared before `x`, meaning that `y` lives longer
than `x`, which is not allowed.


> *commit 6ba9520*

