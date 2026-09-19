# Rei-Src

**Rei** is a planned native source language for the
[ARKlight](https://github.com/ARKlight-Ecosystem/ARKlight) compiler framework.
Dart/Flutter-like in feel (a tree of constructor calls with named props and
positional children), C-spec-minimal by design, starting from Java's grammar and
AST as a reference point that is not expected to survive as-is.

**Status: no Rei code exists yet.** This drop is the workspace: ARKlight, the
OpenJDK reference slices, and a written account of what the design gives up.
Start with [`docs/LOST.md`](docs/LOST.md).

## Where Rei slots in

```
Rei source -> Rei AST -> ARK AST -> Component expansion -> Normalization
           -> Validation -> Website IR -> Backends (HTML, CSS, JS, Android, Desktop)
\_____ new: Rei _____/   \_______________ ARKlight, unchanged _______________/
```

Rei replaces ARKlight's authoring layer, which today is Python: the loader that
executes the site module (`ARKlight/arklight/parser/loader.py`) and the static
Python-AST discovery (`parser/discover.py`). Everything from ARK AST down is
untouched.

**The seam.** ARKlight's pipeline consumes `dict[str, ARKNode]`, one tree per
route, from `Site.build_ark_ast()` (`ARKlight/arklight/api.py`). An `ARKNode`
is `(type, props, children)`: props are keyword arguments, children are
positional. Whatever Rei's frontend produces must land there.

## What the Rei surface has to express

Derived from `ARKlight/examples/hello_site/site.py` and `arklight/ast/nodes.py`,
not invented:

- constructor-style calls with named props and positional children, nested
  (`Container(Text("..."), Button("...", on_click="toggle"), class_name="card")`)
- reusable pieces as plain functions (`nav()` returning a tree)
- route registration (`@site.page("/")`) and user components (`@component`)
- preamble directives (`# include <stdlib.ARKlight>`, `# define A -> B`),
  currently comments, natural candidates for real syntax
- structured references, not arbitrary code: `State(...)`, `Bind.when/model`,
  `Action.*`

Java's grammar has no named arguments, no collection literals with inline
`if`/`for`, and requires `new` for construction. That is the concrete reason
"the Java grammar won't survive as-is".

## Layout

```
Rei-Src/
  README.md
  docs/LOST.md                    what Rei gives up, by layer; open decisions
  ARKlight/                       shallow clone, branch alpha (git history kept)
  reference/
    ARKLIGHT-PROVENANCE.txt       upstream URL, commit, version
    openjdk/                      READ-ONLY reference, no .git
      PROVENANCE.txt              upstream URL, commit, subset, license
      LICENSE ASSEMBLY_EXCEPTION ADDITIONAL_LICENSE_INFO
      src/java.base/              the whole module
      src/jdk.compiler/.../javac/parser   Java lexer + parser (15 files)
      src/jdk.compiler/.../javac/tree     JCTree, the concrete AST (11 files)
      src/jdk.compiler/.../source/tree    public Tree API (78 files)
```

`java.base` is the library and contains no grammar or AST. The parser and tree
slices come from `jdk.compiler`, because that is where the Java grammar and AST
actually live. They are separate from `reference/openjdk/src/java.base/` and can
be deleted independently.

## Licensing, read this before writing code

- `ARKlight/`: GPL-3.0-or-later plus ARKlight's Additional Terms. Keep its
  `LICENSE` and notices intact.
- `reference/openjdk/`: GPL v2 **only** plus the Classpath Exception. Not
  combinable with GPLv3 code in one work. **Do not copy code from it** into
  `ARKlight/` or into Rei's implementation. Read for ideas, write your own.
  Details and caveats: `docs/LOST.md` section 7. This is not legal advice.

## Suggested next steps

1. Decide the build-time evaluator (`docs/LOST.md` section 4). It shapes the
   rest of the language.
2. Write Rei's grammar (EBNF) and a one-page spec.
3. Write the lexer and parser from that spec. Do not adapt javac's.
4. Implement Rei AST to ARK AST lowering at the `build_ark_ast()` seam.
5. Port `examples/hello_site` to Rei as the first end-to-end test.

## Restoring full upstream history

```
cd ARKlight && git fetch --unshallow
```

The OpenJDK slice has no `.git`; re-clone `https://github.com/openjdk/jdk` if
you need history.
