# Exploring Rust

Collection of insights and challenges faced while programming in Rust

The intention of this collection is to focus on someone who
is new to Rust but not new to programming.
Languages such as Kotlin and Swift,
have cultivated a certain mindset,
which is _challenged_ in Rust,
mostly in the form of Rust's ownership system.

It is assumed, the reader has read [Rust Programming Language Book](https://doc.rust-lang.org/book/)

#### Resources

 * [Rust Programming Language Book](https://doc.rust-lang.org/book/)
 * [Rust by Example](https://doc.rust-lang.org/stable/rust-by-example/)
 * [Rust Cookbook](https://rust-lang-nursery.github.io/rust-cookbook/intro.html)

## [Challenge 1: Lazy Loading][challenge-lazy-loading]

Using `Option<T>` and `&mut self` in methods calls
to load data.

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

    fn foo(&mut self) -> &str {
        if self.foo.is_none() {
            let message = String::from("foo");
            self.foo = Some(message);
        }

        self.foo.as_ref().unwrap()
    }

    fn bar(&mut self) -> &str {
        if self.bar.is_none() {
            let message = String::from("bar");
            self.bar = Some(message);
        }

        self.bar.as_ref().unwrap()
    }
}
```

([Read more][challenge-lazy-loading])

## [Demonstration: Coercing Types][demonstrate-coercing-types]

This Demonstration is split into three parts.

1. [Demonstration: Coercing with a String][demonstrate-coercing-string]
2. [Demonstration: Coercing with a struct][demonstrate-coercing-struct]
3. [Demonstration: Coercing a wrapped String][demonstrate-coercing-wrapped-string]

Shows situations where `&String` can be coerced into `&str`
and where it does not.
Also how to define generic types with trait bounds
to get around cases where it does not coerce.
How the three traits -
`Deref`, `Into` and `AsRef` -
can be used and applied.
Extra notes on how currently a trait cannot be set for all `T`.

## [Challenge 2: Contains][challenge-contains]

Look at the cases when `Vec::contains` and `HashSet::contains` fail,
and how to get around it.
Extract that solution into two functions
for `Vec` and `HashSet`.
And go even further, by adapting the function
to work for both `Vec` and `HastSet`

([Read More][challenge-contains])


_Page Last Updated: 2020-03-20_


[challenge-lazy-loading]: ./pages/Challenge-Lazy_Loading.md
[challenge-contains]: ./pages/Challenge-Contains.md
[demonstrate-coercing-types]: ./pages/Demonstrate-Coercing_Types.md
[demonstrate-coercing-string]: ./pages/Demonstrate-Coercing_Types/Demonstrate-Coercing_string.md
[demonstrate-coercing-struct]: ./pages/Demonstrate-Coercing_Types/Demonstrate-Coercing_struct.md
[demonstrate-coercing-wrapped-string]: ./pages/Demonstrate-Coercing_Types/Demonstrate-Coercing_wrapped_string.md