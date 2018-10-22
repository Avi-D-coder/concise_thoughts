---
title: "Ideal Function Semantics"
date: 2018-05-28T12:40:01-04:00
draft: false
---

#### This post is part of the [`ideal_lang` project](https://github.com/Avi-D-coder/ideal_lang.git)

See my upcoming blog posts for the reasons behind my ideal semantics.

- [explicit_proven_and_asserted_constraints]\(avi-d-coder.github.io/posts/**not written yet**)
- [function_overloading_in_depth]\(avi-d-coder.github.io/posts/**not written yet**)
- [constraints_on_functions]\(avi-d-coder.github.io/posts/**not written yet**)

Questions, comments, concerns and alternatives specs should be directed at the [`ideal_lang` project repository](https://github.com/Avi-D-coder/ideal_lang.git).

## Summary


Functions are rust like with:

- a more concise declaration syntax
- default arguments
- optional named arguments when calling
- currying based on patterns matching
- an immutable variant is generated for every mutating function
- logical constraints on function arguments, call locations, and return types

## Declaration syntax
#### Simplest Form:
```rust
function_name(argument_name: Type) -> Type {
        //function_body
}
```

#### Full Form
```rust
function_name: Constraints_on_Function
    // Types are a form of Constraints
    (argument_name: Trait_Argument_Constraints = default_value) Argument_Constraints
    -> Return_Type_Constraints {
        //function_body
    }
```
#### Examples
##### An add function could be written as:
```rust
add(n1: Int, n2: Int) -> Int {
    n1 + n2
}
```
##### or as a more general function:
```rust
add(numbers: Iterator<Summable>) -> Summable {
    numbers.sum()
}
```

##### So far this is almost rust.
##### What is different?
```rust
/// `by != 0` is a compile time constraint value.
/// To call div the complier must prove the constraint expression evaluates to true at compile time.
/// In this case the expression is `by` cannot be 0.
div(n: Int, by: Int) by != 0 -> Int {
    n / by
}
```
##### How can we write this in a less ad hoc fashion?
```rust
// Lift the assertion `!= 0` into the type system NonZero
trait NonZero(n: Number) {
    n != 0
}

/// `n` is constrained take any type that implements the Integer Trait.
/// `by` is constrained take any type that implements the Integer and NonZero Traits.
/// `div` has the signature`(Integer, Integer & NonZero) -> Integer
div(n: Integer, by: Integer & NonZero) -> Integer {
    n / by
}
```

##### Proven Bounds:
```rust
/// This `div` function is defined only where `n >= by` and both arguments are `Integer`s,
/// hence the return type conforms to NonZero
/// `div` has the signature:
/// `(n: Integer, by: Integer & NonZero) n >= by -> Integer & NonZero`
div(n: Integer, by: Integer & NonZero) n >= by -> Integer {
    n / by
}
```

##### Function Overloading
ideal has both constraint and type based Overloading.
A function may have different bodies and signatures based on the constraints the call conforms to, and the caller provided types.
By default the function implementation that is most specialized to the caller's types and constraints is executed.
At compile time, it is proven that the implementations of the called function exist for all caller required types and constraints.
In other words we can have all the previous functions and a `div` function that returns a `Option<Integer>` and we should not have to unnecessarily unwrap `Option`.
More on this in a future commit.
```rust
div(n: Integer, by: Integer & !NonZero) -> Integer {
    match by {
        0 => None,
        _ => Some(n / by)
    }
}
```
Or we can just let the compiler construct all of these definitions of `div`, since by convention anything that could fail is wrapped in a `Option` or a `Result`.
This includes mathematical operations, overflows and even allocation (resource usage is a constraint).
The constraint solver ensures you should almost never have to match against these wrapped values.

##### Default Arguments and Type Variables
```rust
// `qty` and `mult`'s default_value is 1
increment(n: Mut N, qty: N = 1, mult: N = 1) -> N {
    /// Increment `n` by `qty` * `mult`
    n += qty * mult;
    n
}
```

## Call Semantics
##### Call Syntax
```rust
increment(1, 1, 1) == 2

// with optional named arguments
increment(n = 1, qty = 2, mult = 1) == 2
```
##### Default Arguments
```rust
// Default arguments may be omitted when:
// the ommited argument is the last argument,
// or when the out of order arguments are named.

// both expand to increment(n = 1, qty = 2, mult = 1)
increment(1) == 2
increment(n = 1) == 2

// expand to increment(n = 1, qty = 1, mult = 2)
increment(1, mult = 2) == 3
```

##### Pattern Currying
```rust
// currying all arguments returns a function with the type of the original
increment(_, _, _) // returns increment(n: Mut N, qty = N = 1, mult: N = 1) -> N
// returns increment(n: Mut N, qty: N = 1, mult: N = 1) -> N
increment(n =_, qty = _, mult = _)

// to foward a argument use a `_` in a function call
// a new function will be returned with any arguments given as default values
increment(_, _, 2): (n: Mut N, qty: N = 1, mult: N = 2) -> N
increment(n = _, qty = _, mult = 2) // returns (n: Mut N, qty: N = 1, mult: N = 2) -> N

increment(n = 1, qty = _) // returns (n: Mut N = 1, qty: N = 1, mult: N = 1) -> N
increment(qty = 2) == 3
```

##### Constraints on Functions
```rust
Trait Acid {
    // code that guarantees all calls that depend
    // directly or indirectly on the result of `query` occur as one transaction.
    // A Future post will elaborate on this concept.
}

Trait AcidDataBaseTable {
    query: Acid (self, where: Query) -> ReturnType(Query);
    set: Acid (self: Mut, where: Query);
    insert: Acid (self: Mut, item: Item);
    remove: Acid (self: Mut, where: Query);
}
```
##### Alternatives
Pass a transaction constructor object that is passed as a argument in order to use argument constraints.
The Invariant constructor objects approach seems less ergonomic.
As with all of this I would love your feedback.

##### footnotes:
1. () denote values
2. <> denote Types/Traits
3. In the `div` functions `n / by` would really be written `unsafe_div_primitive(numerator= n, denominator= by)`
