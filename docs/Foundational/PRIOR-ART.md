# Prior Art

_Current as of 2026-09-19. External sources were read on that date. All
descriptions of them are paraphrase; no specification text is reproduced.
Cite by section, write ARKVM's own prose, the same rule
`Proposal/REI-LANGUAGE-PROPOSAL.md` sets for C99._

Each entry answers three things: what the source is, what ARKVM takes from
it, and what ARKVM refuses. Takeaways are **[proposed]** unless marked
otherwise; they feed the proposals, they do not settle them.

## 1. The Java Virtual Machine Specification

**Sources.** JVMS SE 25, chapter 1
(<https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-1.html>) and
chapter 4, the class file format
(<https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-4.html>). Chapters
2 (structure of the VM), 5 (loading, linking, initialization) and 6 (the
instruction set) were **not** read for this pass; read them before citing.

**What it says, in short.** The JVM is defined by the class file format,
not by the Java language. Any language whose meaning fits in a valid class
file can run on it. Class files carry a `major.minor` version, and each JVM
release states the range of majors it accepts (SE 25: 45 through 69).
Chapter 4 separates *format checking* (4.8) from *verification* (4.10),
and closes with an explicit list of the VM's own limits (4.11).

**Takes.**

- **The VM reads the artifact, never the language.** ARKVM reads
  `.arklight` and has no Rei or Python front door. This keeps the Python
  frontend, Rei, and a future JS frontend interchangeable.
- **Load-time verification, separate from format checking.** The JVM
  verifies because class files can come from anywhere. `.arklight` can too:
  a file may be written by a Rei compiler, a JS package, or a hand edit.
  ARKlight's compile-time validation cannot be assumed to have run. ARKVM
  should re-check what it loads and fail loudly. Today the decoder only
  does format checking (`ArklightFormatError`).
- **A stated support range for the artifact version.** `.arklight` needs a
  policy: which format versions a given ARKVM accepts, and what an
  unknown-newer file does.
- **A written list of limits.** Maximum tree depth, node count, string table
  size, and so on, stated in the spec rather than discovered by crashing.

**Refuses.** The instruction set, the operand stack, the class model, and
the whole memory, loading and threading apparatus. `.arklight` v1 has no
instructions, and Rei drops the JVM mechanisms (`reference/docs/LOST.md`
section 1).

## 2. WebAssembly

**Sources.** WebAssembly Core Specification, section 1.1 (introduction,
design goals, scope): <https://www.w3.org/TR/2026/CRD-wasm-core-2-20260513/>.
The project's nondeterminism note:
<https://webassembly.org/docs/nondeterminism>. Wasmtime's guide to fully
deterministic execution:
<https://docs.wasmtime.dev/examples-deterministic-wasm-execution.html>.

**What it says, in short.** Wasm is a portable, sandboxed code format that
makes no web-specific assumptions. Its specification is layered: a core
layer defines the instruction set, binary encoding, validation and
execution, while embedding environments, such as the JS API, are specified
separately. Wasm describes itself as having limited, local nondeterminism:
the places where two conforming engines may differ are a short, enumerated
list (NaN payload bits, failed memory growth, host imports, thread
interleaving, relaxed SIMD), and a difference in one place does not leak
elsewhere. Runtimes such as Wasmtime offer switches that remove the
remaining nondeterminism, at a performance cost.

**Takes.**

- **Layer the spec: core versus embedding.** ARKVM core semantics in one
  document; the environment interface (what a host supplies) in another.
  This is the same split the Experimental-Env proposal draws with its
  Environment Interface Contract.
- **Guarantee determinism by enumerating the exceptions, not by claiming
  none.** This is the model for "pixel perfect". Say exactly where two
  conforming ARKVMs are allowed to differ, keep that list short and local,
  and make everything else identical. See
  `Proposal/PIXEL-PERFECT-CONTRACT-PROPOSAL.md`.
- **A strict mode that closes the remaining gaps.** Wasmtime's
  canonicalization switches are the pattern for an ARKVM "reference" mode
  that trades speed for exact reproducibility.

**Refuses.** A general-purpose instruction set and linear memory. Wasm
targets computation; `.arklight` targets a UI tree.

## 3. Flutter's engine and embedders

