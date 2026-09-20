# ARKVM Execution Model: What It Runs, What It Owns

## Status

**Proposed. Not accepted, not scheduled.** Filed 2026-09-19 against ARKlight
`alpha` @ `6983823` (`v0.06502`). Nothing here is committed to. Deleting
this proposal, or replacing it with a different shape, is a supported
outcome.

- **Type:** Architecture.
- **Question:** ARKVM is the runtime beneath Rei, the way the JVM is beneath
  Java. What, concretely, does it execute, and what does it own?
- **Not covered:** what "pixel perfect" means. That is
  [`PIXEL-PERFECT-CONTRACT-PROPOSAL.md`](PIXEL-PERFECT-CONTRACT-PROPOSAL.md),
  the single home for that question.

> **Update 2026-09-20.** Maintainer decisions recorded in
> [`../Foundational/WHAT-REI-IS.md`](../Foundational/WHAT-REI-IS.md) now
> constrain this proposal: ARKVM is **AOT only** (nothing is interpreted on
> the device) and **includes a custom engine**. Shape B says ARKVM owns "the
> interpretation of the closed vocabulary" and shape C says it runs an
> instruction section. Under AOT only, ARKVM lowers the closed vocabulary
> (and any future behavior IR) to native code instead, so A, B and C need
> re-reading under that constraint. The stated justification below is the
> pixel-perfect guarantee; the maintainer's stated motivation is WebView
> limits plus unification, and under an owned engine both point at one
> design (`WHAT-REI-IS.md`, open question 13 holds the conditions). Section
> 10, question 8 is that document's open question 14. The body below is
> unchanged.

## TL;DR

- Today's `.arklight` is a **declarative tree with a closed behavior
  vocabulary**, not code. That makes "the JVM for Rei" a slogan that does
  not yet describe an execution model.
- There are three honest shapes for ARKVM: **A** renderer only, **B**
  renderer plus a reactive core, **C** a general VM with an instruction
  section. They differ mainly in what `.arklight` must grow to carry.
