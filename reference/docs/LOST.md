# LOST.md — What Rei gives up

_Draft v0. Written against ARKlight `alpha` @ `8cafffe` (v0.06501, 2026-09-19)
and OpenJDK @ `852fa4a56` (2026-09-18). Every ARKlight claim below was read from
the cloned source in `ARKlight/`; every JDK claim from `reference/openjdk/`._

Tags used below:

- **[decided]** follows mechanically from the pipeline; nothing to vote on.
- **[proposed]** my expected cut. A maintainer has to confirm it.
- **[open]** genuinely undesigned. See section 9.

---

## 0. Scope, and one correction

Rei slots in as:

```
Rei source -> Rei AST -> ARK AST -> Component expansion -> Normalization
           -> Validation -> Website IR -> Backends (HTML, CSS, JS, Android, Desktop)
```

Everything from **ARK AST** downward is ARKlight's and does not change. So
"replacing everything underneath Java" means something specific: *underneath*
is no longer a JVM, it is ARKlight's pipeline.

**Correction.** Early in the design chat I framed this as a closed-world AOT
compiler to machine code (HotSpot-style lowering, cache layout, instruction
selection). That does not apply. No machine-code stage exists on Rei's path.
The systems-level questions move to the backends and to the compiler's own
runtime (ARKlight is Python, `requires-python >= 3.10`).

---

## 1. Lost with the JVM  **[decided]**

| Mechanism | What Java gave you | In Rei |
|---|---|---|
| Bytecode + verifier | classfile format, stack-map verification | ARK AST, checked by `arklight/ir/validate.py` |
| JIT / tiered compilation / deopt | profile-guided speculation over an open world | none. Output is static. The open world is gone by construction (design agreement 1: "The Compiler Owns Everything It Can Know") |
| GC and object model | heap, headers, identity, allocation | none at language level. The target runtime owns memory (agreement 3: "Do Not Reimplement the Target Runtime") |
| Class loading, linking, init order (JLS 12.4) | lazy, once-only, thread-safe static init; ClassLoader delegation | none. Name binding happens at compile time; a collision is a build error (`PreambleCollisionError`) |
| Threads, monitors, memory model (JLS 17) | `synchronized`, `volatile`, happens-before, `java.util.concurrent` | none. ARKlight's reactive core (`State(...)`, `Bind.*`, `Action.*`) replaces shared-memory concurrency |
| Runtime exceptions and stack traces | `throw`/`catch`; implicit NPE, AIOOBE, ArithmeticException, ClassCastException | build-time diagnostics ("fail loudly at build time"). Runtime error handling is a separate ARKlight proposal (`docs/Proposals/RUNTIME-ERROR-HANDLING-PROPOSAL.md`) |
| Reflection, dynamic proxies, agents, serialization | `Class`, `Method`, JVMTI, `ObjectOutputStream` | none |
| `invokedynamic` / method handles | lambdas, string concat, records and switch bootstraps | none. Lowering targets ARK AST, not bytecode |

Side effect worth noting: the JLS obligations attached to these mechanisms
(class-init checks on first access, volatile fences, implicit-check exceptions)
stop being obligations. Deleting them is a gain as well as a loss.

---

## 2. Lost with `java.base`  **[decided]**

`java.base` is not present at runtime, in the compiled output, or (by default)
in the build-time evaluator. javac hard-wires at least 74 named types
(`Symtab.java`: `Object`, `String`, `Class`, `Throwable`, `Enum`, `Record`,
`Iterable`, ...) plus the box types. None of them mean anything in Rei unless
Rei re-specifies them.

What goes away, by area:

- **`java.lang` core**: the `Object` contract (`equals`/`hashCode`/`toString`/
  `getClass`/`wait`/`notify`), `String`/`StringBuilder`, boxing, and the
  `Integer` cache identity guarantee (JLS 5.1.7).
- **Collections and streams**: `List`, `Map`, `Set`, `Deque`, `PriorityQueue`,
  `Collections`, `Arrays`, `stream`.
- **Concurrency**: `java.util.concurrent`, `ForkJoinPool`, atomics, `VarHandle`.
- **I/O and system**: `java.io`, `java.nio`, `java.net`, `java.time`,
  `java.text`, `java.util.regex`, `java.security`, logging, JPMS.
- **Introspection**: `java.lang.invoke`, `java.lang.reflect`.

What ARKlight offers instead is a **closed UI vocabulary** (`stdlib.ARKlight`,
pulled in with `# include <...>`). That is a component vocabulary, not a
general-purpose library. **Consequence:** Rei has no standard collection or
algorithm library unless we write one.

