# The Pixel-Perfect Contract: What ARKVM Is Allowed to Promise

## Status

**Proposed. Not accepted, not scheduled.** Filed 2026-09-19 against ARKlight
`alpha` @ `6983823` (`v0.06502`). Nothing here is committed to.

- **Type:** Specification strategy.
- **Question:** ARKVM exists to keep output "pixel perfect no matter what
  target backend it is". What does that sentence have to mean for ARKVM to
  be able to keep it, test it, and be held to it?
- **Not covered:** what ARKVM executes and owns. That is
  [`ARKVM-EXECUTION-MODEL-PROPOSAL.md`](ARKVM-EXECUTION-MODEL-PROPOSAL.md).
- **This file is the single home** for the definition of "pixel perfect".
  Other docs link here.

## TL;DR

- Read literally, "pixel perfect on every backend" is either unattainable or
  untestable, and needs a definition before ARKVM can promise it.
- The right model is WebAssembly's: guarantee determinism by **enumerating
  the exact places two conforming implementations may differ**, keeping that
  list short, and treating any unlisted difference as a bug.
- **Proposed guarantee (the author's, for a maintainer to confirm):** three
  tiers. **G0**, every conforming host produces identical *layout geometry*.
  **G2**, ARKVM's own reference renderer produces *bit-identical* pixels on
  every OS and architecture. **G1**, a host that delegates painting to a
  platform surface stays within a stated tolerance of the reference.
- The hard parts are **text**, **the layout model**, and **the size of the
  node vocabulary**. The vocabulary is 90 HTML-shaped node types, including
  form controls, tables, ruby, video and iframes that a browser draws for
  you. A guarantee has to be scoped to a declared subset.
- "Pixel perfect" is a function, not a property:
  `pixels = R(arklight, state, params)`. Every input must be named.

## 1. Why a bare "pixel perfect" cannot stand

Two failure modes.

1. **The literal reading is unattainable.** Different GPUs, font
   rasterizers and color pipelines do not produce identical bits. A promise
   ARKVM cannot keep is worse than a smaller promise it can.
2. **The loose reading is vacuous.** If "close enough" is undefined, no
   test can fail, and the guarantee is decoration.

The project needs a claim that is strong, bounded, and mechanically
checkable.

## 2. What varies

Evidence is labelled. **Documented** means a project's own page says it.
**Reported** means an issue or forum thread, not confirmed by maintainers.
**Analysis** means the author's engineering reasoning, to be tested.

| Source of variance | Why it varies | Evidence |
| --- | --- | --- |
| Text rasterization | Platforms use different font rasterizers, hinting and subpixel positioning. | Reported: text weight differing between platforms in a Flutter issue that attributes it to CoreText versus other rasterizers (`flutter/flutter#67034`). Analysis for the general case. |
| Text shaping and font choice | Shaping, kerning, ligatures and font fallback depend on the shaper and on which fonts exist. | Analysis. |
| Anti-aliasing and coverage | Edge coverage and blending differ between rasterizers. | Documented, in general terms: Flutter's own goldens run on four platforms to account for subtle rendering differences. |
| CPU versus GPU rasterization | Metal, OpenGL, Vulkan and software surfaces are different code paths. | Documented that all are surfaces Flutter draws to (Life of a Flutter Frame). That they differ pixelwise: Reported and Analysis. |
| Image decode and scaling | Decoders and resampling filters differ. | Analysis. |
| Color and blending | Color space, gamma and blend-space choices. | Analysis. |
| Layout arithmetic | Fractional pixels, rounding, and device pixel ratio. | Analysis. |
| Font availability | Base CSS names system font stacks (`ui-monospace, SFMono-Regular, Menlo, Consolas, ...`) and a `--ark-font-family` variable; system fonts differ per device. | Verified: `ARKlight:arklight/backend/css/base_stylesheet.py`. |

Sources: <https://github.com/flutter/flutter/wiki/Writing-a-golden-file-test-for-package:flutter>,
<https://github.com/flutter/flutter/issues/67034>,
<https://github.com/flutter/flutter/wiki/Life-of-a-Flutter-Frame>.

The lesson from Flutter is the important one. An engine that paints its own
pixels through Skia or Impeller still needs multi-platform golden testing.
**Owning the renderer is necessary, not sufficient.** ARKVM's guarantee is
only stronger than Flutter's if ARKVM also owns the text path.

## 3. The precedent to copy: limited, local nondeterminism

WebAssembly does not claim bit-for-bit sameness everywhere. It states that
its nondeterminism is *limited* to a short, enumerated list (for example,
NaN payload bits and thread interleaving) and *local* (a difference in one
place does not leak elsewhere). Everything not on the list is fully
determined. Runtimes such as Wasmtime add a strict configuration that
removes the residual differences at a speed cost.

That is a contract a project can test: every observed cross-implementation
difference is either on the list or a bug. Sources:
<https://webassembly.org/docs/nondeterminism>,
<https://docs.wasmtime.dev/examples-deterministic-wasm-execution.html>.

ARKVM adopts the structure: **a normative list of permitted divergences,
and a strict mode with none.**

## 4. Candidate guarantee levels

| Level | Promises | Cost | Testable by | Reachable on |
| --- | --- | --- | --- | --- |
| **G0 structural** | Same resolved styles, same layout geometry (box positions and sizes in a defined unit), same text runs. Painting unspecified. | Needs a normative layout spec. No rasterizer needed. | Compare a dumped layout tree, exact. | Every host, including constrained ones. |
| **G1 perceptual** | Pixels within a stated metric and threshold of the reference output. | Choose and justify a metric. | Image comparison with tolerance. | Hosts that paint through a platform surface. |
| **G2 reference-exact** | One designated renderer, software, with bundled fonts and its own text stack and arithmetic, is bit-identical on every OS and architecture. | Own the rasterizer and text path; pin the arithmetic. | Exact hash of the framebuffer. | Any target that can run the reference renderer. |
| **G3 exact everywhere** | Every conforming renderer is bit-identical. | Forbids GPU and platform paths, or pins them to one software algorithm. | Exact hash. | Only by giving up native rendering. |

**Proposed.** Require **G0** of every conforming host. Define "pixel
perfect" as **G2** for ARKVM's own reference renderer. Accept **G1** for a
host that delegates painting, stated against the reference. Reject **G3**
as a default; it is what a strict mode could opt into. **[needs maintainer
confirmation]**

The reasoning: G0 is cheap and catches most real "it looks different" bugs
(wrong wrapping, wrong sizes) before any painting exists, so it can ship
first. G2 gives the README's promise an unambiguous meaning on the one
renderer ARKVM controls. G1 keeps native and GPU paths possible without
pretending they are bit-exact.

## 5. Decisions the guarantee forces

### 5.1 Inputs: pixels are a function

Name every parameter. Proposed set: the `.arklight` file, resolved state,
viewport size, device pixel ratio, color scheme, locale, the font set, and
the ARKVM version. The guarantee is: same inputs, same pixels (G2). Anything
outside the set, such as animation timing, is outside the guarantee.

### 5.2 Scope: which targets are inside?

The README says "no matter what target backend". Three readings:

1. **ARKVM-hosted targets only.** Native hosts that embed ARKVM. The
   guarantee is meaningful and reachable.
2. **Plus a web target through canvas or Wasm.** ARKVM draws in the
   browser. Reachable, and deliberately bypasses the browser's layout.
3. **Also the existing HTML backend.** The browser lays out and paints
   ARKlight's generated HTML. Matching that means replicating a browser's
   layout and text engine for every element ARKlight emits. Not credible as
   a promise.

**Proposed:** readings 1 and 2. The HTML backend is outside the guarantee.
That contradicts nothing in ARKlight, which keeps web output ordinary
(`SYSTEM-DESIGN-AGREEMENTS.md` section 9), but it changes what the README
sentence can honestly claim. **[maintainer decision]**

### 5.3 The layout model

Today's styling is CSS-shaped, and the vocabulary is HTML-shaped. Measured
on 2026-09-19:

- The base stylesheet is about 11.9 KB and declares 56 distinct CSS
  properties (excluding `--*` custom properties), among them table
  (`border-collapse`, `caption-side`), ruby (`ruby-position`) and
  form-control (`field-sizing`, `accent-color`, `resize`) properties.
  Layout modes present: `block`, `flex`, `grid`, `none`. Units:
  `px`, `rem`, `em`, `ch`, `vh`, `vw`, `fr`. Functions: `calc`, `clamp`,
  `min`, `minmax`, `repeat`, `var`. One `@media` rule.
- The 90 node types include those a browser draws with no help from the
  site: `Input`, `Select`, `Textarea`, `Datalist`, `Dialog`, `Details`,
  `Meter`, `Progress`, `Video`, `Audio`, `IFrame`, `Picture`; a table family
  (`Table`, `TableRow`, `TableCell`, ...); `Ruby`/`Rt`/`Rp`; and bidi nodes
  (`Bdi`, `Bdo`).
- Default appearance for plain elements comes from the browser's own
  user-agent stylesheet. ARKVM has no UA stylesheet and must define one.

So ARKVM must **specify its own layout algorithm normatively**. Options:

- **(a) A named CSS subset with exact algorithms.** Familiar to authors,
  large to specify, and the tempting mistake is to say "like a browser".
- **(b) A new semantic layout model** lowered from the IR, per
  Experimental-Env's principle "semantic styling, not CSS", which explicitly
  rejects going IR to CSS to native. Smaller and honest, but authors' CSS
  intuition no longer transfers.

Either way, the **capability manifest** (section 5.6) has to say which node
types and properties are inside the guarantee. Form controls, media, iframes
and ruby are candidates to declare unsupported at first, and report loudly,
rather than approximate. **[open]**

### 5.4 The text stack

Text is the largest source of variance and the largest cost. To reach G2,
the reference renderer needs:

- **Bundled fonts** and no system-font lookup in reference mode. This has
  licensing and size consequences, and the base stylesheet's system font
  stacks would map to bundled ones.
- **A defined shaping and line-breaking algorithm**, either written or
  taken as a dependency at a pinned version.
- **Hinting off** and a fixed subpixel-positioning rule.
- **A declared script and language scope.** Complex scripts and bidi
  (`Bdi`, `Bdo`) are expensive. Latin-first with an honest "unsupported"
  report is better than partial support that is not bit-stable.

**[open]**: shaper choice, font choice, initial script coverage.

### 5.5 The rasterizer

- **Embed a mature 2D library such as Skia.** Fastest to good output.
  Conflicts with the ecosystem's zero-dependency binary house style and
  adds a large dependency. Whether a given Skia version is bit-stable
  across platforms in software mode has **not been verified**. Read its
  design notes before relying on it.
- **Write a small software rasterizer.** Full control over arithmetic, so G2
  becomes achievable by construction (integer or fixed-point coverage, no
  platform floating-point surprises). Real engineering cost.

**[open]**. Either way, the arithmetic must be pinned. WebAssembly's
NaN-payload story is the cautionary example: a reasonable float rule that
still leaks into observable behavior.

### 5.6 Capabilities and honesty

The guarantee is conditional on what a host declares it can represent, as
`C_ARKlight:docs/ARKVM.md` section 4 argues for MCU-class targets: without
this, "the same `.arklight` app runs everywhere" fails the first time it
meets real hardware. A host publishes a capability manifest (node types,
properties, fonts, features). A file that uses something outside it gets a
**stable error code** (the `ARK2041` style), never a silent approximation.

## 6. Proposed invariants (each one a test)

- **P1. Reference determinism.** For fixed inputs (section 5.1), the
  reference renderer's framebuffer hash is identical on every supported OS
  and architecture.
- **P2. Host tolerance.** A delegating host's output is within the declared
  G1 threshold of the reference on the conformance corpus.
- **P3. Geometry equality.** Every conforming host produces the same layout
  tree (G0) for the corpus.
- **P4. Refuse, never approximate.** A construct outside the capability
  manifest yields a reported error, not degraded output.
- **P5. No ambient dependence.** In reference mode the output does not
  depend on system fonts, locale settings beyond the declared parameters,
  or the GPU.
- **P6. The divergence list is complete.** Any observed difference not on
  the enumerated list is a defect, and the list is short enough to read in
  one sitting.

## 7. Conformance plan (sketch)

- **Corpus.** `.arklight` files with expected outputs: ARKlight's example
  sites (`examples/hello_site`), plus purpose-built cases for wrapping,
  grid, flex, overflow, images, and each supported node type.
- **Exact tier.** Store hashes, not images, to keep the repo small. On a
  mismatch, dump the framebuffer as an artifact for inspection.
- **Tolerance tier.** Image diff with the chosen metric, and a stored
  reference image per case.
- **Layout-only tests** use a fixed test font whose glyphs are plain boxes,
  a technique the Flutter ecosystem uses to keep goldens independent of
  anti-aliasing (for example the `golden_bricks` package). That isolates
  layout bugs from text bugs.
- **Phase-localized failures.** Dump the tree after state, layout and paint,
  so a failing case names its phase, following the frame pipeline in Life of
  a Flutter Frame.
- **Matrix.** Linux, macOS, Windows, and an Android environment, at minimum,
  against the same hashes.

## 8. Non-goals

- Matching a browser's output for the same site.
- Promising performance, animation timing, or scroll physics.
- Defining accessibility output.
- A decision on the rasterizer, shaper or fonts.
- Any change to the HTML backend.

## 9. Open questions

1. **Scope.** Readings 1 and 2, as proposed, or does the maintainer intend
   the HTML backend to be inside the promise? This one decides how the
   README's sentence should be reworded.
2. **Tier definitions.** Are G0, G1 and G2 the right cut?
3. **Layout model.** CSS subset (a) or semantic model (b)?
4. **Initial supported vocabulary.** Which of the 90 node types are in the
   first guarantee? What does "unsupported" report look like?
5. **Text.** Shaper, bundled font, initial script coverage.
6. **Rasterizer.** Embedded library or written in-house, and, if embedded,
   whether its software output is verifiably bit-stable across platforms.
7. **Tolerance metric** for G1, and the threshold.
8. **Color.** sRGB only? Blend in linear or gamma space?
9. **Who owns the corpus,** and where expected hashes live.

## 10. What would make this accepted

A maintainer answers question 1, chooses the tier definitions, and
picks a layout model. On acceptance, the definition graduates into
`Foundational/` (replacing the "[open]" in `WHAT-ARKVM-IS.md`), this file
is removed or reduced to a pointer, and an `Implementation/` staging file is
created in the same change. The natural first rung is G0 with a layout dump,
since it needs no rasterizer.
