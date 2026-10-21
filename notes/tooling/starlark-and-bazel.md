---
title: Starlark and Bazel
kind: concept
area: tooling
date: 2026-10-21
status: draft
diagram: diagrams/starlark-and-bazel.html
sources:
  - https://github.com/bazelbuild/starlark/blob/master/README.md
  - https://github.com/bazelbuild/starlark/blob/master/design.md
  - https://github.com/bazelbuild/starlark/blob/master/users.md
  - https://github.com/laurentlb/awesome-starlark
  - https://blog.bazel.build/2017/03/21/design-of-skylark.html
  - https://bazel.build/about/intro
  - https://bazel.build/about/why
  - https://bazel.build/about/faq
  - https://bazel.build/rules/language
  - https://bazel.build/extending/concepts
unverified:
  - "The action-graph box in the diagram says outputs are 'keyed by content hash'. The fetched Bazel pages say Bazel tracks changes to file content and build commands; they do not name the hashing scheme."
  - "The claim that Bazel's Starlark interpreter is written in Java comes from the 2017 blog post ('Bazel is written in Java') and the users.md pointer to net.starlark.java; the current docs pages fetched do not restate it."
tags: [starlark, bazel, build-systems, google, configuration-language, monorepo]
---

## Summary

Starlark (formerly Skylark) is a small configuration language with Python 3 syntax that Google designed to describe builds; its spec defines it as "a language intended for use as a configuration language". Bazel is the open-source flavour of Google's internal build tool Blaze, and every `BUILD` file and every `.bzl` extension in Bazel is Starlark. Beyond Bazel, Meta's Buck2, Copybara, Tilt, Drone, Delve and Chromium's lucicfg embed Starlark whenever they need user-written config that can never do I/O, hang, or vary between runs.

## The problem it solves

Google's build history, as told by the Bazel team's 2017 design post:

- **Makefiles** first. They "don't scale well with a large code base".
- **Python scripts generating Makefiles** next: `BUILD` files called Python functions. Too slow, and Make itself was the bottleneck.
- **Blaze (2006)** parsed only function calls, list comprehensions and assignments itself. Anything fancier went through a preprocessing step that ran a real Python interpreter to emit a simplified `BUILD` file. That step caused "maintenance, performance, and safety problems", and because Blaze could not parse the files itself, tooling was hard.
- **Today**: the Python preprocessing step is gone, but the syntax stayed. "We kept the Python syntax, though, in order to migrate our codebase."

The general problem: if build files are a real scripting language, the build tool cannot prove two runs are equivalent (so it cannot cache), cannot evaluate thousands of files in parallel safely, and cannot parse or rewrite a file without executing it. Google's requirements, per the post: `BUILD` files must be "simple and declarative, so that we can build tools to manipulate them"; "we have to guarantee that the execution of the code will always yield the same results" (so no I/O, no date/time, deterministic dict order); and evaluation must be thread-safe because many files load in parallel. Bazel's FAQ adds the business motivation: generated Makefiles "led to slow and unreliable builds, which began to interfere with our developers' productivity and the company's agility."

## How it works

**Starlark's design principles** (from the spec README, verbatim):

- Deterministic evaluation: "Executing the same code twice will give the same results."
- Hermetic execution: "Execution cannot access the file system, network, system clock. It is safe to execute untrusted code."
- Parallel evaluation: "Modules can be loaded in parallel. To guarantee a thread-safe execution, shared data becomes immutable."
- Simplicity: "We try to limit the number of concepts needed to understand the code."
- Focus on tooling: "We recognize that the source code will be read, analyzed, modified, by both humans and tools."
- Python-like: "Python is a widely used language", so the learning curve is small.

