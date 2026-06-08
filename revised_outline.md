# Revised Paper Outline

**Working title**: *Complexity of Structured Data Generators: A Distribution-Aware Framework*

**Target venues** (in order): POPL 2027, PLDI 2027, TOPLAS (journal), LICS 2027 (if logic content strengthened)

---

## 1. Introduction (3 pages)

### 1.1 Motivation: Why Generator Complexity Matters
- PBT is mainstream (QuickCheck, Hypothesis, Rust proptest); generators are the bottleneck
- Practical question: "Can I efficiently generate valid inputs for my SUT?"
- No existing complexity theory answers this

### 1.2 Our Contributions (clearly enumerated)
1. **Formal generator model** as randomized TMs with halting + validity
2. **Unrestricted generability**: GEN = RE (warm-up, *not* the main result — address Reviewer B)
3. **Efficient generation hierarchy**: PG, SPG, EPG, SLG and their relationships to NP
4. **NEW: Distribution-aware classes** (SPG\_vis) that capture "practically generable"
5. **NEW: Structural results** — separation theorems, closure properties, strict hierarchies in budget and visibility
6. **NEW: Coverage-time theory** — occupancy profiles, discovery curves, concentration bounds
7. **Verifier-generator duality** extended with certificate recovery schemes
8. **Real-world mapping**: QuickCheck, Csmith, RustSmith, AFL placed in the framework

### 1.3 Relationship to Prior Work (brief)
- Sanchis 1990: precursor to SPG; we extend with distribution awareness + PBT context
- Transducer theory: generators are transducers, but our contribution is the *complexity hierarchy* specific to generation (address Reviewer C)
- Levin's average-case complexity: we study full-support P-samplable distributions rather than hardness under samplable distributions

---

## 2. Generator Model (2 pages) [mostly from original]

### 2.1 Generators as Randomized TMs
- Definition: 3-tape TM (read-once input, working tape, write-once output)
- Halting condition; rejection on end-of-input
- Generated language: L(G) = {x : ∃b. G(b) = x}

### 2.2 Valid Generators
- Distribution on bitstreams; acceptance probability
- Valid iff μ(A_G^ω) = 1
- Example: natural number encoding (valid); yy-prefix (invalid)
- Rejection sampling is always valid (when predicate has a witness)

### 2.3 Design Space Discussion (NEW — addresses Reviewer A/B)
- Why output-based complexity? The generator has no "input" — output size is the natural parameter
- Sized partitions resolve the "2^n output in 2^{2n} steps" concern (Reviewer B)
- Why halting + full support? Matches PBT library semantics (Hypothesis, QuickCheck)
- Explicit comparison with transducer model (Reviewer C): same machines, different questions

---

## 3. Unrestricted Generability (1 page) [streamlined from original]

- **Theorem**: GEN = RE
- Proof sketch only (dovetailing construction)
- Corollary: non-RE languages are not generable (LOOP, complement of HALT)
- **Framing**: this is a warm-up establishing the setting, not a deep result (Reviewer B)
- Practical implication: compiler testing cannot cover all non-terminating programs

---

## 4. Efficient Generation Hierarchy (3 pages) [from original + cleanup]

### 4.1 Polynomial-Time Generators (PG)
- Definition; PG ⊆ NP via NDTM simulation
- CNF-SAT ∈ PG

### 4.2 Size-Dependent Generators (SPG)
- Sized partitions; sized generators; SPG ⊆ PG ⊆ NP
- CNF-SAT ∈ SPG; relationship to Sanchis's definition (explicit — Reviewer B)
- Conditions for SPG: basis extension, certificate sampling
- **Generator-verifier duality** (Proposition: SPG ↔ certificate recovery schemes)

### 4.3 SPG vs NP
- COLLISION_h ∈ P but COLLISION_h ∉ SPG (under CRHF assumption)
- Corollary: SPG ⊊ NP; P ⊄ SPG
- SPG^NP ⊇ NP (with oracle)

### 4.4 Expected Polynomial (EPG) and Logspace (SLG)
- EPG: PRIMES ∈ EPG; boosting; COLLISION ∉ EPG
- SLG: REACH ∈ SLG; SLG ⊆ SPG ∩ EPG

### 4.5 Summary Diagram
```
SLG ⊆ SPG ⊆ PG ⊆ NP
SLG ⊆ EPG ⊆ NP
SPG ⊊ NP (under CRHF)
EPG ⊊ NP (under CRHF)
```

---

## 5. Distribution-Aware Generation (NEW — 4 pages, core new contribution)

*This entire section addresses Reviewer A's primary concern.*

### 5.1 The Visibility Problem
- A generator can be support-correct but practically useless
- Example: uniform generator on 2^n elements gives each probability 2^{-n}
- Under polynomial sampling budget, most outputs are never observed
- Bridge from SPG (support) to practice (distribution)

### 5.2 Heavy Outputs and Visible Mass
- **Def**: Heavy_T(G), Vis_T(G)
- **Def**: Visible support size VSupp_T(G), visible min-entropy, effective support
- Monotonicity in budget; Hill-number connection
- **Prop**: Visible mass controls observability (coupon-collector style)

### 5.3 Visibility-Restricted Classes
- **Def**: SPG_vis^{T,ρ}
- **Prop**: Continuous hierarchy — decreasing in ρ
- **Prop**: SPG_vis ⊊ SPG (separation via exponentially large languages)
- **Prop**: Strict hierarchy in ρ (at fixed T)
- **Prop**: Strict hierarchy in T (at fixed ρ)
- **Prop**: Non-closure under union at fixed budget
- **Prop**: Closure under intersection with decidable languages (conditional)
- **Prop**: SPG_vis ⊆ EPG (when ρ ≥ 1/poly)

