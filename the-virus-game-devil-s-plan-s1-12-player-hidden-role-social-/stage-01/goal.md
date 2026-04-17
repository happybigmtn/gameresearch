[thinking] The user is asking me to create a research goal document about "The Virus Game" from Devil's Plan S1 — a specific social deduction game with interesting game-theoretic properties. This is a legitimate research planning request. Let me think about this carefully and produce a high-quality research plan.
           
           Let me first understand the game:
           - The Virus Game from Devil's Plan Season 1
           - 12 players in a circular ring arrangement
           - Hidden roles with 3 asymmetric factions
           - Unique IDs 1-12
           - Bounded public signals
           - The research focus is on optimal strategies and equilibrium analysis
           
           
           - The 3-faction structure with asymmetric roles is relatively understudied compared to standard 2-faction hidden-role games, and the circular topology with constrained information flow creates a unique strategic environment worth analyzing.
           - I can model this computationally using CFR algorithms on a manageable...
# Research Goal: Equilibrium Analysis of The Virus Game

## Topic

**Optimal strategies and equilibrium analysis for asymmetric hidden-role social deduction games with topological constraints** — specifically, The Virus Game (Devil's Plan S1), a 12-player game with 3 asymmetric factions, unique ordered IDs on a circular ring, and bounded public signaling.

## Novel Angle

**Why this is underexplored:** The dominant body of work on hidden-role game equilibria focuses on **two-faction symmetric** settings — Mafia/Werewolf variants and The Resistance/Avalon. These games have binary faction structure (informed minority vs. uninformed majority) and treat players as topologically interchangeable. Three key gaps exist:

1. **Three-faction non-zero-sum dynamics.** With 3 asymmetric factions, pairwise incentives shift across game phases — two factions may temporarily align against a third, then defect. Standard Bayesian deduction models for 2-faction games don't capture these coalition-switching equilibria. Recent work on multi-team games in partially observable settings (emerging from multi-agent RL research, 2023-2025) opens the door, but has not been applied to structured social deduction with discrete hidden roles.

2. **Ring topology as an information constraint.** The circular seating with ordered IDs (1-12) creates **locality-dependent inference** — a player's neighbors carry more signal than distant players, and the ID ordering interacts with faction assignment probabilities. This is structurally different from the fully-connected communication graphs assumed in Mafia/Avalon analysis. The topology-information interaction is well-studied in consensus and voting problems, but not in hidden-role deduction games.

3. **Bounded public signaling under asymmetric information.** Players can make public claims (bounded signals), but their credibility depends on hidden faction identity. This creates a **cheap talk game with verifiable structure** — the ID ring constrains which lies are consistent. The interplay between topological constraints and strategic misrepresentation has not been formally analyzed for this class of game.

**Why now:** The convergence of (a) scalable equilibrium-finding algorithms (CFR variants, magnetic mirror descent for extensive-form games) that handle 10^6-10^8 information sets, (b) growing interest in multi-agent social deduction as an RL benchmark (2024-2025 saw several Avalon/Werewolf RL papers), and (c) the fact that Devil's Plan popularized a game with richer structure than existing benchmarks — creates a timely opportunity to formalize and solve a game that sits in a genuine complexity gap between toy models and intractable general social deduction.

**How this differs from standard approaches:** Rather than training RL agents to play a known game, we **formalize the game as an extensive-form game with 3-faction asymmetry and ring-constrained signaling**, derive structural properties of its equilibria analytically (exploiting ring symmetry to reduce the strategy space), and then compute approximate Nash equilibria for the reduced game. The contribution is the formal model and structural analysis, not just empirical agent performance.

## Scope

A single paper that:
1. Formalizes The Virus Game as an extensive-form game with imperfect information
2. Proves structural properties of equilibria exploiting circular symmetry (rotational equivalence classes of faction assignments)
3. Computes approximate equilibria for the full 12-player game using CFR+ on the symmetry-reduced game tree
4. Characterizes how optimal signaling strategies differ across the 3 factions and depend on ring position

Out of scope: full RL training pipelines, human subject experiments, generalization to arbitrary n-player games.

## SMART Goal

**Specific:** Formalize The Virus Game (12 players, 3 factions, circular ring, bounded public signals) as an extensive-form game; exploit rotational symmetry to reduce the information-set space; compute epsilon-Nash equilibria (epsilon < 0.05) using CFR+; characterize faction-asymmetric signaling strategies.

**Measurable:** (1) Formal game model with complete specification, (2) proven symmetry reduction factor (target: >=12x from rotational equivalence), (3) computed equilibrium strategy profiles with exploitability < 0.05, (4) quantified information value of ring position across factions.

**Achievable:** 12-player game with bounded signal space is within CFR+ tractability after symmetry reduction. Similar-scale games (6-10 player poker variants, Avalon) have been solved. Single GPU (A100-class) sufficient for days-scale CFR+ computation on the reduced game.

**Relevant:** Directly advances the frontier of computational game theory for social deduction games, addressing a genuine structural gap (3-faction, topology-constrained) in the literature.

**Time-bound:** 8 weeks — 2 weeks formalization + symmetry analysis, 2 weeks implementation, 2 weeks computation + analysis, 2 weeks writeup.

## Constraints

| Resource | Budget |
|----------|--------|
| Compute | Single GPU (A100 or equivalent), ~100 GPU-hours total |
| Time | 8 weeks |
| Tools | OpenSpiel (extensive-form game framework), custom game implementation in C++/Python, CFR+/DCFR solver |
| Data | No external data needed — game is fully specified; evaluation is against computed exploitability |

## Benchmark

**Name:** No standard benchmark exists for this game class. We define a custom evaluation framework.

**Evaluation metrics:**
- **Exploitability** (epsilon): How much a best-response deviates from the computed equilibrium. Target: epsilon < 0.05 (normalized to game value range [0,1]).
- **Symmetry reduction factor**: Ratio of full information sets to symmetry-reduced information sets. Validates the structural contribution.
- **Faction-specific expected value**: Equilibrium payoff for each of the 3 factions — characterizes the game's inherent asymmetry.
- **Signaling strategy entropy**: Measures how much each faction diversifies its public signals at equilibrium (pure vs. mixed strategies).

**Current SOTA:** No prior computational equilibrium results exist for this specific game. Closest reference points are Avalon equilibrium approximations (2-faction, 5-10 players, no topology) and Werewolf RL agents (learned policies, not equilibria). This work would establish the first baseline.

**Comparison baselines:** (1) Uniform random play, (2) heuristic strategies (always-honest, always-claim-majority-faction), (3) 2-faction reduction (ignore third faction) to measure the strategic impact of the 3-faction structure.

## Success Criteria

Publishable if the paper delivers:
1. A clean formal model that captures the structural novelty (3 factions, ring topology, bounded signals) — reviewable as a standalone contribution
2. Non-trivial symmetry reduction with proof — demonstrates the ring structure is analytically useful, not just thematic
3. Computed equilibria showing **qualitatively different** signaling behavior across factions (e.g., one faction pools signals, another separates) — the game-theoretic insight
4. Exploitability below 0.05, demonstrating the equilibria are meaningful approximations

Target venues: AAAI, AAMAS, NeurIPS (game theory track), or Games and Economic Behavior.

## Generated

2026-04-17T00:00:00Z