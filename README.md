# Exploring Rust

Collection of insights and challenges faced while programming in Rust

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

## [Challenge 1: Lazy Loading][challenge-lazy-loading]

Using `Option<T>` to and `&mut self` in methods calls
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
```

([Read more][challenge-lazy-loading])

## [Challenge 2: Contains][challenge-contains]

Look at the cases when `Vec::contains` and `HashSet::contains` fail,
and how to get around it.
Extract that solution into two functions
for `Vec` and `HashSet`.
And go even further, by adapting the function
to work for both `Vec` and `HastSet`

([Read More](challeng-contains))

---
_Page Last Updated: 2020-03-19_


[challenge-lazy-loading]: ./pages/Challenge-Lazy_Loading.html
[challenge-contains]: ./pages/Challenge-Contains.html