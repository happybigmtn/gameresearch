[thinking] The user is asking me to decompose a research problem about "The Virus Game" from Devil's Plan S1 into prioritized sub-questions. They've provided a detailed research goal document. This is a legitimate research planning request about game theory and social deduction games.
           
           Let me analyze the research goal and decompose it into meaningful sub-questions, then provide the requested markdown structure with Source, Sub-questions, Priority Ranking, and Risks.
           
           This is a pure text-generation task - producing a research decomposition document. I don't need to read any files or use tools for this. Let me produce a high-quality response.
# Research Decomposition: Equilibrium Analysis of The Virus Game

## Source

The Virus Game (Devil's Plan S1) — a 12-player hidden-role social deduction game with 3 asymmetric factions, unique IDs 1–12 arranged on a circular ring, and bounded public signaling. Research goal: formalize the game, exploit structural symmetries, compute approximate Nash equilibria, and characterize faction-asymmetric signaling strategies.

Based on the research goal document specifying an 8-week timeline, single-GPU compute budget, and target venues (AAAI, AAMAS, NeurIPS game theory track).

---

## Sub-questions

### SQ1: Extensive-Form Formalization

**What is the complete extensive-form game representation of The Virus Game, and what are the minimal assumptions needed to close the specification?**

The show presents the game informally. A formal model requires pinning down: the exact faction assignment distribution (uniform? constrained by ring adjacency?), the action space at each decision point, the signal space and its bounds, the information partition for each player-faction pair, and the terminal payoff function. The ring topology must be encoded as a constraint on the information sets — specifically, which observations are available to player $i$ depend on $|i - j| \mod 12$. Key modeling choice: whether signals are simultaneous or sequential within a round, and whether the game has a fixed number of rounds or an endogenous termination condition. This sub-question is prerequisite to everything else.

### SQ2: Symmetry Reduction via Rotational Equivalence

**How much does the $\mathbb{Z}_{12}$ rotational symmetry of the ring reduce the information-set space, and does faction asymmetry break this symmetry partially or completely?**

In a symmetric ring with identical roles, any rotation of a strategy profile yields an equivalent profile, giving a 12x reduction. But faction asymmetry complicates this: if faction sizes are unequal (e.g., 4-4-4 vs. 5-4-3), the number of distinct faction assignments modulo rotation differs. The sub-question asks: (a) what is the orbits-of-assignments count under $\mathbb{Z}_{12}$ for the actual faction partition, (b) does the asymmetry in faction objectives further constrain which orbits are strategically equivalent, and (c) can reflection symmetry ($D_{12}$ dihedral group) provide additional reduction? The answer determines whether the game is computationally tractable on the stated hardware budget.

### SQ3: Faction-Asymmetric Signaling Equilibria

**At equilibrium, do the three factions adopt qualitatively distinct signaling strategies (pooling vs. separating vs. partial pooling), and how does ring position modulate these strategies?**

This is the central game-theoretic insight question. In 2-faction games (Mafia/Werewolf), the informed minority pools (mimics the majority). With 3 factions, the signaling incentives are more complex: faction A may want to separate from faction B but pool with faction C, depending on which faction currently faces elimination pressure. The bounded signal space constrains the degree of separation possible. Ring position matters because a player flanked by same-faction neighbors has different inference and deception incentives than one flanked by opponents. The sub-question asks whether these intuitions hold at computed equilibria and whether the faction-strategy profiles are classifiable into known signaling game categories.

### SQ4: Coalition Dynamics and Phase-Dependent Alignment

**How do pairwise faction incentives shift across game phases, and do computed equilibria exhibit coalition switching (two factions aligning against a third, then defecting)?**

Three-faction games can exhibit non-transitive dominance or shifting coalitions that don't arise in two-faction settings. As players are eliminated and information accumulates, the relative power of each faction changes, potentially flipping which pair of factions benefits from temporary cooperation. This sub-question asks: (a) does the equilibrium strategy profile exhibit identifiable "phase transitions" where coalition structure changes, (b) are these transitions driven primarily by information revelation or by the mechanical effect of eliminations, and (c) can the phase structure be characterized analytically (e.g., via backward induction on a reduced game tree) rather than only observed in the computed equilibrium?

### SQ5: Information Value of Ring Position

**What is the quantified strategic value of ring position (distance from faction boundaries) at equilibrium, and does it differ across factions?**

In a ring topology, a player at a faction boundary (adjacent to a player from a different faction) has access to different local signals than a player surrounded by same-faction members. This sub-question asks whether ring position creates measurable asymmetry in equilibrium expected payoff within the same faction. If so, it implies the game has an inherent positional fairness problem — the random assignment of IDs to the ring creates unequal starting conditions even within a faction. This would be a concrete, novel result: demonstrating that topology-induced information asymmetry has equilibrium payoff consequences in social deduction games, a phenomenon absent from fully-connected models.

### SQ6: Tractability and Exploitability Validation

**Is the symmetry-reduced game tree small enough for CFR+ convergence to $\epsilon < 0.05$ within the compute budget, and how sensitive is the equilibrium to the signal-space discretization?**

This is the feasibility sub-question. Even after symmetry reduction, the game tree may be too large. The sub-question asks: (a) what is the estimated information-set count after reduction, (b) what CFR+ iteration count is needed for the target exploitability based on known convergence rates, (c) does the signal-space discretization (which is a modeling choice from SQ1) significantly affect the equilibrium structure, and (d) if the full game is intractable, what principled abstractions (e.g., bucketing ring positions, reducing round count) preserve the qualitative phenomena of interest?

---

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0** | SQ1 (Formalization) | Everything depends on a clean formal model. Ambiguity here propagates into every downstream result. Must be completed first and reviewed for completeness before implementation begins. |
| **P0** | SQ2 (Symmetry Reduction) | Determines feasibility. If the reduction factor is insufficient, the compute plan fails and the paper must pivot to analytical results only. Must be resolved in week 1–2 alongside SQ1. |
| **P1** | SQ6 (Tractability) | Gate check before committing 100 GPU-hours. A negative answer here redirects effort toward analytical characterization (SQ4) or forces game abstractions. Should be answered via estimation before running full CFR+. |
| **P1** | SQ3 (Signaling Equilibria) | The core contribution. This is what makes the paper interesting — qualitative differences in faction signaling at equilibrium. But it requires SQ1 + SQ2 + SQ6 to be resolved favorably first. |
| **P2** | SQ4 (Coalition Dynamics) | High novelty, but analytically harder and potentially requires larger game trees (multi-phase analysis). May need to be scoped to specific game phases rather than full trajectory. |
| **P2** | SQ5 (Positional Value) | Clean, measurable result but derivative — it falls out of the computed equilibrium rather than requiring separate methodology. Low marginal effort if SQ3 succeeds. |

**Critical path:** SQ1 → SQ2 → SQ6 (gate) → SQ3 → {SQ4, SQ5} in parallel.

---

## Risks

### R1: Specification Ambiguity (Impact: High, Likelihood: High)
The Virus Game's rules as presented on Devil's Plan are partially implicit — player discussions, voting procedures, and elimination mechanics may not be fully specified in available sources. **Mitigation:** Define a canonical rule set explicitly in the paper, noting where assumptions were made. Sensitivity-test key rule variants (e.g., simultaneous vs. sequential voting) to show results are robust.

### R2: Intractability After Reduction (Impact: Critical, Likelihood: Medium)
Even with 12x rotational reduction, the extensive-form game tree for 12 players with multi-round signaling and elimination may exceed $10^{10}$ information sets, beyond CFR+ tractability on a single GPU. **Mitigation:** (a) Compute the information-set count analytically before running CFR+, (b) prepare principled abstraction strategies (action abstraction on signals, player bucketing by ring position class), (c) if necessary, solve a reduced 6- or 8-player variant and extrapolate structural properties.

### R3: Trivial Equilibria (Impact: High, Likelihood: Low-Medium)
The computed equilibria might turn out to be qualitatively similar across factions — all factions babble (random signals) or all pool identically — yielding no interesting insight. **Mitigation:** If babbling equilibria dominate, analyze whether the signal space is too coarse (refine discretization) or whether the game structure genuinely doesn't support informative signaling. A negative result ("bounded signals are uninformative in this topology") is still publishable if well-characterized.

### R4: Symmetry Exploitation Invalidity (Impact: Medium, Likelihood: Low)
The rotational symmetry argument assumes faction assignments are drawn uniformly over valid partitions. If the actual game uses a constrained assignment (e.g., no two adjacent players from the same faction), the symmetry group is smaller and the reduction analysis must account for assignment-conditional symmetries. **Mitigation:** Enumerate the actual symmetry group for the constrained assignment space; the dihedral group analysis in SQ2 should consider this case explicitly.

### R5: Scope Creep into RL (Impact: Medium, Likelihood: Medium)
Reviewers at ML venues may expect learned agent baselines (PPO/MAPPO agents playing the game) in addition to equilibrium computation. This would significantly expand the implementation burden. **Mitigation:** Frame the contribution as computational game theory, not multi-agent RL. Include heuristic baselines (SQ6) to demonstrate the equilibrium's strategic value without training RL agents. Target AAMAS or GEB over NeurIPS if reviewer expectations diverge.

### R6: Novelty Challenge at Review (Impact: Medium, Likelihood: Medium)
Reviewers may view "formalize a TV game show game" as insufficient novelty, treating it as an application paper rather than a contribution. **Mitigation:** Emphasize the structural contribution — the 3-faction ring-topology signaling game is a new game class, not just one instance. The symmetry reduction technique and the faction-asymmetric signaling characterization generalize beyond this specific game. Position the Virus Game as a motivating instance, not the sole contribution.