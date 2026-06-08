# Visibility-Aware Generator Complexity (Preview)

This is a markdown companion for `new_contributions.tex`, so formulas are easier to inspect in Markdown preview.

## Core Definitions

### Heavy Outputs

For generator $G$ with output distribution $D_G$, and budget function $T:\mathbb{N}\to\mathbb{N}$:

$$\mathsf{Heavy}_T(G)=\{x\in \mathcal{L}(G): D_G(x)\ge 1/T(|x|)\}.$$

### Visible Mass

$$\mathsf{Vis}_T(G)=\sum_{x\in \mathsf{Heavy}_T(G)} D_G(x).$$

Interpretation: how much total probability is concentrated on outputs that are "budget-visible."

### Visible Support Size

Condition on heavy outputs:

$$D_{G,T}(x)=D_G(x)\,/\,\mathsf{Vis}_T(G),\quad x\in\mathsf{Heavy}_T(G).$$

Then

$$\mathsf{VSupp}_T(G)=\frac{1}{\sum_{x\in\mathsf{Heavy}_T(G)} D_{G,T}(x)^2}.$$

This is inverse collision probability on visible outputs.

### Visible Min-Entropy and Effective Support

$$\mathsf{H}^{\min}_T(G)=-\log \max_{x\in\mathsf{Heavy}_T(G)} D_{G,T}(x),$$

$$\mathsf{ESupp}_T(G)=2^{\mathsf{H}^{\min}_T(G)}.$$

## Key Results

### Observability Probability

If $X_1,\dots,X_m\sim D_G$ i.i.d., then:

$$\Pr[\exists i\le m,\ X_i\in \mathsf{Heavy}_T(G)] = 1-(1-\mathsf{Vis}_T(G))^m.$$

So if $\mathsf{Vis}_T(G)\ge 1/\mathrm{poly}(n)$, polynomially many samples are enough to see a heavy output with high probability.

### Occupancy Profile (Expected Distinct Heavy Outputs)

$$\mathsf{Occ}_{m,T}(G)=\sum_{x\in\mathsf{Heavy}_T(G)}\left(1-(1-D_G(x))^m\right).$$

This equals expected number of **distinct** heavy outputs seen in $m$ samples.

### Discovery Curve

$$\mathsf{Disc}_{m,T}(G)=\sum_{x\in\mathsf{Heavy}_T(G)} D_G(x)(1-D_G(x))^m.$$

And:

$$\mathsf{Occ}_{m+1,T}(G)-\mathsf{Occ}_{m,T}(G)=\mathsf{Disc}_{m,T}(G).$$

### Coverage-Time Scale

$$\frac{\mathsf{VSupp}_T(G)}{\mathsf{Vis}_T(G)}.$$

Large value means visible outputs are discovered slowly even if support is theoretically correct.

### Concentration

For observed occupancy $\widehat{\mathsf{Occ}}_{m,T}(G)$:

$$\Pr\!\left[\left|\widehat{\mathsf{Occ}}_{m,T}(G)-\mathsf{Occ}_{m,T}(G)\right|\ge t\right] \le 2\exp\!\left(-\frac{2t^2}{m}\right).$$

So observed visible coverage sharply concentrates around expectation.

## Practical Takeaway (for fuzzing / PBT)

- Support correctness is necessary but not sufficient.
- You need non-negligible visible mass.
- You need sufficiently spread visible support (not one dominant output).
- The ratio $\mathsf{VSupp}_T/\mathsf{Vis}_T$ is a practical diagnostic for "why this generator feels slow."

---

## Background: PBT and Generator Combinators

### Property-Based Testing vs Unit Testing

**Unit testing** checks hand-picked input-output examples:
```
assert add(2, 3) == 5
assert add(0, 0) == 0
```

**PBT** states a **property** (universal statement) and uses a **generator** to produce random inputs:
```
@given(integers(), integers())
def test_add_commutative(x, y):
    assert add(x, y) == add(y, x)
```
When a counterexample is found, PBT frameworks **shrink** it to a minimal failing input.

### Generator Combinators

Generators are built compositionally from combinators (QuickCheck style):

| Combinator | Type | Effect |
|---|---|---|
| `choose(lo, hi)` | `Gen Int` | Uniform in [lo, hi] |
| `pure(a)` | `Gen A` | Always return `a` |
| `fmap(f, g)` | `Gen B` | Apply `f` to output of `g` |
| `oneof([g1,...,gk])` | `Gen A` | Uniform choice among generators |
| `frequency([(w1,g1),...])` | `Gen A` | Weighted choice |
| `listOf(g)` | `Gen [A]` | Random-length list from `g` |

