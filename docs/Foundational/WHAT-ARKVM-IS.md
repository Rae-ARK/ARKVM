# What ARKVM Is

_Current as of 2026-09-19. Read against ARKlight `alpha` @ `6983823`
(`v0.06502`), C_ARKlight @ `a87a9ba`, and Experimental-Env @ `b3a9e3e`.
Tags: **[decided]** stated by the maintainer, **[proposed]** filed in
`docs/Proposal/`, **[open]** not designed._

## The definition **[decided]**

ARKVM is the runtime that guarantees ARKlight output stays pixel perfect
whatever the target backend. It sits between `.arklight` (the binary IR)
and a target backend's native code, in the role Skia plays for Flutter.

It is also the layer underneath Rei, the way the JVM is the layer
underneath Java: Rei's compiler targets `.arklight`, and ARKVM is what runs
`.arklight`. (Maintainer framing, 2026-09-19.)

## Where it sits

```
 authoring       Python today  ->  Rei (planned, standalone language)
     |
     v
 compiler        ARK AST -> normalization -> validation -> Website IR
     |            (ARKlight; ARKVM changes nothing here)
     v
 .arklight       binary, versioned, schema-tagged     <- ARKVM's only input
     |
     v
 ARKVM           this repository
     |
     v
 backend         native code for a target (Android, Desktop, ...)
     |
     v
 pixels
```

Two facts about the `.arklight` step, both verified against source:

- Today it is written by `arklight build --emit-arklight` and read back by
  `arklight build site.arklight`. The encoder and decoder are
  `ARKlight:arklight/ir/binary.py`, `FORMAT_VERSION = 1`. C_ARKlight is the
  other intended reader.
- Version 1 covers the structural tree plus each page's route, `state` and
  `computed_initial`. It does **not** yet carry watch, persist, media or
  query specs, custom styles, CSS variable overrides, or several other
  `WebsiteIR` fields; a decoded site gets stock defaults for those. So an
  ARKVM reading v1 cannot yet recover a site's styling. The format has to
  grow before ARKVM can be a complete runtime.

Whether the **web** target sits inside this picture is **[open]**. The
HTML backend emits ordinary HTML/CSS/JS that a browser renders
(`ARKlight:docs/Foundational/SYSTEM-DESIGN-AGREEMENTS.md`, section 9), and
that output is not something ARKVM draws. See
`Proposal/PIXEL-PERFECT-CONTRACT-PROPOSAL.md`, question 1.

## The JVM comparison

| Java | Rei and ARKVM | State today |
| --- | --- | --- |
| Java language | Rei language | Proposal only. No Rei code exists. |
| `javac` | Rei compiler; until then ARKlight's Python pipeline | Python pipeline ships; Rei compiler does not exist. |
| `.class` file | `.arklight` | Exists, version 1, incomplete (see above). |
| Class-file verifier | Load-time verification inside ARKVM | **[open]**, see the execution-model proposal. |
| The JVM (execution engine) | ARKVM | This repo. No code. |
| `java.base` | Rei standard library; today `stdlib.ARKlight`, a closed UI vocabulary | No general-purpose library. `reference/docs/LOST.md` section 2. |
| HotSpot, OS, hardware | Backend native code | Android is a WebView host; Desktop is a WebKit2GTK host (Linux only so far). In both, a browser engine renders. |

### Where the comparison holds

1. **The VM knows the artifact, not the language.** The JVM Specification
   opens by saying the JVM knows nothing of the Java language, only the
   class file format, and that any language expressible as a valid class
   file can be hosted (JVMS chapter 1). The equivalent rule for ARKVM is
   that it reads `.arklight` and never parses Rei or Python. This is
   already the ecosystem's rule: only `.arklight` crosses to a backend
   (`C_ARKlight:docs/TERMINOLOGY.md`).
2. **Many frontends, one artifact.** Python emits `.arklight` today; Rei
   and the planned JS authoring package would emit the same file
   (`C_ARKlight:docs/ADDENDUM.md`, section 3).
