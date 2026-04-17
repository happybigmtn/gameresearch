[thinking] The user is asking me to create a research plan about "Wall Go" - a Go variant with wall placement from Devil's Plan Season 2. This is a legitimate research planning task. Let me first understand the context by looking at what exists in the project.
           
           Let me check the project structure to understand the research-cli and any existing work on this topic.
Let me first understand the project context and any existing work on this topic.

Good, there's existing work. Let me read the Wall Go spec and the existing game implementation.

[thinking] Now I have a solid understanding of:
           1. The Wall Go game mechanics (from the spec and implementation)
           2. The project's research program format (from bagchal.md)
           3. The existing codebase structure
           
           Let me also check the research report for Devil's Plan S2 to get more context on Wall Go's origins, and look at what existing research/literature cache exists.
Let me check the research report and existing literature cache for context.

`★ Insight ─────────────────────────────────────`
- Wall Go is a **perfect-information abstract strategy game** — no hidden state, no randomness. This makes it amenable to classical game tree search (minimax, alpha-beta, MCTS) but also opens interesting questions about RL in games with variable branching factor from the wall-placement mechanic.
- The game has a unique **coupled move+wall structure**: every turn is a two-phase atomic action (move pawn, then place wall). This creates a compound action space where the wall placement after each move is mandatory and constrained to the moved pawn's neighborhood — a feature not seen in standard Go or most studied board games.
- The **3→2 player cascade** (R1 with 3 players, R1 winner locked, R2 with 2) creates a game-theoretic wrinkle: in R1, the optimal strategy isn't just "maximize territory" — it's also about positioning for R2 if you lose R1.
`─────────────────────────────────────────────────`

Now let me look at the existing bot structure to understand what's already been built, and check for any existing autoresearch programs for Wall Go.

[thinking] Now I have a complete picture. Let me also check the GDD for any additional context about the Olympiad project's goals.
Now I have a thorough understanding of the project context. Let me create the SMART research goal.

`★ Insight ─────────────────────────────────────`
- **Wall Go is genuinely novel in academic literature.** It's a TV show game variant, not a studied game. Standard Go has massive literature; Quoridor (wall placement on a grid) has some. But Wall Go's specific mechanic — move pawn THEN mandatory wall adjacent to moved pawn, on a 7×7 grid, with a one-shot wall-break — is unstudied.
- **The compound action space is the key research angle.** In Quoridor, you either move OR place a wall. In Wall Go, you ALWAYS do both (move then wall), making every turn a 2-phase decision. This couples positional play with board modification in a way that isn't captured by existing abstractions in combinatorial game theory.
- **The 3→2 player cascade creates a non-trivial multi-agent problem.** Most abstract strategy game research assumes 2 players. The R1→R2 transition where the R1 winner is "locked safe" creates incentives to either dominate territory (win R1) or strategically position for R2 — a form of meta-game planning rarely studied in CGT.
`─────────────────────────────────────────────────`

Here's the research goal:

---

# Research Goal: Wall Go (Devil's Plan S2) — Coupled Move-Wall Action Spaces in Abstract Strategy Games

## Topic

Wall Go (벽바둑) is a perfect-information abstract strategy game from *The Devil's Plan* Season 2 (Netflix, 2024). Players move pawns on a 7×7 grid and mandatorily place walls after each move, with the game ending when all territories are fully separated by walls. The game combines positional maneuvering with incremental board modification, creating a coupled two-phase action space that differs from standard abstract strategy games.

## Novel Angle

**Why this is unstudied:** Wall Go is a recently invented game (2024 TV broadcast) with no existing academic treatment. The closest studied games are:

- **Standard Go (Baduk):** Territory-based, but stones are placed (not moved) and the board graph is static. AlphaGo/AlphaZero literature is extensive but irrelevant to the move+wall mechanic.
- **Quoridor:** Features wall placement on a grid, but turns are EITHER move OR wall — never both. The coupling is absent. Some MCTS/minimax work exists on Quoridor but doesn't address mandatory post-move wall placement.
- **Blockade / Twixt / other connection games:** Wall-like mechanics exist but serve different strategic roles (blocking opponent paths vs. carving territory).

**What makes this timely:**
1. The game was broadcast in 2024, creating public interest but zero formal analysis.
2. Recent advances in MCTS with policy networks (MuZero, EfficientZero) have shown strong results on games with complex action spaces, but haven't been applied to games where the action space is a *product* of move × wall-placement.
3. This project already has a working Rust implementation of the full game rules (`crates/olympiad/games/wall-go/src/lib.rs`), providing a verified environment for experimentation.

**The specific gap:** How does the mandatory wall-placement coupling affect the game tree structure, branching factor dynamics, and effectiveness of standard search algorithms? In Quoridor, the move-or-wall disjunction makes action enumeration straightforward. In Wall Go, every action is a (move, wall) pair where the wall must touch the destination — this creates a branching factor that varies dramatically with board state and demands different search strategies than either pure-movement or pure-placement games.

