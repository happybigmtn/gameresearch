[thinking] The user is asking me to decompose a research problem about Nine Men's Morris optimal strategy and solver algorithms into sub-questions. This is a legitimate research planning request - they've provided a detailed research goal document and want me to create a structured decomposition.
           
           This is a text-generation task - I can just produce the markdown output directly. No tools needed since this is pure analytical/planning work based on the information provided.
           
           Let me create a thorough decomposition.
## Source

Research goal document: *"Nine Men's Morris Phase-Aware Neural Strategy Learning"* (2026-04-17). Based on the known solved status of NMM (Gasser, 1996) and the proposed comparison of phase-decomposed vs. monolithic neural architectures for policy learning validated against the retrograde analysis database.

---

## Sub-questions

### SQ1: Environment & Ground Truth Infrastructure
**How should the NMM environment and optimal-play validation oracle be implemented to support reproducible architecture comparison?**

- What open-source NMM implementations exist, and do any correctly handle all three phases (placement, movement, flying) with legal-move masking?
- What is the most accessible form of the retrograde analysis database (Gasser/Stahlhacke), and what is the storage/lookup cost for per-state optimal value and optimal action set queries?
- How should state representation encode phase identity — is phase implicit in piece counts, or must it be an explicit input channel?
- What action-space encoding handles the union of placement (24 positions), sliding (variable adjacency moves), and mill-capture (opponent piece removal) without introducing invalid-action noise?

### SQ2: Architecture Design & Phase Decomposition
**What concrete neural architecture variations isolate the effect of phase-awareness as a structural prior?**

- For the phase-decomposed network: should sub-networks share only the value head, or also share early representation layers with phase-specific policy heads branching later? What is the parameter-fairness tradeoff?
- For the phase-conditioned network: is a learned phase embedding sufficient, or does phase information need to modulate intermediate layers (e.g., FiLM conditioning)?
- How should the monolithic baseline handle the variable action space across phases — action masking over a universal action vector, or a fixed-size output with invalid-action penalties?
- What parameter budget keeps all three architectures comparable so that accuracy differences reflect structural priors, not capacity differences?

### SQ3: Training Protocol & Measurement
**What self-play training regime and evaluation protocol yield statistically reliable architecture comparisons on a single-GPU budget?**

- Is pure self-play sufficient, or should training be augmented with expert iteration against the optimal database (and if so, does this confound the architectural comparison)?
- How many training seeds are needed to detect a meaningful effect size given the variance of self-play RL? (The goal states 5+ seeds — is this adequate given expected variance?)
- At what granularity should optimal-move agreement be measured — every N training steps, and over how many held-out states per phase?
- How should "phase-transition states" be formally defined for the transition-accuracy metric — states where the *next* move could trigger a phase change, or states within k plies of a phase boundary?

### SQ4: Phase-Transition Discontinuity Analysis
**Do phase transitions in NMM create measurable policy-learning discontinuities, and does phase-aware architecture reduce them?**

- What does the optimal value landscape look like near phase boundaries — are there sharp value gradients when a player transitions from movement to flying (dropping to 3 pieces)?
- Can the accuracy gap at phase transitions be attributed to action-space distribution shift (new move types become legal) vs. strategic discontinuity (optimal strategy changes character)?
- Is there a correlation between phase-transition proximity and policy error magnitude in the monolithic baseline that disappears in the decomposed architecture?
- Do phase-decomposed networks exhibit negative transfer between phase-specific heads, or does shared representation always help?

### SQ5: Generalizability & Broader Implications
**What do the NMM results imply for phase-aware architecture design in other multi-phase sequential decision problems?**

- Are the findings specific to NMM's phase structure (deterministic, piece-count-triggered), or do they suggest a general principle for any MDP with discrete regime changes?
- How does NMM's phase structure compare to phase transitions in other solved/partially-solved games (e.g., Checkers endgame vs. midgame, Chess opening/middlegame/endgame)?
- Could the phase-decomposition approach be applied to non-game domains with known phase structure (e.g., multi-stage manufacturing, clinical treatment protocols)?

---

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0 — Critical path** | SQ1 (Environment & Ground Truth) | Everything depends on a correct environment and reliable oracle. A bug in phase-transition logic or action masking invalidates all downstream results. This is Week 1 work and the single highest risk item. |
| **P0 — Critical path** | SQ2 (Architecture Design) | The architectural comparison is the paper's core contribution. Parameter fairness between variants must be established before training begins. Wrong design choices here waste all compute budget. |
| **P1 — Core result** | SQ3 (Training & Measurement) | Statistical rigor determines publishability. The training protocol must be locked before the main experiment runs (Week 3). Under-powered experiments or poorly defined metrics kill the paper at review. |
| **P1 — Core result** | SQ4 (Discontinuity Analysis) | This is the paper's *differentiating* claim — that phase transitions create measurable learning difficulties that phase-aware architectures mitigate. Without this analysis, the paper is a routine architecture comparison. |
| **P2 — Strengthening** | SQ5 (Generalizability) | Important for framing and impact, but the NMM-specific results stand on their own. This is discussion-section material, not a prerequisite for the core contribution. Can be addressed qualitatively if time is short. |

---

## Risks

| Risk | Severity | Likelihood | Mitigation |
|------|----------|------------|------------|
| **Null result:** Phase-decomposed architecture shows no significant improvement over monolithic baseline | High | Medium | A null result is still publishable if the analysis is rigorous (showing phase transitions are *not* a bottleneck is informative). Design the paper to frame either outcome as a contribution. Pre-register hypotheses. |
| **Environment bugs in phase logic:** Incorrect legal-move generation at phase boundaries (especially the flying phase trigger at exactly 3 pieces, or mill-capture as a sub-action) | High | Medium-High | Write exhaustive unit tests for phase transitions before any training. Cross-validate against the retrograde database for a sample of known positions. NMM has subtle rules (e.g., can you break a mill to capture?) — document rule choices explicitly. |
| **Retrograde database unavailability or format issues:** Gasser's original database may not be publicly hosted in a usable format | Medium | Medium | Identify the exact source early (Week 1, day 1). Stahlhacke's work and the Mühle program may have accessible databases. Worst case: implement retrograde analysis from scratch for the endgame (movement+flying phases) — feasible but adds ~1 week. |
| **Parameter fairness confound:** Decomposed architecture has more parameters due to separate heads, and any improvement is attributable to capacity, not structure | High | High | Design the monolithic baseline with equivalent total parameters (wider layers). Run an ablation with a "decomposed but randomly assigned" control — same architecture, but states are randomly routed to heads regardless of phase. |
| **Self-play instability on single-GPU budget:** Training doesn't converge within the 8-hour compute cap, or variance across seeds is too high for statistical significance | Medium | Medium | Use the optimal database for warm-starting (supervised pre-training phase) before self-play fine-tuning. Reduce the number of MCTS simulations per move if needed. Consider shorter games via the placement phase database to accelerate episodes. |
| **Phase-transition states are too rare in self-play:** The critical moments (dropping to 3 pieces) may occur infrequently, giving the network few training signals at boundaries | Medium | Medium | Augment training with curated positions near phase boundaries (curriculum learning). Measure the frequency of phase transitions in self-play games to quantify this risk early. |
| **Scope creep into game variants:** Temptation to extend to Twelve Men's Morris, Lasker Morris, or other variants dilutes the contribution | Low | Medium | Hard scope boundary in the goal document (standard 9-piece only). Variants go into Future Work. |