### 5.4 Budget Hardness
- **Prop**: Greedy allocation optimality
- **Corollary**: For |L_n| = 2^Ω(n) and T = poly(n), Vis_T → 0; must under-generate for non-trivial visibility
- This formalizes the reviewer's intuition: "full support + efficiency → bad distribution"

### 5.5 Coverage-Time Theory
- Visible occupancy profile Occ_{m,T}(G)
- Discovery curve Disc_{m,T}(G) = marginal new distinct outputs
- **Prop**: Coverage-time phase transition (linear → saturation at m* ~ (σ/ρ)ln σ)
- **Prop**: McDiarmid concentration for observed occupancy
- **Prop**: Self-bounding variance (Poisson-scale fluctuations)
- Visible coverage time and half-life

### 5.6 Type 1 / Type 2 Error Classes
- **Def**: SPG_{α,β} parameterized by under/over-generation rates
- Conjecture: strict separation via error relaxation (well-typed Rust as candidate)
- Practical mapping: RustSmith = (high α, β=0); AFL = (N/A, high β)

---

## 6. Real-World Generator Taxonomy (NEW — 1.5 pages)

### 6.1 Tool Classification Table
| Tool | Class | Type 1 | Type 2 | Distribution | Vis_T |
|------|-------|--------|--------|-------------|-------|
| QuickCheck | SPG | low | low | engineered | med-high |
| Csmith | SPG | low | zero | hand-tuned | medium |
| RustSmith | SPG | high | zero | heavily engineered | low |
| AFL | PG | N/A | high | coverage-guided | varies |
| Boltzmann | SPG | tunable | zero | analytic | high |

### 6.2 Mapping Engineering Challenges to Theory
- Type conformance → SPG membership
- Semantic validity → RE boundary
- Distribution control → SPG_vis membership
- Borrowing/lifetime conservatism → Type 1 error rate α

### 6.3 RustSmith Deep Dive
- Conservative borrowing = deliberate under-generation
- Formal placement: L_RustSmith ⊊ L_well-typed; L_RustSmith ∈ SPG
- Coverage gaps explained by visible-mass framework

---

## 7. Related Work (2 pages)

### 7.1 Samplable Distributions and Average-Case Complexity
- Levin 1986; Impagliazzo-Levin 1990; Watson 2017 (time hierarchies for sampling)

### 7.2 Enumeration Complexity
- Arenas et al. 2019 (constant-delay → full visible mass)
- Jerrum-Valiant-Vazirani 1986 (generation ↔ counting)

### 7.3 Random Generation of Combinatorial Structures
- Sanchis 1990 (precursor to SPG); Duchon et al. 2004 (Boltzmann sampling)

### 7.4 Fuzzing and Coverage
- Böhme et al. 2016 (AFL as Markov chain); Lemieux-Sen 2018 (FairFuzz = engineering Vis_T upward)

### 7.5 PBT Foundations
- Claessen-Hughes 2000; Lampropoulos et al. 2017 (narrowing); Paraskevopoulou et al. 2015 (Coq proofs)

### 7.6 Diversity Measures
- Hill 1973 (Hill numbers); Good 1953 (missing mass); connection to our visible metrics

---

## 8. Conclusion and Future Directions (1 page)

### Open Problems
1. **Characterize SPG_vis**: Which NP languages admit generators with poly visible mass?
2. **SPG_vis vs counting**: Is #P-hardness of counting a barrier to high visible mass?
3. **DSL for generable languages**: Design a type system/DSL that guarantees SPG_vis membership
4. **Uniform generation in SPG**: When can SPG generators be made (almost-)uniform?
5. **Space-bounded visibility**: Extend SLG to distribution-aware setting

### Broader Impact
- Foundational framework for PBT generator design
- Complexity-theoretic justification for engineering choices (under-generation, distribution tuning)
- Potential for automated generator synthesis guided by visibility metrics

---

## Appendices

- Full proofs for coverage-time results
- Proof of H-is-invalid (from original)
- Rejection sampling validity (from original)
- NDTM simulation (from original)
- CNF-SAT ∈ SPG (from original)
- SLG ⊆ SPG via DFA construction (from original)
- REACH ∈ SLG (from original)
- SPG^NP ⊇ NP (from original)
- PRIMES ∈ EPG (from original)

---

## Key Differences from Original Submission

| Aspect | Original | Revised |
|--------|----------|---------|
| Core contribution | SPG hierarchy | SPG hierarchy + SPG_vis distribution-aware extension |
| Distribution | Agnostic | Explicit visible-mass framework |
| RE=GEN | Presented as main result | Downplayed to warm-up |
| Practical connection | Mentioned in passing | Dedicated section with tool taxonomy |
| Type 1/2 errors | Not formalized | Formal parameterized classes |
| Coverage theory | Absent | Full occupancy/discovery/concentration theory |
| Reviewer C (transducers) | Not addressed | Explicitly acknowledged + novelty argument |
| Venue fit | LICS (rejected) | POPL/PLDI primary target; LICS if logic content added |

---

## Estimated Page Count
- Sections 1-4 (model + hierarchy): ~9 pages (streamlined from 12)
- Section 5 (distribution-aware, NEW): ~4 pages
- Section 6 (taxonomy, NEW): ~1.5 pages
- Sections 7-8 (related work + conclusion): ~3 pages
- **Total body**: ~17.5 pages
- **Appendices**: ~8 pages
- Fits comfortably in POPL/PLDI format (typically ~25 pages with appendix)
