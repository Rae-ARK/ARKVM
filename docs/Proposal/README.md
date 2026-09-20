# Proposal

## Overview

Design proposals not yet decided by a maintainer. Distinct from
[`../Foundational/`](../Foundational/README.md)'s settled record and
[`../Implementation/`](../Implementation/README.md)'s accepted, staged work.

A proposal leaves this folder when a maintainer accepts it (its settled
content graduates into `Foundational/`, and a staging file is created in
`Implementation/` in the same change) or rejects it (the file is removed).
Every proposal here opens with a Status line saying it is not accepted.

## Index

| File | Covers |
| --- | --- |
| [`ARKVM-EXECUTION-MODEL-PROPOSAL.md`](ARKVM-EXECUTION-MODEL-PROPOSAL.md) | What ARKVM executes and owns. Three candidate shapes (renderer only; renderer plus reactive core; general VM). How ARKVM squares with ARKlight's "compiler first, runtime last" agreement. The embedder boundary and load-time verification. Now constrained by the decisions in `WHAT-REI-IS.md` (AOT only, owned engine). **Proposed, not accepted.** |
| [`PIXEL-PERFECT-CONTRACT-PROPOSAL.md`](PIXEL-PERFECT-CONTRACT-PROPOSAL.md) | The guarantee ARKVM exists to make. Why "pixel perfect" needs a definition, the sources of variance, four candidate guarantee levels, a conformance plan. **Proposed, not accepted.** |
| [`REI-LANGUAGE-PROPOSAL.md`](REI-LANGUAGE-PROPOSAL.md) | The Rei language, a Dart-like source frontend for ARKlight, filed against `alpha` @ `8cafffe`. Upstream of ARKVM, and filed here because ARKVM is Rei's execution layer. Its "Fun tier" and "no runtime" non-goals describe the playground phase only, and Rei is now decided to be ARKlight's official native language (`WHAT-REI-IS.md`). **Proposed, not accepted.** |
