[thinking] The user is asking me to decompose a research problem about "Zoo" from Devil's Plan Season 1 into prioritized sub-questions. They've provided a detailed research goal document. This is a legitimate research planning task — I should analyze the provided research goal and produce a structured decomposition.
           
           This is a text-generation task based on the provided context. I don't need to read files or search the codebase — the user has given me all the context I need in the prompt. Let me produce the decomposition.
           
           Let me think carefully about the game mechanics and the research questions that arise:
           
           1. **Formalization** — How do you model this as a formal game? What's the state space, action space, type space, payoff function?
           
           
           2. **Dumping equilibria** — Under what conditions is dumping a dominant or equilibrium strategy? This is the core novel question.
           3. **Hidden information dynami...
# Research Decomposition: Zoo (Devil's Plan S1)

## Source

Research goal document dated 2026-04-17. Topic: Inverse auction theory and combinatorial grid placement with hidden heterogeneous objectives and strategic dumping in a 9-player simultaneous-move game on a 5×5 grid. Primary novelty claim: the intersection of reverse auction mechanics, spatial combinatorial action spaces, and hidden personal conditions creates a game structure not addressed by existing auction theory, combinatorial game theory, or multi-agent RL literature.

---

## Sub-questions

### SQ-1: Formal Game Model — What is the minimal complete formalization of Zoo as a Bayesian simultaneous-move game?

**What this resolves:** Before any analysis, the game needs a precise mathematical specification. The research goal sketches components (state space, action space, hidden types, payoffs) but the formalization choices are load-bearing. Key decisions:

- **Type space structure.** The hidden personal conditions are described as "spatial/numerical predicates" (row sums, adjacency constraints, etc.). Are these best modeled as utility functions over grid configurations, as constraint-satisfaction objectives, or as scoring modifiers on a common payoff? The choice determines whether standard Bayesian game machinery applies or whether you need a constraint-game formulation.
- **Timing model.** "Simultaneous placement" — but is it one-shot (all tiles placed at once) or sequential rounds with partial observation? The show uses rounds, which makes it a stochastic game with belief updating, not a one-shot normal-form game. This distinction fundamentally changes tractability.
- **Dump externality encoding.** The payoff function must capture both self-minimization and opponent-inflation. Is dumping modeled as a direct externality (your tile placement changes opponents' scores) or as a shared-resource competition (grid cells are contested, and occupying them imposes costs)?

**Deliverable:** A formal game tuple (N, S, A, Θ, P, u) with explicit definitions, plus a proof that the game admits a Bayesian Nash equilibrium (via standard existence theorems for finite games, or identifying the specific structure that guarantees existence).

---

### SQ-2: Dumping Equilibrium Characterization — Under what type distributions does pure dumping constitute a Nash equilibrium, and when do mixed or conditional strategies dominate?

**What this resolves:** The core novel claim. "Dumping" — placing high-value tiles to inflate opponents' totals — is the strategic centerpiece, but its equilibrium status depends on hidden-condition distributions in non-obvious ways.

- **When dumping dominates.** If all players have similar conditions (e.g., "minimize your column sum"), dumping into opponents' columns is straightforwardly beneficial. But if conditions are heterogeneous and unknown, dumping without knowing *which* cells matter to which opponent becomes a noisy strategy. Under what heterogeneity threshold does dumping degrade from dominant to dominated?
- **Dumping as a coordination problem.** If multiple players dump into the same region, they may inadvertently help each other (piling high tiles into cells that no one's condition cares about). This creates a coordination externality among dumpers. Is there a "dumping congestion" effect analogous to congestion games?
- **Conditional dumping.** Players might dump only after inferring opponents' types from early-round placements. This introduces a signaling/screening dynamic: does placing a tile in position (i,j) reveal information about your condition? Can you exploit this by placing decoy tiles?

**Deliverable:** For the simplified variant (3×3, 3 players, 2 condition types): a complete characterization of Nash equilibria, including whether pure dumping, conditional dumping, or mixed strategies survive. For the full game: conjectured equilibrium structure informed by RL experiments.

---

### SQ-3: Spatial Topology Effects — Does the 5×5 grid structure materially alter equilibrium behavior compared to an equivalent non-spatial game?

**What this resolves:** The research goal claims the spatial component is "load-bearing" but this needs verification. If you replaced the 5×5 grid with an unstructured set of 25 positions (same cardinality, no adjacency), would equilibria change qualitatively?

- **Adjacency-dependent conditions.** If personal conditions reference adjacency (e.g., "no two of your tiles adjacent"), the grid topology directly shapes the feasible strategy set. Without adjacency, these conditions become vacuous. This is the obvious channel.
- **Spatial dumping targeting.** On a grid, dumping has geometric locality — placing a high tile affects the row, column, and neighborhood of that cell. On an unstructured set, dumping affects only the target cell. Grid structure could make dumping either more or less effective depending on whether conditions aggregate spatially.
- **Controlled comparison.** Run identical RL training on (a) the 5×5 grid game and (b) an isomorphic game where positions are an unstructured set (same payoffs, but conditions rewritten to remove spatial predicates). Compare emergent strategy profiles, win rates, and exploitability.

**Deliverable:** Statistical comparison (effect size + significance test across 1000+ games) of strategy profiles and win rates between grid and non-grid variants. If significantly different: identify which spatial features drive the divergence.

---

### SQ-4: Type Inference and Signaling — Can players infer opponents' hidden conditions from observed placements, and does this inference pressure create signaling equilibria?

**What this resolves:** The hidden-information component is not just noise — it creates an inference game layered on top of the placement game. Each placement reveals information about the placer's condition (rational players place tiles that help their condition, so observing placements constrains the condition space).

- **Observational leakage.** If a player places low tiles in a specific row, opponents can infer that player's condition likely involves that row. This leakage is involuntary (you can't minimize your score without revealing what you're minimizing). How much information leaks per round, formally?
- **Strategic obfuscation.** Knowing that placements leak information, can a player benefit from *suboptimal* early placements that disguise their condition, sacrificing early-round efficiency for late-round information advantage? This is a classic exploration-exploitation tradeoff in a social context.
- **Signaling equilibria.** In the sequential-round version, do separating or pooling equilibria emerge? A separating equilibrium means each condition type plays distinctly (fully revealing); a pooling equilibrium means all types play identically (no information leakage). Semi-separating equilibria (partial revelation) are the interesting middle ground.

**Deliverable:** Information-theoretic analysis of type leakage rate per round in the simplified variant. For the full game: measure mutual information between agent trajectories and hidden types in trained RL agents, testing whether agents learn to obfuscate or signal.

---

### SQ-5: Multi-Agent RL Emergent Strategies — What non-obvious strategy profiles emerge from self-play training, and do they match or contradict game-theoretic predictions?

**What this resolves:** The full 9-player game is analytically intractable. Self-play RL serves as an empirical equilibrium finder. The question is whether emergent strategies are "boring" (converge to simple heuristics like greedy-minimize or pure-dump) or "interesting" (discover conditional cooperation, type-signaling, position-dependent thresholds).

- **Algorithm choice.** MAPPO with parameter sharing is the natural starting point (scalable to 9 agents, handles partial observability). But independent PPO may discover more diverse strategies. Compare both.
- **Strategy taxonomy.** After training, classify agent behavior by analyzing policy rollouts. Key categories: self-minimizer (ignores opponents), dumper (maximizes opponents' totals), conditional dumper (dumps only after inferring opponent types), cooperator (implicitly coordinates with a subset of players).
- **Robustness.** Trained agents should be tested against novel opponents (heuristic baselines, agents trained with different seeds) to check whether emergent strategies are robust or brittle. A strategy that wins in self-play but loses to a simple baseline is not a meaningful equilibrium.

**Deliverable:** Trained agents with >60% win rate against all four baselines. Taxonomy of ≥3 emergent strategy profiles with cluster analysis on action distributions. Comparison of MAPPO vs. independent PPO emergent behavior.

---

### SQ-6: Price of Hidden Information — How much does incomplete information about opponents' conditions cost, and which condition types are most vulnerable?

**What this resolves:** Quantifying the strategic cost of hidden information by comparing agents trained with full observability (oracle agents who see all conditions) against agents trained with partial observability (realistic agents who only see their own condition). The gap measures how much the hidden-information mechanic matters.

- **Aggregate gap.** Average win rate difference between oracle and realistic agents across all condition types. If the gap is small, hidden information is a flavor feature, not a structural one.
- **Type-specific vulnerability.** Some conditions may be easier to play under uncertainty than others. A condition like "minimize your total" is independent of opponents and suffers no information cost. A condition like "have the lowest row sum in row 3" depends critically on knowing what opponents are placing in row 3. Mapping the price of information per condition type reveals which mechanics are strategically deep vs. cosmetic.

**Deliverable:** Win rate gap (oracle vs. realistic) measured per condition type. Identification of high-information-cost and low-information-cost conditions.

---

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0** | SQ-1: Formal Game Model | Everything depends on this. Without a precise formalization, equilibrium analysis and RL environment implementation are impossible. Blocking dependency for all other SQs. |
| **P1** | SQ-2: Dumping Equilibrium Characterization | The primary novelty claim. If dumping equilibria can be cleanly characterized even in the simplified variant, this alone is a publishable contribution. Depends on SQ-1. |
| **P2** | SQ-5: Multi-Agent RL Emergent Strategies | The empirical backbone. Required for full-game results since analytical intractability rules out closed-form equilibria for 9 players. Also generates the data for SQ-3, SQ-4, and SQ-6. Depends on SQ-1 (for environment specification). |
| **P3** | SQ-4: Type Inference and Signaling | High intellectual novelty — the inference layer is what makes Zoo more than a standard auction game. But it's analytically hard and empirically subtle. Partially addressable via SQ-5 data (measuring mutual information in trained agents). |
| **P4** | SQ-3: Spatial Topology Effects | Important for the "grid structure is load-bearing" claim but lower priority because it requires a controlled ablation experiment, which is only meaningful after SQ-5 produces trained agents. |
| **P5** | SQ-6: Price of Hidden Information | Useful for framing the contribution but methodologically straightforward once SQ-5 is complete. Train oracle agents, measure the gap. Deferred to the analysis phase (weeks 6–7). |

**Critical path:** SQ-1 → {SQ-2, SQ-5} in parallel → {SQ-3, SQ-4, SQ-6} as analysis of SQ-5 outputs.

---

## Risks

### R1: Formalization Ambiguity (Severity: High, Likelihood: Medium)
The show's rules may not fully specify edge cases (tie-breaking, placement order within rounds, whether tiles are drawn from a shared pool or individual pools). Incorrect assumptions here propagate through all downstream analysis. **Mitigation:** Watch the relevant episode(s) carefully, cross-reference fan wikis and discussion threads, and explicitly document every rule assumption with a sensitivity analysis for ambiguous rules.

### R2: Simplified Variant May Not Transfer (Severity: Medium, Likelihood: High)
Equilibrium results for 3×3 / 3 players / 2 types may not generalize to 5×5 / 9 players / N types. The simplified variant could be *too* simple — for instance, on a 3×3 grid, every cell is adjacent to most others, collapsing the spatial structure. **Mitigation:** Choose the simplified variant carefully (maybe 4×4 / 4 players is a better middle ground). State transfer limitations explicitly. Use the simplified results as hypotheses to test empirically on the full game via SQ-5.

### R3: RL Training Instability in 9-Player Partial-Observability Games (Severity: High, Likelihood: Medium)
MAPPO with 9 agents and hidden types is at the edge of current multi-agent RL scalability. Training may fail to converge, produce degenerate strategies (all agents learn the same policy regardless of condition), or be sensitive to hyperparameters. **Mitigation:** Start with 3-player training to validate the pipeline, then scale incrementally. Use population-based training (PBT) for hyperparameter search. If 9-player training is intractable, fall back to 5–6 players and acknowledge the scope limitation.

### R4: Novelty Claim Challenged by Adjacent Work (Severity: Medium, Likelihood: Low)
While no existing work addresses Zoo specifically, related work on spite bidding in auctions (Morgan et al. 2003), congestion games with incomplete information, or spatial Bayesian games may be closer than the literature review suggests. A reviewer could argue the contribution is incremental over existing frameworks. **Mitigation:** Conduct a thorough literature search covering auction theory (spite bidding), congestion games, Bayesian persuasion on networks, and spatial voting/placement games. Clearly articulate what's structurally new (the combination, not any single component).

### R5: Condition Design Space Explosion (Severity: Medium, Likelihood: Medium)
The set of possible "hidden personal conditions" is a design choice that dramatically affects results. If conditions are too similar, the game reduces to a standard reverse auction. If too diverse, equilibrium analysis becomes intractable per-type. The TV show uses a specific set of conditions, but for research generality, you need a principled condition taxonomy. **Mitigation:** Start with the show's actual conditions, then define a parametric condition family (e.g., linear combinations of row sums, column sums, and adjacency counts) to enable systematic variation.

### R6: Compute Budget Constraints (Severity: Low, Likelihood: Low)
The stated budget (24–48 GPU-hours, single RTX-class GPU) is tight for 9-agent self-play with population-based training. If the observation/action spaces are large (25 grid positions × tile values), training may require more compute than budgeted. **Mitigation:** Use compact state representations (one-hot grid encoding, not raw pixels). Start with small-scale experiments to estimate per-run cost before committing to full sweeps. If needed, reduce to 5 players or fewer training seeds.