3. **A versioned artifact with a declared support range.** JVMS versions
   the class file `major.minor` and states which majors each release
   accepts. `.arklight` has a `u16` format version and a schema tag; it has
   no stated support-range policy yet. **[open]**
4. **A spec that can be tested against.** A VM is defined by a
   specification and a conformance story, not by one implementation.

### Where it breaks

1. **There is no instruction stream.** `.arklight` v1 is a tree of
   `(type, props, children)` plus state. Behavior is a closed vocabulary,
   not code: 8 actions, 5 behaviors, 12 derivations, 5 predicates and 5
   event modifiers in `ARKlight:arklight/ir/schema.py`. An ARKVM built for
   this input is a renderer and a reactive graph, not an interpreter. A
   real "JVM-like" ARKVM needs `.arklight` to gain an execution section,
   which depends on Rei's build-time-evaluator decision (`LOST.md`
   section 4). **[open]**
2. **The JVM's heavy machinery does not apply.** Garbage collection, class
   loading, threads, JIT and reflection are all things Rei gives up
   (`LOST.md` section 1). ARKVM inherits none of that work, and none of
   those guarantees.
3. **The nearer neighbour is Flutter's engine, not the JVM.** The JVM
   computes; Flutter's engine hosts and paints. In Flutter the Dart VM and
   the drawing engine are separate components. ARKVM's name currently
   covers both roles. **[open]**, see the execution-model proposal.

## What ARKVM guarantees **[open]**

"Pixel perfect" is the goal, and it has no definition yet. There is no
reference renderer, no tolerance, no conformance suite. Even engines that
paint their own pixels, Flutter with Skia, test goldens across several
platforms because output differs subtly between them (`PRIOR-ART.md`).
The candidate definitions and the trade-offs live in
`Proposal/PIXEL-PERFECT-CONTRACT-PROPOSAL.md`, which is the single home
for that question.

## What ARKVM is not

- **Not the compiler.** It does not parse Python or Rei and does not
  validate authoring mistakes. Compile-time failure stays in ARKlight
  ("fail loudly at build time"). Whether ARKVM must also re-validate what
  it loads is **[proposed]**, because it may load files other tools wrote.
- **Not a reader of web output.** It consumes `.arklight`, never generated
  HTML, CSS or JS. Experimental-Env states the same rule ("IR in, native
  views out") and rejects the IR -> CSS -> parse -> native round trip.
- **Not a change to the HTML backend.** Web output stays ordinary web
  output. Whether it falls under ARKVM's guarantee is **[open]**.

## State of the ecosystem

| Repo | Role relative to ARKVM | State |
| --- | --- | --- |
| **ARKVM** (this repo) | The runtime | README and docs. No code. |
| **ARKlight** | Produces `.arklight`; the compiler | Shipping alpha. `.arklight` v1 encoder and decoder present. |
| **C_ARKlight** | C reimplementation; `docs/ARKVM.md` proposes "one name, two modes"; `docs/DESIGN-NOTES.md` floats a shared internal GUI library | Work in progress; tracks ARKlight `v0.0431`. |
| **Experimental-Env** | Tests whether the IR lowers to native Android views; its Stage 4 is "the ARKVM decision" and it holds a C-core plus FFI proposal | Stage 0 scaffold. v1 is non-stateful only. |

## Read next

- [`TERMINOLOGY.md`](TERMINOLOGY.md): which "ARKVM" this repo means, and the glossary.
- [`PRIOR-ART.md`](PRIOR-ART.md): what the JVM Specification, WebAssembly and Flutter teach, and what ARKVM refuses to copy.
- [`../Proposal/ARKVM-EXECUTION-MODEL-PROPOSAL.md`](../Proposal/ARKVM-EXECUTION-MODEL-PROPOSAL.md)
- [`../Proposal/PIXEL-PERFECT-CONTRACT-PROPOSAL.md`](../Proposal/PIXEL-PERFECT-CONTRACT-PROPOSAL.md)
