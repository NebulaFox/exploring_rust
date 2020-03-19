# Challenge 2: Contains 

## When `Vec`s contains Strings

### Problem

```rust
fn main() {
    let array_str = ["a", "b", "c"];
    let array_string = ["a".to_string(), "b".to_string(), "c".to_string()];

    let value_str = "b";
    let value_string = String::from(value_str);

    let result_str = array_str.contains(&value_string);
    let result_string = array_string.contains(&value_str);
    let result_literal = array_string.contains("b");

    println!(
        "result: {} {} {}",
        result_str, result_string, result_literal
    );
}
```
(
[Gist](https://gist.github.com/NebulaFox/e6baa2bc937ab0c5f293c09c1fdd95cb),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=e6baa2bc937ab0c5f293c09c1fdd95cb)
)

Results in a compiler error

```
error[E0308]: mismatched types
  --> src/main.rs:12:41
   |
12 |     let result_str = array_str.contains(&value_string);
   |                                         ^^^^^^^^^^^^^ expected `&str`, found struct `std::string::String`
   |
   = note: expected reference `&&str`
              found reference `&std::string::String`

error[E0308]: mismatched types
  --> src/main.rs:13:47
   |
13 |     let result_string = array_string.contains(&value_str);
   |                                               ^^^^^^^^^^ expected struct `std::string::String`, found `&str`
   |
   = note: expected reference `&std::string::String`
              found reference `&&str`

error[E0308]: mismatched types
  --> src/main.rs:14:48
   |
14 |     let result_literal = array_string.contains("b");
   |                                                ^^^ expected struct `std::string::String`, found `str`
   |
   = note: expected reference `&std::string::String`
              found reference `&'static str`

error: aborting due to 3 previous errors
```

At the very least, it would be expected that `&String` would be
coerced into a `&str`.
The reason is due to
the the type signature of `Vec::contains`
([doc](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.contains))

```rust
impl<T> Vec<T> {

// --snip--

    pub fn contains(&self, x: &T) -> bool
    where
        T: PartialEq,
    {
        // --snip--
    }
}
```

`Vec::contains` only accepts parameters of the same type as its elements.

### Solution

```rust
fn contains<T, Q>(arr: &[T], value: &Q) -> bool
where
    T: std::cmp::PartialEq<Q>,
{
    arr.iter().any(|v| v == value)
}

fn main() {
    let array_str = ["a", "b", "c"];
    let array_string = ["a".to_string(), "b".to_string(), "c".to_string()];

    let value_str = "b";
    let value_string = String::from(value_str);

    let result_str = contains(&array_str, &value_string);
    let result_string = contains(&array_string, &value_str);
    let result_literal = contains(&array_string, &"b"); // needs an &

    println!(
        "result: {} {} {}",
        result_str, result_string, result_literal
    );
}
```
(
[Gist](https://gist.github.com/NebulaFox/b13476b7369c521faa8f9788bc8f869a),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=b13476b7369c521faa8f9788bc8f869a)
)

Here the workaround is used `arr.iter().any(|v| == value)`,
and encapsulated in a function `contains`
with appropriate type signature.

But what if `HashSet` is used?

## When `HashSet`s contains Strings 

### Problem

```rust
use std::collections::HashSet;

fn main() {
    let hs_str: HashSet<&str> = ["a", "b", "c"].iter().copied().collect();
    let hs_string: HashSet<String> = vec![
        "a".to_string(), 
        "b".to_string(),
        "c".to_string()
    ]
        .into_iter()
        .collect();

    let value_str = "b";
    let value_string = String::from(value_str);

    let result_str = hs_str.contains(&value_string);
    let result_string = hs_string.contains(&value_str);
    let result_literal = hs_string.contains("a");

    println!(
        "result: {} {} {}",
        result_str, result_string, result_literal
    );
}
```
(
[Gist](https://gist.github.com/NebulaFox/bb5cf515622246c050d1b6a52e029254),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=bb5cf515622246c050d1b6a52e029254)
)

Gives the compile error

```
error[E0277]: the trait bound `&str: std::borrow::Borrow<std::string::String>` is not satisfied
  --> src/main.rs:13:29
   |
13 |     let result_str = hs_str.contains(&value_string);
   |                             ^^^^^^^^ the trait `std::borrow::Borrow<std::string::String>` is not implemented for `&str`

error[E0277]: the trait bound `std::string::String: std::borrow::Borrow<&str>` is not satisfied
  --> src/main.rs:14:35
   |
14 |     let result_string = hs_string.contains(&value_str);
   |                                   ^^^^^^^^ the trait `std::borrow::Borrow<&str>` is not implemented for `std::string::String`
   |
   = help: the following implementations were found:
             <std::string::String as std::borrow::Borrow<str>>

error: aborting due to 2 previous errors
```

The same problem exists, `&String` is not being coerced into a `&str`. 
Notice this time the `result_literal` is compiled.
Again the reason is due to
the type signature of `HashSet::contains`

```rust

impl<T, S> HashSet<T, S>
where
    T: Eq + Hash,
    S: BuildHasher,
{
    // --snip--
    pub fn contains<Q: ?Sized>(&self, value: &Q) -> bool
    where
        T: Borrow<Q>,
        Q: Hash + Eq,
    {
        //--snip--
    }
}
```

Not that different to `Vec::contains`. In fact, there was an attempt to make the two the same
[Github: rust-lang #43020](https://github.com/rust-lang/rust/pull/43020).
Unfortunately, it broke existing code so the change never
got integrated.

```rust
impl<T> Vec<T> {

    // --snip--

    pub fn contains<Q: ?Sized>(&self, x: &Q) -> bool
    where T: Borrow<Q>,
          Q: PartialEq
    {
     // --snip--
    }

}
```

It is the `Borrow` trait that allows
the string literal to compile.
Since `String` is implements `Borrow<str>`,
so `Q` is a `str`.

### Solution

The `contains` function only needs arr to
be of type `HashSet` instead of `[T]`

```rust
use std::collections::HashSet;

fn contains<T, Q>(arr: &HashSet<T>, value: &Q) -> bool
where
    T: std::cmp::PartialEq<Q>,
{
    arr.iter().any(|v| v == value)
}

fn main() {
    let hs_str: HashSet<&str> = ["a", "b", "c"].iter().copied().collect();
    let hs_string: HashSet<String> = vec!["a".to_string(), "b".to_string(), "c".to_string()]
        .into_iter()
        .collect();

    let value_str = "b";
    let value_string = String::from(value_str);

    let result_str = contains(&hs_str, &value_string);
    let result_string = contains(&hs_string, &value_str);
    let result_literal = contains(&hs_string, &"a"); // need to a &

    println!(
        "result: {} {} {}",
        result_str, result_string, result_literal
    );
}
```
(
[Gist](https://gist.github.com/NebulaFox/17e7be1dc4aad92f04c0c57dfe5f63e3),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=17e7be1dc4aad92f04c0c57dfe5f63e3)
)

## Third Contains 

There are two versions of `contains`, one for `Vec`
and one for `HashSet`.
Another version can support both `Vec` and `HashSet`.

```rust
#[allow(dead_code)]
fn contains<'a, V, T, Q>(arr: V, value: &Q) -> bool
where
    T: 'a + std::cmp::PartialEq<Q>,
    V: std::iter::IntoIterator<Item = &'a T>
{
    arr.into_iter().any(|v| v == value)
}

#[test]
fn test_contains_hash_set() {
    use std::collections::HashSet;

    let hs_str: HashSet<&str> = ["a", "b", "c"].iter().copied().collect();
    let hs_string: HashSet<String> = vec!["a".to_string(), "b".to_string(), "c".to_string()]
        .into_iter()
        .collect();

    let value_str = "b";
    let value_string = String::from(value_str);
    
    assert!(contains(&hs_str, &value_string));
    assert!(contains(&hs_string, &value_str));
    assert!(contains(&hs_string, &"a"));
}

#[test]
fn test_contains_vec() {
    let array_str = ["a", "b", "c"];
    let array_string = ["a".to_string(), "b".to_string(), "c".to_string()];

    let value_str = "b";
    let value_string = String::from(value_str);

    assert!(contains(&array_str, &value_string));
    assert!(contains(&array_string, &value_str));
    assert!(contains(&array_string, &"b"));
}
```
(
[Gist](https://gist.github.com/NebulaFox/7181c8da0ccfe94ad2f4132bf2382a8f),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=7181c8da0ccfe94ad2f4132bf2382a8f)
)

## References

* [StackOverflow: Why does a &str not coerce to a &String when using Vec::contains?](https://stackoverflow.com/questions/48985924/why-does-a-str-not-coerce-to-a-string-when-using-veccontains)
* [StackOverflow: Difference between iter() and into_iter() on a shared, borrowed Vec?](https://stackoverflow.com/questions/32234954/difference-between-iter-and-into-iter-on-a-shared-borrowed-vec)
* [StackOverflow: What is the difference between iter and into_iter?](https://stackoverflow.com/questions/34733811/what-is-the-difference-between-iter-and-into-iter)

_Page Last Updated: 2020-03-19_