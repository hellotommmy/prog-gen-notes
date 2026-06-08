# Study Session Notes — 2026-04-03

## Agenda
Continue from Phase 3 → Phase 4 (reviewer responses) → Phase 5 (literature survey).
Ensure all content is persisted in `new_contributions.tex` and `new_contributions_preview.md`.

## Phase 4 — Reviewer Response Formalizations

### 4.1 Distribution-aware classes
- Define `SPG_vis^{T,ρ}`: SPG generators with visible mass ≥ ρ under budget T.
- Separation: show SPG_vis ⊊ SPG (exponentially flat distributions).
- Continuous hierarchy: ρ₁ ≤ ρ₂ ⟹ SPG_vis^{T,ρ₁} ⊇ SPG_vis^{T,ρ₂}.

### 4.2 Practical complexity metrics
- Coverage rate = Occ_{m,T}(G)
- Rejection rate β(G) = 1 − Pr[G(b) ∈ L]
- Throughput TP(G) = (1 − β(G)) / time(G)

### 4.3 Type 1 vs Type 2
- SPG_{α,β} formalization.
- Conjecture: L_well-typed ∈ SPG_{α,0} for some α, but not SPG_{0,0}.

### 4.4 Real-world mapping
| Tool | Class | Type 1 | Type 2 | Distribution |
|------|-------|--------|--------|-------------|
| QuickCheck | SPG | low | low | engineered |
| Csmith | SPG | low | zero | hand-tuned |
| RustSmith | SPG | high | zero | heavily engineered |
| AFL | PG | N/A | high | coverage-guided |

## Phase 5 — Literature Survey
Agent produced `related_work.md` with 70+ papers across 11 sections.
Key connections:
- Samplable distributions → generator as sampler
- Enumeration complexity → constant-delay = ideal generator
- Boltzmann sampling → principled distribution control
- Coverage-guided fuzzing → Markov chain model, visible mass analogue
- FairFuzz → practical visible-mass argument (reweight rare branches)
