
# Demonstrate: Coercing with a string

[Demonstration: Coercing Types][demonstrate-coercing-types]
1. Demonstration: Coercing with a String
2. [Demonstration: Coercing with a struct][demonstrate-coercing-struct]
3. [Demonstration: Coercing a wrapped String][demonstrate-coercing-wrapped-string]

## Deref Coercing

A `&String` can be coerced into a `&str`.
This is called _deref coercing_.
Wherever you expecting a `&str`
you can pass `&String`.

_Deref coercing_ into a variable definition

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

_Deref coercing_ into a function parameter

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

## Trait "Coercing"

A `String` cannot be _coerced_ into a `&str`.
One is a concrete type the other is a reference.
However, using the conversion traits, a function can have a type signature
that accepts a `&str`, `String` and `&String`.

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
    let slice = "Hello World"; // &'static str
    let string = String::from(slice); // also slice.to_string() or slice.to_owned()
    let moved_r_string = string.clone();
    let moved_i_string = string.clone();
    let ref_string = &string;

    // how long is a piece of string
    // coercing into &str
    let s_literal = make_a_string_with("panic");
    let s_slice = make_a_string_with(slice);
    let s_string = make_a_string_with(&string); // have to reference
    let s_ref_string = make_a_string_with(ref_string);

    // matching type with trait AsRef<str>
    let r_literal = make_a_string_with_ref("panic");
    let r_slice = make_a_string_with_ref(slice);
    let r_string = make_a_string_with_ref(moved_r_string); // moves
    let r_ref_string = make_a_string_with_ref(ref_string);

    // matching type with trait Into<String>
    let i_literal = make_a_string_with_into("panic");
    let i_slice = make_a_string_with_into(slice);
    let i_string = make_a_string_with_into(moved_i_string); // moves
    let i_ref_string = make_a_string_with_into(ref_string);

    let array:[(&str, String); 12] = [
        ("panic", s_literal),
        (slice, s_slice),
        (&string, s_string),
        (ref_string, s_ref_string),
        ("panic", r_literal),
        (slice, r_slice),
        (&string, r_string),
        (ref_string, r_ref_string),
        ("panic", i_literal),
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

### Conversion Traits

* `AsRef` ([doc][as-ref]) is a reference-reference conversion trait.
* `AsMut` ([doc][as-mut]) is similar but between mutable references.
* `Into` ([doc][into]) is a value-value conversion trait.
It consumes the input value.
* `From` ([doc][from]) is the reciprocal of `Into`.
  If a type implements `Into` its better off implementing `From`.

None of the above is allow to fail when converting between types.
If there is type conversion can fail, its better to implement

* `TryInto` ([doc][try-into]) is a failable version of `Into`
* `TryFrom` ([doc][try-from]) is a failable version of `From`

[PREV Demonstrate: Coercing Types][demonstrate-coercing-types] •
[NEXT Demonstrate: Coercing String][demonstrate-coercing-struct]

_Page Last Updated: 2020-03-20_

[from]: https://doc.rust-lang.org/std/convert/trait.From.html
[into]: https://doc.rust-lang.org/std/convert/trait.Into.html
[as-ref]: https://doc.rust-lang.org/std/convert/trait.AsRef.html
[as-mut]: https://doc.rust-lang.org/std/convert/trait.AsMut.html
[try-into]: https://doc.rust-lang.org/std/convert/trait.TryInto.html
[try-from]: https://doc.rust-lang.org/std/convert/trait.TryFrom.html

[demonstrate-coercing-types]: ../Demonstrate-Coercing_Types.md
[demonstrate-coercing-string]: ./Demonstrate-Coercing_string.md
[demonstrate-coercing-struct]: ./Demonstrate-Coercing_struct.md
[demonstrate-coercing-wrapped-string]: ./Demonstrate-Coercing_wrapped_string.md