`java.base` stays in `reference/openjdk/` as a design and algorithm reference
only (see section 7 before copying anything from it).

---

## 3. Lost with javac's back half  **[decided]**

What we keep as reference: `javac/parser` (15 files, 12,597 lines),
`javac/tree` (11 files, 12,882 lines, including `JCTree.java`), and the public
`com.sun.source.tree` interfaces (78 files, 5,834 lines).

What we do not carry: `comp/` (`Attr`, `Flow`, `TransTypes`, `Lower`,
`LambdaToMethod`), `jvm/` (`Gen`, classfile writing), and `util/` (diagnostics).

Java's *semantics* live in those packages, not in the parser. Rei inherits
syntax ideas and **none of Java's static semantics**:

- overload resolution (JLS 15.12) and type inference (JLS 18)
- definite assignment and reachability (JLS 16)
- checked-exception analysis (JLS 11.2)
- wildcard generics and capture conversion
- erasure and bridge methods

Whatever Rei's type checker promises has to be written from scratch. Realistic
scope is far smaller than `Attr`, but nothing is inherited for free.

A second, structural loss: `JCTree` models classes, interfaces, generics,
annotations and modules. `ARKNode` is `(type, props, children)`. Java's central
abstraction, the class, has **no counterpart in ARK AST**. Classes can survive
only as compile-time constructs that are evaluated away, or not at all.

---

## 4. Lost from Python as the authoring host  **[decided]** (largest impact on ARKlight)

Today `arklight/parser/loader.py` executes the site module in a real Python
namespace. Its own docstring lists what that buys: "name resolution, imports,
loops, conditionals, helper functions/components: all ordinary Python."
`arklight/parser/discover.py` does static analysis over Python's AST. Rei
replaces both. Lost:

- **Free build-time computation.** Loops, conditionals, helper functions,
  comprehensions, f-strings, dict/list operations all came from CPython.
- **Python stdlib and PyPI at authorship time.** Reading JSON/CSV/Markdown
  and generating pages from data was one `import` away.
- **Decorators as registration.** `@site.page("/")` and `@component` need Rei
  syntax.
- **Python tooling.** Debugger, linters, type checkers, IDE support, `pytest`
  over authoring code.
- **Python as the specification.** "What does this expression mean" was
  answered by CPython. Rei has to answer it itself (section 6).

**Build-time evaluation is the central open decision [open].** Options:

1. **Rei interpreter hosted in ARKlight (Python).** Full power, but Rei needs
   a specified abstract machine and its own semantics.
2. **Restrict Rei to what lowers statically.** Declarative trees plus
   compile-time-expandable macros. Smallest, easiest to validate before
   evaluation, least expressive.
3. **Python escape hatch for computation.** Cheapest, but the pipeline then
   depends on Python anyway and Rei is a syntax skin.

---

## 5. Audience premise  **[decided]**, a product risk and not a technical one

