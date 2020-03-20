# Demonstration: Coercing Types

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

[NEXT Demonstration: Coercing with a String][demonstrate-coercing-string]

_Page Last Updated: 2020-03-20_

[demonstrate-coercing-types]: ./Demonstrate-Coercing_Types.md
[demonstrate-coercing-string]: ./Demonstrate-Coercing_Types/Demonstrate-Coercing_string.md
[demonstrate-coercing-struct]: ./Demonstrate-Coercing_Types/Demonstrate-Coercing_struct.md
[demonstrate-coercing-wrapped-string]: ./Demonstrate-Coercing_Types/Demonstrate-Coercing_wrapped_string.md