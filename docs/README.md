# ARKVM Documentation

This folder is the documentation index for ARKVM. Start here, then follow
the links into the subfolder for the topic you need.

_Current as of 2026-09-19. ARKVM has no code yet; everything below is
documentation, and most of it is deliberately marked unsettled._

## What ARKVM is, in one paragraph

ARKVM is the runtime that guarantees ARKlight output stays pixel perfect
whatever the target backend. It sits between `.arklight` (the binary IR)
and a backend's native code, in the role Skia plays for Flutter. When Rei
becomes its own language, ARKVM is what sits underneath her, the way the
JVM sits underneath Java. The full statement, with its limits, is
[`Foundational/WHAT-ARKVM-IS.md`](Foundational/WHAT-ARKVM-IS.md).

## Folder Guide

Each folder is one *state* in a doc's lifecycle, not a topic. This mirrors
the convention ARKlight's own `docs/` uses, scaled to the states this repo
needs today.

| State | Folder | Leaves the folder when... |
| --- | --- | --- |
| Proposed, not yet decided | [`Proposal/`](Proposal/README.md) | A maintainer accepts it (graduates to `Implementation/`, and its settled content to `Foundational/`) or rejects it (removed). |
| Accepted, staged, in flight | [`Implementation/`](Implementation/README.md) | Every stage ships. The outcome is captured in `Foundational/` and the staging file is trimmed or removed. |
| Settled, permanent design record | [`Foundational/`](Foundational/README.md) | Never. Updated in place, not deleted. |

### [`Foundational/`](Foundational/README.md) permanent

| File | Covers |
| --- | --- |
| [`WHAT-ARKVM-IS.md`](Foundational/WHAT-ARKVM-IS.md) | The definition, where ARKVM sits in the stack, where the JVM comparison holds and where it breaks, what is and is not guaranteed today. |
| [`TERMINOLOGY.md`](Foundational/TERMINOLOGY.md) | The five meanings "ARKVM" has had across the ecosystem and which one this repo uses; a glossary (IR vs `.arklight`, backend vs environment, embedder, the two Reis). |
| [`PRIOR-ART.md`](Foundational/PRIOR-ART.md) | What ARKVM takes from, and refuses from, the JVM Specification, WebAssembly, Flutter's engine, and the ecosystem's own earlier designs. External sources cited by section, with dates. |

### [`Proposal/`](Proposal/README.md) unsettled

| File | Covers |
| --- | --- |
| [`ARKVM-EXECUTION-MODEL-PROPOSAL.md`](Proposal/ARKVM-EXECUTION-MODEL-PROPOSAL.md) | What ARKVM actually executes: renderer only, renderer plus reactive core, or a general VM. Reconciliation with "compiler first, runtime last". The embedder boundary. |
| [`PIXEL-PERFECT-CONTRACT-PROPOSAL.md`](Proposal/PIXEL-PERFECT-CONTRACT-PROPOSAL.md) | What "pixel perfect" is allowed to mean, why it is hard, candidate guarantee levels, a conformance plan. |
| [`REI-LANGUAGE-PROPOSAL.md`](Proposal/REI-LANGUAGE-PROPOSAL.md) | The Rei language (upstream of ARKVM, not part of it). Imported from the Rei-Src workspace; filed against ARKlight `alpha` @ `8cafffe`. |

### [`Implementation/`](Implementation/README.md) accepted, staged

Empty. No proposal has been accepted, so no staging ladder exists yet.

## Material outside `docs/`

`reference/` holds read-only research material, not project documentation.

| Path | What it is | Caveat |
| --- | --- | --- |
| `reference/docs/LOST.md` | What the Rei language gives up relative to Java, by layer. | Written for the Rei-Src workspace. Its "no runtime / no VM underneath" framing describes the playground phase, before Rei is standalone. See the execution-model proposal, "Effect on existing docs". |
| `reference/README.md` | Describes the Rei-Src layout. | Stale here: it lists an `ARKlight/` clone and `PROVENANCE.txt` files that are not in this repo. |
| `reference/reference/openjdk/` | A GPL-2.0-only slice of OpenJDK (`java.base`, plus the javac parser and tree). | Design reference only. Do not copy code from it (see `LOST.md` section 7). |

## Notation

Files in sibling repositories are written `Repo:path`.

| Prefix | Repository |
| --- | --- |
| `ARKlight:` | [`ARKlight-Ecosystem/ARKlight`](https://github.com/ARKlight-Ecosystem/ARKlight), branch `alpha` |
| `C_ARKlight:` | [`Rae-ARK/C_ARKlight`](https://github.com/Rae-ARK/C_ARKlight) ("carklight", the C reimplementation) |
| `Experimental-Env:` | [`Rae-ARK/ARKlight-Experiemental-Env`](https://github.com/Rae-ARK/ARKlight-Experiemental-Env) (sic, the URL carries that spelling) |

Status tags used across these docs: **[decided]** stated by the maintainer,
**[proposed]** filed in `Proposal/`, **[open]** not designed yet.

## Adding a new doc

Two checks before writing a file or a section.

1. **Does this fact already have a home?** Search first. If it does, link
   to it instead of restating it. Copied facts drift the moment one copy is
   updated and the other is not. Examples of single homes here: the
   glossary lives in `TERMINOLOGY.md`, the definition of "pixel perfect"
   lives in `PIXEL-PERFECT-CONTRACT-PROPOSAL.md`, the JVM comparison lives
   in `WHAT-ARKVM-IS.md`.
2. **Which folder matches the doc's *state*?** Use the table above. If a
   file matches no row, it is probably a section of an existing file.

Then, **in the same change, not a follow-up**:

1. Add or update the doc.
2. Add or update its row in its folder's own `README.md`.
3. Add or update its row in the Folder Guide above.
4. If a proposal is accepted, create its `Implementation/` staging file in
   that same change, and move its settled content into `Foundational/`.

The reverse holds too. When a file leaves a folder, deleting it and
removing its rows in both indexes is one change. A stale row is the same
bug as a missing one.