`docs/Foundational/WHAT-ARKLIGHT-IS.md` names two audiences: the Python
community, defined as people "structurally unwilling, uninterested, or simply
not equipped to reach for `npm`, a JS build chain, or a second language", and
the Education community, for whom "every extra language ... is a teaching
cost". **Rei is a second language.** The only credible mitigation is
minimalism: small enough to learn in minutes, with the same posture ARKlight
already has (a student's mistake fails loudly at `arklight build`). The Rei
compiler narrator is a natural home for that voice. Note it is **accepted but
not yet shipped**: `PROGRESS.md` lists it as PLANNED for v0.065, and there is
no `--narrate` flag in the code today.

---

## 6. Semantics that stop being free

Java specified these precisely. The hosts Rei runs on (CPython in the
evaluator, JS in the output) specify them differently.

| Java semantics | Host default | Cost or decision |
|---|---|---|
| `int`/`long` wrap in two's complement (JLS 4.2.2) | Python ints are unbounded; JS numbers are doubles | mask and sign-extend per operation, e.g. `((x + 2**31) & 0xFFFFFFFF) - 2**31`; JS side needs `\|0`, `Math.imul`, `BigInt.asIntN(64, ...)`. Or Rei defines ints as unbounded/checked **[open]** |
| Integer `/` truncates toward zero; `%` takes the dividend's sign (JLS 15.17.2-3) | Python `//` floors, `%` takes the divisor's sign | `-7 / 2` is `-3` in Java, `-4` with `//`; `-7 % 2` is `-1` vs `1`. Must be implemented explicitly |
| `String` is UTF-16 code units | Python `len` counts code points, JS `.length` counts UTF-16 units | `"😀".length()` is 2 in Java, 1 in Python. Pick one and specify it, or evaluator and JS output disagree |
| `float` is binary32 | Python has only binary64; JS needs `Math.fround` | drop `float`, or emulate |
| Left-to-right evaluation order (JLS 15.7) | mostly the same in Python; C leaves it unspecified | specify it. See the note below |

**Note on the C-spec inspiration.** C stays small partly by leaving behavior
unspecified or undefined. ARKlight's doctrine is "fail loudly at build time",
which cannot tolerate undefined behavior in the front end. Rei should be
minimal **by omission** (fewer constructs), not **by unspecification** (more
things the spec shrugs at).

### Java features and their likely fate  **[proposed]**

| Java feature | Proposed fate | Why |
|---|---|---|
| classes, inheritance, interfaces, abstract | drop, or compile-time-only records for props | no ARK AST counterpart. Components are functions (`@component`) |
| generics (wildcards, inference) | drop | erased away anyway; the type-checker cost is the largest single item in section 3 |
| method overloading | drop | named props do the job; also matches ARKlight's "collisions are errors" rule |
| checked exceptions | drop | no runtime exceptions to check |
| `synchronized`, `volatile`, threads | drop | no shared-memory model (section 1) |
| inner, anonymous, local classes and capture rules | drop | no class model |
| static fields and static init | drop | class-init order is gone |
| annotations | keep syntax as decorator-like markers; no reflection | registration (`@page`, `@component`) still needs it |
| `new` for construction | drop | `Heading("x")` call shape is the target; Dart made it optional for the same reason |
| lambdas | restricted **[open]** | ARKlight's event handling is a closed vocabulary of structured refs (`Action.*`, `ActionRef`), not arbitrary closures |
| enums, records | keep as compile-time data **[open]** | useful for props and state shapes |
| `switch` / pattern matching | keep for compile-time evaluation **[open]** | tied to the evaluator decision |

---

## 7. Legal: what we cannot copy

Verified from file headers: `JavacParser.java` and `String.java` are both
**GPL version 2 only**, with the Classpath Exception. ARKlight is
**GPL-3.0-or-later** plus its own Additional Terms under GPLv3 section 7
(attribution on conveyed copies, among others).

GPLv2-only code and GPLv3 code cannot be combined into one work, and the
Classpath Exception covers linking independent code against the runtime
library. It is not a relicensing grant for copying source.

**Consequences:**

- Do not paste or transliterate javac parser/tree or `java.base` code into
  `ARKlight/` or into Rei's implementation.
- Write Rei's lexer and parser from a Rei grammar and spec of our own. Read
  javac for ideas (precedence handling, error recovery) without transcribing.
- Shipping both trees side by side in this zip is generally treated as mere
  aggregation, as long as each keeps its own license text. That is why
  `reference/openjdk/` carries `LICENSE`, `ASSEMBLY_EXCEPTION` and
  `ADDITIONAL_LICENSE_INFO`.

I am not a lawyer. If Rei will be distributed, have someone qualified confirm.

---

## 8. What survives

- **All of ARKlight below ARK AST**: component expansion, normalization,
  validation, Website IR, the binary IR (`.arklight`), and the HTML, CSS, JS,
  Android and Desktop backends, plus search/doc retrieval, PWA, packer/seal and
  SBOM.
- **The doctrine**: compiler first, runtime last; fail loudly at build time.
  A statically analyzable language fits that better than Python did, because
  Rei code can be validated before any evaluation.
- **A C-shaped seam already in the tree**: the preamble directives
  `# include <stdlib.ARKlight>` and `# define A -> B` are compiler-owned name
  binding, structurally the C preprocessor. That matches the C-spec direction
  and is worth promoting from comments to real syntax.
- **C-family syntax knowledge**: tokenizer structure, precedence climbing,
  error recovery. As ideas only (section 7).

---

## 9. Open decisions, ordered by blast radius

1. **Build-time evaluator** (section 4): interpreter, static-only, or Python
   escape hatch.
2. **Numeric and string semantics** (section 6): wrap vs unbounded ints;
   code points vs UTF-16.
3. **Fate of classes, generics, overloading** (section 6).
4. **Event handlers**: closures vs ARKlight's closed `Action` vocabulary.
5. **The seam**: Rei AST lowers to ARK AST at the point where
   `Site.build_ark_ast()` produces `ARKNode` trees today. Does Rei replace
   `load_site` or add a second loader alongside it?
6. **Diagnostics**: Rei-language errors, possibly in the voice of the planned
   Rei narrator (v0.065, not yet built). `arklight search` already exists as
   the tool that proposal points errors toward.
7. **Clean-room hygiene** (section 7): who reads javac, who writes the parser.