**Sources.** Flutter architectural overview
(<https://docs.flutter.dev/resources/architectural-overview>), Life of a
Flutter Frame
(<https://github.com/flutter/flutter/wiki/Life-of-a-Flutter-Frame>), and the
Flutter team's golden-file testing page
(<https://github.com/flutter/flutter/wiki/Writing-a-golden-file-test-for-package:flutter>).

**What they say, in short.** Flutter is a layered system. A platform
embedder, written in a language natural to the platform, provides the
entry point, coordinates rendering surfaces, accessibility and input with
the OS, and runs the message loop. A frame is produced by a UI thread and
rasterized on a separate raster thread: the layer tree is walked (a
preroll pass, then paint), drawing goes through Skia or Impeller onto a
Metal, OpenGL, Vulkan or software surface, and the embedder is called back
to present. Flutter's own golden tests run on Linux, Windows, macOS and web,
managed through Skia Gold, to account for occasional subtle rendering
differences between those platforms.

**Takes.**

- **The embedder pattern.** A small platform-specific host owns the
  window, input, surface and event loop; the engine owns everything else.
  This is the shape of ARKVM's environment implementations.
- **A staged frame pipeline** with named phases (state change, tree,
  layout, paint, present), so conformance can be tested per phase and a
  failure localized.
- **Support for a software surface.** A CPU rasterizer path is what makes
  an exact reference renderer possible.

**Refuses / warns.**

- **Painting your own pixels does not by itself make them identical.** The
  Flutter team tests goldens across four platforms because output differs
  subtly between them. Issue reports describe text
  weight differences attributed to platform font rasterizers (for example
  <https://github.com/flutter/flutter/issues/67034>; that thread is
  anecdotal and unconfirmed by the engine team). ARKVM must own the text
  and anti-aliasing path if its guarantee is to be stronger than
  Flutter's.
- **The Dart VM is a separate component from the drawing engine.** ARKVM
  should decide deliberately whether it is one thing or two.

## 4. The ecosystem's earlier designs

These are internal prior art. Each is the home of its own facts; this
section only says what ARKVM inherits.

| Design | Inherited | Where |
| --- | --- | --- |
| Fixed backend interface, compile-time-registered array, no per-backend special cases in the core | The shape of ARKVM's target/environment registration. | `C_ARKlight:docs/ADDENDUM.md` section 4.1 |
| Hand-rolled, zero-dependency binary house style | A constraint on any ARKVM dependency, including a 2D rasterizer. It is in tension with embedding Skia. | `ARKlight:arklight/ir/binary.py`; `C_ARKlight` README ("zero external dependencies") |
| `--minimal` / `--full` modes, `StateProvider`, target capability advertisement, and the MCU warning that "the same `.arklight` app runs everywhere" stops being true unless capabilities say which component types are representable | The idea that ARKVM's guarantee is conditional on a declared capability set. | `C_ARKlight:docs/ARKVM.md` sections 2 to 4 |
| C core behind a stable C ABI; environment callbacks for rendering and events; `ARK2041`-style unsupported-construct reports; open questions on threading, memory ownership, versioning | A candidate execution-core boundary. | `Experimental-Env:docs/Proposals/ARKVM-FFI-ENVIRONMENT-CONTRACT-PROPOSAL.md` |
| "Static UI first, no behavioral runtime yet"; "semantic styling, not CSS" | A staging order, and the rejection of an IR -> CSS -> native round trip. | `Experimental-Env:docs/Foundational/ARCHITECTURE.md` |
| The JVM mechanisms Rei gives up, and the semantics that stop being free (integer wrap, division rounding, string length) | The list of things ARKVM does **not** have to provide, and the things it would have to specify if Rei code ever ran at runtime. | `reference/docs/LOST.md` sections 1, 4, 6 |
| "Compiler first, runtime last", especially the decision order in section 16 | The doctrine ARKVM has to be reconciled with. | `ARKlight:docs/Foundational/SYSTEM-DESIGN-AGREEMENTS.md` |

## Not yet read

- JVMS chapters 2, 5 and 6.
- The Skia and Impeller design documents, for their actual determinism
  claims and the text stack (shaping, hinting) each uses.
- The State-Driven-UI-Streaming-Prototype repository, for meaning 2 in
  `TERMINOLOGY.md`.
- LVGL or a similar immediate-mode renderer, relevant to the MCU-class
  targets `C_ARKlight:docs/ARKVM.md` mentions.
