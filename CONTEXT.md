# ur2

A new language that keeps Ur's core type system (type-level computation over records) with a readable syntax and a Rust toolchain. No web features.

## The type system

**Row**:
A compile-time, unordered collection of named fields, such as `[A = int, B = string]`. It is not itself the type of any value.
_Avoid_: field list, object type

**Record type**:
The type of runtime values built from a row: `$[A = int, B = string]`, for example.
_Avoid_: object, struct

**Kind**:
The "type of a type-level thing". `Type`, `Name` and `{Type}` (a row of types) are kinds.

**Constructor**:
Ur's word for any type-level thing: a type, a name, a row, or a function over them. Unrelated to a class or a data constructor.

**Disjointness**:
The fact that two rows share no field names, written `r1 ~ r2`. It is proved at compile time and needed before two records can be concatenated.
_Avoid_: non-overlap

**Row unification**:
Deciding whether two rows, possibly with unknown parts, can be made equal, and what the unknowns must be.

**Unification variable**:
A placeholder for a type (or row) the checker doesn't know yet, filled in as unification solves equations. Written `?a` or `?r` in our notes.
_Avoid_: type hole, inference variable, metavariable (all common in the literature; we use one name)

**Normalization**:
Rewriting a type-level expression into a standard form so that two expressions can be compared.

**Folder**:
A value that fixes an order for walking a row's fields, so code can loop over a record in a typed way.
_Avoid_: iterator

## How we work

**Kata**:
A small, standalone implementation of one core algorithm, written by the user to learn what it does and whether it can be simpler.