- **Recommendation (the author's, for a maintainer to confirm):** aim at
  **B**, build **A** first as its first rung, and defer **C** until Rei's
  build-time-evaluator decision is made, because C only earns its cost if
  Rei code must run at runtime.
- ARKVM is the *last* thing ARKlight's own decision rule prefers, so it
  needs an explicit justification. The proposed justification is the
  pixel-perfect guarantee, which a target-native renderer cannot give.
- ARKVM must **not depend on Rei**. It reads `.arklight` and nothing else.

## 1. What is true today

Each fact was checked against source on 2026-09-19.

1. **`.arklight` v1** (`ARKlight:arklight/ir/binary.py`, `FORMAT_VERSION =
   1`) encodes the site name, language, app-shell flag, and per page a
   route, an IR tree, `state`, and `computed_initial`. It does not carry
   watch, persist, media or query specs, custom styles, CSS variable
   overrides, raw postprocessors, or CSP settings. Those come back as
   stock defaults after decoding.
2. **The tree vocabulary is 90 built-in node types**
   (`ARKlight:arklight/ir/schema.py`, `SCHEMA`), 37 of them text-only. The
   names are HTML-shaped: `Header`, `Footer`, `Nav`, `Section`, `Article`,
   `Details`, `Summary`, `Figure`, and so on.
3. **Behavior is a closed vocabulary**, all structured references and no
   closures: 8 actions, 5 behaviors, 12 derivations, 5 predicates, 5
   event modifiers, plus reactive state (`State`, `Bind`, `Computed`,
   `Watch`, `Repeat`, `Show`).
4. **Styling is CSS-shaped**: `--ark-*` custom properties
   (`arklight/backend/css/design_tokens.py`), `Site.style(...)` classes,
   and `@media` responsive rules.
5. **Android and Desktop backends are WebView hosts** today (Android via
   `androidx.webkit`; Desktop via GTK3 and WebKit2GTK, Linux only so far).
   A browser engine renders; no ARKlight code draws pixels.
6. **Rei runs at build time** in every design so far. The Rei proposal
   places it before the ARK AST, and its own non-goals say "no runtime"
   and "never shipped to the browser".

## 2. The tension with "compiler first, runtime last"

`ARKlight:docs/Foundational/SYSTEM-DESIGN-AGREEMENTS.md` is the ecosystem's
core doctrine, and an ARKVM cuts against several of its lines.

| Agreement | What it says (paraphrased) | Pressure from ARKVM |
| --- | --- | --- |
| Section 3, do not reimplement the target runtime | ARKlight should not become a replacement for the environment its output runs in. | A pixel-perfect renderer *is* a replacement for the platform's own layout and painting. |
| Section 9 | Web output stays ordinary HTML/CSS/JS. | Does not forbid ARKVM elsewhere, but implies the web target stays the browser's job. |
| Section 15 | Portability comes from standard output, not a big ARKlight runtime everywhere. | ARKVM's portability comes from *owning* the output. |
| Section 16, the decision rule | Prefer, in order: compiler, target-native capability, minimal generated bridge, ARKlight runtime abstraction. The lower on the list, the stronger the justification. | ARKVM is the last item. |
| `C_ARKlight:docs/ADDENDUM.md` section 3 | ARKVM is a compile-time dispatcher: no on-device execution model, no bytecode interpreter, ever. | Directly contradicted by a runtime. `C_ARKlight:docs/ARKVM.md` section 2 already notes the sentence would have to change. |

**Proposed reconciliation.** Read section 16 as it is written: a runtime
abstraction is allowed when the first three answers are "no". For visual
fidelity they are:

1. *Can the compiler solve it?* No. Rendering is not knowable at build time.
2. *Can the target solve it?* Not with a guarantee. Target-native rendering
   is exactly what varies across targets, which is the problem ARKVM
   exists to remove.
3. *Can a minimal bridge solve it?* A bridge to a platform's own renderer
   inherits that renderer's variance.

So ARKVM is justified, and only for **targets that have no renderer worth
delegating to, or where a guarantee is required**. The rest of the doctrine
still holds: the compiler bakes everything static, ARKVM receives a tree
that is already normalized and validated, and ARKVM ships only what a given
`.arklight` needs. This should be written into the doctrine as an
explicit, bounded exception, not left as an unspoken contradiction.

## 3. Three candidate shapes

### A. Renderer only

ARKVM turns a static tree into pixels. Layout and paint live in ARKVM.
Behavior stays wherever it is today (JS in the browser, or nothing on a
native target at first).

- **Owns:** layout, paint, text, images, hit testing.
- **`.arklight` must add:** the styling and layout data v1 lacks.
- **Fits:** Experimental-Env's "static UI first" staging; the smallest
  useful first rung.
- **Gap:** a UI with no interaction. `State`, `Bind` and `Action` do
  nothing.

### B. Renderer plus reactive core

A adds the reactive graph that evaluates `State`, `Computed`, `Watch`,
`Bind.*` and the closed action, derivation and predicate vocabulary,
re-rendering what changes. This is the `--full` mode in
`C_ARKlight:docs/ARKVM.md`, and the "ARKVM core" of the Experimental-Env
proposal.

- **Owns:** everything in A, plus dependency tracking, action dispatch,
  and the interpretation of the closed vocabulary.
- **`.arklight` must add:** the watch, persist, media and query specs v1
  omits.
- **Fits:** the closed-vocabulary doctrine. There is nothing to execute
  that is not already a named, validated primitive.
- **Boundary:** a C ABI, per the Experimental-Env proposal's reasoning
  (no stable C++ ABI across compilers; every target can already call C).

### C. General VM

B plus an execution section in `.arklight` so that Rei functions can run at
runtime.

- **Owns:** everything in B, plus an instruction set, a value model, and
  evaluation rules.
- **Costs:** integer and string semantics must be specified (`LOST.md`
  section 6: wrap versus unbounded integers, division rounding, UTF-16
  versus code point string length), plus a memory story and error model.
- **Justified only if** Rei code must run after the build, which
  contradicts "compiler first, runtime last" and Rei's own "no runtime"
  non-goal, and is the open Rei question 3 (build-time evaluator).
- **Fits the JVM analogy best,** and the doctrine worst.

### Comparison

| | A | B | C |
| --- | --- | --- | --- |
| Owns layout and paint | yes | yes | yes |
| Runs `State`/`Watch`/`Action` | no | yes | yes |
| Runs user (Rei) code at runtime | no | no | yes |
| `.arklight` growth needed | styling, layout | plus behavior specs | plus instructions |
| Fit with the doctrine | good | good | poor |
| Fit with "JVM for Rei" | weak | moderate | strong |
| Needs a Rei decision first | no | no | yes (Rei question 3) |

**Recommendation.** Take **B** as the target and **A** as its first rung.
The closed vocabulary means B is a complete runtime for everything ARKlight
can express today. Revisit **C** only when Rei's evaluator decision says
runtime code is needed. Until then, "the JVM for Rei" is best read as "the
one runtime every Rei-authored site runs on", not "a bytecode interpreter".
**[proposed, needs maintainer confirmation]**

## 4. One thing or two

Flutter separates the Dart VM from the drawing engine. ARKVM's single name
covers a renderer and a reactive core. They have different dependencies
(the renderer needs fonts and a rasterizer; the reactive core needs
neither) and different test strategies. Option: one repo, two
independently testable components inside it, joined by an internal
interface, with the C ABI only at the outer edge. **[open]**

## 5. The boundary with a host

Every design so far gives ARKVM a small platform-specific host: Flutter's
embedder, Experimental-Env's environment implementation, carklight's
desktop and Android shells. Requirements drawn from the prior art
(`Foundational/PRIOR-ART.md`), not a spec:

- **Layered spec.** Core semantics in one document; the environment
  interface in another (the WebAssembly split).
- **Versioned and capability-declaring.** A host states which node types and
  features it can represent. The guarantee is conditional on that
  declaration (`C_ARKlight:docs/ARKVM.md` section 4).
- **Fail loudly.** A construct a host cannot represent is reported with a
  stable code (the `ARK2041` style), never dropped or approximated.
- **Explicit ownership.** Threading and event-loop ownership, and who frees
  what across the boundary, are stated. Both are open in the
  Experimental-Env proposal and must be closed by any real contract.
- **The host owns** window, surface, input, accessibility and clocks.
  **ARKVM owns** the tree, state, layout, and paint.

## 6. Load-time verification

Because `.arklight` can be written by tools other than ARKlight-py (a Rei
compiler, a JS package, a hand edit), ARKVM cannot assume compile-time
validation ran. Proposed: a **verification stage** on load, separate from
format checking, that rejects unknown node types, required props missing,
illegal nesting (text-only nodes), dangling action or state references, and
limit violations. It applies the same fail-loudly rule, one layer down. The
JVM's format-checking versus verification split is the model
(`Foundational/PRIOR-ART.md`).

## 7. Independence from Rei

ARKVM reads `.arklight` and never Rei source. Consequences:

- The Python frontend, Rei and a future JS frontend all work unchanged.
- Rei can change syntax freely without touching ARKVM.
- Anything Rei promises about *language* semantics (integer overflow,
  string length) is a compile-time property, unless option C is chosen.
- ARKVM's obligations to Rei reduce to one: keep `.arklight` sufficient
  to express what Rei can produce.

## 8. Effect on existing docs (if accepted)

Listed so the change is one pass, not a chain of follow-ups.

| Doc | What would need to change |
| --- | --- |
| `C_ARKlight:docs/ADDENDUM.md` section 3 | The "no on-device execution model, ever" sentence, per `docs/ARKVM.md` section 2. |
| `ARKlight:docs/Foundational/SYSTEM-DESIGN-AGREEMENTS.md` | A bounded exception naming ARKVM in sections 3, 15 and 16. |
| `reference/docs/LOST.md` section 0 and 1 | "No JVM underneath" is true only while Rei is inside the Python pipeline; a standalone Rei has ARKVM beneath it. |
| `Proposal/REI-LANGUAGE-PROPOSAL.md`, non-goals and section 6 | "No runtime" scoped explicitly to the playground phase. |
| `C_ARKlight:docs/ARKVM.md` | Reconcile `--minimal`/`--full` with this repo's single runtime. |
| `Experimental-Env` Stage 4 | Point its placeholder at this repo. |

## 9. Non-goals

- Not a JVM, CLR or Wasm implementation; no bytecode compatibility.
- Not a replacement for the HTML backend.
- Not a decision on the Rei evaluator (Rei question 3).
- Not an implementation-language decision beyond noting that the boundary
  is a C ABI.

## 10. Open questions

1. **Shape.** A, B or C? Recommendation above is B, via A.
2. **One component or two** (section 4).
3. **Doctrine edit.** Is a bounded exception acceptable, and where is it
   written?
4. **`.arklight` growth.** The order in which styling, then behavior
   specs, then anything else is added to the format, and how each is
   versioned. A format version bump policy is needed either way.
5. **Modes.** Does `--minimal` (meaning 1) survive as a mode of ARKVM, or
   as a separate tool?
6. **Implementation language.** C, matching carklight and the FFI proposal,
   or another.
7. **Verification scope** (section 6): what exactly is checked, and does
   it duplicate ARKlight's validator or share code with it.
8. **The web.** Is a browser target an ARKVM target (canvas or Wasm), or
   permanently the HTML backend's alone?

## 11. What would make this accepted

A maintainer chooses a shape (or a revision of one), resolves question 3,
and names an owner for the `.arklight` growth in question 4. On
acceptance, the settled content graduates into `Foundational/`, this file
is removed or reduced to a pointer, and an `Implementation/` staging file
is created in the same change.
