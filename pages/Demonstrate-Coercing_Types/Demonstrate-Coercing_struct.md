
# Demonstrate: Coercing with a struct

[Demonstration: Coercing Types][demonstrate-coercing-types]
1. [Demonstration: Coercing with a String][demonstrate-coercing-string]
2. Demonstration: Coercing with a struct
3. [Demonstration: Coercing a wrapped String][demonstrate-coercing-wrapped-string]

## Deref Trait

Otherwise known as _deref coercing_. A complete explanation is given in [The Rust Programming Language,
Chapter 15.2: Treating Smart Pointers Like Regular References
with the Deref Trait](https://doc.rust-lang.org/book/ch15-02-deref.html)

```rust
use std::ops::Deref;
use rand; // 0.7.3
use rand::Rng;

#[derive(Debug, PartialEq)]
struct Wrap<T> {
    inside: T
}

impl<T> Wrap<T> {
    fn new(inside: T) -> Wrap<T> {
        Wrap {
            inside
        }
    }
}

impl<T> Deref for Wrap<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.inside
    }
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
    let string = String::from("Hello World");
    let wrap = Wrap::new(string);
    let result = make_a_string_with(&wrap); // deref coercions
    
    println!("len: {}, string: {}", result.len(), result);
}
```
(
[Gist](https://gist.github.com/NebulaFox/583a88f6d373997e8e7fdfbfa48461a5),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=583a88f6d373997e8e7fdfbfa48461a5)
)

## Into Trait

```rust
use rand; // 0.7.3
use rand::Rng;

#[derive(Debug, PartialEq)]
struct Wrap<T> {
    inside: T
}

impl<T> Wrap<T> {
    fn new(inside: T) -> Wrap<T> {
        Wrap {
            inside
        }
    }
}

impl<T> Into<String> for Wrap<T>
where
    T: Into<String>,
{
    fn into(self) -> String {
        self.inside.into()
    }
}

fn make_a_string_with_into<S>(string: S) -> String
where
    S: Into<String>,
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
    let slice = "Hello World";
    let string = String::from(slice);
    
    let wrap_literal = Wrap::new("panic");
    let wrap_slice = Wrap::new(slice);
    let wrap_string = Wrap::new(String::from(slice));
    let wrap_ref_string = Wrap::new(&string);
    
    // wraps are moved
    
    let array: &[String] = &[
        make_a_string_with_into(wrap_literal),
        make_a_string_with_into(wrap_slice),
        make_a_string_with_into(wrap_string),
        make_a_string_with_into(wrap_ref_string)
    ]; 
    
    for s in array.iter() {
        println!("len: {}, string: {}", s.len(), s)
    }
    
}
```
(
[Gist](https://gist.github.com/NebulaFox/6c950c1b8563cb23d9fe5623c4253c2c),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=6c950c1b8563cb23d9fe5623c4253c2c)
)

## AsRef Trait

```rust
use rand; // 0.7.3
use rand::Rng;

#[derive(Debug, PartialEq)]
struct Wrap<T> {
    inside: T
}

impl<T> Wrap<T> {
    fn new(inside: T) -> Wrap<T> {
        Wrap {
            inside
        }
    }
}

impl<T> AsRef<str> for Wrap<T>
where
    T: AsRef<str>,
{
    fn as_ref(&self) -> &str {
        self.inside.as_ref()
    }
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

fn main() {
    let slice = "Hello World";
    let string = String::from(slice);
    
    let wrap_literal = Wrap::new("panic");
    let wrap_slice = Wrap::new(slice);
    let wrap_string = Wrap::new(String::from(slice));
    let wrap_ref_string = Wrap::new(&string);
    
    // wraps are moved
    
    let array: &[String] = &[
        make_a_string_with_ref(wrap_literal),
        make_a_string_with_ref(wrap_slice),
        make_a_string_with_ref(wrap_string),
        make_a_string_with_ref(wrap_ref_string)
    ]; 
    
    for s in array.iter() {
        println!("len: {}, string: {}", s.len(), s)
    }
    
}
```
(
[Gist](https://gist.github.com/NebulaFox/4b9f3823a728a69f07ce784ede3299f9),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4b9f3823a728a69f07ce784ede3299f9)
)

## Default Implemention

Note that `Wrap<T>` does not implement `Into` generically
```rust
impl<T> Into<T> for Wrap<T> {
    fn into(self) -> T {
        self.inside
    }
}
```
_or_
```rust
impl<T> Into<T> for Wrap<T>
where
    T: Into<T>,
{
    fn into(self) -> T {
        self.inside.into()
    }
}
```


It will result in an [E0119](https://doc.rust-lang.org/stable/error-index.html#E0119) error.
In short, its a preventive measure for a future type
defining its own implementation.
Rust at the time of writing does not support specialization
([rust-lang/rfcs#1210](https://github.com/rust-lang/rfcs/blob/master/text/1210-impl-specialization.md) and issue tracking
[rust-lang/rust#31844](https://github.com/rust-lang/rust/issues/31844)).
This means, a generic version for type `T` cannot be written
and then latter specialized for a concrete type.

_Last Update 2020-03-19_

[PREV Demonstrate: Coercing with a string][demonstrate-coercing-string] •
[NEXT Demonstrate: Coercing a wrapped string][demonstrate-coercing-wrapped-string]

_Page Last Updated: 2020-03-20_

[demonstrate-coercing-types]: ../Demonstrate-Coercing_Types.md
[demonstrate-coercing-string]: ./Demonstrate-Coercing_string.md
[demonstrate-coercing-struct]: ./Demonstrate-Coercing_struct.md
[demonstrate-coercing-wrapped-string]: ./Demonstrate-Coercing_wrapped_string.md