**How this differs from standard approaches:** Rather than applying AlphaZero-style RL to yet another board game, the research focuses on the *structural properties* of coupled action spaces — a class of games poorly represented in the CGT/RL literature. The findings would generalize to any game where turns consist of mandatory multi-phase compound actions.

## Scope

Single paper analyzing Wall Go's game-theoretic properties and bot performance:

1. **Formal characterization** of the Wall Go state space: branching factor distribution, game length distribution, and how wall density affects both.
2. **Comparative bot benchmarks:** Random baseline → greedy heuristic → depth-limited alpha-beta → MCTS, measuring how each algorithm handles the coupled action space.
3. **Action-space factorization study:** Compare "joint" search (enumerate all move×wall pairs) vs. "sequential" search (choose move first, then choose wall) — does factoring the compound action improve or degrade play quality?

## SMART Goal

**Specific:** Characterize the game-theoretic properties of Wall Go's coupled move-wall action space, and build a bot that achieves ≥0.80 win-rate against a random baseline over 200 matches, using action-space factorization to reduce the effective branching factor.

**Measurable:**
- Branching factor statistics (mean, median, p95) at different game stages
- Win-rate of each bot tier (greedy, alpha-beta, MCTS) vs. random baseline
- Win-rate of factored vs. joint action enumeration at equivalent search budgets
- Game-length distribution and territory-balance metrics

**Achievable:** The game engine is already implemented in Rust with a `GamePlugin` trait, `enumerate_legal_moves` support, and a `SessionHarness` for testing. The 7×7 board keeps the state space manageable (49 cells, ~96 possible wall edges). A single GPU is not needed — all algorithms are CPU-based tree search on a small board.

**Relevant:** Advances understanding of multi-phase compound action spaces in combinatorial games — a class that includes many real-world games but is underrepresented in academic game AI. Directly supports the project's Olympiad bot pipeline.

**Time-bound:** 4 weeks to completion:
- Week 1: State-space characterization (branching factor analysis, game-length Monte Carlo)
- Week 2: Greedy + alpha-beta bots with evaluation function design
- Week 3: MCTS implementation + factored vs. joint action comparison
- Week 4: Analysis, ablation studies, writeup

## Constraints

- **Compute:** Single machine, CPU-only. No GPU required — game tree search on 7×7 grid.
- **Available tools:** Existing Rust game engine (`olympiad_game_wall_go` crate), bot framework (`olympiad_bots::Bot` trait), session harness for benchmarking, `olympiad_autoresearch` for automated iteration.
- **Data access:** No external dataset needed — all data is generated through self-play on the existing engine.
- **Dependencies:** No new crates required beyond what the project already uses.

## Benchmark

- **Name:** Wall Go Bot Benchmark (custom, project-internal)
- **Source:** Self-play via `SessionHarness` with `WallGoPlugin`, matching the pattern in `crates/olympiad/autoresearch/programs/big-small.md`
- **Metrics:**
  - Primary: Win-rate vs. random baseline over 200 matches (both as P1 and P2 seat positions)
  - Secondary: Average game length (turns), average territory differential at settlement, branching factor per turn
- **Current SOTA:** No bots exist yet for Wall Go in this project. The random baseline is the floor. The Bagchal greedy bot achieves 1.000 vs. random (per recent commit `a5d61b05d`); the Nine Men's Morris greedy bot achieves 0.885 (per commit `724cbeb2a`). A comparable ≥0.80 target for Wall Go's first iteration is realistic.
- **Evaluation protocol:** Each benchmark run plays 200 matches with alternating seat assignments. Bot must handle both 3-player (R1) and 2-player (R2) configurations. Win-rate computed as `wins / total_matches`.

## Success Criteria

Results would be publishable (workshop paper or short paper at AAAI/CoG/AIIDE) if:

1. **Novel game characterization:** First formal analysis of Wall Go's state space properties (branching factor, game tree depth, territory dynamics) — filling a gap since no academic work on this game exists.
2. **Actionable algorithmic finding:** Demonstrate that action-space factorization (sequential move→wall selection) achieves comparable or superior win-rates to joint enumeration at 2-4× lower node counts — this would generalize to other compound-action games.
3. **Strong bot performance:** ≥0.80 win-rate vs. random with greedy heuristics; ≥0.90 with depth-limited search or MCTS — showing the game is tractable for classical methods despite the coupled action space.
4. **3-player analysis:** Characterize how the 3→2 player cascade affects optimal R1 strategy — does maximizing R1 territory correlate with overall game wins, or is R2 positioning sometimes preferable?

## Generated

2026-04-17T00:00:00Z