**Distribution implications**: If a branch in `frequency` has weight $w$ out of total $W$, its probability is $w/W$. If $w/W = 2^{-\Omega(n)}$, deep structures appear with exponentially small probability — exactly the "exponentially rare outputs" problem.

### SPG Recap

SPG (Size-Dependent Polynomial Generator) from our paper:

$$\text{SPG} = \{\mathcal{L}(G) : G \text{ is an } l\text{-sized polynomial-time generator}\}$$

Key results: SPG $\subseteq$ PG, SPG $\subseteq$ NP, CNF-SAT $\in$ SPG, and under collision-resistant hash: SPG $\subsetneq$ NP.

### Mapping RustSmith to SPG

RustSmith builds well-typed Rust programs **constructively** (no rejection sampling):

1. **SPG membership**: Polynomial-time in output size → in SPG
2. **Type 1 errors**: Deliberately under-generates (conservative borrowing): $L_{\text{RustSmith}} \subsetneq L_{\text{well-typed}}$
3. **Zero Type 2 errors**: Every generated program compiles
4. **Distribution**: Some optimization passes get 100% coverage, others 0%

### Engineering Challenges

| Challenge | Simple case | Hard case |
|---|---|---|
| Type conformance | C basic types (P-time) | Rust traits (logic programming) |
| Semantic validity | Wrappers guard UB | Exact generation is undecidable |
| Distribution control | Constructive generation | Rejection sampling too expensive |

### Well-Typed Rust in the Complexity Hierarchy

$$L_{\text{well-defined}} \subseteq L_{\text{well-typed}} \subseteq L_{\text{Rust}}$$

- $L_{\text{Rust}} \in \text{SPG}$: Syntactic generation is polynomial-time
- $L_{\text{well-typed}} \in \text{SPG}$? Likely yes without generics/traits
- $L_{\text{well-defined}} \in \text{SPG}$? Unlikely — undecidable in general

### Type 1 vs Type 2 Errors

$$\text{SPG}_{\alpha,\beta} = \{ \mathcal{L}(G) : G \in \text{SPG}, \Pr[\text{Type 1}] \le \alpha, \Pr[\text{Type 2}] \le \beta \}$$

RustSmith = $\beta = 0$ (zero over-generation). In differential testing, Type 2 errors → false-positive bug reports; Type 1 errors → reduced coverage.

### The Visible-Mass Bridge

Expected samples to hit target $T$: $1/D_G(T)$. If $D_G(T) = 2^{-\Omega(n)}$, impossible in practice.

- **SPG** guarantees: $\text{supp}(D_G) = \mathcal{L}(G)$ (correct in theory)
- **Visible mass** asks: under budget $T$, how much probability is practically reachable?
- A generator in SPG with $\mathsf{Vis}_T(G) \to 0$ is correct but **useless in practice**

---

## Phase 4: Distribution-Aware Classes & Reviewer Responses

### 4.1 Visibility-Restricted Generation Classes

$$\text{SPG}_{\text{vis}}^{T,\rho} = \{ \mathcal{L}(G) : G \in \text{SPG},\; \mathsf{Vis}_T(G) \ge \rho \}$$

**Continuous hierarchy**: For fixed $T$, $\rho_1 \le \rho_2 \Rightarrow \text{SPG}_{\text{vis}}^{T,\rho_1} \supseteq \text{SPG}_{\text{vis}}^{T,\rho_2}$.

**Separation**: Languages in $\text{SPG} \setminus \text{SPG}_{\text{vis}}^{T,1/\text{poly}(n)}$ exist — e.g., $|L \cap \{0,1\}^n| = 2^n$ with uniform distribution has $\mathsf{Vis}_T(G) = 0$ for poly $T$.

### 4.2 Practical Complexity Metrics

| Metric | Definition | Interpretation |
|---|---|---|
| Rejection rate | $\beta(G) = 1 - \Pr[G(b) \in L]$ | Fraction of wasted outputs |
| Throughput | $\text{TP}(G) = (1-\beta(G))/\text{time}(G)$ | Valid outputs per unit time |
| Coverage rate | $\mathsf{Occ}_{m,T}(G) / |\mathsf{Heavy}_T(G)|$ | Fraction of heavy outputs discovered |

### 4.3 Type 1 vs Type 2 Error Classes

$$\text{SPG}_{\alpha,\beta} = \{ \mathcal{L}(G) : G \in \text{SPG}, \Pr[\text{Type 1}] \le \alpha, \Pr[\text{Type 2}] \le \beta \}$$

**Conjecture**: $L_{\text{well-typed}} \in \text{SPG}_{\alpha,0}$ for some $\alpha > 0$, but $L_{\text{well-typed}} \notin \text{SPG}_{0,0}$ (exact generation requires data-flow analysis beyond SPG).

### 4.4 Real-World Mapping

