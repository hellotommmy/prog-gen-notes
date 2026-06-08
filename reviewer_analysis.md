# LICS Review Analysis & Improvement Directions

## Reviewer A (Most Thorough)
**Verdict**: Reject, Knowledgeable
**Tone**: Sympathetic but unconvinced on fit

### Key Concerns
1. **Distribution-agnostic = loses practicality**
   - "An input generated with exponentially small probability never appears in practice"
   - This is the most fundamental criticism
   - **Action needed**: Define distribution-aware complexity classes

2. **Type 1 vs Type 2 errors not distinguished**
   - Type 1: failing to generate some string in L (under-generation)
   - Type 2: generating a string not in L (over-generation)
   - RustSmith deliberately accepts Type 1 errors (conservative borrowing)
   - **Action needed**: Formalize this tradeoff

3. **Definitional choices not sufficiently motivated**
   - Why these definitions over alternatives?
   - Need discussion of design space
   - **Action needed**: Add section on alternative formalizations and why this one

4. **Standard methods, limited logic content**
   - "Not a natural fit for LICS"
   - **Action needed**: Either add more logic content or target a different venue (POPL? ICFP? PLDI?)

## Reviewer B (Most Critical of Definitions)
**Verdict**: Reject, Knowledgeable

### Key Concerns
1. **Time measured by output size is "weird"**
   - "A generator that outputs 2^n doing (2^n)^2 steps — is this polynomial?"
   - This is a legitimate concern about the definition
   - **Action needed**: Better justify or consider input-parameterized complexity

2. **RE = GEN is "trivial"**
   - "You guess an input that halts and produces output; not surprising"
   - **Action needed**: Either downplay this or frame it as a warm-up

3. **SPG definition may differ from original**
   - **Action needed**: Clarify relationship to Sanchis's definitions explicitly

4. **Undecidability of checking if something is a generator**
   - This is a valid meta-concern
   - **Action needed**: Address explicitly

## Reviewer C (Dismissive)
**Verdict**: Reject, Knowledgeable

### Key Concern
1. **"This is just transducers"**
   - Model = TM transducer, generated language = image of transducer
   - Proofs resemble standard transducer ↔ TM equivalence
   - **Action needed**: Acknowledge connection, argue novelty is in the complexity hierarchy + PBT application

## Improvement Strategy

### High Priority
1. **Distribution-aware classes**: Define GEN_δ where δ controls minimum probability
2. **Practical complexity metrics**: Coverage rate, rejection rate, throughput
3. **Type 1/Type 2 error formalization**: Under-generation vs over-generation classes
4. **Better venue targeting**: Consider POPL, PLDI, or TOPLAS if logic content insufficient for LICS

### Medium Priority
5. **Concrete examples from real tools**: RustSmith, Csmith, CSmith → map to our classes
6. **Input-parameterized complexity**: Address reviewer B's concern about output-based measurement
7. **Stronger separation results**: Go beyond hash collision assumption

### Research Questions to Explore
- What is the complexity of generating well-typed Rust programs? (without traits/generics: probably P-generable)
- What is the complexity of generating semantically valid C programs? (undecidable in general, but restricted subsets?)
- Can we formalize the "coverage rate" of a generator as a complexity measure?
- Is there a natural distribution-aware analog of SPG?
- What is the relationship between generator complexity and the "hardness" of the SUT's input language?

## Papers to Read
- Sanchis 1990: "Efficient random generation" (the predecessor)
- Arenas et al. 2019: "Efficient enumeration" (used in SLG proof)
- Levin 1986: Average-case complexity (P-samplable distributions)
- Goldreich 2011: "Computational complexity: A conceptual perspective" (samplable distributions)
- Jerrum et al. 1986: "Random generation of combinatorial structures" (uniform generation)
- Impagliazzo 1995: "A personal view of average-case complexity"
- Claessen & Hughes 2000: QuickCheck (original PBT)
- Lampropoulos et al. 2017: "Generating good generators" (narrowing)
- Paraskevopoulou et al. 2015: "Foundational PBT" (Coq proofs)
