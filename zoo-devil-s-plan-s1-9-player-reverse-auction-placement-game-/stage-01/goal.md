[thinking] The user is asking me to create a research goal document about "Zoo" from Devil's Plan Season 1 — a specific game from a Korean reality TV show. This is a legitimate research planning request about game theory applied to a specific game mechanic. Let me think about this carefully and generate a well-structured research goal.
           
           Let me think about what Zoo actually is:
           - It's a game from the Korean Netflix show "Devil's Plan" Season 1
           - 9 players place bids on a 5x5 grid
           - It's a reverse auction — lower totals win
           - Players have hidden personal conditions (secret objectives/constraints)
           
           
           - There's a "dumping" mechanic where players try to place low values
           
           This combines several distinct areas of game theory that don't usually intersect: reverse auction mechanics from economics, combinatorial grid placement from game theory, hidden information dynamics, and the strategic dumping equilibr...
# Research Goal: Zoo (Devil's Plan S1)

## Topic

**Inverse auction theory and combinatorial grid placement with hidden heterogeneous objectives and strategic dumping in multiplayer settings.** Zoo is a 9-player game played on a 5×5 grid where players simultaneously place numbered tiles. The twist: each player has a hidden personal condition (a secret scoring rule), the lowest aggregate total wins, and players can strategically "dump" — place high-value tiles to sabotage opponents' grid regions. This creates a game with incomplete information, combinatorial action spaces, and misaligned incentive structures that don't map cleanly onto standard auction or placement game frameworks.

## Novel Angle

**Why this is underexplored:** Standard reverse auction theory (procurement auctions, Vickrey-style mechanisms) assumes homogeneous goods and known valuation structures. Combinatorial auction theory handles heterogeneous bundles but typically with a single auctioneer. Grid placement games (hex, combinatorial game theory) assume complete information and zero-sum payoffs. Zoo breaks all three assumptions simultaneously:

1. **Hidden heterogeneous objectives** — Each player's personal condition creates a different induced utility over grid positions. Unlike Bayesian games with common priors over types, the conditions in Zoo are drawn from a finite but diverse set of spatial/numerical predicates (e.g., "your row must sum to X," "avoid adjacency to tiles > Y"). This means players face a *type inference* problem with a structured type space, not just a scalar valuation.

2. **Dumping as a strategic externality** — The "lower total wins" mechanic creates an inverse incentive: you want low values for yourself but can place high values to inflate opponents' totals. This is structurally different from standard auction overbidding or combinatorial blocking. It resembles "spite bidding" in auction theory but applied to a spatial domain where the externality depends on grid topology. The interaction between dumping and hidden conditions — where you don't know *which* cells matter to which opponents — creates a novel information-action coupling.

3. **Timeliness** — Recent work (2024–2025) on large-action-space games has advanced MCTS and policy gradient methods for games like Stratego and Diplomacy, but these are adversarial or cooperative. Zoo occupies an underexplored middle ground: competitive but non-zero-sum, with simultaneous moves and a spatial action space. The trend toward studying "social deduction" and hidden-role games with RL (e.g., Avalon, Werewolf, Hanabi) has not yet reached reverse-auction-placement hybrids. Additionally, mechanism design research has recently revisited spite and externality-aware bidding, but not in combinatorial spatial settings.

**What's specifically new:** Characterizing the *dumping equilibrium structure* — under what hidden-condition distributions does a pure dumping strategy dominate? When do mixed strategies emerge? How does the grid topology (5×5, adjacency structure) shape equilibrium compared to an unstructured action space? No existing work addresses this specific combination.

## Scope

Formalize Zoo as a Bayesian simultaneous-move game on a grid. Analyze dumping equilibria under a tractable subset of hidden conditions. Train RL agents via self-play and compare emergent strategies against game-theoretic predictions. Specifically:

- Formalize the game (state space, action space, hidden types, payoff function)
- Derive analytical results for simplified variants (e.g., 3×3 grid, 3 players, 2 condition types)
- Run self-play RL on the full 5×5, 9-player game
- Classify emergent strategy profiles (pure dump, conditional dump, cooperative equilibria)
- Measure the *price of hidden information* — performance gap between complete and incomplete information agents

## SMART Goal

**Specific:** Develop a formal game-theoretic model of Zoo, derive dumping equilibrium conditions for simplified variants, and train self-play RL agents on the full game to characterize emergent strategy profiles under hidden heterogeneous objectives.

**Measurable:** (1) Formal model with proven equilibrium existence for the simplified variant, (2) RL agents achieving >60% win rate against heuristic baselines (random, greedy-minimize, pure-dump), (3) taxonomy of ≥3 distinct emergent strategy profiles with statistical significance (p<0.05 across 1000+ games).

**Achievable:** The simplified variant (3×3, 3 players) is analytically tractable. The full game has ~25 positions × ~N tiles per player per round — large but within reach of PPO/MAPPO with parameter sharing on a single GPU. The game can be fully simulated (no external data dependency).

**Relevant:** Advances understanding of strategic behavior in multiplayer incomplete-information games with spatial externalities — relevant to mechanism design, multi-agent RL, and computational social choice.

**Time-bound:** 8 weeks. Weeks 1–2: formalization and simplified analysis. Weeks 3–5: environment implementation and RL training. Weeks 6–7: analysis of emergent strategies. Week 8: writeup.

## Constraints

- **Compute:** Single GPU (RTX-class), training budget ~24–48 GPU-hours total
- **Data:** No external dataset required — game is fully simulatable from rules
- **Tools:** Python, PettingZoo or custom multi-agent env, PyTorch, RLlib or CleanRL for MAPPO
- **Scope limit:** Analysis of simplified variants is analytical; full-game analysis is empirical via RL. No attempt at full equilibrium computation for the 9-player game (intractable)

## Benchmark

- **Name:** No standard benchmark exists. This is a novel game formalization.
- **Evaluation method:** Custom simulation environment implementing Zoo rules. Metrics measured via round-robin tournaments between trained agents and baselines.
- **Metrics:**
  - Win rate against baselines (random, greedy-minimize, pure-dump, oracle-with-info)
  - Exploitability approximation (NashConv) for simplified variants
  - Strategy profile entropy (diversity of emergent play)
  - Price of hidden information (win rate gap: complete-info agent vs. incomplete-info agent)
- **Current SOTA:** None — this game has not been formally studied. The TV show provides qualitative observations of human play but no quantitative analysis.
- **Baselines to establish:** Random placement, greedy self-minimization (ignore opponents), pure dumping (maximize opponents' totals), and Bayesian best-response with uniform prior over opponent conditions.

## Success Criteria

A publishable result would demonstrate at least two of:

1. **Analytical:** A formal proof characterizing dumping equilibria in the simplified variant — specifically, conditions on the hidden-type distribution under which dumping is (or is not) a Nash equilibrium strategy, with a clean separation result.
2. **Empirical:** RL agents that discover non-obvious strategies not predicted by naive game-theoretic analysis (e.g., conditional cooperation, position-dependent dumping thresholds, type-signaling through early placements).
3. **Structural:** Evidence that grid topology materially affects equilibrium structure — showing that the same payoff rules on a non-spatial (set-based) action space produce qualitatively different equilibria, establishing that the spatial component is load-bearing.

**Target venue:** Workshop paper at a game theory / multi-agent systems venue (AAMAS, GAIW, or GameSec workshop track), or a short paper at a computational intelligence venue.

## Generated

2026-04-17T00:00:00Z