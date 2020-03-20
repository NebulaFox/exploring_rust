# Challenge 1: Lazy Loading

## Problem

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

The above code will not compile
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
    
    fn build_context(self) -> Context {
        let r = self.load_foo()
            .load_bar();
        
        Context {
            foo: r.foo.unwrap(),
            bar: r.bar.unwrap()
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
        .build_context();

    print(&context);
}
```
(
[Gist](https://gist.github.com/NebulaFox/8222ec0676cc3ba8aefcf6d001215c65),
[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=8222ec0676cc3ba8aefcf6d001215c65)
)

_Page Last Updated: 2020-03-19_