## Lifetimes

HIGHKEY forgot everything I learned in this video, will have to rewatch

## Declarative Macros

Helpful reference (little book of rust macros): https://danielkeep.github.io/tlborm/book/README.html

Declarative macros are like C function like macros, except they work on the AST rather than direct text. Declarative macros begin with the `macro_rules!` macro. The high level explanation is that a declarative macro is a large switch statement of *grammatically* correct patterns which consume the input and return a single output, (which can be a block!).

Let's take a look at an example, which let me simplify creating an instruction enum by making all binary enums the same pattern.

```rust
#[macro_export]
macro_rules! define_instructions {
    (
        custom {
            $(
                $custom_variant:ident
                    $( ( $( $tuple_ty:ty ),* $(,)? ) )?
                    $( { $( $field:ident : $field_ty:ty ),* $(,)? } )?
            ),* $(,)?
        }
        binary {
            $( $bin_variant:ident ),* $(,)?
        }
    ) => {
        #[derive(Debug, Clone, PartialEq, Eq)]
        enum Inst {
            $(
                $custom_variant
                    $( ( $( $tuple_ty ),* ) )?
                    $( { $( $field : $field_ty ),* } )?,
            )*
            $(
                $bin_variant {
                    l: InstId,
                    r: InstId,
                },
            )*
        }
    };
}
```
This only takes one pattern, a `custom` block and a `binary` block. We allow two patterns inside custom, a tuple variant, and a struct variant. `$tuple_ty:ty`, for example, expects any type as input, and names it `tuple_ty`. The rest of the syntax is very similar to regex, `*` means zero or more, `+` means one or more, `?` means zero or one. The names that are included in the body repeat as many times as the repetitions in the input, which is how Rust knows when to stop. For something with more patterns, lets take a look at the standard library `vec![]` macro.
```rust
macro_rules! vec {
    () => (
        $crate::vec::Vec::new()
    );
    ($elem:expr; $n:expr) => (
        $crate::vec::from_elem($elem, $n)
    );
    ($($x:expr),+ $(,)?) => (
        <[_]>::into_vec(
            $crate::boxed::box_new([$($x),+])
        )
    );
}
```
The exact implementations aren't too important, but we can see the pattern matching in play! The first pattern, `() => ( .. )`, is the empty case, which corresponds to `vec![];`. The second case is the *count* case, `vec![0, 32];`, for example. The last case corresponds to just placing in elements, `vec![1, 2, 3];`, for example. We can see that we expect at least one comma separated expression, with an optional trailing comma. We can see that we can use similar regex operators in the body of the final pattern.

A few misc. things to note:
- To make a macro public, you must add `#[macro_export]` above it
- If you try to take two inputs both with repetition, be careful including them together into the same expression, as Rust won't know how to expand something with 5 inputs and 3 inputs into the same expression.