| Tool | Class | Type 1 | Type 2 | Distribution |
|---|---|---|---|---|
| QuickCheck | SPG | low (filter) | low | engineered |
| Csmith | SPG | low (wrappers) | zero | hand-tuned |
| RustSmith | SPG | high (conservative) | zero | heavily engineered |
| AFL | PG | N/A | high | coverage-guided |

### 4.5 Separation and Closure Properties of SPG\_vis

**Closure under intersection (conditional)**: If $L \in \text{SPG}_{\text{vis}}^{T,\rho}$ and $S \in P$ with $D_G(L \cap S) \ge 1/\text{poly}(n)$, then $L \cap S \in \text{SPG}_{\text{vis}}^{T',\rho'}$ (via rejection sampling against $S$).

**Non-closure under union**: There exist $L_1, L_2 \in \text{SPG}_{\text{vis}}^{T,1}$ with $L_1 \cup L_2 \notin \text{SPG}_{\text{vis}}^{T,1/2}$ for the natural combined generator — because merging halves each output's probability, pushing them below the heavy threshold. Budget must grow with the number of components.

**Strict hierarchy in ρ**: For every $0 < \rho_1 < \rho_2 \le 1$ and polynomial $T$: $\text{SPG}_{\text{vis}}^{T,\rho_2} \subsetneq \text{SPG}_{\text{vis}}^{T,\rho_1}$. Witnessed by languages whose size forces uniform mass below $\rho_2$ but above $\rho_1$.

**Strict hierarchy in T**: For polynomial $T_1 < T_2$ and fixed $\rho$: $\text{SPG}_{\text{vis}}^{T_1,\rho} \subsetneq \text{SPG}_{\text{vis}}^{T_2,\rho}$. Witnessed by languages with $|L_n|$ between $T_1(n)$ and $T_2(n)$.

**Relationship to EPG**: $\text{SPG}_{\text{vis}}^{T,\rho} \subseteq \text{EPG}$ when $\rho \ge 1/\text{poly}(n)$ — because repeated sampling with $1/\rho$ expected trials gives expected polynomial time.

### 4.6 Budget Hardness Theorem

For $|L \cap \{0,1\}^n| = 2^{\Omega(n)}$ and budget $T(n) = \text{poly}(n)$:

$$\mathsf{Vis}_T(G) \le \text{poly}(n) / 2^{\Omega(n)} = 2^{-\Omega(n)}$$

**Implication**: Practical generators **must** accept Type 1 errors (under-generation) to achieve non-trivial visible mass. This directly justifies RustSmith's conservative approach.

### 4.6 Coverage-Time Phase Transition

For $(T, \rho, \sigma)$-visibility-rich generator:
1. $m \ll \sigma/\rho$: $\mathsf{Occ}_{m,T} \approx m\rho$ (linear discovery)
2. $m \gg (\sigma/\rho)\ln\sigma$: $\mathsf{Occ}_{m,T} \approx \sigma$ (saturation)
3. Transition at $m^* \sim (\sigma/\rho)\ln\sigma$

### 4.7 Enumeration Connection

Constant-delay enumeration → full visible mass ($\mathsf{Vis}_T = 1$). Harder enumeration → lower visible mass.

---

## Phase 5: Related Work Summary

### Key Connections

| Topic | Connection to our paper |
|---|---|
| Samplable distributions (Levin, Impagliazzo) | Generator = sampler; P-samplable = polynomial-time |
| Enumeration complexity (Arenas, Jerrum) | Constant-delay = ideal generator |
| Jerrum-Valiant | Counting hardness propagates to generation |
| Boltzmann sampling (Duchon, Flajolet) | Principled distribution control |
| Coverage-guided fuzzing (Böhme) | Markov chain model; visible mass analogue |
| FairFuzz (Lemieux & Sen) | Practical visible-mass argument (reweight rare branches) |
| Hill numbers / Rényi entropy | Diversity profile = thresholded Hill numbers |
| Good-Turing (Good 1953) | Missing mass = 1 - visible mass |

### Core Papers (10 most relevant)

1. Levin 1986 — Average-case complexity, P-samplable distributions
2. Jerrum, Valiant, Vazirani 1986 — Random generation ≈ approximate counting
3. Sanchis 1990 — Predecessor of SPG, efficient generation from specs
4. Claessen & Hughes 2000 — QuickCheck (original PBT)
5. Duchon et al. 2004 — Boltzmann samplers
6. Böhme, Pham, Roychoudhury 2016 — Coverage-based greybox fuzzing as Markov chain
7. Lampropoulos et al. 2017 — Generating good generators
8. Hill 1973 — Diversity measures / Hill numbers
9. Arenas et al. 2019 — Constant-delay enumeration
10. Watson 2017 — Time hierarchies for sampling