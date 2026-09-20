# Rei Programming: Course Syllabus (Draft)

_Current as of 2026-09-20. Drafted against `WHAT-REI-IS.md`. Rei has no
compiler yet, so every unit is tagged with the language layer it depends on
and the decision that blocks it. Adopting institutions should treat this as a
template to fill in, not an approved curriculum._

## Course particulars

| | |
| --- | --- |
| Course code | REI-101 (placeholder) |
| Semester | to be assigned by the adopting institution |
| Category | to be assigned (core or elective) |
| L / T / P / C | 3 / 0 / 0 / 3 (45 periods, mirroring a one-semester theory course) |
| Course title | Rei Programming |

Suggested lab exercises are listed per unit. They are not credit-bearing here;
an institution that wants a lab component can attach a practical slot.

## Prerequisites

- Problem solving and Python programming.
- Basic familiarity with HTML and CSS (what a page is made of).
- Programming in C is helpful, not required.
- No prior OOP course is assumed. Object-oriented design is taught in
  Unit III.

## Course objectives

1. To understand how a Rei program becomes a website, and to write basic
   Rei programs.
2. To design useful programs with procedural decomposition, and to judge where
   object-oriented structure earns its place.
3. To apply inheritance, interfaces, polymorphism and GoF design patterns.
4. To apply error-handling mechanisms and guardrails.
5. To build programs that read project files at build time and generate pages
   from data.

## Course outcomes

At the end of the course students will be able to:

| CO | Outcome | Bloom's level |
| --- | --- | --- |
| CO1 | Apply Rei's types, expressions, control flow and functions to solve simple problems. | C3 |
| CO2 | Develop programs that combine procedural decomposition with object-oriented structure to generate a site. | C6 |
| CO3 | Apply inheritance, interfaces and polymorphism, and identify GoF patterns in a given design. | C3 |
| CO4 | Illustrate error-handling mechanisms and predict a program's build-time behavior. | C3 |
| CO5 | Construct programs that read and validate data files and generate pages from them. | C6 |

CO to program-outcome (PO/PSO) mapping is left to the adopting institution.

## Syllabus (total contact hours = 45 periods, credits = 3)

### Unit I: Introduction and fundamentals (10)

_Layer L1. Blocked on nothing; teachable against the tree subset._

The ARKlight pipeline: Rei source, Rei AST, ARK AST, after which ARKlight
handles the rest (HTML/CSS/JS for the web). Where REIlight extends ARKlight
through ARKVM to native apps, and why one language for every target
(unification) matters. Rei and Python side by side. The C99 philosophy: trust the programmer, keep the
language small, one way to do an operation, fail loudly at build time.
Translation phases: from source text to tokens to tree. Lexical elements:
tokens, identifiers, keywords, literals, `//` comments. Types: fixed-width
integers, booleans, strings (UTF-8). Variables and `const`. Operators,
precedence, evaluation order, integer division truncating toward zero.
Selection and iteration. **Input and output at build time**: the tree is the
output, diagnostics are the console, configuration is the input. Trees of
constructor calls: named props, positional children, trailing commas.

Lab: `hello_site` in Rei; tokenize a snippet by hand; predict the ARK AST for a
given tree; compare with the Python version.

### Unit II: Building something useful, procedural core with object-oriented structure (9)

_Layer L2. Blocked on the build-time evaluator decision._

Functions: parameters, named arguments, defaults, scope. Records as plain
data. Modules and `#include`; `#define` as compiler-owned name binding.
Components as functions. Collection `if` / `for`, spread, string
interpolation. Composition first. **When a record becomes a class**:
state plus behavior, constructors, `this`, encapsulation, access control.
Uniform call syntax: `f(x)` and `x.f()` as one operation. Decomposing a
problem: a blog index page from a list of posts.

Lab: a data-shaped page built with functions only; refactor one piece into a
class and compare the two versions' output (they must be identical).

### Unit III: Object-oriented design, inheritance, polymorphism and patterns (9)

_Layer L3. Blocked on the evaluator decision and the class-fate decision._

Inheritance and its types; abstract classes; interfaces; overriding, and why
Rei has no overloading. Sealing and finality. Polymorphic dispatch and its
cost model: static resolution versus a dynamic class-chain lookup, and why it
costs build time only, today. GoF patterns in the ARK domain: Composite (the ARK
tree), Command (`Action` values as data), Observer (`State` / `Watch`),
Visitor (compiler passes). Patterns that a language feature absorbs: Builder
becomes named props, Strategy becomes a first-class function. Patterns as
constraints: what the compiler checks. Comparison with Java, Dart and Kotlin.

