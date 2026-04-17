[thinking] The user is asking me to decompose a research problem about the "Montage" game from Devil's Plan Season 1 into prioritized sub-questions. They've provided a detailed research goal document as context. This is a legitimate research planning request - they want me to generate a structured markdown document with Source, Sub-questions, Priority Ranking, and Risks sections.
           
           This is a text generation task - I can just write this out directly as a response. I don't need to use tools for this since it's about generating a research decomposition based on the provided context.
           
           Let me think about the sub-questions carefully:
           
           
           The core research problem centers on multi-agent signal detection when penalties accumulate during cooperative buzzer games. I need to break this down into foundational questions about signal detection theory in multi-agent settings, the speed-accuracy tradeoffs that eme...
# Research Decomposition: Montage — Multi-Agent Signal Detection Under Penalty Accumulation

## Source

**Primary:** Research goal document for "Multi-Agent Signal Detection Under Penalty Accumulation in Cooperative Buzzer Games," based on the Montage game from Devil's Plan Season 1. The game is a 6-player buzzer challenge where players receive noisy face-similarity signals, race to buzz in, and accumulate escalating penalties for incorrect responses — creating a three-way coupling of preemption externality, penalty-driven non-stationarity, and cooperative team scoring.

**Theoretical foundations:** Signal detection theory (Green & Swets), drift-diffusion models (Ratcliff), Bayes-Nash equilibrium in timing games, multi-agent RL (IPPO/PPO in PettingZoo environments).

---

## Sub-questions

### SQ1: What is the symmetric Bayes-Nash equilibrium threshold policy for 2-player Montage, and how does it depend on penalty state and signal noise?

**Why this matters:** This is the analytical anchor for the entire project. Without a closed-form (or at least numerical) characterization of the 2-player equilibrium, you have no ground truth to validate RL training against. The 2-player case is tractable because each player's optimal threshold depends only on their own penalty state and the *distribution* of the opponent's threshold — a fixed-point problem in one dimension.

**Key sub-parts:**
- Formalize the per-round decision as a single-shot SDT problem conditioned on penalty state
- Model the timing component: does signal evidence accumulate within a round (DDM-style), or is it a single noisy draw?
- Derive the best-response threshold function: given opponent plays threshold τ*, what is my optimal τ as a function of my penalty count and signal noise σ?
- Characterize the fixed point (symmetric BNE) and check uniqueness

---

### SQ2: How does penalty accumulation reshape the optimal detection threshold over the course of a game?

**Why this matters:** This is the core novelty claim — that penalties make the game non-stationary in a way that creates qualitatively different equilibrium structure compared to penalty-free buzzer games. If the threshold just shifts linearly with penalties, the result is incremental. If there are phase transitions (e.g., a "give up" threshold where a player stops buzzing entirely), regime switches, or non-monotonic dynamics, that's a publishable finding.

**Key sub-parts:**
- In the penalty-free baseline, what is the static equilibrium? (This is a standard preemption game with noisy signals.)
- As penalty accumulation rate increases, does the equilibrium threshold increase monotonically (more conservative), or is there a crossover where trailing players become *more* aggressive (gambling to recover)?
- Is there a critical penalty level beyond which a rational agent should never buzz? How does this interact with finite-horizon effects (desperation in final rounds)?
- Map the threshold-vs-penalty-state curve for different signal noise levels σ

---

### SQ3: Does the 6-player game exhibit emergent role differentiation or symmetry breaking among identical agents?

**Why this matters:** This is the most interesting empirical question and the one most likely to produce a surprising result. In many multi-agent settings, identical agents trained via independent learning converge to symmetric equilibria. But the preemption + penalty structure of Montage creates strong incentives for *niche specialization* — some agents may learn to be aggressive early-buzzers (accepting higher false alarm rates to capture easy rounds) while others play conservatively (waiting for high-confidence signals, accepting fewer opportunities). If this emerges from independent PPO without any explicit role assignment, it demonstrates a self-organizing division of labor.

**Key sub-parts:**
- Train 6 identical IPPO agents and inspect the learned threshold policies — are they symmetric or differentiated?
- Measure the variance in learned thresholds across agents at each penalty state
- Test whether differentiation depends on the penalty accumulation rate (low penalty → symmetric; high penalty → differentiated?)
- Under team scoring, does cooperation increase or decrease role differentiation?

---

### SQ4: How do cooperative (team) scoring rules alter individual SDT criterion placement compared to individual scoring?

**Why this matters:** The cooperative layer is the third leg of the "three-way coupling" that distinguishes this from standard competitive buzzer models. Under team scoring, a false alarm hurts everyone, so individual thresholds should internalize this externality and shift conservative. But the preemption externality pushes the opposite direction — if you don't buzz, a teammate might buzz incorrectly anyway. The equilibrium between these opposing forces is non-obvious and likely depends on team size, penalty schedule, and signal correlation.

**Key sub-parts:**
- Compare equilibrium thresholds under individual vs. team scoring in the 2-player analytical case
- In the 6-player RL case, measure the aggregate false alarm rate under both scoring regimes
- Does team scoring produce implicit coordination (e.g., agents learning to defer to the agent with the lowest penalty count)?
- What happens with mixed incentives (partially cooperative, partially individual)?

---

### SQ5: Can IPPO agents recover the analytical 2-player BNE, validating the environment and training pipeline?

**Why this matters:** This is the calibration gate. If RL agents can't match the analytical solution in the tractable 2-player case, none of the 6-player results are trustworthy. This sub-question also stress-tests the environment implementation — any bug in reward calculation, signal generation, or turn mechanics will show up as a deviation from the analytical baseline.

