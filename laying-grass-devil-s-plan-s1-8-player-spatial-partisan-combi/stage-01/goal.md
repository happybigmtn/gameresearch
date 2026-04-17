[thinking] The user is asking me to create a research goal document about a specific game theory topic - "Laying Grass" from Devil's Plan Season 1. This is a legitimate research planning request. Let me think about this carefully and generate a well-structured research goal.
           
           Let me analyze the game first:
           - 8-player spatial partisan game on 30x30 grid
           - Inverted adjacency rules: own tiles must be adjacent, enemy tiles must be separate
           - Power cells: steal, stone, tile-swap
           - This is a combinatorial game theory problem for spatial territory games
           
           This is actually a fascinating and genuinely novel research topic. 
           
           Most combinatorial game theory work focuses on two-player games, so extending the standard analytical tools like Sprague-Grundy theory to an eight-player spatial game with these inverted adjacency constraints is genuinely underexplored territory. The power cells add another layer of c...
# Research Goal: Laying Grass — Connectivity-Constrained Spatial Partisan Games

## Topic

Combinatorial game theory and multi-agent reinforcement learning for **connectivity-constrained spatial partisan games**, using "Laying Grass" (Devil's Plan S1) as the canonical instance: an 8-player territorial game on a 30×30 grid where players must maintain contiguous own-territory while enforcing separation from opponents, with asymmetric power cells (steal, stone, tile-swap) introducing controlled disruption.

## Novel Angle

**Why this is underexplored:**

Standard combinatorial game theory (CGT) — Sprague-Grundy values, thermography, surreal number valuations — is built almost entirely on the 2-player partisan/impartial framework. Multi-player CGT exists (Cincotti's work on n-player combinatorial games, Li's coalition analysis) but remains sparse and largely theoretical. The spatial-territorial dimension makes this harder: classical CGT tools decompose games into independent sub-positions, but the **global connectivity constraint** (your tiles must form a contiguous region) defeats local decomposition. Each move's value depends on the entire board topology, not just a local neighborhood.

**What creates the opportunity now:**

1. **Graph Neural Networks for board evaluation** — recent advances in GNN-based game state evaluation (AlphaZero descendants, PolyMatrix games) make it feasible to learn value functions over irregular spatial graphs, which is exactly what connectivity-constrained territory produces.
2. **Multi-agent RL maturation** — algorithms like MAPPO, QMIX, and mean-field MARL have matured enough for 4–8 agent settings, but have been applied primarily to cooperative or simple competitive tasks (StarCraft micro, traffic), not to tight spatial-partisan games with combinatorial structure.
3. **Power cell mechanics as controlled disruption** — the steal/stone/swap cells create a game where the combinatorial structure is periodically perturbed. This is analogous to "stochastic CGT" (Calistrate, Albert & Nowakowski's probabilistic Nim) but in a spatial, multi-player setting — a combination that has no existing treatment.

**How this differs from standard approaches:**

- **Not a Go/Hex variant study** — In Go, groups can exist anywhere; connectivity is local and optional (ladders, life-and-death). Here, connectivity is a hard global constraint on every move. In Hex, connectivity is the win condition, not a per-turn constraint. The "inverted adjacency" rule (own-adjacent, enemy-separated) creates dynamics closer to competitive graph coloring than to territory-counting games.
- **Not a standard MARL benchmark** — Existing MARL benchmarks (Hanabi, Overcooked, SMAC) test cooperation or simple competition. Laying Grass is a free-for-all with shifting alliances implicit in spatial proximity, which is structurally different.
- **Not pure CGT theory** — We combine formal game-theoretic analysis (temperature of positions, zugzwang detection) with learned policies, bridging the CGT/RL gap.

## Scope

Formalize Laying Grass as a connectivity-constrained spatial partisan game. Analyze game-theoretic properties (temperature bounds, first-mover advantage, phase transitions between territorial expansion and defensive consolidation) on reduced boards (8×8 to 15×15, 2–4 players). Train multi-agent RL policies using GNN-based state representations and evaluate whether learned strategies discover game-theoretic structure (e.g., do agents learn to play "hot" positions first, consistent with CGT temperature theory?).

## SMART Goal

**Specific:** Implement the Laying Grass ruleset as a PettingZoo-compatible environment, formally characterize the game's CGT properties on small instances (8×8, 2-player), and train MAPPO/IPPO agents on 15×15 4-player boards using GNN state encoders. Analyze whether learned policies correlate with CGT-derived positional values.

**Measurable:**
- Environment correctness validated against hand-traced game logs
- CGT analysis: compute exact game values for all 8×8 2-player endgame positions with ≤12 empty cells
- RL performance: trained agents achieve ≥70% win rate vs. heuristic baselines (greedy-area, greedy-connectivity, random)
- Correlation metric: Spearman ρ ≥ 0.4 between CGT temperature estimates and agent move preferences

**Achievable:** Single GPU (RTX 3090/4090 class), 8×8 exact analysis tractable via alpha-beta with connectivity pruning, 15×15 RL training in ~20 hours of self-play.

**Relevant:** Advances both CGT (first spatial multi-player connectivity game analysis) and MARL (novel structured environment with formal game-theoretic ground truth for interpretability).

**Time-bound:** 8 weeks — 2 weeks environment + formalization, 2 weeks CGT analysis, 3 weeks RL training + analysis, 1 week writeup.

## Constraints

- **Compute:** Single GPU, ≤48 GPU-hours total for RL training
- **Software:** Rust game engine (already in this repo), Python RL stack (PettingZoo, Stable-Baselines3 or CleanRL, PyTorch Geometric for GNNs)
- **Data:** Self-generated via self-play; no external dataset needed. Optional: hand-transcribe 2–3 Devil's Plan S1 episodes for qualitative validation
- **Scope limit:** Full 30×30 8-player is computationally prohibitive for exact CGT analysis — use reduced boards for theory, larger boards for RL only

## Benchmark

- **Name:** No existing benchmark — this is a novel game formalization. We define our own evaluation protocol.
- **Evaluation environment:** Laying Grass (custom PettingZoo env), board sizes 8×8 (exact), 10×10 (heuristic), 15×15 (RL)
- **Metrics:**
  - *CGT:* Number of positions solved exactly, temperature distribution statistics, first-mover advantage measurement (% of 2p games won by P1 under optimal play on 8×8)
  - *RL:* Win rate vs. baselines (random, greedy-area, greedy-connectivity), Elo rating from round-robin tournament, sample efficiency (games to reach 60% win rate)
  - *Interpretability:* Spearman ρ between CGT temperature and agent move ranking, percentage of agent moves that match CGT-optimal in solved positions
- **Current SOTA:** None — no prior computational study of this game exists. Nearest reference points: AlphaZero-style approaches to Go (but 2-player, no connectivity constraint) and MARL results on SMAC/Overcooked (but no spatial-partisan structure).

## Success Criteria

A publishable result requires **at least two** of:

1. **Novel CGT characterization:** Prove or empirically demonstrate a structural property of connectivity-constrained territorial games (e.g., temperature bounds as a function of board fill ratio, phase transition point between expansion-dominant and defense-dominant play, or a decomposition theorem for positions with separated components)
2. **RL agents that exhibit CGT-consistent behavior:** Trained agents preferentially play high-temperature moves and avoid zugzwang positions at rates significantly above baseline, demonstrating emergent game-theoretic reasoning
3. **Power cell impact analysis:** Quantify how steal/stone/swap cells shift game-theoretic values and whether RL agents learn qualitatively different strategies with vs. without power cells (e.g., measured by policy divergence or territory distribution shift)
4. **Scalability insight:** Demonstrate that GNN-based state representations enable meaningful policy transfer from smaller to larger boards (≥10% win rate improvement over training from scratch on the larger board)

**Target venue:** AAAI, IJCAI, or CoG (IEEE Conference on Games) — all accept work at the CGT/RL intersection.

## Generated

2026-04-17T00:00:00Z