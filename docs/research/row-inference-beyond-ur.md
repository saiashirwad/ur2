# Row type inference beyond Ur

## TL;DR

- **No verified drop-in replacement emerged:** the algorithms with strong inference guarantees cover less than Ur; the newer systems that cover the four requested capabilities do not supply a complete inference algorithm for that combination. [R, G, L, Ro, H, T]
- **Rose is the strongest reference for abstract records plus disjoint concatenation**, but its 2019 inference results do not include arbitrary type-level mapping or a loop over unknown fields. [Ro §4.5; H §§2.3–3]
- **Rω (2023) and its 2026 successor are the closest feature matches.** They add mapping and generic field operations, but change the story about traversal order, and the 2026 paper explicitly leaves inference to future work. [H §§3, 7; T §3 n.11]
- **PureScript demonstrates a practical alternative:** mapping and folding through library constraints, with an ordered view generated from a fully known row—not Ur's built-in type-level `map` plus a caller-selected folder. [P1–P5]
- **v0 implication:** borrow small row-solving techniques, not a claim of complete inference for all of Ur. Preserving all four features while requiring more annotations is a plausible engineering direction, not a theorem established by this survey. [U, H, T]

Research for [ticket #8](https://github.com/saiashirwad/ur2/issues/8), 2026-09-29. Recommendations below are research conclusions, not a change to the project's standing decision that Ur's core rules are the specification.

## What exactly are we comparing?

A **row** is a compile-time collection of field names and their associated types; a **record** is a runtime value whose shape is described by a row. An **abstract row** is a row parameter whose fields are not known inside a generic function. **Row polymorphism** means one function works for many choices of that row. [U, Core Syntax; H §2.2]

The four tests are deliberately stronger than “supports extensible objects”: [U, Core Syntax and `Top`]

1. Use a record with an abstract row, without listing all fields.
2. Concatenate two abstract records, requiring **disjointness**: neither row contains a field name found in the other.
3. Compute `map F r` at the type level: preserve the labels of `r`, replacing each field type `T` with `F T`, even while `r` is abstract.
4. Write a **typed fold**, a loop whose step knows the current field's type. Ur additionally permits an accumulator type that depends on which fields have been visited, and a **folder**, a value specifying visitation order.

For TypeScript intuition, test 3 resembles `{ [K in keyof R]: Array<R[K]> }`, but here we also need a generic runtime operation that traverses the matching record without losing the relationship between each key and its value type. This is explanatory pseudocode, not an equivalence claim about TypeScript and Ur. [U, `map` and `Top`; H §3.3]

**Type inference** fills in omitted types; **type checking** validates types supplied by the programmer. An algorithm is **decidable** if it always finishes with an answer, **sound** if its answers obey the typing rules, and **complete** if it can find a typing whenever those rules allow one. A **principal type** is a most-general type from which the other allowed types follow by specializing parameters. These guarantees are always relative to particular rules—not transferable to every feature of another language. [R §2.3; G §4; L §7; Ro §4.5]

## Capability map

“Yes” means supported by the cited system, not identical to Ur in all details. “Not provided” describes the cited calculus or API, not an impossibility result about extending it. A **calculus** is a small formal language used to specify and study language features.

| System | Abstract records | Two abstract rows + disjoint concatenation | General type-level `map` | Typed field loop | Inference result and boundary |
|---|---|---|---|---|---|
| Rémy, natural extension of ML | Yes | Not in this paper | Not provided | Not provided | Decidable inference and principal types for its record system. [R §§2.2–2.4] |
| Gaster & Jones, 1996 | Yes | Not provided; adds one field with an absence condition | Limited `to`/`from` mapping proposals, not general `map` | Not an Ur-style fold | Sound, complete inference with principal types for the base system; mapping extensions have an unresolved empty-row issue. [G §§3–4, 6.1] |
| Leijen, scoped labels, 2005 | Yes | Not provided; field extension permits duplicates rather than requiring disjointness | Not provided | Not provided | Sound and complete row unification; carries into the chosen surrounding inference system. [L §§6–7, 9] |
| Rose, 2019, simple-row theory | Yes | Yes | Not provided | Not provided | Principal qualified types; paper reports decidable inference and proves principality using sound/complete inference correspondences. Scope is Rose, not its later higher-order extensions. [Ro §§1, 3–4.5; H §2.3] |
| PureScript + record/heterogeneous libraries | Yes | Yes, `disjointUnion` | Encoded as a relation between input/output rows | Yes, library mapping/folding | Working compiler/library machinery; no whole-system completeness or decidability theorem established by the inspected sources. [P1–P5] |
| Koka's 2014 effect-row calculus | Effect rows, not this record API | Not a record-concatenation feature | Not provided for records | Not provided for records | Sound, complete, principal inference for the paper's type-and-effect rules; terminating row unification. [K §3, Appendix A] |
| Rω, 2023, simple rows | Yes | Yes | Yes, lifting type operations over rows | Yes, with different order/accumulator guarantees | Formal soundness model; no complete inference algorithm established here. [H §§3–5, 7] |
| Toohey et al., 2026 | Yes | Yes for unique-label commutative rows | Yes, `Lift` | Yes, `ind`, with ordering choices | Sound elaboration proved; inference explicitly future work. [T §§2–3, 6; §3 n.11] |

## 1. The classic algorithms solve a smaller problem

**Unification** solves equations between types containing unknowns, choosing values for those unknowns. Rémy gives decidable row unification with a most-general solution (Theorem 1), then connects it to principal typing and decidable inference (Theorem 2 and its record application). The paper explicitly excludes concatenation from its supported record operations. This makes it a strong implementation reference for ordinary open records, but not a replacement for all of Ur's row computations. [R abstract, §§2.2–2.4]

Gaster & Jones use **qualified types**: types carrying conditions that must be satisfied at use sites. Their “lacks” condition states that a row does not contain a particular label. It safely types adding one field; it is not the same operation as combining two arbitrarily large unknown rows. Theorems 1 and 2 establish most-general unification and sound/complete inference for their base rules. [G §§3.3–4.2]

There is an important near miss: §6.1 proposes `to A r`, replacing each field type `T` by `T -> A`, and the corresponding `from` operation. But `to A empty = to B empty` does not imply `A = B`. The authors explicitly identify the empty-row complication, discuss excluding empty rows or distinguishing empty from nonempty rows, and leave further investigation to future work. Do not attach the base-system completeness theorem to unrestricted mapping. [G §6.1]

Leijen's **scoped labels** allow repeated field names, with a newer field hiding an older one until removed. This avoids needing an absence condition for each extension. The paper supplies soundness/completeness results for row unification and explains how those results carry into surrounding inference algorithms. Its primitive is field extension, not general record concatenation. Adopting it would simplify one problem by changing Ur's unique-label/disjointness semantics, not by solving the same problem. [L §§2–3, 7, 9]

**v0 trade-off:** these designs offer an understandable, well-specified first row solver if general concatenation, mapping, and folders are deferred. Keeping those features means further work beyond the classic guarantees. Rémy also wrote later work on concatenation; the “not provided” entry above is intentionally about *Natural Extension of ML*, not a claim about his entire research. [R abstract; L §9 bibliography; Ro §5]

## 2. Rose: keep the relationship instead of guessing

A **row theory** specifies which rows are valid and how they can be combined. Rose parameterizes its typing rules by such a theory. Its simple-row theory requires distinct names and allows disjoint combination; its scoped-row theory supports repeated labels. [Ro §3]

Consider this illustrative function:

```text
pickX(left, right) = concatenate(left, right).x
```

Which argument supplies `x`? Committing to either one prematurely loses valid calls. Rose instead keeps conditions equivalent to “`left` and `right` combine into `whole`, and `whole` contains `x`.” A **constraint** is just such a condition retained for later resolution. This is the central benefit of qualified types for concatenation, and the paper's version of this example motivates the design. [Ro §§1, 2.2]

Rose's Theorem 11 establishes principal types; the proof outline adapts an inference algorithm and establishes soundness/completeness correspondences. Its introduction claims decidable inference. However, Rose is a framework: its results should not be advertised as a solver for arbitrary extra row operations or arbitrary added theories. [Ro §§1, 4.5]

The cost is a richer constraint language and **evidence**, compiler-generated values that justify constraints and implement operations such as locating record fields. Rose also needs a condition ensuring **coherence**, meaning different valid ways of checking a program do not change its runtime meaning. The paper exhibits ambiguous programs and states its coherence result with an explicit restriction (Theorem 15). [Ro §§4.3–4.5]

**v0 trade-off:** Rose is the best reference here if abstract concatenation matters but mapping/folding can wait. It does not itself supply the other two features: the later Rω paper explicitly identifies Rose's inability to express generic operations over the types within a row. [H §§2.3–3]

## 3. PureScript and Koka: useful, but different evidence

PureScript's compiler-provided `Union` combines rows **including duplicate labels**. `Nub` removes duplicates; `Lacks` asserts absence of one label. The record library's `disjointUnion` combines `Union r1 r2 r3` with `Nub r3 r3`, requiring a result that already has no duplicates. So “PureScript supports disjoint concatenation” is supported by an actual API, but “all PureScript row union is disjoint” is false. [P1, P2]

A **type class** describes an operation or relationship at the type level, with implementations selected for particular types. PureScript's heterogeneous library expresses mapping as classes relating input and output rows, and folds as classes relating successive accumulator types. A **heterogeneous** record has fields of different types. Its record instances use `RowToList`, whose compiler documentation explicitly says it generates an ordered, label-sorted list from a **closed row**, one with all fields known. [P3–P5]

Generic functions can retain these class constraints while their row is abstract; the restriction does not mean all generic functions must be rejected. It means constructing the needed list/evidence is not arbitrary evaluation of Ur's `map` over an unknown row. **v0 trade-off:** workable typed traversals, but extra class machinery, different inferred signatures, and a default label order rather than a caller-supplied folder. No completeness claim for this combination was confirmed. [P3–P5; U, `Top`]

An **effect row** tracks computational behaviors such as exceptions rather than a record's fields. Koka's 2014 paper uses duplicate effect labels to support principal unification without extra absence constraints, and proves sound/complete inference in Appendix A. That is strong evidence for an effect-row solver, not evidence that Koka supplies the four record features. The theorem is for the paper's rules, not automatically every feature in today's Koka compiler. [K §§2.2, 3, Appendix A]

## 4. Newer work reaches the feature set, not complete inference

**Rω** is a formal language with type-level functions and row-generic operations. Its **lifting** operation applies a type operation to each field of a row—the role required of `map`. It adds generic record synthesis and a fold; it supplies a formal interpretation checked in Agda, a language used to machine-check proofs. These are substantial advances over Rose 2019, not just alternate row-unification implementations. [H §§3–5]

The differences matter. Rω's fold reduces fields to a common result type, then combines those results. For unordered rows the combining operation should give the same answer regardless of grouping and order; otherwise its behavior is unspecified. Ur instead carries the traversal order in a folder and can vary the accumulator type with the processed subrow. Thus Rω's “yes” for a typed loop does **not** establish a drop-in replacement for Ur's full folder interface. [H §3.4, §6; U, `Top`]

Toohey, Chen, Jamalzadeh, and Xie's January 2026 paper adds `All` conditions over field types, `Lift` for mapped rows, and `ind` for folding, including an accumulator type indexed by a row. It distinguishes ordered from unordered row combination. It proves **elaboration soundness**: translating source programs into an explicitly typed core preserves their typing, with proofs checked in Lean. But §3 footnote 11 explicitly leaves type inference to future work and flags the difficulty of type-level functions. This is the closest newer design lead, **not an available complete inference algorithm**. [T §§2.2–2.5, 3, 6]

## What v0 could gain or lose

These are engineering options inferred from the comparisons, not decisions made on behalf of the project:

- **Reduce the feature set:** use Rémy/Gaster–Jones-style open records first. Gain a smaller algorithm with strong guarantees; lose the central abstract concatenation/map/folder combination until later. [R, G]
- **Choose a constraint-oriented language:** use Rose for concatenation, possibly PureScript-style classes for traversal. Gain explicit relationships instead of premature guesses; take on evidence generation and different source-level constraints. Adding mapping still needs its own inference account. [Ro, P3–P5]
- **Preserve Ur's feature set:** keep annotations and explicit folder arguments available, and separate ordinary row equations from equations involving `map`. Ur's manual already documents incomplete cancellation and reverse-map inference. Gain compatibility with the intended core; do not promise complete inference. [U, Type Inference]

**Research recommendation:** do not replace Ur's entire approach solely because a candidate says “principal rows.” First compare small examples: projection from an open record, concatenation where either operand may provide a selected field, mapping an abstract row including the empty case, and a fold whose accumulator type grows with the visited fields. Those examples expose exactly the boundaries documented above. [Ro §2.2; G §6.1; U, `Top`; H §3]

## What I could not confirm

- No primary source inspected establishes decidable, complete inference for **all four features with Ur's exact semantics together**. This is a bounded literature finding, not a proof that such an algorithm cannot exist.
- I did not verify a complete-inference theorem for PureScript plus the heterogeneous libraries, nor infer one from their existence.
- Rω supplies a soundness model, and the 2026 paper supplies sound elaboration; neither is treated here as a proof of complete source inference or of a terminating checker for every configurable row theory.
- No benchmark, compiler build, or cross-language example suite was run. Runtime cost and annotation burden are not measured. The Ur PLDI paper download was blocked; the first-party Ur manual supplies the baseline instead.
- The survey covers the named approaches and especially relevant 2023/2026 successors, not every record calculus or every later Rémy/Pottier development.

## Primary sources

Citations refer to the following author papers, language documentation, or implementation sources. Web sources accessed 2026-09-29; moving source branches may change.

- **[U]** Adam Chlipala, [Ur/Web manual, source](https://github.com/urweb/urweb/blob/master/doc/manual.tex): Core Syntax; Type Inference; `Top` folder/fold definition. Explicitly states heuristic inference with no completeness claim.
- **[R]** Didier Rémy, [*Type Inference for Records in a Natural Extension of ML*](https://gallium.inria.fr/~remy/ftp/taoop1.pdf), author-hosted chapter/report text: abstract, §2, Theorems 1–2.
- **[G]** Benedict R. Gaster and Mark P. Jones, [*A Polymorphic Type System for Extensible Records and Variants*](https://web.cecs.pdx.edu/~mpj/pubs/96-3.pdf), NOTTCS-TR-96-3 (1996): §§3–4, especially Theorems 1–2; §6.1 mapping extensions.
- **[L]** Daan Leijen, [*Extensible Records with Scoped Labels*](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/scopedlabels.pdf), TFP 2005: §§2–3, 6–7, 9. [Author's publication page](https://www.microsoft.com/en-us/research/publication/extensible-records-with-scoped-labels/) also identifies the separate constructive-proofs report.
- **[Ro]** J. Garrett Morris and James McKinna, [*Abstracting Extensible Data Types; Or, Rows by Any Other Name*](https://jgbm.github.io/pubs/morris-popl2019-rows.pdf), POPL 2019, [DOI](https://doi.org/10.1145/3290325): §§2–4.5, Theorems 11–15.
- **[P1]** PureScript, [Prim.Row](https://pursuit.purescript.org/builtins/docs/Prim.Row): `Union`, `Nub`, `Lacks`, `Cons` compiler classes.
- **[P2]** PureScript record library v4.0.0, [Record.Builder](https://pursuit.purescript.org/packages/purescript-record/4.0.0/docs/Record.Builder): `merge`, `union`, `disjointUnion`.
- **[P3]** PureScript, [Prim.RowList](https://pursuit.purescript.org/builtins/docs/Prim.RowList): `RowToList` closed-row and ordering contract.
- **[P4]** Nate Faubion et al., [Heterogeneous.Mapping implementation](https://github.com/natefaubion/purescript-heterogeneous/blob/master/src/Heterogeneous/Mapping.purs): `hmapRecord`, `MapRecordWithIndex` instances.
- **[P5]** Nate Faubion et al., [Heterogeneous.Folding implementation](https://github.com/natefaubion/purescript-heterogeneous/blob/master/src/Heterogeneous/Folding.purs): `hfoldlRecord`, `FoldlRecord` instances.
- **[K]** Daan Leijen, [*Koka: Programming with Row Polymorphic Effect Types*](https://arxiv.org/pdf/1406.2061), 2014: §§2–3 and Appendix A, especially inference and unification results.
- **[H]** Alex Hubers and J. Garrett Morris, [*Generic Programming with Extensible Data Types*](https://arxiv.org/html/2307.08759v2), ICFP 2023, [DOI](https://doi.org/10.1145/3607843): §§2.3, 3, 4–7.
- **[T]** Matthew Toohey, Yanning Chen, Ara Jamalzadeh, and Ningning Xie, [*Extensible Data Types with Ad-Hoc Polymorphism*, extended paper](https://xnning.github.io/papers/popl26extensible-appendix.pdf), POPL 2026, [DOI](https://doi.org/10.1145/3776662): §§2–3, 6; §3 footnote 11 explicitly defers inference.