**Key sub-parts:**
- Train 2-agent IPPO and compare learned threshold policy to analytical BNE
- Measure exploitability: can a best-response agent improve unilaterally against the learned policy?
- Convergence diagnostics: does training converge stably, or do agents oscillate?
- Sensitivity to hyperparameters (learning rate, entropy bonus, reward normalization)

---

### SQ6: What is the geometry of the speed-accuracy-penalty tradeoff surface across parameter regimes?

**Why this matters:** This is the "main results figure" — a 3D characterization of how expected score, false alarm rate, and response latency co-vary as you sweep signal noise (σ), penalty rate, and number of players. The shape of this surface tells the full story: are there Pareto-optimal operating points? Sharp transitions? Regions where small parameter changes cause large strategy shifts?

**Key sub-parts:**
- Define the parameter grid: σ ∈ {low, medium, high} × penalty rate ∈ {0, linear, quadratic} × players ∈ {2, 4, 6}
- For each configuration, extract: mean score, hit rate, false alarm rate, mean buzz latency, threshold at penalty=0 vs. penalty=max
- Identify phase boundaries where strategy qualitatively changes
- Generate the tradeoff surface visualization

---

## Priority Ranking

| Rank | Sub-question | Rationale |
|------|-------------|-----------|
| **P0** | **SQ1** — 2-player analytical BNE | Everything downstream depends on this. No analytical anchor → no validation → no credible results. Week 1 deliverable. |
| **P0** | **SQ5** — RL recovers 2-player BNE | Inseparable from SQ1 — this is the validation that the computational pipeline works. Must pass before scaling to 6 players. |
| **P1** | **SQ2** — Penalty accumulation dynamics | The core novelty claim. If penalties don't create qualitatively new dynamics, the paper's contribution is thin. This is where the "publishable finding" lives or dies. |
| **P1** | **SQ3** — 6-player symmetry breaking | The most likely source of a surprising result. Tightly coupled with SQ2 (penalty rate is the key control variable for symmetry breaking). |
| **P2** | **SQ4** — Cooperative vs. individual scoring | Important for completeness and the "cooperative layer" framing, but secondary to establishing the penalty dynamics. Can be a clean extension once SQ2-SQ3 are solid. |
| **P2** | **SQ6** — Full tradeoff surface | This is synthesis, not discovery. Depends on all prior sub-questions being answered. Week 3-4 work. |

---

## Risks

### R1: The 2-player BNE may not admit a clean closed-form solution
**Likelihood:** Medium. **Impact:** High.
If the penalty-state dependence makes the fixed-point equation intractable, you'll need numerical solutions, which weakens the "analytical anchor" story. **Mitigation:** Accept a numerical BNE computed via iterated best response — this is standard in mechanism design. The key is that it's *exact* (not learned), even if not closed-form.

### R2: IPPO may not converge to equilibrium in the 6-player case
**Likelihood:** Medium. **Impact:** High.
Independent learning in multi-agent settings is known to be unstable. Agents may cycle, diverge, or converge to poor local equilibria. **Mitigation:** (1) Use population-based training or multiple seeds to check robustness; (2) try centralized-critic variants (MAPPO) if IPPO fails; (3) the 2-player validation (SQ5) catches pipeline issues early.

### R3: Penalty accumulation may produce only quantitative, not qualitative, differences
**Likelihood:** Medium. **Impact:** High (undermines the novelty claim).
If the optimal threshold just shifts smoothly with penalty count — no phase transitions, no symmetry breaking, no regime changes — the contribution reduces to "we added a parameter to SDT." **Mitigation:** Sweep a wide range of penalty schedules (linear, quadratic, step-function). If even extreme penalties don't produce qualitative shifts, pivot the framing to "robustness of symmetric equilibria under cost asymmetry."

### R4: The environment model may not capture the perceptual richness of face recognition
**Likelihood:** Low (by design). **Impact:** Low.
The 1D signal model is deliberately abstract — it captures the SDT structure without requiring a vision model. But a reviewer might argue the results don't generalize to actual face recognition. **Mitigation:** Acknowledge this explicitly as a scope constraint. The contribution is game-theoretic, not perceptual.

### R5: Scope creep into human behavioral experiments
**Likelihood:** Low. **Impact:** Medium.
The research goal explicitly excludes human data collection, but reviewers or collaborators may push for behavioral validation. **Mitigation:** Frame the paper as "computational modeling" with qualitative comparison to observed Devil's Plan gameplay. Leave human experiments as explicit future work.

### R6: Compute budget insufficient for full parameter sweep
**Likelihood:** Low (environment is lightweight). **Impact:** Medium.
6-agent IPPO across a full parameter grid could exceed the single-GPU, 4-hour-per-run budget. **Mitigation:** Prioritize the parameter combinations most likely to produce phase transitions (high penalty rate, medium noise). Use the 2-player analytical results to narrow the interesting regime before running expensive 6-player sweeps.

---

`★ Insight ─────────────────────────────────────`
- **SQ1 and SQ5 are a validation pair, not separate questions.** The analytical solution (SQ1) only matters because it lets you calibrate the RL pipeline (SQ5). Together they form a "trust gate" — if this pair fails, nothing else in the project is credible. This is a common pattern in computational game theory: always have a tractable special case that your general method must recover.
- **The priority ranking reflects a "novelty funnel."** P0 questions establish correctness. P1 questions are where the novel contribution lives — penalty dynamics (SQ2) and emergent asymmetry (SQ3). P2 questions are extensions that strengthen the paper but don't define it. If you're time-constrained, a strong P0+P1 result is publishable; P2 is polish.
- **Risk R3 is the "so what?" risk.** Every research project has a version of this — the possibility that the phenomenon you're studying turns out to be boring. The mitigation (wide penalty schedule sweep + pivot strategy) is the right move: plan for the interesting case but have a graceful degradation path.
`─────────────────────────────────────────────────`