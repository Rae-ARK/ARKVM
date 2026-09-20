# What Rei Is

_Current as of 2026-09-20. Read against the ARKVM docs @ `464b0a8` and
ARKlight `main` @ v0.54.0. Claims about `ARKNode`, `ActionRef` and the
build-time validator were checked directly in `arklight/ast/nodes.py` and
`arklight/ir/validate.py`. The sibling docs cite ARKlight `alpha`; nothing
here was checked against `alpha`. Tags: **[decided]** stated by the
maintainer, **[proposed]** a recommendation that needs a maintainer to accept
it, **[open]** not designed yet._

## The definition **[decided]**

Rei is the official native source language of ARKlight.

- It follows the **philosophy of ISO C99**. It is not C, and it is not a
  systems language.
- It is an **application-authoring language** in the family of Dart and
  Kotlin: a reimagining of what Dart could and should be, built from first
  principles.
- It is **procedural by default**. Object-oriented design and the GoF
  patterns are native to it, not bolted on, so both paradigms feel at home.
  Where they overlap or reinforce each other, that is intended.
- It uses procedural design and OOP/GoF patterns together as a set of
  constraints for solving the problem in front of it.
- It **helps its users itself**. Rei is also the voice of ARKlight's
  compiler narrator, so the language and the help are one thing, not two
  products sharing a name.

**Python authoring stays.** Rei is added beside it, not put in its place.
The reason to own a language is to control the whole chain: authoring,
compiler, IR, runtime.

**Unification is the promise. [decided]** As Dart gives Flutter one
language for every target, Rei promises one for every target, while keeping
ARKlight's philosophy: compiler first, fail loudly at build time. A WebView
host has real limits on Android, on desktop and elsewhere, and that is why
ARKVM exists. Delivering the promise is REIlight's job (next section). The
Rei language itself stops at the ARK AST.

```
.rei source -> Rei AST -> ARK AST -> ARKlight handles the rest
```

This is the Rei language for ARKlight. It changes nothing below the ARK AST.

Rei's front end started from Java's: the AST and the stages before it
(lexing, parsing) were studied in `reference/openjdk/`. That is a design
reference only. **No javac or `java.base` code is copied**, because those
files are GPL-2.0-only and ARKlight is GPL-3.0-or-later (`LOST.md`,
section 7). Rei's lexer, parser and spec are written from scratch.

## Three names

| Name | What it is | Status |
| --- | --- | --- |
| **ARKlight** | The Python-first compiler for static sites. Unchanged; Python authoring stays. | shipping |
| **The Rei language** (for ARKlight) | The frontend: `.rei` -> Rei AST -> ARK AST, then ARKlight handles the rest. | **[decided]** |
| **REIlight** | A superset of ARKlight, and distinct from it. | **[decided]** that it is a superset and that the distinction stands. Its contents are **[open]**. |

I found no definition of REIlight in the ARKlight or ARKVM repos, or on the
web, so none is invented here. **My reading, to confirm:** REIlight is
ARKlight plus what "Where it sits" below describes (ARKVM, its engine and the
native targets), with the Rei language as its native authoring language.
Under that reading:

- The Rei language works with plain ARKlight and needs nothing from REIlight.
  A `.rei` site goes through the same pipeline and reaches whatever backends
  ARKlight has.
- ARKVM reads `.arklight` and never Rei
  (`ARKVM-EXECUTION-MODEL-PROPOSAL.md`, section 7), so a Python-authored site
  can reach it too. REIlight is where the two meet; the language is not what
  makes it a superset.
- "Superset" is testable **[proposed]**: every site that builds under
  ARKlight builds under REIlight with byte-identical web output (a golden
  test over `examples/`, the same shape as invariant I1 in the Rei
  proposal). Today `sbom.txt` carries a timestamp, so the test must exclude
  it until alpha's issue-register #19 is fixed.

## Where it sits

The diagram shows the full stack. The Rei language is the left-hand
authoring box and stops at the ARK AST. ARKVM and everything below it is the
REIlight half, under the reading above.

```
 authoring          Python (stays)   |   Rei (native)
     |                               |
     +------------- ARK AST ---------+
     v
 compiler           normalization -> validation -> Website IR
     v
 .arklight          binary IR; the only artifact that crosses to a target
     v
 ARKVM              gatekeeper: verifies, then lowers ahead of time;
                    includes the engine (custom, browser-like)
     v
 target interface   Android | Desktop | ...   (one host per platform)
     v
 native app         lowered code + engine; the host supplies input, window
```

