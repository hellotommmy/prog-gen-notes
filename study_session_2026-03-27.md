# Study Session Notes — 2026-03-27

## Phase 1.1 — Property-based testing vs unit testing
- `property-based testing (PBT)` states a *property* or invariant, then uses a `generator` to produce many inputs automatically.
- `unit testing` checks a small set of handpicked examples; coverage depends on the author’s imagination.
- PBT is especially good for corner cases, algebraic laws, round-trip properties, idempotence, and metamorphic relations.
- When a failing input is found, PBT frameworks usually `shrink` it to a minimal counterexample, which makes debugging much easier.

## Theory takeaway
- For generators, practical usefulness depends on exploration quality of the input space, not only on formal correctness of the generated language.
- A generator can be correct but still weak if most probability mass lands on boring or redundant cases.
- This is exactly where distribution and throughput matter later in the RustSmith / complexity discussion.

## Phase 1.2 — What is a generator?
- A `generator` turns a size parameter, seed, or context into concrete test inputs.
- In tools like `QuickCheck` and `Hypothesis`, generators are usually *compositional*: small generators are combined with `map`, `flatMap` / `bind`, `oneOf`, `frequency`, `listOf`, and recursion.
- For structured data, a generator often carries state such as a symbol table, scope stack, or borrow/lifetime context so that it can build valid objects by construction.
- The main theoretical lesson is that generator quality is not just about `support`; it is also about the probability mass assigned to useful regions of the input space.

## Phase 1.3 — Differential testing / compiler fuzzing
- `differential testing` compares the behavior of multiple implementations on the same input; disagreement is evidence of a bug in at least one implementation.
- `compiler fuzzing` is a special case where the generator produces programs and the oracle is consistency across compilers, optimization levels, or execution modes.
- `Csmith` is the classic example for C: generate many random, side-effect-rich but well-defined programs, compile them, and compare outputs.
- `RustSmith` does the same style of testing for Rust, but must respect borrowing, lifetimes, and undefined-behavior avoidance much more carefully.
- The theory point is that the generator must be judged by how well it feeds the *oracle*: valid, diverse, and bug-revealing inputs matter more than raw randomness.

## Phase 1.4 — What makes a generator good in practice?
- A good generator is one that converts time into *useful* test cases.
- Main practical metrics: `validity rate`, `throughput`, `coverage`, and `bug-finding rate`.
- A generator with perfect theoretical support can still be poor if it spends most of its time rejecting candidates or repeats the same boring shapes.
- For fuzzing, the best generator is often the one that reaches new states or new code paths per second, not the one that is most elegant mathematically.
- Distribution matters because coverage depends on where probability mass lands, not just on which inputs are possible.

## Phase 1.5 — Key engineering challenges
- `type conformance`: the generator must produce inputs that satisfy the language's typing rules.
- `semantic validity`: the generator must avoid undefined behavior or other invalid executions.
- `distribution control`: the generator must steer probability mass toward useful structures, not just any legal output.
- `stateful constraints`: many structured inputs depend on scope, borrow state, symbol tables, or recursion depth, so generation is inherently context-sensitive.
- `constructive generation` is usually preferable to `rejection sampling` because rejection can make valid outputs exponentially expensive.
- This is exactly why RustSmith and similar tools deliberately under-approximate the target language: exactness can be too costly.

# Phase 2 — RustSmith Deep Dive

## Phase 2.1 — Problem statement and motivation
- RustSmith is the first random differential compiler-testing tool for Rust.
- Its goal is to generate Rust programs that are both well-typed and well-defined, so that differential testing against `rustc` and `mrustc` is meaningful.
- Rust is harder than C in this setting because the generator must respect ownership, borrowing, and lifetimes, not just syntax.
- The paper's practical motivation is clear: compilers still contain optimization bugs and miscompilation bugs, and random program generation can find them.

## Phase 2.2 — Design decisions and complexity implications
- RustSmith builds an abstract syntax tree recursively and uses a symbol table to track variables, types, borrows, and lifetimes.
- It uses wrappers to avoid dangerous operations such as division by zero, overflow, and out-of-bounds access.
- It controls output size and block depth with generation parameters; the default output is about 3,000 lines.
- The complexity lesson is that the generator is not merely enumerating syntax; it is maintaining contextual invariants while producing large programs quickly.
- That makes RustSmith a practical example of a generator whose cost is driven by constraint maintenance, not just tree size.

## Fuzzer taxonomy
- `Mutation-based greybox fuzzing` (AFL, libFuzzer) starts from existing inputs and mutates them, using coverage feedback to keep promising cases.
- `Grammar-based fuzzing` generates structured inputs from grammars or abstract syntax, which is better when validity is hard to preserve by mutation.
- `Differential compiler fuzzing` (Csmith, RustSmith) generates semantically valid programs and checks whether multiple compilers or optimization levels agree.
- `Whitebox / concolic fuzzing` uses symbolic execution or constraint solving to steer input generation toward hard-to-reach paths.
- RustSmith sits in the structured-generator + differential-testing corner: it is not primarily mutation-driven, and it is not trying to infer structure from crashes.

## RustSmith in the big picture
- RustSmith is a generator-first tool: it constructs valid programs by design, then uses compilation and execution as the oracle.
- Compared with mutation fuzzers, it pays more upfront cost to satisfy language constraints but gains much higher validity for complex structured inputs.
- Compared with grammar fuzzers, RustSmith must handle semantic constraints that go beyond syntax, especially ownership, borrowing, and lifetimes.
- In the theory paper, RustSmith is the kind of real system that motivates a distinction between support-based generability and distribution-aware visibility.
