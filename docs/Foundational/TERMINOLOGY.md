# Terminology

_Current as of 2026-09-19. This file is the single home for the glossary.
Other docs link here rather than redefining a term._

## One name, five meanings

"ARKVM" is used for five different things across the ecosystem's own
repositories. This repo uses the fifth. The others are listed so nobody
reads a sibling repo's "ARKVM" as this repo's.

| # | Where | What it means there | Relation to this repo |
| --- | --- | --- | --- |
| 1 | `C_ARKlight:docs/ADDENDUM.md` section 3 | A **compile-time multi-target dispatcher**: `.arklight` in, one backend runs, output out, done. Explicitly not a VM in the execution sense; the closer analogy given is LLVM IR feeding codegen backends. | **Conflicts.** This repo's ARKVM is a runtime. That addendum's "no on-device execution model, ever" sentence needs an edit if this repo's meaning is accepted. _(Added 2026-09-20: ARKVM is now decided as AOT only, interpreting nothing on the device, so the sentence may survive as written. See `WHAT-REI-IS.md`, "Effect on existing docs".)_ |
| 2 | State-Driven-UI-Streaming-Prototype, `client/src/arkvm/ARKVM.js` (as described in `C_ARKlight:docs/ARKVM.md` section 1; the prototype repo was not read for these docs) | A browser-side JS runtime that watches per-field update latency and promotes fields between rendering paths. | **Unrelated** unless a maintainer says otherwise. Flagged so it does not become a sixth meaning by accident. |
| 3 | `C_ARKlight:docs/ARKVM.md` | One tool, two modes. `--minimal` is meaning 1. `--full` is an on-device runtime that loads `.arklight`, binds a `StateProvider`, renders and re-renders. Design draft. | **Closest.** This repo's ARKVM is `--full`, plus the pixel-perfect guarantee. Whether the mode split survives is **[open]**. |
| 4 | `Experimental-Env:docs/Proposals/ARKVM-FFI-ENVIRONMENT-CONTRACT-PROPOSAL.md` | A C core for the `State`/`Watch`/`Action` execution model behind a stable C ABI, with each native environment implementing a contract for rendering and events. | **A candidate shape** for part of this repo's ARKVM. Examined in the execution-model proposal. |
| 5 | This repo | The runtime between `.arklight` and backend native code that guarantees pixel-perfect output; the layer under Rei as the JVM is under Java. | This is it. |

Meanings 3, 4 and 5 are compatible. Meaning 1 contradicts 5. Meaning 2 is
separate. Reconciling 1 with 5 is in
`Proposal/ARKVM-EXECUTION-MODEL-PROPOSAL.md`.

## Glossary

| Term | Meaning | Source |
| --- | --- | --- |
| **Rei (language)** | The official native source language of ARKlight **[decided]**. No compiler exists yet. Say "the Rei language" wherever confusion is possible. | `Foundational/WHAT-REI-IS.md`; `Proposal/REI-LANGUAGE-PROPOSAL.md`, open question 7 |
| **REIlight** | A superset of ARKlight, and distinct from it **[decided]**. Its contents are **[open]**. The working reading is ARKlight plus ARKVM, its engine and the native targets, with the Rei language as its native authoring language. | `Foundational/WHAT-REI-IS.md`, open question 16 |
| **Rei (narrator)** | The compiler-narrator persona, an opt-in `--narrate` build flag in ARKlight. A different thing sharing the name. | same |
| **ARK AST** | The tree of `ARKNode(type, props, children)` produced by authoring. | `ARKlight:docs/Foundational/ARCHITECTURE.md` |
| **Website IR** | The normalized, validated, backend-independent in-memory tree (`WebsiteIR`/`IRNode`). Models intent, not markup. | same |
| **IR vs `.arklight`** | IR is the human-facing shape; `.arklight` is the machine-facing binary encoding of it. They are not two names for one thing. Only `.arklight` crosses a process boundary. | `C_ARKlight:docs/TERMINOLOGY.md` |
| **`.arklight`** | Versioned binary encoding of the Website IR: magic `ARKL`, `u16` format version, schema-generation tag, string table, body. `FORMAT_VERSION = 1`. | `ARKlight:arklight/ir/binary.py` |
| **`.ark` bundle** | A packed, optionally sealed multi-file site bundle. A different artifact from `.arklight`. | `ARKlight:docs/Foundational/AUTHORING-GUIDE.md` |
| **Backend** | The compilation mechanism that turns IR into output files. | `ARKlight:arklight/backend/base.py` |
| **Environment** | The target execution context a mechanism produces for. Kept distinct from "backend" so Android is not treated as merely another output format. | `Experimental-Env:docs/Foundational/ARCHITECTURE.md` |
| **Embedder** | Flutter's word for the platform-specific host that supplies an entry point, a drawing surface, input, accessibility and an event loop to the engine. The likely analogue of an ARKVM environment implementation. | Flutter architectural overview (`PRIOR-ART.md`) |
| **Environment Interface Contract** | The fixed, versioned C-ABI boundary between an ARKVM core and an environment implementation. Proposed, not decided. | Experimental-Env proposal, section 3 |
| **Compiler first, runtime last** | ARKlight's design agreement: do at compile time everything that can be known, delegate the rest to the target, ship the smallest bridge, and add a runtime abstraction only last. | `ARKlight:docs/Foundational/SYSTEM-DESIGN-AGREEMENTS.md` |
| **Pixel perfect** | The goal. **[open]**, not yet defined. | `Proposal/PIXEL-PERFECT-CONTRACT-PROPOSAL.md` |
| **Reference renderer** | A renderer whose output defines correct. Proposed, not decided. | same |
