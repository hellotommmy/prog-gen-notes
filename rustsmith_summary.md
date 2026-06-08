# RustSmith Paper Summary (ISSTA 2023 Tool Demo)

**Title**: RustSmith: Random Differential Compiler Testing for Rust
**Authors**: Mayank Sharma, Pingshi Yu, Alastair F. Donaldson (Imperial College London)

## What It Does
RustSmith is the first random Rust program generator for end-to-end differential testing of Rust compilers. It generates programs conforming to Rust's advanced type system (borrowing, lifetimes) that are guaranteed to yield well-defined results.

## Key Design Decisions (Relevant to Complexity of Generators)

### 1. Context-Aware AST Construction
- Recursively builds a random AST conforming to Rust grammar
- Maintains a **symbol table** tracking: variable names/types, borrowing states, lifetime scopes
- Generation is context-aware: `break` only inside loops, `if` conditions restricted to `bool`

### 2. Avoiding Undefined Behavior
- Uses **wrappers** to guard dangerous operations (div-by-zero, overflow, out-of-bounds)
- This ensures programs have well-defined semantics — crucial for differential testing
- **Complexity implication**: The generator must produce only programs in the language of "well-defined Rust programs", which is a strict subset of "syntactically valid Rust programs"

### 3. Borrowing Rules (Over-Approximation)
- RustSmith enforces **stricter** borrowing rules than Rust actually requires
- Real Rust allows violations when rustc can prove references aren't live
- Mirroring exact rules would require data-flow analysis during generation
- **Complexity implication**: The generator produces L ⊂ L_target (under-generates)
- This is a **practical tradeoff**: exact generation is harder (possibly intractable for partial programs)

### 4. Lifetime Handling (Conservative)
- Structs created "on demand" with fixed lifetime annotations
- Does not exercise lifetime coercion mechanism
- **Complexity implication**: Another deliberate under-approximation for feasibility

### 5. Output Size Control
- Default mode produces ~3000 line programs
- Parameters control block sizes and recursive depths
- Command-line arguments as opaque values to inhibit compiler optimizations

## Bugs Found
- 5 historic rustc bugs (independently rediscovered)
- 2 new mrustc bugs
- Achieved complementary mutation coverage vs. official test suite

## Coverage Results
- 51.59% line / 54.06% function coverage of MIR optimizer (vs. 82.12% / 72.78% for official suite)
- Outperformed official suite on specific optimizations (add_call_guards.rs: 100% vs 55.56%)
- 3 mutants killed only by RustSmith (not by official test suite)

## PBT/Fuzzing Insights for Our Paper

### Practical Complexity Barriers in Real Generators
1. **Type system conformance**: Generating well-typed programs is at least as hard as type-checking (in P for Rust without generics/traits)
2. **Semantic validity**: Generating programs with well-defined semantics is harder — involves reasoning about runtime behavior
3. **Borrow checker conformance**: Exact generation requires data-flow analysis = higher complexity
4. **Practical generators deliberately under-approximate**: RustSmith generates L ⊂ L_target

### Distribution Issues Visible in Practice
1. RustSmith uses tuned parameters for output size → distribution is heavily engineered
2. Some optimizations get 100% coverage while others get 0% — distribution matters enormously
3. Mutation coverage experiment shows 3-minute timeout per mutant — throughput matters
4. Rejection sampling not used (too expensive) — generator must be **constructive**

### What "Efficient Generation" Means in Practice
1. ~3000 lines per program, generated in milliseconds
2. Compilation + execution is the bottleneck, not generation
3. Generator complexity should be measured by: time per valid program, coverage per time budget
4. A generator that is O(n^2) in output size is fine; one that rejects 99.99% of attempts is not
