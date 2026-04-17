[thinking] The user is asking me to create a research goal document about Nine Men's Morris optimal strategy and solver algorithms. This is a legitimate research planning request. Let me think about this carefully and produce a high-quality research plan.
           
           Nine Men's Morris is a well-studied game from a game-theory perspective - it was solved by Ralph Gasser in 1996, proving it's a draw with optimal play from both sides. The solution used retrograde analysis. So I need to find a genuinely novel angle.
           
           Let me think about what's NOT well-studied:
           
           1. The game was solved using retrograde analysis, but the database is enormous. Modern approaches using neural networks for compression of the endgame database haven't been deeply explored.
           
           
           2. The phase-based structure of NMM - with distinct placing, moving, and flying phases - creates unique challenges for reinforcement learning since the action spac...
# Research Goal: Nine Men's Morris Phase-Aware Neural Strategy Learning

## Topic

Optimal strategy learning and solver algorithms for Nine Men's Morris (NMM), a combinatorial game with distinct phase-based mechanics (placement, movement, flying), studied through the lens of reinforcement learning architecture design.

## Novel Angle

**Nine Men's Morris was solved via retrograde analysis by Gasser (1996), proving optimal play yields a draw.** This makes the core "solve the game" question settled. Most subsequent work has treated NMM as a monolithic game environment. The unexplored gap lies in **how the game's phase structure interacts with learned policy architecture**.

**Why this is timely:**
- Recent RL and game-playing research has moved toward architectures that handle variable action spaces (e.g., entity-based transformers in StarCraft, action-space masking in card games). However, these advances have not been systematically studied in classical board games with *discrete, well-defined phase transitions* where ground truth exists.
- NMM is uniquely suited as a testbed because: (1) it has three phases with fundamentally different action semantics (place a piece vs. slide a piece vs. fly a piece anywhere), (2) optimal play is known for every reachable state, enabling exact measurement of policy quality against the solved database, and (3) the state space (~10^10 states) is large enough to be non-trivial but small enough for single-GPU experiments.

**What's specifically unexplored:**
- Whether **phase-decomposed policies** (separate sub-networks per phase with a shared value head) learn faster and generalize better than monolithic policies in NMM.
- The **critical transition points** between phases (e.g., when a player drops to 3 pieces and enters flying phase) create strategic discontinuities. No existing work quantifies how well standard RL architectures handle these discontinuities vs. phase-aware alternatives.
- Using the **solved game as a dense supervision signal** — not just win/loss, but per-state optimality labels — to benchmark policy distillation techniques for phase-based games.

**How this differs from standard approaches:**
Standard NMM AI work uses alpha-beta search with handcrafted evaluation functions or flat MCTS. Standard RL game work (AlphaZero-style) treats the game as a single MDP. This work explicitly models the phase structure as an architectural prior and measures its impact against ground truth.

## Scope

A single paper comparing phase-decomposed vs. monolithic neural architectures for NMM policy learning, validated against the known optimal solution database. Limited to the standard board (no variants).

## SMART Goal

**Specific:** Implement and compare three architectures — (1) monolithic AlphaZero-style network, (2) phase-decomposed network with separate policy heads per game phase and shared representation, (3) phase-conditioned network using phase embedding as auxiliary input — for self-play learning in Nine Men's Morris.

**Measurable:** Report (a) convergence speed (training steps to reach X% agreement with optimal play), (b) per-phase policy accuracy against the solved database, (c) head-to-head win rates between architectures, and (d) accuracy specifically at phase-transition states.

**Achievable:** NMM state space fits in memory for lookup validation. Self-play training on a single GPU is feasible within hours (the game tree is orders of magnitude smaller than Go/Chess). Retrograde analysis databases for NMM are publicly available.

**Relevant:** Contributes to understanding how structural game priors in architecture design affect RL policy quality — applicable beyond NMM to any game or decision process with phase transitions (e.g., economic simulations, multi-stage negotiations).

**Time-bound:** 4 weeks — Week 1: environment + baseline, Week 2: phase-decomposed architectures, Week 3: training + evaluation, Week 4: analysis + writing.

## Benchmark

| Attribute | Detail |
|---|---|
| **Name** | NMM Optimal Play Database (Gasser/Stahlhacke retrograde solution) |
| **Source** | Publicly reconstructible via retrograde analysis; Stahlhacke's database covers all reachable positions with optimal values |
| **Metrics** | (1) **Optimal move agreement** — % of states where the policy selects an optimal action. (2) **Phase-specific accuracy** — agreement broken down by placement/movement/flying phase. (3) **Transition accuracy** — agreement at states within 1 move of a phase change. (4) **Training efficiency** — steps to 90%/95%/99% optimal agreement. |
| **Current SOTA** | The game is solved (draw). For learned policies: no published benchmark exists for neural NMM play accuracy against optimal — this is part of the contribution. Traditional alpha-beta engines with tuned evaluation functions play near-optimally but are not learned systems. |

## Constraints

- **Compute:** Single consumer GPU (RTX-class), training runs capped at ~8 hours each
- **Tools:** PyTorch, custom NMM environment (to be implemented or adapted from open-source), retrograde analysis database for validation
- **Data:** Self-play generated; validation against precomputed optimal play database
- **Scope limit:** Standard 9-piece NMM only; no game variants, no multi-game transfer

## Success Criteria

A publishable result requires:

1. **Statistically significant difference** in convergence speed or final policy accuracy between at least two architectures (p < 0.05 across 5+ seeds)
2. **Phase-transition analysis** showing measurable accuracy gaps at phase boundaries for the monolithic architecture that the decomposed architecture reduces
3. **Ablation evidence** isolating the contribution of phase-awareness (not just extra parameters)
4. **Reproduction package** — open-source environment, training code, and evaluation scripts

A strong result would additionally show that the phase-decomposed architecture reaches 95%+ optimal move agreement with fewer training steps than the monolithic baseline, and that the accuracy gap is concentrated at phase-transition states.

## Generated

2026-04-17T00:00:00Z