Lab: model a component library with an interface and two implementations;
identify the pattern in each of three given designs; write a Visitor that
counts node types.

### Unit IV: Error handling and guardrails (9)

_Layer L4. Blocked on the source-span table._

Failure kinds: syntax errors, semantic errors, faults during evaluation.
Build diagnostics: `file:line:col` and codes; reading a diagnostic, in Rei's voice (how it explains a failure and points to `arklight search`). The C99
behavior taxonomy (undefined, unspecified, implementation-defined,
locale-specific) and why Rei has no undefined behavior. Result values and
must-use rules. Typed faults, ordered handler chains (Chain of
Responsibility), cleanup with `finally`. Defining your own faults. Java's
exception model compared with Rei's. Guardrails: validation at the ARK AST
boundary, the closed action vocabulary, undeclared state names as build
errors. Finding the right vocabulary with `arklight search`.

Lab: predict and fix a set of broken programs; design a `Fault` hierarchy for a
data loader.

### Unit V: File handling at build time (8)

_Layer L5. Blocked on L2 and the write-policy decision._

The build-time I/O model: tracked inputs, project-relative paths, no
parent-directory search, no network. Text and bytes; UTF-8. Reading JSON, CSV
and Markdown into records. Validating data. Faults for missing and malformed
files. Generating routes and pages from data. The `assets/` folder. The
data-only project config (`arklight.config.rei`) and why it executes nothing.
Reproducible and incremental builds. Writing files: policy and its limits.

Lab / mini-project: a data-driven site (a portfolio, a course page or a
gallery) built from a data file, with complete error handling.

**Total: 45 periods**

## Course-outcome coverage

| Unit | Periods | COs |
| --- | --- | --- |
| I | 10 | CO1 |
| II | 9 | CO1, CO2 |
| III | 9 | CO2, CO3 |
| IV | 9 | CO4 |
| V | 8 | CO2, CO5 |

## Relationship to a Java course

This course is shaped like a standard one-semester Java course, with the
same five-part arc: fundamentals, useful programs, OOP, errors, files. What
does not carry over, and why:

| Java course topic | In Rei |
| --- | --- |
| Packages, CLASSPATH | `#include` and modules, resolved at build time |
| Multithreading, synchronization | not taught. There is no shared-memory model; the reactive core (`State`, `Watch`, `Bind`) takes its place (`LOST.md`, section 1) |
| Networking, sockets, JDBC | not taught. Rei has no general-purpose library, and its programs run at build time |
| Checked exceptions | not taught. Rei drops them, as Dart and Kotlin do |

## Learning resources

**Course texts (to be written).** The Rei language specification, organized
the way ISO C99 is; `WHAT-REI-IS.md`; the Rei language proposal.

**References**

1. ISO/IEC 9899:1999, *Programming languages: C*. The public working draft
   N1256 is freely available.
2. *Rationale for International Standard, Programming Languages: C* (C99
   Rationale).
3. Brian W. Kernighan, Dennis M. Ritchie, *The C Programming Language*, 2nd
   ed., Prentice Hall, 1988.
4. Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides, *Design
   Patterns: Elements of Reusable Object-Oriented Software*, Addison-Wesley,
   1994.
5. Cay S. Horstmann, *Core Java Volume I: Fundamentals*, Pearson (for the
   Java comparison in Units III and IV).
6. Robert Nystrom, *Crafting Interpreters* (for lexing, parsing and tree
   evaluation).
7. The Dart language tour: <https://dart.dev/language>
8. The Kotlin documentation: <https://kotlinlang.org/docs/home.html>
9. The ARKlight repository and its `docs/` folder:
   <https://github.com/ARKlight-Ecosystem/ARKlight>

## Note to adopters

- Rei is a second language for students who may already know Python. The only
  credible mitigation is minimalism: small enough to learn in minutes, and a
  student's mistake fails loudly at build time rather than at run time.
- Unit I can be taught as soon as the tree subset (L1) exists. Units II to V
  cannot be taught as written until the decisions named under each unit are
  made. Until then, the units are a plan, and their contents may change.