**What was cut from Python, and why** (Bazel's language page plus the design doc):

| Removed | Rationale given |
|---|---|
| `while`, recursion | Unbounded loops "would encourage users to write more complex code"; every program terminates. |
| `class`, `import`, `yield`, generators, `global`/`nonlocal`, most builtins | Fewer concepts; `load()` replaces `import`. |
| `try`/`raise`/`except` | All errors are fatal. Omitting exceptions "makes the language simpler", and because nothing can catch an error, later versions can relax behaviour without breaking code. |
| `is`, chained comparisons, implicit string concatenation, iterable strings, bools as ints, duplicate dict keys | Each was "a common source of bugs in user code" or added an unneeded concept. |
| Top-level `for`/`if`, reassigning globals | Non-trivial code belongs in functions; single assignment helps readers and IDEs. |
| 64-bit ints | Integers are 32-bit signed and overflow raises an error (Bazel's dialect). |

Kept: dynamic typing, because `**kwargs` with heterogeneous dicts made static types impossible to migrate to. The design doc's framing: "The goal for Starlark is to provide a configuration language. CPU-intensive computations are better written in another language."

**Freezing and parallel loading.** Loading forms a dependency graph: leaf files load first and in parallel, each file loads at most once, and its result is cached and frozen read-only so threads can share it. The extension docs state the contract: loading a `.bzl` file "has no visible side effects"; it only defines values and functions.

**How Bazel uses it.** Two kinds of file on one interpreter: `.bzl` files define reusable functions (macros and rules), `BUILD` files call them, and each rule call adds a target to a graph. Three phases:

1. **Loading**: Bazel evaluates the `.bzl` and `BUILD` files the build needs. Macros, which are plain functions that call rules, expand here. Bazel 8 added symbolic macros next to legacy ones.
2. **Analysis**: each rule's `implementation` function runs, declares outputs, and instantiates actions. The target graph becomes an action graph. Rules, unlike macros, can access Bazel internals and pass information to other rules.
3. **Execution**: an action (inputs, a command such as a compiler invocation, declared outputs) runs when one of its outputs is needed. Tests run here. An action that fails to produce an output fails the build.

Starlark only runs in the first two phases. It describes the graph; Bazel's core does the work.

**What the hermeticity buys Bazel.** Because every action's inputs are declared and actions run in sandboxes, Bazel "tracks changes to both file content and build commands", reruns only what changed, and caches "in memory, on disk, and on a remote build farm"; Google says it routinely sees cache hit rates "north of 99%". The same discipline gives the correctness claim: "the same source code always results in the same binary." Bazel claims to stay agile at "100k+ source files" and to run "thousands of build or test commands at the same time" when the graph allows.

**Starlark outside Bazel.** The spec README lists implementations in Go (google/starlark-go), Java (in Bazel's tree) and Rust (facebookexperimental/starlark-rust), noting none is "fully compliant to the specification yet". awesome-starlark's user list includes Buck2 (Meta's build system), Copybara (moving code between repos), Tilt (Kubernetes dev loops), Drone and Vela (CI config as an alternative to YAML), lucicfg (Chromium CI config), Delve (debugger scripting), Isopod and ytt (Kubernetes/YAML generation), qri, Kurtosis, Pixlet and others. The common thread: each tool wants users to write logic, but needs that logic to be sandboxed, terminating and reproducible.

## When to use it, when to skip it

| Use it when | Skip it when |
|---|---|
| Bazel's FAQ list: a large codebase, multiple compiled languages, deployment on multiple platforms, extensive tests. | A small single-language project where cargo, go, or npm already gives reproducible builds; the FAQ's own strengths list implies this. |
| You are embedding a config language in your own tool and need it safe for untrusted input, deterministic, and parseable by tools. | The config needs network, clock, or heavy computation. The design doc says CPU-heavy work belongs in another language. |
| You want many files evaluated in parallel with results cached across builds. | Build operations "whose outputs should not be cached", or "a few long, sequential steps" that cannot be split into smaller targets (both from the FAQ). |

## Diagram

[diagrams/starlark-and-bazel.html](../../diagrams/starlark-and-bazel.html): shows the Python-vs-Starlark contrast, the loading, analysis and execution phases, how `.bzl` and `BUILD` files feed the analysis phase and produce an action graph, and the use/skip table. It does not show rule-writing syntax (`rule()`, `ctx.actions`, providers), the Bzlmod module system, remote execution protocols, or how Buck2's use of Starlark differs from Bazel's.

## Sources

- https://github.com/bazelbuild/starlark/blob/master/README.md: backs the "configuration language" definition, the six design principles verbatim, and the Skylark rename.
- https://github.com/bazelbuild/starlark/blob/master/design.md: backs "designed at Google to replace Python as the build description language", the "CPU-intensive computations" quote, determinism as a requirement, freezing for thread safety, and the rationale for dropping `while`, exceptions, `is`, chained comparisons and keeping dynamic typing.
- https://github.com/bazelbuild/starlark/blob/master/users.md: backs the Go, Java and Rust implementations and the "not fully compliant" note.
- https://github.com/laurentlb/awesome-starlark: backs the list of non-Bazel Starlark users (Buck2, Copybara, Tilt, Drone, Vela, lucicfg, Delve, Isopod, ytt, qri, Kurtosis, Pixlet).
- https://blog.bazel.build/2017/03/21/design-of-skylark.html: backs the Makefile to Python to Blaze (2006) history, the preprocessing problems, "We kept the Python syntax", the "simple and declarative" and "always yield the same results" requirements, and parallel loading with frozen modules.
- https://bazel.build/about/intro: backs "similar to Make, Maven, and Gradle", caching of file content and build commands, sandboxing for reproducibility, and the 100k+ source files claim.
- https://bazel.build/about/why: backs thousands of parallel commands, in-memory/disk/remote caching, the 99% cache hit rate, sandboxed actions, "same source code always results in the same binary", and extensibility via Starlark.
- https://bazel.build/about/faq: backs Bazel as a flavour of Blaze sharing most of its code, the generated-Makefile motivation, the four "shines at" traits, and the two poor-fit cases.
- https://bazel.build/rules/language: backs the full list of Python features Starlark removes, 32-bit ints, immutable globals, no top-level `for`/`if`, deterministic dict order, and no recursion.
- https://bazel.build/extending/concepts: backs BUILD vs .bzl roles, the three phases, macros vs rules vs actions, symbolic macros in Bazel 8, "no visible side effects", and load-once caching.
