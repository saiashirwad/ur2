# Ur/Web anatomy: the language inside the web compiler

## TL;DR
- Ur's distinctive core is **typed computation over unordered rows**, not HTML: kinds, higher-kinded polymorphism, row `map`, disjointness, and typed record traversal.
- XML and SQL *use* that core; removing those DSLs does not logically remove the record machinery. The implementation is nevertheless interleaved, not a ready-made standalone Ur compiler.
- The hardest front-end work is concentrated in `elaborate.sml` (5,228 lines), normalization, environments, and the 285-line disjointness solver—not all 57,349 lines of SML.
- A folder is an ordinary, higher-rank Ur value encoding a traversal order; **automatic folder synthesis is compiler-special-cased**.
- There are two separate limitations: incomplete type inference, and well-typed programs rejected by the closure-eliminating server backend.
- The Coq development models **Featherweight Ur**, not the production compiler. Inference completeness and the proposed equality-normalization algorithm are not proved.
- This is an orientation report, not a Rust architecture or a decision about which features to retain. Evidence and boundaries follow.

## Scope and how to read the citations

Inspected the vendored `urweb/` tree in this repository (introduced by commit `1669e94b44a6b51ee5b2f33d19da324f556cb1fa`). Paths below are repository-relative; `file:20–40` means lines 20 through 40. Counts are physical lines, including comments and blanks, measured with `wc -l`; they are **not effort estimates**. `urweb/src/*.sml` totals 57,349 lines; the lexer/grammar add 2,982. No compiler build or snippet execution was performed.

Paper abbreviations:

- **[Ur10]** Adam Chlipala, [*Ur: Statically-Typed Metaprogramming with Type-Level Record Computation*](https://adam.chlipala.net/papers/UrPLDI10/UrPLDI10.pdf), PLDI 2010.
- **[Web15]** Adam Chlipala, [*Ur/Web: A Simple Model for Programming the Web*](https://adam.chlipala.net/papers/UrWebPOPL15/UrWebPOPL15.pdf), POPL 2015.

The manual is especially useful, but its phase overview omits passes and repetitions present in today's source. The pipeline below follows **`compiler.sml`, not the manual's shorter narrative**.

## 1. A TypeScript expert's map of Ur

### First distinction: a row is not yet an object type

Ur has three levels worth keeping separate:

| Level | Example | Meaning |
|---|---|---|
| Kind | `{Type}` | Classifies a compile-time record whose entries are types. |
| Constructor | `[A = int, B = string]` | That compile-time record (a row). |
| Type and value | `$[A = int, B = string]`; `{A = 1, B = "hi"}` | The type of runtime records with those fields, and one such value. |

`{A : int, B : string}` is convenient type syntax for the `$` version. Ur calls compile-time entities **constructors**; this is broader than either a TS constructor function or an algebraic-data-type constructor. Kinds classify constructors, while types classify expressions. Sources: `urweb/doc/manual.tex:454–509,619–646`; [Ur10] §§2, 3.1.

### Kinds: types of compile-time things

Kinds include `Type`, `Name`, `Unit`, arrows, records `{K}`, tuples, and kind polymorphism. `Name` holds field labels such as `#A`; `Unit` permits set-like rows `[A, B]` where payloads carry no information. Higher-kinded functions and functions polymorphic over kinds are first-class at the constructor level. (`urweb/doc/manual.tex:456–511`.)

```ur
con boxed :: Type -> Type = fn t :: Type => {Value : t}
con field :: Name = #A
con flags :: {Unit} = [A, B]
```

**TS analogue:** `type Boxed<T> = { Value: T }`, a literal key `"A"`, and a key set. **No direct TS equivalent** to Ur's explicit kind language or general kind-polymorphic constructor functions. A TS generic alias is not itself a freely passable value of kind `Type -> Type`.

### Rows and `$`: structural programming without width subtyping

Rows can be abstract, passed as parameters, and concatenated when their domains do not overlap. They are unordered for type equality. `$` turns a row of types into a runtime record type; it does not create a value. (`urweb/doc/manual.tex:484–497,806–880`; [Ur10] §§3.1–3.2.)

```ur
con fields = [A = int, B = string]
val example : $fields = {A = 1, B = "hi"}
```

**TS analogue:** `type Fields = { A: number; B: string }`. TS collapses the row/type distinction in this example. Do not import TS's usual width-subtyping intuition: Ur deliberately lacks subtyping; record polymorphism supplies many of the same use cases. `[Ur10] §3.2` explains the ambiguity subtyping would create for symmetric concatenation.

### `map`: mapped types, with algebraic equalities built in

`map` changes every row payload while retaining names. The compiler understands identity, distribution over concatenation, and map fusion even for abstract rows. This is stronger than merely evaluating a mapped type on a known finite object. (`urweb/doc/manual.tex:839–880`; [Ur10] §3.2, Figure 3.)

```ur
con fields = [A = int, B = string]
con optionalFields = map option fields
(* $optionalFields has A : option int and B : option string. *)
```

**TS analogue:** `type OptionalValues<R> = { [K in keyof R]: R[K] | undefined }`. Here `option` is an explicit sum type, **not optional-property syntax**: Ur still requires both fields. Ur's type-level map can also operate on rows of names, tuples, other rows, etc., according to their kinds.

### Disjointness: no duplicate keys, not “last write wins”

`[r1 ~ r2]` asserts that two rows have no field names in common. It is a static obligation, not a runtime test. Concatenation needs it. The solver decomposes concatenations and maps, then checks pairwise facts; mapping does not change domains. (`urweb/src/disjoint.sml:156–285`; [Ur10] §4.1.)

```ur
fun merge [r1 :: {Type}] [r2 :: {Type}] [r1 ~ r2]
          (x : $r1) (y : $r2) : $(r1 ++ r2) = x ++ y
```

**TS analogue:** a helper constrained by `Extract<keyof A, keyof B> extends never`. That is an approximation, not a built-in proof system. Neither `A & B` nor `{...a, ...b}` means disjoint union: intersections can have overlapping keys and JS spread overwrites them.

### Guarded types: a function usable under a row fact

A guarded type `[r1 ~ r2] => T` means “a `T`, provided those rows are disjoint.” Inside the guard the fact is available; at use sites it must be discharged. It is **not** a TS conditional type selecting one of two results. Guards may occur inside other types, which matters to higher-rank folds. (`urweb/doc/manual.tex:502,806–837,882–976`; `urweb/lib/ur/top.ur:26–37`; [Ur10] §§2, 3.2.)

```ur
(* Signature fragment: a guarded function type. *)
val merge : r1 :: {Type} -> r2 :: {Type} -> [r1 ~ r2]
            => $r1 -> $r2 -> $(r1 ++ r2)
```

**TS analogue:** none directly. An extra proof-like argument or conditional generic constraint can emulate particular APIs, but TS has no matching general guard abstraction/application. The compiler erases these static constraints in Explify (`urweb/doc/manual.tex:2647`).

### Folders: a typed iterator over a heterogeneous row

Rows are unordered. Iteration needs an order, so `folder r` is a **value encoding a permutation of `r`**. `fold` accepts a type function `tf` describing the accumulator after each subset of fields; its step is polymorphic in the next name, payload and remaining row. This makes heterogeneous, type-changing traversal safe. The representation and implementation are written in Ur itself. (`urweb/lib/ur/top.ur:3–24`; `urweb/doc/manual.tex:1451–1465`.)

```ur
(* Count fields, with an int accumulator at every intermediate row. *)
fun countFields [r ::: {Type}] (fl : folder r) =
    @fold [fn _ :: {Type} => int]
         (fn [nm :: Name] [t :: Type] [rest :: {Type}]
             [[nm] ~ rest] (n : int) => n + 1)
         0 fl
```

Here `@fold` makes the normally inferred folder argument explicit (`urweb/doc/manual.tex:645`).

**TS analogue:** runtime key iteration plus a generic fold API, but **no direct equivalent** to this higher-rank, row-indexed accumulator with a disjointness fact at each step. The example's count is deliberately simple; the same interface can grow a typed record one field at a time.

Important boundary: `folder` is library-defined, but the elaborator recognizes `Top.folder` and synthesizes values using `Top.Folder.nil/cons` when the row structure is known (`urweb/src/elaborate.sml:4973–5028`). Do not confuse this with a primitive, order-sensitive type-level fold over unordered rows; [Ur10] §3.1 explains why that would be problematic.

### Constructor classes: implicit dictionaries, not JS classes

Ur classes are constructors specially marked for instance search. Values whose types apply that constructor become instances in scope. A function argument of class type is implicitly filled from that database; the dictionary remains an ordinary value and can also be passed explicitly. Modules can expose abstract classes and hide their dictionary representation. (`urweb/doc/manual.tex:1399–1407`.)

```ur
fun render [t ::: Type] (s : show t) (x : t) : string = show x
```

**TS analogue:** `interface Show<T> { show(x: T): string }` passed to `render`, **without automatic instance search**. Ur's `show` signature is `t ::: Type -> show t -> t -> string` (`urweb/lib/ur/basis.urs:105–113`). Classes are not restricted to `Type -> Type`; SQL uses multi-argument and higher-kinded classes too. The manual warns that general Prolog-like resolution may not terminate.

### Inference: infer uses, annotate polymorphic definitions

Ur does not automatically generalize definitions as ML does. Type parameters must be introduced explicitly. `::` marks explicit constructor arguments, `:::` implicit ones; square-bracket arguments supply constructors. Inference handles many omitted arguments, but not general higher-order unification. (`urweb/doc/manual.tex:470–474,1383–1415`.)

```ur
fun identity [t :: Type] (x : t) : t = x
val one = identity [int] 1
```

**TS analogue:** `const identity = <T>(x: T): T => x`. In both cases generic parameters are explicit at definition and often inferable at calls. But TS expectations about solving arbitrary generic relationships do not transfer to Ur.

A concrete historical limit from **[Ur10] §4**, presented there as an inference failure:

```ur
fun id [f :: Type -> Type] [t] (x : f t) : f t = x
val x = id 0
```

The engine would need to guess a type function `f`, not merely an ordinary type. This is the paper's example, **not a newly reproduced failure of this checkout**. Current documentation independently promises only heuristic inference. Reverse-engineering `map f ?row = knownRow` is a targeted heuristic, not a complete inverse-map algorithm (`urweb/doc/manual.tex:1411`).

## 2. The actual compiler pipeline

### Reading the table

**C** = general language mechanism; **W** = web/SQL/JS-specific; **M** = mixed implementation. “C” does not mean the file can be copied unchanged: exports, FFI and runtime conventions still matter. Every repeated optimization is shown, in actual dependency order. Lines are for the implementation file **once**, not additional code for every invocation. Helper IR/environment/printer files are excluded unless specified.

The complete ordinary executable path is `compile → toChecknest` (`urweb/src/compiler.sml:1685–1688`). Transform composition runs the right-hand transform first (`:1001–1013`). Phase wiring citations are in the last column; implementation filenames identify the measured source.

| # | Phase | IR in → out | Role / classification | Implementation lines; `compiler.sml` wiring |
|---:|---|---|---|---|
| 1 | parseJob | project path → job | Read `.urp`, libraries, settings (**M**) | compiler job reader ~560, `:441–999` |
| 2 | parse | job → Source | Parse files; desugar syntax and add exports (**M**) | grammar 2,404 + lexer 578; driver `:1028–1269` |
| 3 | elaborate | Source → Elab | Kind/type inference, modules, constraints (**M**, mostly C) | elaborate 5,228; `:1271–1291` |
| 4 | unnest | Elab → Elab | Lift nested named functions (**C**) | unnest 568; `:1293–1298` |
| 5 | explify | Elab → Expl | Remove resolved unification scaffolding/static constraints (**C**) | explify 215; `:1307–1312` |
| 6 | corify | Expl → Core | Eliminate modules/functors, expose FFI and exports (**M**) | corify 1,332; `:1314–1319` |
| 7 | core_untangle | Core → Core | Split recursive groups (**C**) | core_untangle 237; `:1333–1338` |
| 8 | shake1 | Core → Core | Remove unreachable declarations from export roots (**M**) | shake 234; `:1340–1345` |
| 9 | especialize1' | Core → Core | Specialize function argument patterns (**C**) | especialize 717; `:1347` |
| 10 | shake1' | Core → Core | Dead-code removal again (**M**) | shake 234; `:1348` |
| 11 | rpcify | Core → Core | Turn mixed-tier calls into RPCs (**W**) | rpcify 168; `:1350–1355` |
| 12 | core_untangle2 | Core → Core | Split recursion again (**C**) | core_untangle 237; `:1357` |
| 13 | shake2 | Core → Core | Dead-code removal (**M**) | shake 234; `:1358` |
| 14 | especialize1 | Core → Core | Argument specialization (**C**) | especialize 717; `:1360` |
| 15 | core_untangle3 | Core → Core | Split recursion (**C**) | core_untangle 237; `:1362` |
| 16 | shake3 | Core → Core | Dead-code removal (**M**) | shake 234; `:1363` |
| 17 | tag | Core → Core | Assign URL identities to handlers (**W**) | tag 356; `:1365–1370` |
| 18 | reduce | Core → Core | Definitional simplification/inlining (**C**) | reduce 954; `:1372–1377` |
| 19 | shakey | Core → Core | Dead-code removal (**M**) | shake 234; `:1379` |
| 20 | unpoly | Core → Core | Specialize polymorphic functions (**C**) | unpoly 336; `:1381–1386` |
| 21 | specialize | Core → Core | Specialize parameterized datatypes (**C**) | specialize 340; `:1388–1393` |
| 22 | shake4 | Core → Core | Dead-code removal (**M**) | shake 234; `:1395` |
| 23 | especialize2 | Core → Core | Argument specialization (**C**) | especialize 717; `:1397` |
| 24 | shake4' | Core → Core | Dead-code removal (**M**) | shake 234; `:1398` |
| 25 | unpoly2 | Core → Core | Function monomorphization again (**C**) | unpoly 336; `:1399` |
| 26 | specialize2 | Core → Core | Datatype specialization again (**C**) | specialize 340; `:1400` |
| 27 | shake4'' | Core → Core | Dead-code removal (**M**); log label repeats `shake4'` | shake 234; `:1401` |
| 28 | especialize3 | Core → Core | Argument specialization (**C**) | especialize 717; `:1402` |
| 29 | specialize3 | Core → Core | Datatype specialization (**C**) | specialize 340; `:1403` |
| 30 | reduce2 | Core → Core | Simplify/inline again (**C**) | reduce 954; `:1405` |
| 31 | shake5 | Core → Core | Dead-code removal (**M**) | shake 234; `:1407` |
| 32 | marshalcheck | Core → Core | Reject disallowed serialized types (**W**) | marshalcheck 157; `:1409–1414` |
| 33 | effectize | Core → Core | Classify handler effects, check safe GET behavior (**W**) | effectize 208; `:1416–1421` |
| 34 | monoize | Core → Mono | Erase higher types; lower XML/SQL/transactions/signals (**M**) | monoize 4,683; `:1430–1435` |
| 35 | endpoints | Mono → Mono | Collect HTTP endpoint metadata (**W**) | endpoints 117; `:1442–1447` |
| 36 | mono_opt1 | Mono → Mono | Local optimizations, especially strings/HTML (**M**) | mono_opt 713; `:1437–1439,1449` |
| 37 | untangle | Mono → Mono | Split recursion (**C**) | untangle 214; `:1451–1456` |
| 38 | mono_reduce | Mono → Mono | Simplify/inline monomorphic code (**C**) | mono_reduce 923; `:1458–1463` |
| 39 | mono_shake1 | Mono → Mono | Dead-code removal (**M**) | mono_shake 166; `:1465–1470` |
| 40 | mono_opt2 | Mono → Mono | Local optimization (**M**) | mono_opt 713; `:1472` |
| 41 | iflow | Mono → Mono | Optional information-flow policy checking (**W**; otherwise identity) | iflow 2,184; `:1474–1479` |
| 42 | namejs | Mono → Mono | Name JS fragments for reuse/reference (**W**) | name_js 173; `:1481–1486` |
| 43 | namejs_untangle | Mono → Mono | Split recursion after naming (**C**) | untangle 214; `:1488` |
| 44 | scriptcheck | Mono → Mono | Classify script/RPC/server-push needs (**W**) | scriptcheck 182; `:1490–1495` |
| 45 | dbmodecheck | Mono → Mono | Classify required database access mode (**W**) | dbmodecheck 86; `:1497–1502` |
| 46 | jscomp | Mono → Mono | Compile client computations into JS fragments (**W**) | jscomp 1,369; `:1504–1509` |
| 47 | mono_opt3 | Mono → Mono | Local optimization (**M**) | mono_opt 713; `:1511` |
| 48 | fuse | Mono → Mono | Push page-output writes into recursive producers (**W**) | fuse 152; `:1513–1518` |
| 49 | untangle2 | Mono → Mono | Split recursion (**C**) | untangle 214; `:1520` |
| 50 | mono_reduce2 | Mono → Mono | Simplify/inline (**C**) | mono_reduce 923; `:1522` |
| 51 | mono_shake2 | Mono → Mono | Dead-code removal (**M**) | mono_shake 166; `:1523` |
| 52 | mono_opt4 | Mono → Mono | Local optimization (**M**) | mono_opt 713; `:1524` |
| 53 | mono_reduce3 | Mono → Mono | Simplify/inline (**C**) | mono_reduce 923; `:1525` |
| 54 | fuse2 | Mono → Mono | Page-output fusion again (**W**) | fuse 152; `:1526` |
| 55 | untangle3 | Mono → Mono | Split recursion (**C**) | untangle 214; `:1527` |
| 56 | mono_shake3 | Mono → Mono | Dead-code removal (**M**) | mono_shake 166; `:1528` |
| 57 | pathcheck | Mono → Mono | Reject duplicate public paths/names (**W**) | pathcheck 115; `:1530–1535` |
| 58 | sidecheck | Mono → Mono | Reject client-only primitives in server code; track environment reads (**W**) | sidecheck 84; `:1537–1542` |
| 59 | sigcheck | Mono → Mono | Thunk values depending on request signature strings (**W**, not module signatures) | sigcheck 97; `:1544–1549` |
| 60 | filecache | Mono → Mono | Instrument file/blob caching operations (**W**) | filecache 233; `:1551–1556` |
| 61 | sqlcache | Mono → Mono | Optional SQL caching instrumentation, preceded by full inlining (**W**) | sqlcache 1,732 + mono_inline 28; `:1558–1566` |
| 62 | cjrize | Mono → Cjr | Lower to C-like first-order representation (**M**) | cjrize 746; `:1568–1573` |
| 63 | prepare | Cjr → Cjr | Extract prepared SQL statements (**W**) | prepare 357; `:1575–1580` |
| 64 | checknest | Cjr → Cjr | Annotate nested prepared-statement use if DBMS needs it (**W**) | checknest 187; `:1582–1587` |
| 65 | C emission | Cjr → C text | Emit program/runtime glue, optionally SQL schema and endpoint report (**M**) | cjr_print 3,959; driver `:1714–1753` |
| 66 | C compile/link | C → object → executable | Invoke external toolchain and link protocol/runtime/DB libraries (**M**) | driver ~150, `:1608–1683,1718–1756` |

Implementation names in the count column are `urweb/src/<name>.sml`. Descriptions of the central transformation families are corroborated by `urweb/doc/manual.tex:2641–2737`. Less obvious checks can be read directly in `urweb/src/sidecheck.sml:43–79`, `sigcheck.sml:35–94`, `dbmodecheck.sml:34–86`, and `prepare.sml:60–357`.

### Branches, not steps in that executable path

These are easy to misread as additional sequential phases:

| Branch | Input → output | What actually happens |
|---|---|---|
| `parseUrs`, `parseUr` | individual source file → Source signature/declarations | Parsing helpers used above, not extra whole-program passes (`compiler.sml:215–277`). |
| `parseJob'` | project path → job plus library list | Alternative project reader, retaining library information (`compiler.sml:988–999`). |
| `termination` | Elab → Elab | 396-line general termination checker; branches from Unnest. **Explify uses `toUnnest`, not `toTermination`**, so normal compilation does not run it (`compiler.sml:1300–1312`). |
| `reduce_local` | Core → Core | 398-line local simplifier; its wiring is commented out (`compiler.sml:1321–1326`). |
| `css` | Core → CSS summary | 322-line CSS report branch from Shake5, not on the ordinary path (`compiler.sml:1423–1428`). |
| `sqlify` | Mono → Cjr, printed as SQL | Reuses Cjrize after MonoOpt2; separate inspection/output transform (`compiler.sml:1589–1594`). Normal `compile` instead optionally emits SQL from the final Cjr result (`:1732–1740`). |

The IR definitions themselves are small: Source 194, Elab 205, Expl 167, Core 147, Mono 174 and Cjr 141 lines (`urweb/src/{source,elab,expl,core,mono,cjr}.sml`). The size lives in traversals, environments, inference and lowering, not just AST declarations.

## 3. Where the difficult core lives

**Do not equate “type system” with only `disjoint.sml`.** The disjointness prover is small because constructor normalization and scoped unification do much of the work around it.

| Area | Location and physical size | Why it matters |
|---|---|---|
| Kind checking and constructor elaboration | `urweb/src/elaborate.sml:95–616`, ~520 lines | Kinds, constructor abstractions/applications, row formation, guard checking and generated disjointness obligations. |
| Constructor/row unification | `elaborate.sml:621–1465`, ~845 lines; row entry point `:801`, summary normalization `:826–879`, summary solving `:881–1159` | Normalizes rows into explicit fields and unknown pieces; cancels matching pieces; solves/delays constraints; reverse-engineers maps. This is a principal novel core. |
| Scope-sensitive normalization and substitution support | `urweb/src/elab_ops.sml`, 517 lines; head normalization `:157–325`, full reduction `:327–515` | Implements constructor computation, including map over rows and map composition; needed for definitional equality, not a web feature. |
| Disjointness facts and goals | `urweb/src/disjoint.sml`, 285 lines; decomposition `:168–221`, assertion `:223–251`, proving `:253–285` | Tracks atomic domain-disjointness facts, decomposes concatenations/maps, and returns unresolved goals. |
| Implicit arguments and expression checking | `elaborate.sml:1467–1577,2139–2535`, ~510 lines combined | Threads disjointness/class goals through expression elaboration and inserts inferred arguments. |
| Environment and class search | `urweb/src/elab_env.sml`, 1,721 lines total; class representation `:179–215`, resolution `:696–827` (~130 lines) | Binders, modules, scopes and instance rules are inseparable from getting inference right. Not all 1,721 lines are class search. |
| Modules/signatures and declaration elaboration | `elaborate.sml:2600–4840`, ~2,240 lines, mixed | Abstract types, signature matching, functors and hidden constraints; also table/cookie/style declarations. Large, and not all novel row logic. |
| Final constraint solving and folder synthesis | `elaborate.sml:4843–5228`, ~386 lines | Initializes built-ins, revisits deferred goals, generates folder terms and reports remaining failures. |
| Error explanation | `urweb/src/elab_err.sml`, 446 lines; `elab_print.sml`, 916 lines | Rich internal constructors must become comprehensible diagnostics; failure cases include scope depth, row leftovers and unresolved classes. |

Supporting traversals are substantial: `elab_util.sml` 1,320 lines and `elab_util_pos.sml` 917. They should not be counted as additional novel inference algorithms. The whole-file total for `elaborate + elab_ops + disjoint + elab_env` is **7,751 lines**, but contains both generic and web-related material. The ranges above are reading landmarks, not non-overlapping effort buckets.

The distinctive interaction is:

1. **Normalize enough** to expose constructor structure.
2. **Unify rows algebraically**, rather than compare object-property lists syntactically.
3. **Prove domain separation** under scoped hypotheses.
4. **Retry delayed problems**, since solving one unknown can expose another row/map.
5. **Synthesize traversal witnesses** once enough row structure is available.

This directly matches [Ur10] §§4.1–4.4 and `elaborate.sml:777–1159,4973–5100`. It is not dependent typing over arbitrary runtime values: [Ur10] §§1, 3 deliberately isolates type-level computation without general term dependency.

## 4. What is web-entangled—and what is not?

### XML and SQL typing: generic substrate, special front/back ends

The right answer to “library or special case?” is **both, at different layers**.

| Layer | Evidence | Consequence |
|---|---|---|
| Type-level substrate | `Basis.xml :: {Unit} -> {Type} -> {Type} -> Type`; `tag`/`join` use ordinary guards and row concatenation (`urweb/lib/ur/basis.urs:762–791`). | XML contexts, form inputs and bindings exercise the same row system available to user code. |
| SQL typing API | Nested rows represent table environments; `sql_subset` uses `map` over pairs, and joins use disjointness (`basis.urs:375–419`). | SQL's structural typing is largely encoded as ordinary constructors, classes and functions, not a separate SQL unifier. [Ur10] §5 explicitly describes this generic-inference approach. |
| Syntax | XML grammar expands to `Basis.cdata`, `Basis.join`, etc. (`urweb/src/urweb.grm:1452–1479,1611–1628`); SQL syntax is another explicit extension (`manual.tex:2287–2393`). | Removing literals is a parser change, not removal of `$`, map or guards. XML syntax is Ur/Web's own notation, not literally JSX. |
| Declarations/elaboration | Named built-ins `sql_table`, `sql_view`, `http_cookie`, `css_class` are referenced by elaboration (`elaborate.sml:2537–2543`); table constraints get dedicated treatment (`:2772–2783,4456`). | The production front end is not perfectly DSL-neutral. |
| Lowering/runtime | Monoize recognizes `Basis.xml`, `transaction`, `signal`, `sql_query` by name (`urweb/src/monoize.sml:257–292`), and lowers their operations (`:1142–1290,1758–1784,3032–3038`). | “Encoded in the library” does **not** mean implemented with an ordinary portable Ur library. |
| Tier/effect checks | Client/server restrictions are checked during whole-program compilation, not by the core type system (`manual.tex:1470`; `sidecheck.sml:43–79`). | Removing tiering does not remove a special tier kind from Ur's kind system. |

The logical dependency therefore runs **web DSL → generic row machinery**, not the reverse. That observation does not establish a clean source-code deletion boundary.

### Standard library footprint

`lib/ur/` has **19 files and 4,742 physical lines**: nine `.ur` implementations and ten `.urs` signatures. Basis has only a signature; the other nine modules have implementation/signature pairs. The inventory below separates API entanglement from file size.

| Module/file group | Lines, implementation + signature | Entanglement found |
|---|---:|---|
| `basis.urs` | 1,234 | Generic primitives/classes plus transactions, FRP, HTTP, SQL, HTML, tasks, security. SQL section `:256–737` alone is 482 lines; XML/HTML begins `:738`. |
| `top.ur` + `top.urs` | 774 | Generic folders, maps and folds (`top.urs:1–132`) mixed with XML maps/query helpers (`:133–305`), HTTP parsing (`:309`), and generic tail utilities. |
| `list.ur` + `list.urs` | 749 | Mostly lists; XML functions `list.urs:33–48`, SQL query combinators `:91–107`. |
| `listPair.ur` + `listPair.urs` | 101 | Generic paired lists plus XML mapping (`listPair.urs:3–5`). |
| `string.ur` + `string.urs` | 150 | Generic string helpers plus HTML newline conversion (`string.urs:33`). |
| `datetime.ur` + `datetime.urs` | 175 | Mostly date conversion; `now : transaction t` (`datetime.urs:33`). |
| `char.ur` + `char.urs` | 38 | Generic character helpers (`char.urs:1–19`). |
| `option.ur` + `option.urs` | 84 | Generic option helpers (`option.urs:1–18`). |
| `monad.ur` + `monad.urs` | 230 | Generic monad combinators (`monad.urs:1–90`); a monad need not be a web transaction. |
| `json.ur` + `json.urs` | 1,207 | Generic typed serialization API (`json.urs:1–57`), but implementation uses HTML-valued errors (`json.ur:7–8,36–47`). |

**This is an API/source footprint, not a certified removable percentage.** The two automatically opened modules, Basis and Top, are intentionally mixed (`manual.tex:1420–1424`; `compiler.sml:1271–1286`). Even apparently generic errors use HTML: `Basis.error : t ::: Type -> xbody -> t` (`basis.urs:1165–1167`). A core-only runtime would therefore encounter dependencies outside obviously web-named functions.

The type-system ingredients behind folders, records, polymorphic variants, module abstraction and implicit dictionaries do not require SQL or FRP. By contrast, transaction retry, channel delivery, request-local storage and reactive GUI updates are implementation semantics for web effects, not consequences of row soundness ([Web15] §§3.1–3.3). This report leaves the replacement effect/runtime model undecided.

## 5. Known friction: evidence, not folklore

### Build setup

The manual expects a Unix toolchain, MLton, OpenSSL and ICU; database development libraries are additional. It warns about MLton's memory needs and platform-specific configuration (`urweb/doc/manual.tex:54–101`). Repository builds also have generated/build tooling, so this is not a one-command Rust-style dependency graph. Historical user reports include:

- [Issue #268](https://github.com/urweb/urweb/issues/268): the documented `apt-get install urweb` package could not be found on Debian 12/Linux Mint 22.
- [Issue #260](https://github.com/urweb/urweb/issues/260): Homebrew/macOS ICU configuration and MySQL linking trouble, culminating in `dyld: missing symbol called`.
- [Issue #255](https://github.com/urweb/urweb/issues/255): newer GCC warnings promoted to errors during the C runtime build.

These are reported experiences, not proof the current checkout fails on those systems. I did not install the build toolchain or reproduce them.

### Error messages expose the inference engine

Row errors deliberately show unmatched normal-form remainders (`manual.tex:1397`). Implementation diagnostics include “Can't unify record constructors,” “TooDeep,” and constructor-scope failures (`urweb/src/elab_err.sml:111–175`). This can be useful to a language implementer while being intimidating to someone expecting TS-style property errors.

Concrete reports:

| Report | What it demonstrates—not a stronger conclusion |
|---|---|
| [#266: `firstSome`](https://github.com/urweb/urweb/issues/266) | A short recursive list helper hit “too-deep unification variables.” Evidence of inference ergonomics, not a row-soundness counterexample. |
| [#267: table inside a structure](https://github.com/urweb/urweb/issues/267) | Hidden SQL constraint rows yielded “Couldn't prove field name disjointness”; moving the table changed behavior for the reporter. Shows module/DSL/constraint interaction. |
| [#269: GROUP BY / ORDER BY](https://github.com/urweb/urweb/issues/269) | Reporter found an invalid query accepted and a valid aggregate variant rejected with final record-unification leftovers. Distinguish correctness of a DSL signature/encoding from soundness of the host calculus. |
| [#242: folders and “Unsupported expression”](https://github.com/urweb/urweb/issues/242) | A functor-based workaround changed compilation success. The symptom may be downstream compilation, not merely failure to infer a folder. |

Issue bodies were read through the upstream GitHub API; the reports are samples, not prevalence estimates or verified current regressions. No mailing-list evidence is needed to establish these particular observations, and no mailing-list survey was performed.

### Syntax has a real learning cost

- `:` versus `::` versus `:::`, `->` versus `=>`, and several kinds of square-bracket arguments encode genuinely different layers (`manual.tex:470–509`). [Issue #197](https://github.com/urweb/urweb/issues/197) explicitly asks why guarded types use `=>`; the reporter describes confusion in folder signatures.
- [Issue #204](https://github.com/urweb/urweb/issues/204) complained that reserved `Name` blocked a natural record/table field name. This is **historical**, not a claim that the restriction persists after subsequent parser work.
- Project files distinguish directives from source names using the first blank line. The parser has a dedicated diagnostic for directives placed afterward (`compiler.sml:1193–1199`). This is build-file syntax friction rather than a type-theoretic issue.

### Type-correct does not imply this backend can compile it

The manual expressly describes **heuristic compilation**: server programs that remain “too higher order” can fail, despite valid typing (`manual.tex:2633–2635`). Argument order affects specialization; mixing a function and runtime data in one tuple can prevent closure elimination (`:2657–2659`). Polymorphic recursion can defeat monomorphization (`:2685–2691`), and remaining functions-as-data cause Cjrize failure (`:2731–2733`).

This is separate from undecidable/incomplete inference. A Rust rewrite retaining the type system would not automatically inherit—or automatically solve—these particular backend restrictions.

## 6. What is actually formalized?

### Three claims that should not be collapsed

| Claim | Evidence and limit |
|---|---|
| **Featherweight Ur has a type-preserving meaning in CIC.** | [Ur10] §3.3 translates kinds, constructors and typed expressions to Coq/CIC. Soundness is inherited by construction from the target, rather than established through a separate Ur small-step progress/preservation proof. It also inherits strong normalization, which the paper explicitly says does **not** hold for full Ur. |
| **Ur inference is not complete.** | [Ur10] §4 states inference is undecidable (already implicating System F's impredicative polymorphism), uses explicit polymorphism annotations and first-order heuristics, and leaves possible restricted completeness results for future work. The manual confirms no completeness claim (`manual.tex:1383–1411`). |
| **The proposed normalization strategy is not a proved decision procedure here.** | [Ur10] §4 calls the oriented rewrite system *conjecturally* terminating and confluent, treating commutative row concatenation separately. This does not justify claiming either a proved decidable full checker or that all fully annotated Ur type checking has been proved undecidable. |

### What's in `src/coq/`

Only a small simplified development—not a verification of the SML compiler:

- `Syntax.v` **186 lines**: kind-indexed constructors, equality/disjointness judgments and typed expressions. Its kinds are just Type, Name, arrows and records (`urweb/src/coq/Syntax.v:36–57`); the syntax is visibly smaller than production Ur. Typed expressions end at `:172–186`, without the production web runtime or module language.
- `Semantics.v` **251 lines**: denotations for kinds/constructors, substitution correctness, `deq_correct` and `disj_correct`, and a typed expression interpreter (`:43–79,133–145,169–251`). These establish that the modeled equality/disjointness rules respect the denotation.
- `Axioms.v` **47 lines**: assumes dependent functional extensionality as `ext_eq` (`:31–33`), then derives two consequences. This is an explicit assumption, not an axiom-free development.
- `Name.v` **31 lines**, plus build files. The README says it has only been tested with **Coq 8.3pl2** (`urweb/src/coq/README:1–3`). I did not rerun it on either that version or modern Coq.

No `Admitted` was found in these files, but that fact alone is not a compiler-correctness claim. No proof was found here for the SML inference implementation, its termination/completeness, module elaboration, whole-program optimization, SQL emission, C/JS code generation or runtime behavior. The development's actual modeled syntax and semantics delimit its scope.

### What the web paper adds (and an abstract discrepancy)

The fetched **[Web15] PDF** is principally a tutorial, implementation account and evaluation. §§2–3 describe atomic server actions, cooperative client threads, transaction restart and FRP. §3.1 explicitly notes that database backends with weaker isolation weaken the intended semantics; §§3.2–3.3 explain commit-time message delivery and GUI dependency tracking.

The author's [landing-page abstract](https://adam.chlipala.net/papers/UrWebPOPL15/) says it “formalize[s] the basic programming model with operational semantics.” **That sentence is absent from the linked PDF's abstract**, and I found no formal rules/theorem section in that PDF (its sections are Introduction, Tutorial, Implementation, Evaluation, Related Work, Conclusion). I therefore do **not** attribute a mechanized distributed-runtime soundness proof to this paper. This is a source-version discrepancy, not evidence that no other version or development exists.

## What you could read first (three items)

1. **[Ur10] §2, then §§3–4** — build the mental model from record projection and folders, then see why algebraic row equality and incomplete inference matter.
2. **`urweb/lib/ur/top.ur:3–48` alongside `top.urs:1–20,61–131`** — see the folder encoding and useful generic record APIs without starting in SQL or HTML.
3. **`urweb/src/elaborate.sml:801–1159` alongside `disjoint.sml:168–285`** — the implementation's row-unification and disjointness center; keep `elab_ops.sml:157–310` open for normalization.
