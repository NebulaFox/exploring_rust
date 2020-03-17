# Exploring Rust

Collection of challenges faced while programming in Rust

The intention of this collection is to focus on someone who
is new to Rust but not new to programming.
Languages such as Kotlin and Swift,
have cultivated a certain mindset,
which is _challenged_ in Rust,
mostly in the form of Rust`s ownership system.

It is assumed, the reader has read [Rust Programming Language Book](https://doc.rust-lang.org/book/)

#### Resources

 * [Rust Programming Language Book](https://doc.rust-lang.org/book/)
 * [Rust by Example](https://doc.rust-lang.org/stable/rust-by-example/)
 * [Rust Cookbook](https://rust-lang-nursery.github.io/rust-cookbook/intro.html)

## Challenge 1: `&str` and `String`

The difference between `&str` and `String` is, put simply, `&str` is a view slice and `String` is the dynamic heap string.

As you may notice, `&str` is a reference.
`str` itself is, sequence of UTF-8 bytes of dynamic length somewhere in memory, but `str` will always be seen as a reference, `&str`.
The reason is `str` is not a fixed size at
compile time, compared to `[u32; N]`.

```rust
use rand; // 0.7.3
use rand::Rng;

fn make_a_string() -> String {
    let mut s = String::new();
    s.push_str("Hello World");

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn main() {
    // how long is a piece of string
    let string: String = make_a_string();
    let s: &str = &string;
    println!("len: {}, string: {}", s.len(), s)
}
```
(
[Gist](https://gist.github.com/NebulaFox/7c51a2127c0ea745f385687202a04a12),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=7c51a2127c0ea745f385687202a04a12)
)

Internally, `String` has an array of `char`s that is a `str`.
Rust is setup so a `&String` can be coerced `&str`.

```rust
use rand; // 0.7.3
use rand::Rng;

fn make_a_string() -> String {
    let mut s = String::new();
    s.push_str("Hello World");

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn make_a_string_with(string: &str) -> String {
    let mut s = String::new();
    s.push_str(string);

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn main() {
    // how long is a piece of string
    let string: String = make_a_string();
    let s = make_a_string_with(&string); // compiler error without &
    println!("len: {}, string: {}", s.len(), s)
}
```
(
[Gist](https://gist.github.com/NebulaFox/e18b981cdc36b7323758854b87a0a373),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=e18b981cdc36b7323758854b87a0a373)
)

It would not make sense to coerce a `String` into `&str`,
as one is a concrete type while the other is a reference.

You cannot pass a `String` as it, as that would involve a move semantic.

```rust
use rand; // 0.7.3
use rand::Rng;

fn make_a_string_with(string: &str) -> String {
    let mut s = String::new();
    s.push_str(string);

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn make_a_string_move(string: String) -> String {
    let mut s = string;

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn main() {
    // how long is a piece of string
    let s = make_a_string_with("Hello World");
    println!("len: {}, string: {}", s.len(), s);
    
    let moved_string = String::from("PANIC")
    let m = make_a_string_move(moved_string); // move
    println!("len: {}, string: {}", m.len(), m);
    // println!("len: {}, string: {}", moved_string.len(), moved_string);
    // ^^^ Compiler error if uncommented
}
```
(
[Gist](https://gist.github.com/NebulaFox/476ba895333b1efa36892d630e68e230),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=476ba895333b1efa36892d630e68e230)
)

### Summary

It is important to understand `&str` and `String`
as it has ramifications later on.

### References

* [StackOverflow: What are the differences between Rust's `String` and `str`?](https://stackoverflow.com/questions/24158114/what-are-the-differences-between-rusts-string-and-str)
* [thoughtram: String vs &str in Rust](https://blog.thoughtram.io/string-vs-str-in-rust/)

## Extra: Trait Coercing

Although, just because `String` does not coerce into `&str`
does not mean you cannot write a function type that
accepts a `String`.
But passing `String` and not a `&String` will mean the
`String` is moved.

```rust
use rand; // 0.7.3
use rand::Rng;

fn make_a_string_with(string: &str) -> String {
    let mut s = String::new();
    s.push_str(string);

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn make_a_string_with_ref<S>(string: S) -> String
where
    S: AsRef<str>,
{
    let mut s = String::new();
    s.push_str(string.as_ref());

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn make_a_string_with_into<S>(string: S) -> String
where
    S: Into<String>
{
    let mut s = string.into();

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn main() {
    let slice = "Hello World"; // &'static str
    let string = String::from(slice); // also slice.to_string() or slice.to_owned()
    let moved_r_string = string.clone();
    let moved_i_string = string.clone();
    let ref_string = &string;

    // how long is a piece of string
    let s_slice = make_a_string_with(slice);
    let s_string = make_a_string_with(&string); // have to reference
    let s_ref_string = make_a_string_with(ref_string);
    
    let r_slice = make_a_string_with_ref(slice);
    let r_string = make_a_string_with_ref(moved_r_string); // moves
    let r_ref_string = make_a_string_with_ref(ref_string);
    
    let i_slice = make_a_string_with_into(slice);
    let i_string = make_a_string_with_into(moved_i_string); // moves
    let i_ref_string = make_a_string_with_into(ref_string);
    
    let array = [
        (slice, s_slice),
        (&string, s_string),
        (ref_string, s_ref_string),
        (slice, r_slice),
        (&string, r_string),
        (ref_string, r_ref_string),
        (slice, i_slice),
        (&string, i_string),
        (ref_string, i_ref_string),
    ];
    
    for (o, s) in array.iter() {
        println!("len: {}, orginal:{}, string: {}", s.len(), o, s)
    }
    
}
```
(
[Gist](https://gist.github.com/NebulaFox/2faa580ae76bd1e008a31ec2952b82cd),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=2faa580ae76bd1e008a31ec2952b82cd)
)

## Challenge 2: Lazy Loading

### Problem

```rust
struct Context {
    foo: Option<String>,
    bar: Option<String>
}

impl Context {
    fn new() -> Self {
        Context {
            foo: None,
            bar: None
        }
    }

    fn foo(&mut self) -> &String {
        if self.foo.is_none() {
            let message = String::from("foo");
            self.foo = Some(message);
        }

        self.foo.as_ref().unwrap()
    }

    fn bar(&mut self) -> &String {
        if self.bar.is_none() {
            let message = String::from("bar");
            self.bar = Some(message);
        }

        self.bar.as_ref().unwrap()
    }
}

fn print(context: &mut Context) {
    let foo = context.foo();
    let bar = context.bar();

    println!("{}{}", foo, bar);
}

fn main() {
    let mut context = Context::new();

    print(&mut context);
}
```
(
[Gist](https://gist.github.com/NebulaFox/897a513604a9612597e9ebc5d31bd624), 
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=897a513604a9612597e9ebc5d31bd624)
)

The above code will not complie
```
error[E0499]: cannot borrow `*context` as mutable more than once at a time
  --> src/main.rs:35:15
   |
34 |     let foo = context.foo();
   |               ------- first mutable borrow occurs here
35 |     let bar = context.bar();
   |               ^^^^^^^ second mutable borrow occurs here
36 | 
37 |     println!("{}{}", foo, bar);
   |                      --- first borrow later used here

error: aborting due to previous error

For more information about this error, try `rustc --explain E0499`.
error: could not compile `playground`.
```

My assumption was the first `&mut` would be released before the second one occurs.
I asked this on [StackOverflow](https://stackoverflow.com/questions/60591782/unexpected-second-mutable-borrow-on-a-struct)
and got the following explanation.

> The assumption that the first `&mut` is released before the second one is incorrect.
>
> The function: `fn foo(&mut self) -> &String` has the lifetimes elided,
> so more fully is `fn foo<'a>(&'a mut self) -> &'a String`, so the lifetime of the `&mut`
> is that of the returned String which is assigned to foo.
>
> Since the lifetime of foo will be until the `println!` macro,
> the `&mut` reference will have the same lifetime,
> preventing it from being borrowed by the second reference.

### Solution: Builder Pattern

```rust
struct Context {
    foo: String,
    bar: String
}

struct ContextBuilder {
    foo: Option<String>,
    bar: Option<String>
}

impl ContextBuilder {
    fn new() -> Self {
        ContextBuilder {
            foo: None,
            bar: None
        }
    }

    fn load_foo(mut self) -> Self {
        if self.foo.is_none() {
            let message = String::from("foo");
            self.foo = Some(message);
        }

        self
    }

    fn load_bar(mut self) -> Self {
        if self.bar.is_none() {
            let message = String::from("bar");
            self.bar = Some(message);
        }

        self
    }
    
    fn build(self) -> Context {
        
        Context {
            foo: self.foo.unwrap(),
            bar: self.bar.unwrap()
        }
    }
}

fn print(context: &Context) {
    let foo = &context.foo;
    let bar = &context.bar;

    println!("{}{}", foo, bar);
}

fn main() {
    let context = ContextBuilder::new()
        .load_foo()
        .load_bar()
        .build();

    print(&context);
}
```
(
[Gist](https://gist.github.com/NebulaFox/897a513604a9612597e9ebc5d31bd624),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=8222ec0676cc3ba8aefcf6d001215c65)
)

