# What ur2 should learn from firsthand Ur/Web experience

Research for [ur2 #17](https://github.com/saiashirwad/ur2/issues/17), under [map #16](https://github.com/saiashirwad/ur2/issues/16). Searched/read 2026-09-30. These are research recommendations, **not language-design decisions**. Ur2 retains Ur's core rules, changes syntax/toolchain, and excludes web features from v0.

## Answer in brief

Preserve the ability to express useful abstractions and have the checker keep related data consistent; do not equate advanced types with the user benefit. Production users often valued ordinary composition, refactoring, purity, and less glue code more than metaprogramming itself. There is nevertheless a concrete core-language success: Logitext adopted polymorphic variants for reliable generated JSON serialization [A2].

Change the experience of getting stuck. Four independent long-form authors describe oversized or opaque errors [A3–A6]; one describes repeated abandonment before becoming productive [A5]. More recent, small reproductions show confusing constructor-argument syntax and unification failures even without SQL/XML [A11]. A syntax redesign alone is not shown to solve this.

Protect the edit/check/run loop, explain inference boundaries, and make examples and library documentation discoverable. Do not carry Ur/Web's C/JavaScript code-generation limitations, transaction runtime, or deployment assumptions into the definition of Ur's core. Some historical deficiencies have demonstrably changed: current documentation includes LSP support and inline FFI declarations [T1]. Other reports remain unverified against today's compiler.

## Method, scope, and stopping rule

This is a bounded qualitative corpus, not a survey of the user population. Inclusion requires a first-person account of trying, building, maintaining, or debugging Ur/Web, with at least an identifiable action/outcome. Accounts are primary evidence of what their authors experienced, not automatic proof of a compiler defect or its cause. Tutorials with actual project experience qualify; descriptions of advertised features alone do not. Creator explanations are technical context, not independent satisfaction reports.

The main ledger has **11 distinct authors**: six longer project/tutorial authors (A1–A6), two brief discussion participants (A7–A8), and three issue reporters (A9–A11). Chris Double/doublec, Daniel Patterson/dbp, and steen/steinuil are each counted once across their posts and comments. A2 collaborated with the creator on a technique; A6 contributed LSP support, so these are not disinterested observers. No claim is made that all authors are statistically independent or that anonymous handles identify distinct people with certainty.

Stop after covering: historical and more recent sources; production success, small projects, failed starts and stopping; blogs, public threads, a mailing list and issue discussions; and targeted current checks for potentially misleading historical claims. Stop expanding when the principal themes recur, retaining a core-specific counterexample and a recent core diagnostic example. This session met that coverage rule; it did **not** exhaust the mailing archive, HN, Reddit, or issues. Evidence dates run from 2010 through 2025; 2024 retrospectives sometimes describe much older use.

### Search log

All queries below ran on 2026-09-30. General searches had **no publication-date filter** (historical coverage deliberately allowed). Year-token queries sought newer material, not an enforced date range. Search results were discovery leads; findings below use opened sources.

| Venue / action | Exact query or boundary | Result and disposition |
| --- | --- | --- |
| Web search, initial broad positive/negative | `Ur/Web experience using urweb frustrating review`; `Ur/Web experience blog built application liked` | No results / mostly irrelevant web-development hits. Slash/name ambiguity is a real retrieval limitation. |
| Web search, quoted discovery | `"Ur/Web" "experience"`; `"urweb" "blog"`; `"Ur/Web" "tried"` | Found BazQux, Steen, ezyang, Lobsters, Patterson through links, Bluish Coder, and Yorhel. |
| Web search, negative and historical | `"Ur/Web" "gave up"`; `urweb experience abandoned programming`; `"Ur/Web" "Simple Ur/Web Example"` | First mostly noise; second found 2024 HN thread; third located tutorial/resource list. |
| Web search, recent | `"Ur/Web" "experience" "2024"`; `urweb "experience" "2025"`; `urweb "experience" "2026"` | Mostly irrelevant results, papers, and generic pages; no additional substantive recent retrospective verified. Not evidence that none exist. |
| Link traversal | [awesome-urweb](https://github.com/docelic/awesome-urweb), tutorial links and cross-posts | Found Van Casteren and Double; followed both ezyang records/variants posts. Resource directory is discovery, not satisfaction evidence. |
| Mailing list | BazQux's linked 2014 original | HTTPS timed out; HTTP original readable. Read full account, not merely promotional blog pointer. No archive-wide crawl. |
| Discussion threads | Lobsters 2019, HN 2014 and 2024 (links in ledger) | Read threads; retain explicit users, exclude speculation and customers reviewing BazQux rather than its language. Did not follow all older HN links. |
| GitHub issues via `gh` | `gh issue list --repo urweb/urweb --state all --limit 80 --json number,title,createdAt,url` | Screened newest 80 issues returned, spanning 2018-03-06 through 2025-12-10; read full bodies/comments of #242, #243, #266, chosen for folders, installation, and core checking respectively. Titles alone are not coded as experiences. |
| Current technical check via `gh api` | `doc/manual.tex` on master; then pinned revision `55a881ff9b50d9e5c3b2fd564f5cd44a5cc5e6bc` | Checked LSP, effect annotations, inline FFI, static-file directive and heuristic compilation. No runtime reproduction or benchmark. |
| Other lead | Kent Vilem Liepelt talk announcement [X5] | Production e-commerce use mentioned, but talk itself not watched; year absent from rendered page. Kept as follow-up, not another counted author. |

Coverage gaps: English, publicly indexed, technically articulate authors dominate; long-form successful users are unusually persistent and often experienced functional programmers. No team interview, representative beginner cohort, non-English search, private abandonment history, systematic issue resolution audit, or contemporary compiler installation was performed. Reddit results/cross-posts were discovered but not independently read; this is not a Reddit survey. No date/version is inferred from a page's current copyright year.

## Source ledger: accounts and their contexts

Categories: **C** core language/type-system behavior; **D** syntax, learning or diagnostics; **T** libraries/editor/build/deployment tooling; **W** web-specific behavior; **A** adoption/community. “Current unverified” means the experience is historical and no present-day reproduction establishes persistence. “Open” is tracker state at access time, not proof of a current bug.

### A1 — Chris Double / doublec: simple start, later production and continued use

- Sources: [Simple Ur/Web Example](https://bluishcoder.co.nz/2010/12/14/simple-urweb-example.html), **2010-12-14**; [Lobsters production-use comment](https://lobste.rs/c/dvbnja), **2019-01-05**. No compiler version stated.
- Task/outcome/background: a clock page calling server-side `date` through `uw-process`, refreshed every second via RPC/reactive rendering; later reports “a couple of production systems” and ongoing personal web-UI projects. The 2010 post is explicitly a first experiment, not evidence of production at that time; later production applications are not named. Broader experience is not specified in these excerpts.
- Positive/concrete: calls the manual good and demo/tutorial excellent; complete clock code makes the small cross-tier task tangible. Later values statically linking the executable with musl for portability across Linux machines. No substantive dislike or abandonment reported here.
- Explanation/inference/status: author credits browser threads/RPC simplicity and static linking. Researcher inference: deployable artifacts can matter independently of type sophistication; this does not establish a requirement for native compilation in interpreted ur2 v0. **W/T/D**, historical; present portability not tested.

### A2 — Edward Z. Yang: core metaprogramming paid off, but required tricks

- Sources: [How Ur/Web records work](https://blog.ezyang.com/2012/04/how-urweb-records-work-and-what-it-might-mean-for-haskell/), **2012-04-20**; [Polymorphic variants in Ur/Web](https://blog.ezyang.com/2012/07/polymorphic-variants-in-urweb/), **2012-07-29**. No compiler version stated.
- Task/outcome/background: firsthand work on Logitext; explains records to Haskell readers. Adopted polymorphic variants, although not initially planned, as the most reliable way they found to quickly implement JSON serialization via metaprogramming. Collaborated with Adam Chlipala on the local-type-class technique.
- Positive/concrete: row operations and automatically inferred disjointness obligations; reusable variant tags; generated operations. The conclusion says they appreciated metaprogrammability beyond serialization.
- Pain/concrete: tutorial omitted variants and manual gave only a paragraph at the time. Matching many tactic constructors was repetitive; local type classes reduced it. A naive fold generating variant constructors fails because the checker cannot establish that the current name belongs to the full row; a polymorphic accumulator carries the missing relationship. Lack of recursive variants required a module-system encoding.
- Explanation/inference/status: the author supplies code and an explanation, stronger mechanism evidence than a bare complaint. Researcher inference: retain working row/variant abstractions, but teach accumulator invariants and annotation boundaries; don't infer that a simpler surface syntax removes that reasoning. **C/D**, historical; examples not rerun today. Statements about Haskell are dated and are not adopted as current comparisons.

### A3 — Daniel Patterson / dbp: complete small application with integration rough edges

- Sources: [A Literate Ur/Web Adventure](https://dbp.io/essays/2013-05-21-literate-urweb-adventure.html), **2013-05-21 (date in URL)**; [2019 retrospective](https://lobste.rs/c/biayja) and [comparison comment](https://lobste.rs/c/so8lkm), **2019-01-05**. “Current version” in original, no version number.
- Task/outcome/background: built a Democracy Now video player remembering position across devices, with complete source/build directions, tested on then-current Debian and macOS. In 2019 reports several small Ur/Web applications and substantial Haskell web experience, but limited Yesod experience. Demo no longer hosted by 2019; not presented as language abandonment.
- Positive/concrete: smooth reactive/client-server combination, handlers as ordinary functions, typed SQL/XML; calls overall HTML system neat and effective despite rough edges.
- Pain/concrete: errors scrolling pages; minimal date functions; wrote C random bindings before discovering built-in `rand`; manual schema migration when module/table names change; absent tags/media support needing FFI; separate static hosting in original setup.
- Explanation/inference/status: author explicitly acknowledges missed `rand`, so that is API **discovery**, not missing functionality. Table naming/migrations and browser media/cache workarounds are **W**, not flaws in row theory. Other concerns **D/T**. Historical; current manual documents a `file URI FILENAME` static-asset facility [T1], so blanket “cannot serve static files” is obsolete. Other gaps/current migrations unverified.

### A4 — Vladimir Shabanov: sustained production value despite substantial compiler friction

- Original: [Ur/Web in production](http://www.impredicative.com/pipermail/ur/2014-January/001608.html), **2014-01-16**. [Blog pointer](https://blog.bazqux.com/2014/01/urweb-and-bazqux-reader.html) and [HN cross-post](https://news.ycombinator.com/item?id=7072437) are **the same account**, not extra votes. Version unspecified; patches during development described.
- Task/outcome/background: BazQux Reader, reportedly thousands of paying daily users; Haskell backend/data-processing experience. Initially Postgres data shared between Haskell and Ur/Web, later Riak/Haskell linked through FFI. Published frontend/Haskell interface source.
- Positive/concrete: interactive widgets are composable functions; typed SQL/RPC, algebraic data types and pattern matching reduce glue and make refactoring easier. Explicitly values plain-code simplicity more than the homepage's advanced-type messaging. Reports server performance was sufficient (about 2K requests/second in his early test, **not an independently reproduced benchmark**).
- Pain/concrete: sparse libraries led to Haskell integration; FFI declarations awkward, especially local data types. Missing pattern facilities, including tuple patterns in monadic binds. Huge record/annotated-code errors for small mistakes. Compilation once took ten minutes for a few thousand lines, improved by patches to roughly a minute for more code; still slow for layout edits. Attributes much bloat to inlining and describes indirection/fake-recursion workarounds. Reports an uncertain effect-elimination bug. Thousands of simultaneous `dyn` elements hurt mobile performance; used server-generated HTML plus JS.
- Consequence/explanation: considered rewriting in JavaScript during buggy/slow builds but repeatedly chose repair over reimplementing Ur/Web's benefits. Inlining explanation is the author's informed diagnosis, not measured here; the effect bug was already of uncertain fix status in the post. **C/D/T/W**. Historical partial improvements are explicit. Current manual supports inline FFI with an unsafe opt-in [T1]; historical blanket absence no longer holds. Compile times, pattern coverage, effect bug and mobile workload not rerun.

### A5 — steen / steinuil: repeated failed starts, productivity, then leaving

- Sources: [I survived Ur/Web](https://sgt.hootr.club/blog/urweb/), **2018-01-22 (page date)**; [Lobsters follow-up](https://lobste.rs/c/ayvtsy), **2019-01-07**; [HN retrospective](https://news.ycombinator.com/item?id=39160664), **2024-01-27**. Do not mistake the Lobsters submission date (2019) for the blog's date. Versions unspecified; 2024 describes an old project, not necessarily 2024 compiler behavior.
- Task/outcome/background: unnamed early attempts, later web application work; 2024 identifies [negoto](https://github.com/steinuil/negoto). Did not know monads on first trying the language. Reports SQLite patch contribution later.
- Positive/concrete: after repeated attempts it clicked; typed tables, cookies, forms and RPC eliminated busywork; signature files helped hide tables/cookies inside modules. Initially confusing `transaction` separation became natural/useful.
- Pain/concrete: straying from examples brought unreadable errors; repeatedly deleted project/compiler and stopped for months. Unexplained `queryL` versus `queryL1`, sparse signature comments, parse failures and huge desugared XML/SQL dumps. Disliked `end`; wanted string buffers and richer C binding data. In 2019 says ecosystem and those problems usually lead to choosing other stacks.
- Later consequence: 2024 says ultimately stopped after that project because of server/deployment features, FFI/transaction integration and missing compiler-supported web behavior. Cannot recall exact standalone-server problem; multi-file form recollection is explicitly uncertain. Found compiler SML difficult to extend.
- Explanation/inference/status: trust the reported stopping and frustration, **not** a precise unremembered deployment defect. The blog's joking attribution of roughness to intentional elitism and speculation about creator priorities do not establish causes. **D/T/A**, module/effect benefits **C**, final blockers largely **W/T**. Historical; later retelling is corroboration of the same author, not independent evidence or a contemporary reproduction.

### A6 — Simon Van Casteren: experienced solo production user with a different view of documentation/FFI

- Source: [Using Ur/Web: Pro's and Con's](https://frigoeu.github.io/urweb1.html), **2019-05-22**, includes an undated LSP update. Version unspecified.
- Task/outcome/background: 1.5 years part-time building classy.school, music-school SaaS. Senior solo developer with Haskell, PureScript, OCaml and extensive frontend experience; no hiring/convincing-team constraint. Explicitly warns his situation differs from others'.
- Positive/concrete: tier consistency without handwritten serialization; typed SQL; reactive frontend scales to complex interactions (including his Elm-style architecture); purity makes effects visible. Runtime performance felt good; generated 1.7MB JS compressed to about 140KB/90KB in his reported setup, not a universal size benchmark. Documentation praised once found: demos, tutorial, tests, manual and helpful mailing list.
- Pain/concrete: bounced off homepage four times thinking abandoned/irrelevant. Initially huge cryptic errors; learned to inspect first/last lines and maintain a phrase-to-cause list. Missed typed hover/completion, then built an initial LSP. Four dependencies managed with custom Nix solution/manual work. Full executable build about two minutes; largest-file type feedback about five seconds, most files instant. Some SQL date/time types need compiler changes.
- Disagreement/consequence: tiny ecosystem not a major problem for his app; both FFIs “pretty easy,” used OpenSSL and requestAnimationFrame. This conflicts usefully with A4/A5/A8, not a reason to discard them. Continues; slow full builds are biggest gripe.
- Explanation/inference/status: author connects prior functional experience and solo context to ease. Researcher inference: documentation findability and task/library needs mediate outcomes; no controlled causal comparison. **C/D/T/A/W**. LSP absence superseded even within updated post and confirmed by current manual [T1]; other timings/needs historical, current unverified.

### A7 — igorclark: short corroboration of giving up

- Source: [Lobsters comment](https://lobste.rs/c/153nsm), **2019-01-05**. Version, project and background unstated.
- Reports giving up sooner than Steen over the same problems and asks for accessible learning material. This is first-person negative experience, but lacks a reproducer or an independently itemized diagnosis. **D/A**, outcome stopped attempt; current status unverified. Count as one brief corroboration of onboarding friction, **not** a fifth detailed compiler-error case.

### A8 — banana_feather: personal attempt stopped at server IO boundary

- Source: [HN comment](https://news.ycombinator.com/item?id=39158487), **2024-01-27**. Attempt date/version/background unspecified.
- Says personal projects were stopped by server isolation: thinks task was reading a JSON file, needing C FFI. No completed application reported.
- **T/W** (IO/library/runtime boundary), not evidence against rows. Author's recollection is qualified (“I think”); remarks about BazQux bypassing the security model are a separate, unsupported interpretation, not adopted here. Current file-reading alternatives were not audited; [T1]'s static asset embedding is **not** proof of general runtime file IO. Researcher implication: make supported IO and escape hatches explicit in a general-purpose redesign.

### A9 — Fabrice Ferreira Leal / fabriceleal: folder-related compilation workaround

- Source: [urweb/urweb #242](https://github.com/urweb/urweb/issues/242), **2021-11-20**, open with no comments at access. Version/background unspecified.
- Attempt: multilingual menu/page code using folds; supplies failing and working repository files and error dump links. Reports “Unsupported expression”; a dummy functor sharing a type/folder makes it compile.
- Author suspects two inferred folders for one type, explicitly tentative. **C-adjacent/T/D**: compiler acceptance of higher-order/folder programs is not necessarily a core typing problem. Current manual acknowledges valid programs can fail heuristic compilation [T1], but does **not** diagnose this exact report. Outcome workaround, no abandonment claim; not reproduced/current unverified.

### A10 — Chris Bailey / ammkrn: installation and demo failure

- Source: [urweb/urweb #243](https://github.com/urweb/urweb/issues/243), **2022-02-12**, open without comments at access.
- Version/environment: **20200209**, Homebrew, macOS **11.4**, Apple clang **12.0.0**. Background not stated. Official demo command produces parsing errors and no executable; source build also fails after OpenSSL/ICU setup work, with `true`/`false` macro collisions in C shown in the report.
- **T/D**, reached attempted installation/demo, no successful later outcome reported. C macro diagnostics support that immediate build conflict; relationship between compiler/demo versions and parser failures not established here. Do not interpret this as rejection of Ur's type system or a failure on all current macOS installs. Historical/current unverified.

### A11 — Vilem Liepelt / buggymcbugfix: core inference/syntax confusion in a tiny function

- Source: [urweb/urweb #266](https://github.com/urweb/urweb/issues/266), **2025-02-20**, replies **2025-02-28 / 2025-03-01**; open at access. Version not stated. Background not specified in issue (separate talk lead below).
- Task: recursive `firstSome` over a list, with `a`/`b` type parameters; repeatedly hits “too-deep unification variables.” No web construct required. dan-nectry points out explicit application to implicit constructor parameters requires `@@`, and suggests an inner recursive helper avoiding repeated type passing. Reporter confirms workaround typechecks but still asks why annotations/application did not fix it.
- **C/D**, outcome workaround, unresolved explanation. Supported: forgotten explicit-argument syntax and successful restructuring as reported. Unsupported: asserting the full failure is solely syntax error or a proven unsound/incomplete inference rule. This is a recent report of the *class* of opaque diagnostic trouble, not proof that an old bug persists in today's compiler. No reproduction undertaken.

## Additional sources: classified rather than silently counted

- **X1, non-user evaluation:** [Yorhel, An Opinionated Survey of Functional Web Development](https://dev.yorhel.nl/doc/funcweb), 2017-05-28. Explicitly places Ur/Web on a to-try list, citing perceived one-person/small ecosystem risk. Useful evidence of an adoption perception, **not** actual Ur/Web frustration. Other framework experiences in that article must not be misattributed to Ur/Web.
- **X2, non-use/unclear use:** [zem on Lobsters](https://lobste.rs/c/kqae10), 2019-01-06: considered Ur/Web several times, chose js_of_ocaml for ecosystem and browser-heavy work. No demonstrated Ur/Web implementation; excluded from user counts.
- **X3, speculation/creator context:** [HN 2024 thread](https://news.ycombinator.com/item?id=39156207): vmsp's “ML rabbit-hole” diagnosis does not document personal use; creator achlipala describes work on Nectry. Neither counts as independent user validation. Product-customer praise of BazQux is not developer experience.
- **X4, small learning comment:** [numberten](https://lobste.rs/c/oqe9n6), 2019-01-05, liked a blog-building tutorial but struggled with typos/vagueness and reports an unanswered PR for over three years. Excluded from the main 11-author corpus/counts because the project outcome/version and PR weren't followed up; a worthwhile community-response lead, not evidence of all maintainers' responsiveness.
- **X5, talk lead:** [Vilem Liepelt at Kent](https://www.kent.ac.uk/whats-on/event/80876/ur-web-programming-language-vilem-liepelt), rendered “Monday 20 July” with no year established. Abstract promises a production e-commerce experience and jokes about 10,000-line errors; warns initial Nix build may take 20 minutes if MLton builds. Full talk not reviewed; do not count as a second author or use the joke as a measurement.
- **T1, current technical context only:** [Ur/Web manual source, pinned revision](https://github.com/urweb/urweb/blob/55a881ff9b50d9e5c3b2fd564f5cd44a5cc5e6bc/doc/manual.tex), accessed 2026-09-30. Sections on LSP, project directives, Less Safe FFI, and Compiler Phases. It documents basic completion/hover/errors; `file` compile-time asset embedding; default effectfulness of transaction-based FFI; inline FFI under `lessSafeFfi` with invariant-breaking warning; and heuristic server compilation that can reject valid, too-higher-order programs to obtain efficient native representations. This establishes documented facilities/trade-offs, not their usability or that historical bugs are fixed.

## Synthesis with explicit author counts

Counts below are **within the selected 11-author ledger**, not prevalence estimates. Each row counts an author once regardless of number of posts; rows overlap. Counts are intentionally narrow rather than treating vague agreement as every detailed complaint.

| Finding | Count and evidence | Meaning and counterevidence |
| --- | --- | --- |
| Integrated, typed web composition makes useful work easier | **5 authors: A1, A3, A4, A5, A6** | From clock to production apps. Much of this is W: ur2 cannot promise the same payoff without the web stack. A4 explicitly prizes ordinary composition over advanced-type marketing. |
| Core row/variant metaprogramming solves a concrete application problem | **1 detailed author: A2** | Strong, specific serialization/fold evidence, but not a broad vote for every advanced feature. |
| Oversized/opaque compiler errors impede development | **4 detailed long-form authors: A3, A4, A5, A6** | Different examples: records, elaborated code, SQL/XML, opaque phrases. A7 briefly corroborates onboarding; A11 adds recent core-only diagnostic confusion, separately. A6 became competent at decoding errors; this does not negate initial cost. |
| Full builds interrupt the loop | **2 production authors: A4, A6** | Different years/projects/timings. A6 distinguishes fast checking from slow executable generation. Not evidence that core type checking inherently takes minutes. |
| Thin libraries or discovery require workarounds | **4 authors: A3, A4, A5, A6** | Missed `rand`, Haskell integration, poor API explanation, custom dependency handling. A6 says available library scope mostly suffices. Avoid “no ecosystem” absolutism. |
| FFI/integration friction is material | **3 negative authors: A4, A5, A8** | Severity ranges from annoyance to stopping. A6 explicitly finds both FFIs easy enough; tasks, types and boundary semantics differ. |
| Documentation assessment differs | **Positive A1/A6; gaps A2/A5** | Not a single quality ranking. A6 praises content yet reports discoverability trouble; A3's unnecessary random binding reinforces that distinction. |
| Explicit effect separation is valued | **2 authors: A5, A6** | A5 had an initial monad-learning barrier. A5 later also finds transactional integration hard. Core purity and web rollback semantics are not the same requirement. |
| Native runtime/deployment can be an asset or obstacle | **A1 portability positive; A5 deployment negative; A10 install failure** | Different environments and stages. No contradiction resolved by claiming one universal deployment quality. |
| Reported stopping, not just complaints | **3 authors: A5, A7, A8** | A5's sequence includes repeated returns, productivity, and later cessation; A7/A8 are brief/underspecified. No basis for an abandonment rate. |

### Symptoms, causes, and confidence

“High” here means confidence in the narrowly stated evidence, not generalization to all users.

| Symptom | Supported explanation | Researcher inference / uncertainty | Confidence |
| --- | --- | --- | --- |
| Huge errors for small mistakes | A4 names full records/annotated AST; A5 XML/SQL desugaring; A6 supplies small row mismatch examples | Preserve source-oriented context and summarize differences. Does not show the underlying type rules must change. | High report confidence; medium general remedy |
| Basic recursive function fails with a deep-variable message | A11 confirms helper workaround and forgotten `@@` syntax | Need a phase-specific explanation/reproducer; don't label every failure a row-unification issue. | High symptom, low full cause |
| Fold program fails while restructuring works | A9 reproduction links; T1 says valid higher-order programs may fail later compilation | Could be compiler specialization rather than typing; exact phase/root cause not established. | High reported workaround, low exact cause |
| Long builds/bloat | A4 attributes to inlining and reports patch gains; A6 separates checker and full build time | Measure phases before optimizing inference or cutting features. No claim that two projects share identical cause. | Medium causal evidence |
| Library/ecosystem friction differs across users | A4 needs Haskell services; A6 needs few dependencies; A3 missed an existing API | Task coverage and discoverability plausibly mediate friction; no controlled comparison. | High differing needs, medium inference |
| Abandonment after proficiency | A5 explicitly names deployment and FFI/transaction limitations in retrospective | Core sophistication alone is not a sufficient causal story; exact server issue forgotten. | High stated decision, low technical specifics |
| “Documentation is poor” versus “manual is amazing” | Both assessments are direct, at different dates/backgrounds | Better navigation, task examples and advanced explanations might reconcile them; cannot prove which intervention dominates. | High disagreement, medium recommendation |

## Implications for ur2 (candidates, not settled choices)

| Finding → evidence | Relevance to ur2 | Candidate preservation/change or follow-up |
| --- | --- | --- |
| Metaprogramming earned its keep → A2 | **Core** rows, `map`, folders, variants and modules; a row is a compile-time collection of named fields, not a record value | Preserve expressiveness by checking the feature decision against a small serialization/traversal example. Which pieces are essential, and which can wait beyond the first milestone? |
| Ordinary typed composition matters → A4/A6, plus A3/A5 | Core records/functions/abstraction support this, but the demonstrated whole-stack payoff is **web-specific** | Show a non-web typed data transformation rather than advertising metaprogramming alone. Do not add SQL/XML/RPC back to v0. |
| Error floods and obscure inference boundaries → A3–A6, A11 | Directly applicable to checker UX even after removing XML/SQL | Candidate diagnostic tests: misspelled field in a large record; wrong kind; failed disjointness; explicit-vs-implicit constructor arguments; a too-deep unification variable. State expected/actual local difference and where it arose; keep full internal output opt-in. |
| Syntax annoyances and pattern wishes → A4/A5, explicit-argument confusion A11 | Syntax decision should distinguish readability from elaboration complexity | Evaluate bind patterns and visible type-argument notation on real snippets. One `end` complaint is not enough to select a syntax family; keep Ur's core rules as spec. |
| Documentation discovery is part of usability → A2/A3/A5/A6 | Core concepts and eventual standard library | Ship a single start path, searchable signature explanations, runnable examples and a map from diagnostic to concept. Teach row versus record type, kind, disjointness and folders with successful and failing examples. |
| Full build cost differs from check cost → A4/A6; heuristic compilation T1/A9 | Interpreted v0 need not reproduce Ur/Web specialization limits | Candidate milestone measures parse/check/run separately, including a small edit. Keep typed program acceptance separate from later backend limitations. Don't import ten-minute compilation claims as a checker benchmark. |
| Purity helps; effects/FFI can obstruct tasks → A5/A6 versus A4/A8 | Effects/IO are undecided in map; web transactions/rollback are **not core obligations** | Ask for a minimal non-web IO use case and explicit boundary/error semantics before choosing an effect API. Consider how foreign calls declare effects; do not infer a need for Ur/Web's transactional server. |
| Tooling/installation can block otherwise useful language → A6/A10; LSP now exists T1 | Rust toolchain and v0 distribution, independent of row theory | Candidate reproducible install-and-run check on supported platforms, version-matched examples, and a typed-information interface that can later serve editors. Full IDE/package registry is not justified as first-milestone scope by this corpus alone. |
| Sparse ecosystem is not uniformly fatal → A4/A6/A8 | Adoption and library scope | Identify the tiny set of tasks v0 promises. Say what is missing and how to cross boundaries; don't mistake specialist success for general ecosystem readiness. |
| Mobile `dyn`, table migrations, file-upload/RPC and CGI limitations → A3/A4/A5 | **Web-only**, outside v0 | Keep as cautionary examples of leaky domain integration, not requirements to fix in ur2's core. |

## Recommendations to bring to the existing decisions

1. **Feature set:** use A2's variant/serialization case as a concrete preservation test, alongside ordinary record composition. The corpus supports useful abstraction, not maximizing feature count. Ask which Ur facilities that example really requires.
2. **Syntax and diagnostics:** review syntax with errors in mind. A11 is a small non-web test of whether users can understand implicit arguments and recovery; A4/A6 supply large-record typo cases. A readable successful program is only half the evaluation.
3. **Tooling and documentation:** propose a short, executable onboarding path plus local, bounded errors before a broad ecosystem plan. Learn from the discovery failures without falsely saying present Ur/Web has no LSP.
4. **First milestone:** consider one end-to-end non-web program that parses, checks and runs, one rejected variant with a helpful error, and a timed edit/check/run path. This is a candidate acceptance test for the milestone decision, not authorization to implement or finalize it here.
5. **Effects/IO:** separate pure core reasoning from Ur/Web's transaction/runtime restrictions. A small file-reading or foreign-call use case should expose this boundary before broad general-purpose claims are made.

### Unanswered questions and bounded follow-up

- Which failures still occur on a pinned current Ur/Web build? A focused follow-up should reproduce **#266 and #242**, record the actual phase, and compare source-level versus elaborated diagnostics. This is two cases, not an open-ended compiler audit.
- How much of A2's metaprogramming benefit survives a minimal v0 feature subset? Extract one non-web serialization/traversal example for the feature discussion; do not decide variants/modules from popularity counts.
- What helps a functional-language novice versus an expert? A few observed tasks with row/record/kind errors would distinguish syntax friction from unfamiliar type-level reasoning; these public essays cannot.
- What became of A5's specific deployment blockers, A4's compilation workarounds and A6's full-build timings? Need versions and runnable projects before claiming fixes or persistent regressions.
- Team adoption, hiring, maintenance handoff and long-term abandonment remain largely unmeasured. A6 explicitly avoids the hiring problem as a solo developer. Do not turn sparse public reports into a representative market explanation.

The strongest redesign lesson is therefore modest: **keep the capability, make failure understandable, and show a complete usable path through the intended non-web scope**. The evidence does not require sacrificing Ur's core type system, nor does it justify restoring Ur/Web's web framework to ur2 v0.
