# Demonstrate: Coercing a wrapped String

[Demonstration: Coercing Types][demonstrate-coercing-types]
1. [Demonstration: Coercing with a String][demonstrate-coercing-string]
2. [Demonstration: Coercing with a struct][demonstrate-coercing-struct]
3. Demonstration: Coercing a wrapped String

## Using Option\<T> 

If there is a function with parameter type `Option<&str>`,
it might be expected that an `Option<&String>`
would _deref coerce_ into `Option<&str>`.
From [Demonstrate: Coercing Structs][demonstrate-coercing-struct],
_deref coercing_ is just the `Deref` trait.
Dereferencing an `Option` or `Result` does not make much sense,
and would not coerce into a wrapped `&str`.

Trait bounds can be used in the case of function parameter

```rust
use rand; // 0.7.3
use rand::Rng;

fn make_a_string_with_optional<S>(option: Option<S>) -> String
where
    S: AsRef<str>,
{
    let mut s = String::from("Hello");
    if let Some(string) = option {
        s.push_str(" ");
        s.push_str(string.as_ref());
    }

    let mut rng = rand::thread_rng();
    let r = rng.gen_range(0, 100);
    for _ in 0..r {
        s.push_str("!");
    }
    s
}

fn main() {
    let world_slice = "World";
    let world_string = String::from(world_slice);

    let none = make_a_string_with_optional(None::<&str>);
    let some_literal = make_a_string_with_optional(Some("Ferris"));
    let some_slice = make_a_string_with_optional(Some(world_slice));
    let some_string = make_a_string_with_optional(Some(String::from(world_slice)));
    let some_ref_string = make_a_string_with_optional(Some(&world_string));

    let array: &[String] = &[none, some_literal, some_slice, some_string, some_ref_string];

    for s in array.iter() {
        println!("len: {}, string: {}", s.len(), s)
    }
}
```
(
[Gist](https://gist.github.com/NebulaFox/5456bb20fadeb06ec7cb893b3b55fa18),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=5456bb20fadeb06ec7cb893b3b55fa18)
)

For a variable definition, a convert function is needed

```rust
fn convert<S>(option: Option<S>) -> Option<String>
where
    S: Into<String>,
{
    match option {
        Some(string) => Some(string.into()),
        None => None
    }
}

fn is_ferris<S>(option: Option<S>) -> bool
where
    S: AsRef<str>,
{
    match option {
        Some(string) => string.as_ref() == "Ferris",
        None => false
    }
}

fn main() {
    let ferris: Option<String> = Some("Ferris").map(|s| s.into());
    let world = convert(Some("World"));

    let array: &[Option<String>] = &[None, ferris, world];

    for o in array.iter() {
        match o {
            Some(string) => println!("Hello {}", string),
            None => println!("PANIC!")
        }
        
    }
    

    let ferris: Option<&str> = Some("Ferris").map(|s| s.as_ref());
    let world: Option<String> = Some(String::from("World"));
    println!("Is it Ferris: none: {}, ferris: {}, world: {}",
        is_ferris(None::<&str>),
        is_ferris(ferris),
        is_ferris(world)
    );
}
```
(
[Gist](https://gist.github.com/NebulaFox/ddf29b2505ab57bded9348db817730b1),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=ddf29b2505ab57bded9348db817730b1)
)

## References

* [StackOverflow: Why does Option<&String> not coerce to Option<&str>?](https://stackoverflow.com/questions/34974732/why-does-optionstring-not-coerce-to-optionstr?rq=1)

[PREV Demonstrate: Coercing a struct][demonstrate-coercing-struct] •
[NEXT Demonstrate: Coercing Types][demonstrate-coercing-types]

_Page Last Updated: 2020-03-20_

[demonstrate-coercing-types]: ../Demonstrate-Coercing_Types.md
[demonstrate-coercing-string]: ./Demonstrate-Coercing_string.md
[demonstrate-coercing-struct]: ./Demonstrate-Coercing_struct.md
[demonstrate-coercing-wrapped-string]: ./Demonstrate-Coercing_wrapped_string.md