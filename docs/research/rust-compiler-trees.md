# How Rust compilers represent trees

## TL;DR

- **Recommendation, not a settled v0 decision:** use a simple abstract syntax tree (AST: source structure without every punctuation/whitespace detail), then a checker representation with typed integer IDs and a separate table for unknown types. rust-analyzer and rustc supply precedents for separating these concerns.[R1], [R3], [A2]
- Intern names: store one copy of each spelling and pass around a small identifier. Do not confuse name equality with two uses referring to the same declaration.[R4], [G4]
- Arena allocation (keeping many nodes in one owner-managed pool) and interning (deduplicating equal values) solve different problems. rustc combines them for types; Gleam instead demonstrates shared types with mutable unknowns.[R2], [G2]
- Rowan's lossless trees preserve the source text; Salsa reuses computations after edits. Neither is required to implement a row checker.[A1], [S1]
- A row is an unordered compile-time collection of named fields. Gluon's explicit row-plus-rest representation is the closest comparison here, but it is not evidence that Ur's type-level computation or disjointness rules come for free.[U1], [L2], [L4]
- The most important trap: an interned type's identity is not the same question as whether two types can be made equal.[R3]

Research for [ticket #9](https://github.com/saiashirwad/ur2/issues/9), read 2026-09-29. Code links below are commit-pinned; documentation links are live unless versioned. Costs and the v0 proposal are engineering deductions from the cited representations, not benchmark results.

## 1. The three things called “a tree”

**Syntax** records what the programmer wrote. **Intermediate representation (IR)** is a compiler's working data model between stages. **Lowering** translates one representation into a simpler one; **desugaring** replaces convenient syntax with more basic forms. rustc's high-level IR (HIR) removes constructs such as `for`, while keeping source locations. Its semantic type representation is separate: two written `u32`s have different source locations but can share one semantic type.[R1], [R3]

**Lossless syntax** keeps all source text, including comments and whitespace. A **span** is a source range used to point back to code. An abstract tree can carry spans without containing enough text to reconstruct the original file. Rowan supports lossless syntax; Gleam's expression nodes instead show semantic fields, spans, children, and type annotations.[A1], [G1]

These are separate choices: retaining comments does not determine how unknown types are solved, and using an arena does not require storing types inside syntax nodes. rust-analyzer explicitly keeps syntax independent of semantic information and lowers it into separate representations.[A1], [A3]

## 2. Rust ownership vocabulary, translated

- **`Box<T>`:** one owner of a separately allocated `T`; it makes recursive enums possible without infinitely large values. rustc's AST uses boxed expression children.[R5]
- **`Vec<T>`:** a growable sequence of values; **slice** means a view of a contiguous sequence. Gleam uses vectors for argument lists.[G1]
- **`Arc<T>`:** shared ownership through an atomic reference count; **reference counting** tracks how many owning handles remain. Gluon's type wrapper explicitly uses it.[L2]
- **`RefCell<T>`:** mutation behind shared references, with borrow rules checked while the program runs. Gleam uses it for unknown types.[G2], [B1]
- **Lifetime**, such as `'ast` or `'tcx`: Rust's way to express how long borrowed data remains valid. Gluon's syntax references and rustc's arena types carry these lifetimes.[L1], [R2]
- **Typed index/ID:** an integer-like handle that can only index a particular category of data, like an `ExprId` rather than a plain number. It still needs the correct owning store; its type alone does not identify which function's store it belongs to.[A2]

In TypeScript terms, the ID-based approach resembles `Map<ExprId, Expr>` plus branded numbers, rather than a web of mutable object references. That is an explanatory analogy, not the literal storage layout of every compiler below.

## 3. rustc: different storage for different stages

The current AST is not “everything in an arena.” Selected expression variants own boxed children. `ThinVec` is rustc's compact vector container; only its role as a sequence matters here.[R5]

```rust
// Verbatim variants from rustc's ExprKind (other variants omitted).
Call(Box<Expr>, ThinVec<Box<Expr>>),
Binary(BinOp, Box<Expr>, Box<Expr>),
```

HIR then mixes references with IDs: a module names its contained items by `ItemId`, with item contents stored separately. Looking an item up lets the compiler record that it was read. `HirId` combines an owner and an owner-local ID; it is not simply a globally stable array offset.[R1]

For semantic types, rustc uses interned, arena-backed `Ty<'tcx>` handles. Construction looks up an existing equal representation before allocating another. The arena is released together, and the lifetime prevents references escaping its lifetime. Names separately use `Symbol(SymbolIndex)` and a string interner.[R2], [R3], [R4]

**Unification** means finding assignments to unknowns that make types equal. A **unification variable** is one such unknown, e.g. `?a`. A **union-find table** groups IDs that have been proven equal and finds a representative for each group. rustc keeps type-variable origins in an indexed vector and equality relationships/known values in a separate unification table; it also records changes so speculative work can be undone.[R6]

```rust
// Selected fields from TypeVariableStorage; comments/other fields omitted.
values: IndexVec<TyVid, TypeVariableData>,
eq_relations: ut::UnificationTableStorage<TyVidEqKey<'tcx>>,
```

**Cost / lesson:** an arena makes lifetime-wide cleanup straightforward, but retains allocations until that lifetime ends and threads ownership lifetimes through APIs. Interning adds a lookup on construction and keeps a deduplication table. These buy sharing and cheap representation comparison, not a complete equality solver: rustc explicitly warns that inference and normalization can make distinct representations denote compatible/equal types.[R2], [R3] **Normalization** means rewriting type-level expressions to a standard form before comparison.[U1], [R3]

## 4. rust-analyzer + Rowan: source fidelity, then IDs

Rowan's **green tree** is immutable shared syntax data; its **red/syntax node** is a navigable view that adds parent/position context. rust-analyzer adds typed AST wrappers over those syntax nodes. Thus an AST wrapper here does not mean a second, separately copied abstract tree. Incomplete input is allowed, so expected children can be absent.[A1], [A3]

The older syntax design guide is explicitly dated 2020. Its conceptual layers remain useful, but do not copy its incidental storage details as current facts: the inspected Rowan source has token trivia fields (comments/whitespace attached to tokens), and its node cache skips nodes with more than three children. It is not universal deduplication of every subtree.[A1], [A4]

After lowering, rust-analyzer uses index-based expression references, separate stores, and source maps (tables connecting internal nodes back to syntax).[A2]

```rust
// Verbatim excerpts from hir.rs and expr_store.rs, collected together.
pub type ExprId = Idx<Expr>;
// An Expr variant:
Call {
    callee: ExprId,
    args: Box<[ExprId]>,
},
// In ExpressionOnlyStore:
exprs: Arena<Expr>,
```

Here `Arena` comes from `la_arena`, an index-addressed store, rather than exposing an arena borrow on every expression edge. Equal-looking expressions retain distinct IDs: the two `1`s in `1 + 1` are different occurrences.[A2]

Types are shared separately. At the inspected commit, `Ty<'db>` wraps an interned reference and uses `rustc_type_ir::TyKind`; inference imports its context from `next_solver`. Do not rely on old descriptions that treat Chalk as the entire current type-checking implementation: the architecture guide still mentions it, but current code has moved on in this area.[A3], [A5]

```rust
// Verbatim from next_solver/ty.rs.
pub struct Ty<'db> {
    pub(super) interned: InternedRef<'db, TyInterned>,
}
```

Names/text can be interned independently of the green tree: the inspected HIR uses `intern::Symbol` and calls `Symbol::intern` for literal text. This is a different concern from Rowan's syntax-node cache.[A2], [A4]

**Cost / lesson:** lossless trees support source-preserving edits but retain text/token structure a checker does not need. Layered wrappers and separate source maps add machinery. IDs avoid borrowed references on each edge, but every access needs the right store and an ID from an old rebuilt store must not be reused blindly. Rowan is justified by editor/refactoring requirements, not by row unification itself.[A1], [A2], [A3]

## 5. Gleam: the simpler shared-object alternative

Gleam's typed expressions use Rust enums with `Box` and `Vec` children, `SrcSpan` locations, and `Arc<Type>` annotations. Selected call fields demonstrate a direct recursive tree.[G1]

```rust
// Verbatim Call variant; its optional-parenthesis comment omitted.
Call {
    location: SrcSpan,
    type_: Arc<Type>,
    fun: Box<Self>,
    arguments: Vec<CallArg<Self>>,
    open_parenthesis: Option<u32>,
},
```

The type graph (a network whose nodes may be shared rather than having one parent) uses `Arc<Type>`. An unknown is a shared mutable cell. The following variants are exact, with unrelated variants/comments omitted.[G2]

```rust
// In Type:
Var { type_: Arc<RefCell<TypeVar>> },
// In TypeVar:
Unbound { id: u64 },
Link { type_: Arc<Type> },
Generic { id: u64 },
```

`Unbound` means “not known yet”; `Link` points to the type discovered for that unknown; `Generic` is a parameter standing for arbitrary types, not an unknown to solve in place. Gleam's named types use `EcoString` for package/module/name and `Vec<Arc<Type>>` for arguments.[G2]

`EcoString` is a compact, reference-counted string that can store short text inline; **clone-on-write** means sharing until a mutation requires a private copy. That is not the same as interning independently created equal strings into one identity. The declarations inspected establish `EcoString` use, not a global symbol-ID scheme.[G2], [G3]

**Cost / lesson:** this is close to ordinary mutable objects in TypeScript and avoids pervasive arena lifetimes. The trade is separate allocations/shared-pointer bookkeeping and runtime borrow checks for cells. `Arc<RefCell<_>>` does not magically make those cells safe to share across threads, and `Arc` sharing alone does not guarantee independently constructed equal types have the same pointer.[G2], [B1], [B2] Gleam is a storage-pattern example here, not evidence of an Ur-style row solver.

## 6. Gluon: especially useful for rows

Gluon is another language implemented in Rust. Its syntax uses arena-borrowed children: `Expr::App` contains `&'ast mut SpannedExpr` and slices of arguments. Its AST arena macro builds typed arenas for node categories. Its semantic `ArcType` instead contains an `Arc` to a type plus cached flags; it also defines a type interner.[L1], [L2]

Gluon's types distinguish a row from a record built from that row. Selected variants below omit serialization attributes, not semantic fields.[L2]

```rust
Record(T),
EmptyRow,
ExtendRow {
    fields: T::Fields,
    rest: T,
},
Variable(TypeVariable),
```

**Open row** means the fields listed so far have an unknown or abstract remainder, represented here by `rest`. **Kind** means the type of a type-level thing: Gluon's comments give `Record` kind `Row -> Type`. Its variables have IDs and kinds, while a `Substitution` stores variable equalities, known type assignments, and undo state separately.[L2], [L3] **Substitution** means a mapping from unknowns to the types that replace them.

```rust
// Verbatim selected Substitution fields.
union: RefCell<ena::unify::InPlaceUnificationTable<UnionByLevel>>,
variables: FixedVec<T>,
types: FixedVecMap<T>,
types_undo: RefCell<Vec<u32>>,
```

Its row unifier handles fields and remaining rows in dedicated code; merging variable IDs alone is not the row algorithm. Gluon also interns names through `Symbols`, while the resulting `Symbol` is reference-counted rather than simply an integer index.[L3], [L4], [G4]

**Cost / lesson:** arena syntax and shared semantic types can coexist. A mutable substitution table can change what a variable means without rewriting every type containing it. The cost is consistently resolving variable IDs through that table, managing rollback where needed, and implementing the actual row rules separately.[L1], [L3], [L4] Gluon provides a useful representation reference, not a drop-in implementation of Ur's richer constructors (type-level values including names, rows and functions) or disjointness (proof that rows share no field names).[U1]

## 7. Salsa is a computation cache, not a tree format

**Incremental computation** reuses earlier work after inputs change. A **query** computes a result from inputs; **memoization** caches that result. Salsa records which inputs a tracked function reads, caches results in its database, and validates/recomputes them after changes. Its current API separates input structs, tracked computations/structs, and interned structs.[S1]

```rust
// Verbatim example from Salsa's current overview.
#[salsa::interned]
struct Word<'db> {
    #[returns(deref)]
    pub text: String,
}
```

The generated `Word` is an integer ID into the database; equal field values produce equal IDs. This is not the same as a local expression's position in an arena.[S1], [A2]

rust-analyzer uses Salsa and deliberately summarizes file structure separately from function bodies, so editing a body need not invalidate unrelated global information. rustc has its own query infrastructure: its guide distinguishes that from using Salsa directly. The guide's old Salsa `query_group` examples should not be treated as the current API.[A3], [R7], [S1]

**Cost / lesson:** retained results and dependency records consume memory, and the program must expose deterministic computations with suitable boundaries. Deterministic means identical inputs yield identical results. A checker may mutate private working tables during one query, but should return an immutable result rather than expose an unknown that a later query silently changes. This recommendation follows from Salsa's input/determinism contract and the mutable-cell alternatives above.[S1], [G2], [L3]

## 8. Proposed starting point for ur2

These are recommendations for a later design decision, not claims that the ticket author has selected an architecture. They apply the precedents above to the project's stated parse → check → interpret destination.[U2]

1. **Start with boxed AST nodes and spans.** Keep original source text separately. Add Rowan only if source-preserving edits or robust editor parsing become a near-term goal. This borrows Gleam's simplicity while leaving the syntax/semantic separation intact.[G1], [A3]
2. **Give checker nodes typed IDs in an append-only store.** Keep source spans and inferred results in separate ID-keyed tables. Own stores per module/body or compilation, and never interpret an ID without that owner. This follows rust-analyzer's separation without its whole editor infrastructure.[A2]
3. **Intern field spellings, but distinguish spelling IDs from declaration IDs.** Use one name interner per chosen compilation context. The same written `x` may name different declarations; a repeated row label needs cheap spelling equality, not accidental scope equality.[R4], [G4]
4. **Keep unknowns in one mutable solver table.** Give each unknown its own ID, kind, origin span, and unknown/solved state. Initially use explicit equality handling; add a union-find implementation or rollback only when the algorithm needs it. rustc and Gluon demonstrate why this state need not live in immutable type nodes.[R6], [L3]
5. **Represent rows explicitly, including their rest.** Preserve the distinction between a row and `$row`, its record type. Do not replace all constructors with a flat field map: ur2 also needs names, type-level functions, normalization, and disjointness. A sorted collection of known labels may be a convenient implementation choice, but does not solve unknown names or row computation.[U1], [L2], [L4]
6. **Delay full type interning and Salsa until measured needs justify them.** Keep construction/equality behind small APIs so storage can change. For interning, immutable nodes containing stable unknown IDs are safer than keys whose contents change when a cell is solved; resolving/normalizing remains the solver's job. The warning is derived from rustc's representation-versus-semantic-equality distinction and Gleam's mutable links.[R3], [G2], [S1]

Suggested validation cases before optimizing: two equal spellings in different scopes; two syntax occurrences of the same type with different error spans; equal rows written in different orders; unifying an open row and checking its remaining fields; rejecting self-containing unknown solutions (**occurs check**: rejecting an assignment that contains the unknown being assigned); disjointness failures; and a type-level computation that must normalize before equality. These are proposed tests, not claims that all four implementations implement Ur's rules.[U1], [R3], [L3], [L4]

## 9. What this research does not establish

- No benchmarks were run. No evidence here chooses `Box`, an arena, or an interner as universally fastest for ur2.
- Source snapshots establish concrete storage patterns, not an exhaustive census of every IR, allocator, interner, or cache in each repository.
- I did not confirm Salsa usage in Gleam or Gluon and make no claim either way. rustc's explicit non-Salsa statement is dated in its guide; the report does not infer future integration plans from it.[R7]
- The Rowan design guide is historical; current inspected source differs in trivia storage and cache policy. Likewise, rust-analyzer's current type code is a better guide than the architecture page's older Chalk wording.[A1], [A4], [A5]
- Gluon's row representation does not confirm compatibility with Ur's higher-kinded row computation, name computation, or disjointness discipline. Those remain language/algorithm questions, not allocation choices.[U1], [L2], [L4]
- The excerpts are explanatory fragments, not a compiling prototype. No implementation or kata was written.

## Primary sources

[R1]: https://rustc-dev-guide.rust-lang.org/hir.html
[R2]: https://rustc-dev-guide.rust-lang.org/memory.html
[R3]: https://rustc-dev-guide.rust-lang.org/ty.html
[R4]: https://github.com/rust-lang/rust/blob/ea6bb45b74c24a026dab4187b1550e97b6985c8a/compiler/rustc_span/src/symbol.rs
[R5]: https://github.com/rust-lang/rust/blob/ea6bb45b74c24a026dab4187b1550e97b6985c8a/compiler/rustc_ast/src/ast.rs#L1749-L1768
[R6]: https://github.com/rust-lang/rust/blob/ea6bb45b74c24a026dab4187b1550e97b6985c8a/compiler/rustc_infer/src/infer/type_variable.rs#L16-L121
[R7]: https://rustc-dev-guide.rust-lang.org/queries/salsa.html
[A1]: https://rust-analyzer.github.io/book/contributing/syntax.html
[A2]: https://github.com/rust-lang/rust-analyzer/blob/03fcb77246f2568adb0e9b2fa60d19c6cc1686f4/crates/hir-def/src/hir.rs
[A3]: https://rust-analyzer.github.io/book/contributing/architecture.html
[A4]: https://github.com/rust-analyzer/rowan/blob/b6f1cb6494a99a2264186d110b1b170989774814/src/green/node_cache.rs#L17-L110
[A5]: https://github.com/rust-lang/rust-analyzer/blob/03fcb77246f2568adb0e9b2fa60d19c6cc1686f4/crates/hir-ty/src/next_solver/ty.rs#L47-L85
[G1]: https://github.com/gleam-lang/gleam/blob/9520541ebda6d01f1c079ed0156a28c257bfbe13/compiler-core/src/ast/typed.rs#L15-L94
[G2]: https://github.com/gleam-lang/gleam/blob/9520541ebda6d01f1c079ed0156a28c257bfbe13/compiler-core/src/type_.rs
[G3]: https://docs.rs/ecow/0.3.1/ecow/
[G4]: https://github.com/gluon-lang/gluon/blob/38ee70113b58b4950ca4924a5ddcf1146edf75f1/base/src/symbol.rs
[L1]: https://github.com/gluon-lang/gluon/blob/38ee70113b58b4950ca4924a5ddcf1146edf75f1/base/src/ast.rs
[L2]: https://github.com/gluon-lang/gluon/blob/38ee70113b58b4950ca4924a5ddcf1146edf75f1/base/src/types/mod.rs
[L3]: https://github.com/gluon-lang/gluon/blob/38ee70113b58b4950ca4924a5ddcf1146edf75f1/check/src/substitution.rs
[L4]: https://github.com/gluon-lang/gluon/blob/38ee70113b58b4950ca4924a5ddcf1146edf75f1/check/src/unify_type.rs#L814-L1000
[S1]: https://salsa-rs.github.io/salsa/overview.html
[U1]: ../../CONTEXT.md
[U2]: https://github.com/saiashirwad/ur2/issues/16
[B1]: https://doc.rust-lang.org/std/cell/struct.RefCell.html
[B2]: https://doc.rust-lang.org/std/sync/struct.Arc.html

Additional code supporting [A2]'s arena/source-map discussion: [ExpressionStore](https://github.com/rust-lang/rust-analyzer/blob/03fcb77246f2568adb0e9b2fa60d19c6cc1686f4/crates/hir-def/src/expr_store.rs#L112-L270). Supporting [A5]'s inference-context discussion: [infer/unify.rs](https://github.com/rust-lang/rust-analyzer/blob/03fcb77246f2568adb0e9b2fa60d19c6cc1686f4/crates/hir-ty/src/infer/unify.rs#L1-L35).
