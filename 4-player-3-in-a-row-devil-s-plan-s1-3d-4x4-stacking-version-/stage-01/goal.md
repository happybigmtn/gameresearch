[thinking] The user is asking me to create a research goal document about a specific game theory topic. This is a legitimate research planning request - they want me to generate a structured markdown document about 4-Player 3-in-a-Row (from Devil's Plan S1), focusing on the 3D 4x4 stacking version.
           
           This is a text generation task that doesn't require tools. Let me write a thorough research plan.
           
           Let me think about what makes this topic interesting:
           
           1. Devil's Plan is a Korean reality show where contestants play various strategy games. Season 1 featured a 3D tic-tac-toe variant (4x4x4) with 4 players.
           
           
           2. The core research opportunity here is that N-player combinatorial game theory is vastly understudied compared to traditional 2-player games, and the 4-player 3D variant specifically hasn't been solved despite the 2-player version being well-understood. The "no communication" constraint adds a...
# Research Goal: N-Player Minimax in 3D Stacking Tic-Tac-Toe

## Topic

Multi-player combinatorial game theory applied to 4-player 3D 4×4×4 stacking tic-tac-toe (3-in-a-row win condition), as featured in Devil's Plan Season 1. Intersection of N-player search algorithms, emergent coordination without communication, and computational game solving under gravity constraints.

## Novel Angle

**Why this is underexplored:** Two-player 4×4×4 tic-tac-toe (Qubic) was solved in 1980 — first player wins with perfect play. But **the 4-player variant with a stacking/gravity constraint and 3-in-a-row win condition is a fundamentally different game** that has not been solved or deeply analyzed. Three specific gaps motivate this work:

1. **N-player search algorithm comparison in constrained 3D topology.** The three dominant N-player search strategies — maxⁿ (each player maximizes own utility), paranoid (treat all opponents as a coalition), and best-reply search (only the strongest opponent plays optimally, others play randomly) — have been studied primarily in card games and classic board games. Their behavior in a **stacking game with spatial structure** is unexplored. The gravity constraint dramatically prunes the action space (≤16 legal moves vs. ≤64 in free-placement), which may invert the usual paranoid-dominates-maxⁿ finding from games with larger branching factors.

2. **Emergent implicit coordination without communication.** In the Devil's Plan format, players cannot communicate. Yet rational play in a 4-player free-for-all creates implicit coordination dynamics: blocking a near-winner benefits all non-winners, but *who* blocks is a public-goods problem. This "alliance-free coordination" regime sits between fully cooperative and fully adversarial game theory and is timely given recent interest in multi-agent coordination without explicit communication (driven by multi-agent RL research).

3. **Skill-asymmetric equilibria.** The show introduced a chess professional as a guest player. When one player is known to be stronger (deeper search / better heuristic), does it shift the equilibrium toward implicit 3-vs-1 ganging? This connects to the growing body of work on opponent modeling and adaptive search depth in multi-agent settings.

**What creates the opportunity now:** The recent wave of AlphaZero-style self-play methods being extended to N>2 players (e.g., multiplayer poker, Diplomacy, Hanabi research) provides both algorithmic tools and reviewer interest. Yet the combinatorial game theory community has not revisited classic spatial games under these N-player lenses. This game is small enough to be tractable (upper bound ~16^16 ≈ 10^19 states before pruning, but gravity + symmetry reduce this by orders of magnitude) yet complex enough to exhibit non-trivial multi-agent phenomena.

**How this differs from standard approaches:** Standard work either (a) solves 2-player variants exactly, or (b) applies MCTS/RL to large N-player games where exact analysis is impossible. This project occupies the **middle ground**: a game small enough for deep analytical search yet complex enough that N-player dynamics produce genuinely novel strategic phenomena.

## Scope

A single paper characterizing optimal and near-optimal play in 4-player 3D 4×4×4 stacking 3-in-a-row:

- Implement the game engine with legal move generation and win detection
- Implement maxⁿ, paranoid, and best-reply search with iterative deepening and alpha-beta pruning (where applicable)
- Design a position evaluation heuristic (line threats, blocking value, positional control)
- Compare algorithms by: win rate in round-robin tournaments, search depth achieved in fixed time, and quality of play against a baseline (random, 1-ply greedy)
- Analyze emergent blocking/coordination behavior through game trace statistics
- Optionally: test skill-asymmetric scenarios (one player searches deeper)

## SMART Goal

**Specific:** Implement and comparatively evaluate maxⁿ, paranoid, and best-reply search algorithms for 4-player 3D 4×4×4 stacking 3-in-a-row, measuring win rates, search efficiency, and emergent coordination patterns across symmetric and asymmetric skill configurations.

**Measurable:** 
- Win rate matrix across all algorithm pairings (at least 1,000 games per configuration)
- Search depth achieved per algorithm at 1s/move and 5s/move time controls
- Coordination metric: frequency of "blocking moves" (moves that prevent an opponent's immediate win) vs. "advancing moves"
- First-player advantage quantification across algorithms

**Achievable:** Game state fits in <1KB. Branching factor ≤16 with gravity. A well-optimized Rust implementation should reach depth 8-12 in 1 second. No GPU needed — this is a tree search project. Total compute: single modern CPU, ~48 hours of tournament play.

**Relevant:** Directly advances the understanding of N-player minimax variants in spatial/combinatorial games, a gap between solved 2-player games and intractable large-scale N-player domains.

**Time-bound:** 6 weeks — 1 week engine + heuristic, 1 week search algorithms, 1 week tournament infrastructure, 2 weeks experiments, 1 week analysis and writing.

## Benchmark

- **Name:** No standard benchmark exists for this specific game variant. This work establishes the first systematic evaluation.
- **Evaluation method:** Self-play round-robin tournaments between algorithm variants. Metrics:
  - **Win rate** (primary): percentage of games won per algorithm in N-way matchups
  - **Draw rate**: frequency of board-full draws
  - **Search efficiency**: maximum depth reached under fixed time controls (1s, 5s per move)
  - **Coordination index**: ratio of blocking moves to total moves, measured per-player
  - **First-mover advantage**: win rate delta between seat positions (P1/P2/P3/P4)
- **Baselines:** Random play, 1-ply greedy (maximize own threats), 2-ply greedy
- **Existing SOTA:** Qubic (2-player, 4-in-a-row, no gravity) was solved by Patashnik (1980). Connect Four (2-player, gravity, 7×6) solved by Allis (1988). No known solution or systematic study exists for the 4-player, 3-in-a-row, gravity, 4×4×4 variant.

## Constraints

- **Compute:** Single CPU (no GPU required). Tree search is CPU-bound. Budget: ~48-72 hours total tournament compute.
- **Tools:** Rust game engine (already partially built in this repository's `crates/olympiad/` workspace), custom search implementations, Python/matplotlib for analysis plots.
- **Data:** Self-generated through self-play. No external dataset needed.
- **Scope limitation:** Full game solving (proving first-player win/loss/draw) is likely intractable for 4 players without significant retrograde analysis infrastructure. This work characterizes strong play, not perfect play.

## Success Criteria

Publishable if the paper demonstrates **at least two** of:

1. **Algorithm ranking reversal:** Show that the relative strength of maxⁿ vs. paranoid vs. best-reply differs in this stacking game compared to their known rankings in card/abstract games — with a mechanistic explanation tied to the gravity constraint or spatial structure.
2. **Quantified coordination emergence:** Measure statistically significant implicit coordination (e.g., players disproportionately block the leader) emerging from purely self-interested search, with analysis of how this varies by algorithm type.
3. **Skill asymmetry phase transition:** Identify a threshold at which one player's search depth advantage triggers coalition-like behavior from the other three, shifting from "each for themselves" to "implicit 3-vs-1."
4. **Tractability result:** Establish tight bounds on the game's complexity (state space, game tree) and demonstrate that partial solving (e.g., endgame tablebases for ≤K pieces) is feasible.

**Target venue:** AAAI, IJCAI, or CoG (IEEE Conference on Games) workshop/main track.

## Generated

2026-04-17T00:00:00Z

---

`★ Insight ─────────────────────────────────────`
- **Paranoid vs maxⁿ is the core tension in N-player search.** Paranoid search assumes all opponents cooperate against you (conservative, but prunes well with alpha-beta). Maxⁿ assumes each player maximizes independently (realistic, but can't prune as effectively). Which wins depends on the game's branching factor and whether opponents *actually* coordinate — this game is a natural testbed because implicit coordination emerges from the blocking incentive.
- **Gravity/stacking is a huge deal for search.** Free-placement 4×4×4 has branching factor 64 → 63 → 62... Stacking caps it at 16 (one move per column) and creates forced spatial structure. This is why the game is tractable for deep search despite being 4-player.
- **Your existing `crates/olympiad/` workspace already has Nine Men's Morris infrastructure** — the game engine pattern (legal moves, apply/undo, evaluation) transfers directly to this new game, which makes the 1-week engine timeline realistic.
`─────────────────────────────────────────────────`