**ARKVM is AOT only, and it is the gatekeeper [decided].** Nothing is
interpreted on the device. ARKVM accepts `.arklight`, verifies it, and lowers
it through the target's interface to that platform's native runtime. On a
phone the result is a native Android app; on desktop, the same story. Only
verified `.arklight` reaches a target, and ARKVM never sees Rei or Python
source (the ecosystem's existing rule, `TERMINOLOGY.md`).

**The nearer comparison is Flutter's release mode, not the JVM.** The JVM
comparison still holds for "the layer under the language that knows only the
artifact" (`WHAT-ARKVM-IS.md`). But AOT-only means there is no execution
engine in the JVM sense. Flutter release builds are the closer shape: Dart
compiled ahead of time to native code, plus an engine and a per-platform
embedder. ARKVM's per-target interface plays the embedder's role.

**ARKVM includes an engine [decided].** It is a custom, browser-like engine
that stands in for the HTML and CSS a WebView would have supplied. It is not
a full browser: it covers ARKlight's own closed vocabulary. Interactivity
comes from the platform as far as the platform allows. The host owns the
window, input and accessibility, and ARKVM owns the tree, layout and paint
(`ARKVM-EXECUTION-MODEL-PROPOSAL.md`, section 5). A native app is therefore
the lowered code plus this engine. This rules out lowering to each platform's
own views (the Experimental-Env line, "IR in, native views out"). It is
option (a) of the pixel-perfect proposal's layout choices (section 5.3): a
named CSS subset with exact algorithms.

**The web target is unchanged.** The HTML backend still emits ordinary HTML,
CSS and JS. My reading is that ARKVM replaces the WebView-host paths that the
Android and Desktop targets use today (`WHAT-ARKVM-IS.md`, JVM comparison
table) and that the web target sits outside it. Confirm. **[open]**

Today ARKVM has no code, and `.arklight` v1 carries only a closed vocabulary
of behavior, so everything Rei does today happens at build time.

## Effect on existing docs

| Doc | What changes |
| --- | --- |
| `Proposal/REI-LANGUAGE-PROPOSAL.md` | The "playground", "Fun tier" and "not accepted" framing no longer describes Rei. Its C99 mapping, file types and stage ladder remain useful input. Whether an alpha-only guard survives as a *maturity* gate is **[open]**. |
| `reference/docs/LOST.md`, section 6 | The "likely fate" table proposed dropping classes, inheritance, interfaces and abstract. The hybrid model reverses that. See open question 3. |
| `Foundational/TERMINOLOGY.md` | "Rei (language)" should read "the official native source language", not "planned". It also needs a "REIlight" entry (a superset of ARKlight, distinct from it; contents pending open question 16). |
| `Proposal/ARKVM-EXECUTION-MODEL-PROPOSAL.md` | Shape B says ARKVM owns "the interpretation of the closed vocabulary", and shape C says it runs an instruction section. Under AOT-only, ARKVM lowers the closed vocabulary (and any future behavior IR) to native code instead of interpreting it, so A, B and C need re-reading under that constraint. Its stated justification is the pixel-perfect guarantee; the maintainer's stated motivation is WebView limits plus unification. Under an owned engine both motivations point at one design (open question 13 holds the conditions). |
| `C_ARKlight:docs/ADDENDUM.md`, section 3 | The execution-model proposal says its sentence "no on-device execution model, ever" must change. Under AOT-only it may survive as written, because nothing is interpreted on the device. Whether it does depends on how behavior is lowered (open question 2). |
| `docs/README.md` | Add rows for this file and for `REI-SYLLABUS.md` in the same change. |

## Design stance

### 1. C99 philosophy in one screen

The translation-phase model and the tenet table live in
`REI-LANGUAGE-PROPOSAL.md`, sections 3 and 4. Three points to add or fix:

- **"No hidden costs" is Rei's own tenet.** As I recall the C99 Rationale,
  its "Spirit of C" list has five items: trust the programmer; do not prevent
  the programmer from doing what needs to be done; keep the language small
  and simple; provide only one way to do an operation; make it fast, even if
  not guaranteed portable. "No hidden costs" is not one of them, so the
  proposal should label it as Rei's addition. Verify against the Rationale
  before citing.
- **Division and remainder follow C99.** C99 (6.5.5) fixes integer division
  as truncation toward zero, with `%` taking the dividend's sign. C90 left it
  implementation-defined. Python's `//` floors, so the build-time evaluator
  must implement truncation explicitly. **[proposed]**
- **No undefined behavior in the front end.** Adopt the C99 taxonomy
  (undefined, unspecified, implementation-defined, locale-specific) to
  classify every corner of the spec, but Rei either defines or rejects at
  build time anything C99 leaves undefined. "Minimal by omission, not by
  unspecification" (`LOST.md`, section 6). **[proposed]**

### 2. Procedural by default **[decided]**

- A program is declarations and functions. There is no mandatory wrapper
  class; Java's `public class Main` ceremony is what Rei deletes.
- Data is plain records: fields, no required behavior.
- A call like `Card(title: "x")` is an ordinary function call that returns an
  `ARKNode`. The most common Rei program uses no classes at all.

### 3. OOP native, where the domain already has objects **[decided]**

Rei's classes are not imported from Java. They name structures the pipeline
already has (verified in source):

- The ARK AST is a **Composite**: `ARKNode(type, props, children)`, uniform
  and recursive.
- `ActionRef`, `ClassBindSpec` and `PredicateRef` are frozen, structured,
  validated objects: **Command** values as data, never closures or strings
  that get executed.
- Compiler passes over the tree (normalize, validate) are **Visitor**-shaped.
- `State` / `Watch` / `Bind` are **Observer**-shaped: a `Watch` names a state
  key and an action to run when it changes, and the validator checks the key
  against the page's declared state.

**Constraint to state plainly.** `ARKNode` has no class counterpart
(`LOST.md`, section 3), and `.arklight` v1 carries no behavior beyond a closed
vocabulary. So today Rei's classes are **build-time constructs**: instantiate,
call methods, evaluate away into `ARKNode` trees. Because ARKVM is AOT only,
the future question is not "an interpreter for objects". It is whether
`.arklight` gains a behavior IR that ARKVM can lower to native code (open
question 2). Do not promise runtime objects until that is decided.

### 4. Two paradigms, one operation **[proposed]**

**Rule:** anything reachable through both paradigms lowers to the same ARK AST
and the same diagnostics. Neither paradigm gets a private feature.

This collides with C99's "provide only one way to do an operation". Resolve it
by letting *notation* differ while the *operation* stays single: **uniform
call syntax**, where `f(x, a)` and `x.f(a)` are one call. The compiler resolves
`x.f(a)` against `x`'s static type first, then against free functions whose
first parameter accepts that type. Ambiguity is a build error, matching the
existing "collisions are errors" rule (`PreambleCollisionError`). The same
idea appears as extension functions in Kotlin and as UFCS in D.

Where the cost lives: resolution of free functions is static. Dispatch of an
overridden method is the one dynamic step, and in the build-time evaluator it
is a lookup along the class chain (a dict walk in CPython). That affects build
time only. Nothing "dynamic" reaches emitted HTML, CSS, JS or the lowered native app,
unless a behavior IR is added (open question 2).

### 5. GoF patterns as constraints **[proposed]**

The GoF book was written against C++ and Smalltalk, and many of its patterns
are workarounds for missing language features. Rei keeps the ones that carry
an invariant the compiler can check, and absorbs the ones a language feature
replaces.

| Pattern | Where it already lives | What Rei constrains |
| --- | --- | --- |
| Composite | `ARKNode` tree | Every UI value is a node; a non-node child is rejected at the boundary. |
| Command | `ActionRef` (closed vocabulary) | Event handlers are Command values, not arbitrary closures (`LOST.md`, decision 4). Lambdas only where evaluated at build time. |
| Observer | `State` / `Watch` / `Bind` | Subscriptions are declared, and their targets are checked against declared state at build. |
| Visitor | normalize / validate passes | Rei's own AST passes use the same shape, so a new pass does not edit node classes. |
| Builder | named props | Dart-style named arguments make a hand-written Builder unnecessary. Absorbed by syntax. |
| Chain of Responsibility | fault handling | See section 6. |

"Constraint" here means the compiler enforces the pattern's invariant instead
of leaving it to convention. It is "fail loudly at build time" applied to
design patterns.

### 6. Error handling and guardrails **[proposed]**

Precision on the Java model: `try`/`catch` dispatch is **Chain of
Responsibility**. Handlers are tried in order, the first type match wins, and
an unhandled exception propagates outward up the call stack. The exception
class hierarchy makes catching by supertype polymorphic. `finally` and
try-with-resources are *not* GoF; they are deterministic cleanup (the
Dispose/RAII family). Checked exceptions are a compile-time analysis rule, not
a pattern, and `LOST.md` proposes dropping them. Dart and Kotlin have none
either.

Three layers:

1. **Build diagnostics.** Compiler-owned, with `file:line:col` and a stable
   code. Needs the source-span side table (Rei proposal, open question 8).
2. **Procedural default.** A fallible operation returns a tagged result
   (value or fault), and ignoring it is a build error. C's `errno` and return
   codes have the same explicitness, but omitting the check is silent. Rei
   makes it loud.
3. **OOP layer.** A typed `Fault` hierarchy, an ordered handler chain and
   `finally`. A fault that reaches the top unhandled becomes a build error
   carrying its span.

A result can be raised into a fault and a fault caught into a result, and both
lower identically. That is the overlap from section 4, on purpose. Whether Rei
has both, or only one, is open question 4.

Guardrails that already exist stay: validation at the ARK AST boundary rejects
unknown actions and undeclared state names. Runtime error handling in shipped
output is a separate ARKlight proposal and out of scope here.

### 7. Types and numbers **[proposed]**

- Static, nominal, null-safe. No implicit narrowing conversions.
- Fixed-width integer names (`int32` and friends, after `<stdint.h>`) with
  **checked** overflow: exceeding the range is a build error. Reason: the
  CPython evaluator has unbounded ints, and JavaScript output has doubles that
  are exact only to 2^53. A checked 32-bit range agrees on every host, where
  wraparound would need per-operation masking on each (`LOST.md`, section 6).
- Strings are UTF-8 source. Length semantics (code points, UTF-16 units or
  bytes) must be chosen and specified. **[open]**

### 8. File handling at build time **[proposed]**

- Reads are **tracked inputs**: paths resolve relative to the entry file, with
  no parent-directory search (the same rule as `arklight.config.py`) and no
  network access at build time.
- A missing or malformed file is a fault (section 6), never a silent skip.
- UTF-8 by default.
- Tracked inputs are what make builds reproducible and incremental.
- `arklight.config.rei` stays data-only, so it carries no code-execution
  surface.
- Writes from Rei programs: none by default. Output is the compiler's job.
  **[open]**

## Feature layers **[proposed]**

Build order and teaching order are the same, so the syllabus
(`REI-SYLLABUS.md`) follows these layers unit for unit.

| Layer | Adds | Blocked on |
| --- | --- | --- |
| L1 Basics | lexer, parser, constructor-call trees, literals, named props, comments, `#include` (proposal Stage 1) | nothing |
| L2 Procedural core | functions, records, modules, `const`, control flow | build-time evaluator (open question 1) |
| L3 OOP | classes, interfaces, abstract, overriding, uniform call syntax | evaluator, plus open questions 2 and 3 |
| L4 Faults | results, `Fault` hierarchy, handlers, spans | source-span table |
| L5 Files | tracked reads, data formats | L2, plus the write policy |

## What Rei is not

- **Not a systems language.** No pointers, manual memory, `union`,
  `goto`/`setjmp`, VLAs or `stdio` (proposal, section 3.2).
- **Not a Python replacement.** Python authoring is permanent.
- **Not Java.** No JVM, no `java.*`, no bytecode. Rei inherits syntax ideas,
  not Java's static semantics.
- **Not shipped, and not interpreted.** Rei source never reaches a browser
  or a device. ARKVM is AOT only: what runs on a target is lowered native
  code.

## Open questions, by blast radius

1. **Build-time evaluator.** Interpreter hosted in ARKlight, static-only
   subset, or Python escape hatch (`LOST.md`, section 4). Decides L2 onward.
   One candidate escape hatch that stays inside the closed model: a Rei
   program pulls in an ACC capability with `#include <acc.some_module>`, the
   same preamble seam Python uses (`# include <acc.some_module>`), so Rei
   reaches Python's build-time ecosystem through declared capabilities, never
   through arbitrary Python (ACC design, sections 9 and 20). Caveat: ACC's
   docs predate the preamble, and alpha now binds an `acc.` include by module
   path plus an `__all__` contract, while the ACC design separates capability
   identity from the import path. That gap needs reconciling first.
2. **Behavior IR.** ARKVM is AOT only, so it cannot interpret behavior. Does
   `.arklight` gain a behavior representation that ARKVM lowers to native
   code? If not, classes stay build-time forever.
3. **Which of `LOST.md`'s dropped features come back.** The hybrid model
   reverses classes, interfaces and abstract. Recommended to stay dropped:
   overloading (named props and uniform call cover it), checked exceptions,
   threads and `synchronized`, inner and anonymous classes, static init.
   Generics are undecided.
4. **Fault model.** Results only, faults only, or both convertible.
5. **Event handlers.** Closures or Command data (`ActionRef`).
6. **Numeric and string semantics** (section 7).
7. **Uniform call syntax.** Accept or reject (section 4).
8. **Gating.** Retire the Fun tier, or keep an alpha guard until L1 ships.
9. **Route naming and source spans** (proposal, open questions 2 and 8).
   Precedent for spans: alpha 0.06506 already captures `file:line` at
   component call sites with `sys._getframe(1)` (`api.py`). Generalizing that
   into a node-to-span side table would give Python-authored sites the same
   error locations Rei will have.
10. **Named-argument spelling.** `name: value` or `.name = value` (proposal,
    open question 5).
11. **Build-time writes** (section 8).
12. **One Rei. [decided]** The language and the voice are the same thing:
    the language helps its users from the inside. Today that voice is the
    compiler narrator (`--narrate`, the `[Rei]` heavy-reliance nudge, the
    README status block). When `.rei` exists, its diagnostics speak the same
    way. Keep the narrator proposal's rule: help is deterministic templates
    plus pointers to real tools such as `arklight search`, never advice
    invented on the spot. Keep a stable machine-readable channel (exit
    status, error codes) beside the voice, because 0.06508 already broke
    log filters keyed on the old `[Rae ARK]` tag.
13. **Pixel-perfect conditions.** An owned engine is necessary but not
    sufficient. Flutter draws its own pixels, and its gallery still runs
    golden tests on one OS because text renders differently across platforms.
    Chromium screenshots differ across OSes until LCD text and hinting are
    disabled. Same pixels need bundled fonts, no hinting, a defined shaping
    and line-breaking algorithm, and pinned rasterizer arithmetic: level G2
    of the pixel-perfect proposal (sections 4, 5.4 and 5.5). A GPU path
    cannot be bit-exact by that standard, so it is at best G1. Which level
    defines "pixel perfect" is the maintainer's call.
14. **Web target.** In or out of ARKVM (see "Where it sits"). This is the
    execution-model proposal's question 8.
15. **Dev loop without a JIT.** Flutter's stateful hot reload relies on the
    Dart VM's JIT in debug builds. AOT-only rules that out, so the fast edit
    loop needs another design.
16. **REIlight's boundary.** What it contains (ARKVM, the engine, the native
    targets, the Rei language), whether it has its own CLI or is ARKlight plus
    extensions, how its version relates to ARKlight's, and how the "superset"
    guarantee is tested.

## Sources for the engine claims

- Blitz, a modular HTML/CSS engine that is not a full browser (pre-alpha):
  <https://github.com/DioxusLabs/blitz>
- Flutter gallery golden tests, run on one OS because of text differences:
  <https://dart.googlesource.com/external/github.com/flutter/gallery/+/HEAD/test_goldens>
- Chromium text rendering varies by OS, GPU and font stack, and the flags
  that remove it: <https://argos-ci.com/docs/learn/reliability-and-flakiness/flaky-tests/stabilize-text-rendering>

## Read next

- [`REI-LANGUAGE-PROPOSAL.md`](../Proposal/REI-LANGUAGE-PROPOSAL.md): file types, C99 mapping, stage ladder.
- [`REI-SYLLABUS.md`](REI-SYLLABUS.md): the course built on the layers above.
- [`WHAT-ARKVM-IS.md`](WHAT-ARKVM-IS.md): what runs underneath.
- `reference/docs/LOST.md`: what Rei gives up relative to Java, by layer.
