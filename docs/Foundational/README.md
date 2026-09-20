# Foundational

## Overview

The permanent design record for ARKVM: what it is, the words used to
describe it, and the prior art its design leans on. Reference material for
the project as a whole, not a record of one in-flight task.

**These docs are not deletable.** They are updated in place as decisions
land. Anything still undecided belongs in [`../Proposal/`](../Proposal/README.md)
until a maintainer decides.

Because ARKVM is at day zero, most of what is *decided* is small: the
definition and the boundary. The rest is tagged **[open]** and points at the
proposal that owns the question.

## Index

| File | Covers |
| --- | --- |
| [`WHAT-ARKVM-IS.md`](WHAT-ARKVM-IS.md) | The definition, the stack position, the JVM comparison (where it holds, where it breaks), what is and is not guaranteed today, and the state of the sibling repos. |
| [`TERMINOLOGY.md`](TERMINOLOGY.md) | The five meanings "ARKVM" has had across the ecosystem, which one this repo uses, and the shared glossary. |
| [`PRIOR-ART.md`](PRIOR-ART.md) | What ARKVM takes from and refuses from the JVM Specification, WebAssembly, Flutter's engine, and the ecosystem's earlier designs, with sources. |
| [`WHAT-REI-IS.md`](WHAT-REI-IS.md) | What Rei (ARKlight's official native source language) and REIlight are, the full stack from authoring to native app, where ARKVM and its engine sit, the design stance, feature layers, and open questions. Tags each claim **[decided]**, **[proposed]** or **[open]**. |
| [`REI-SYLLABUS.md`](REI-SYLLABUS.md) | Draft course syllabus for Rei, built unit for unit on `WHAT-REI-IS.md`'s feature layers. A template for adopters, not an approved curriculum. |
