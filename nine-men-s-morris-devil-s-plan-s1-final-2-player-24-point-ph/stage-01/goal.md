[thinking] The user is asking me to create a research goal document about Nine Men's Morris. This is a legitimate research planning task related to their project. Let me first understand the context by looking at their project structure and existing work.
           
           Let me look at the existing files to understand what they've already done with Nine Men's Morris in their codebase.
Let me first understand what you've already built in this project before proposing a research direction.

Now let me see the existing minimax bot and greedy mill bot, plus the autoresearch harness.

[thinking] Now I have a thorough understanding of the project. Let me analyze:
           
           1. The project has a Nine Men's Morris implementation with:
              - A game plugin (`olympiad_game_nine_mens_morris`) with full rules
              - A random baseline bot
              - A greedy mill bot (depth-1, ~88.5% vs random per commit msg)
              - A minimax alpha-beta bot (depth 2 default)
              - An autoresearch harness for benchmarking
           
           2. The autoresearch program doc outlines a progression: greedy → alpha-beta → iterative deepening + transposition table → endgame tablebase, with neural evaluation deferred to later phases.
           
           
           3. The game is solved (Gasser 1996, perfect play is a draw), but that solution came from retrograde analysis rather than learned representations. What's interesting is that NMM has three distinct phases with very different evaluation dynamics, and the phase transitions are abrupt and asymmetric—one player might be in...
`★ Insight ─────────────────────────────────────`
Your project has a clean progression ladder: random → greedy (0.885) → minimax (depth 2) → [tablebase] → [neural]. The autoresearch harness benchmarks each rung against random. The interesting research gap isn't in any single rung — it's in **how phase structure affects the transition from search-based to learned evaluation**, which NMM's three explicit phases make uniquely measurable.
`─────────────────────────────────────────────────`

Here's the research goal document:

---

# Research Goal: Phase-Conditioned Evaluation Transfer in Nine Men's Morris

## Topic

Phase-aware neural strategy learning in solved combinatorial games. Specifically: how explicit game-phase structure (placement / movement / flying) affects the efficiency of knowledge transfer from classical search to lightweight neural evaluators in Nine Men's Morris.

## Novel Angle

**Gap:** NMM was solved in 1996 by Gasser via retrograde analysis, and optimal-play databases exist (Stahlhacke). Research since then has largely treated NMM as a closed book — a checkbox on the "solved games" list. Meanwhile, the expert iteration / AlphaZero line of work (Silver et al. 2017, Anthony-Tian-Barber 2017) demonstrates self-play learning for large unsolved games, but does not study how **explicit phase transitions** within a game affect learning dynamics. Most board games (chess, Go, Othello) have implicit phases (opening/middlegame/endgame) with fuzzy boundaries. NMM is unusual: phases are hard-coded into the rules with discrete transitions (placement → movement → flying) and qualitatively different legal-move sets.

**What's unexplored:** No published work systematically measures how phase-conditioned evaluation — separate learned evaluators per game phase, or a shared evaluator with phase-embedding inputs — compares against monolithic evaluation when distilling knowledge from a classical minimax oracle. The closest work is phase-aware heuristic design (handcrafted, not learned) and AlphaZero-style self-play (which doesn't leverage solved-game ground truth or phase structure).

**Why timely:** Recent trends in RL efficiency — particularly "offline RL from demonstrations" and "curriculum learning from expert trajectories" — create a natural opportunity: use classical search as an oracle to generate expert trajectories, then study whether phase-conditioned architectures learn faster and generalize better across phase boundaries. NMM's small state space (approximately 10^10 positions, ~7×10^9 reachable) makes this feasible on a single GPU in hours, not days. The 24-point board admits compact neural representations (24×3 one-hot + phase indicator + hand counts = ~80 input features).

**How this differs from standard approaches:** Standard NMM bot research stops at "use minimax + hand-tuned eval" or "link a retrograde database." Standard neural game-playing research targets unsolved games where ground truth is unavailable. This work occupies the intersection: using the solved game's ground truth to rigorously measure whether phase conditioning produces measurable learning efficiency gains — a question that transfers to larger phase-structured games (Stratego, many card games, real-time strategy games with tech trees).

## Scope

Single paper focused on a controlled experiment:
1. Train three neural evaluator architectures (monolithic MLP, phase-conditioned MLP with separate heads, phase-embedding MLP with shared trunk) on expert trajectories generated by the existing minimax bot at increasing depths.
2. Measure: (a) learning speed (samples to reach X% agreement with oracle), (b) per-phase accuracy (does the monolithic model systematically fail at phase boundaries?), (c) playing strength when the learned evaluator replaces the handcrafted one in the existing alpha-beta search framework, (d) optimality gap vs. the known perfect-play solution.
3. Ablation: phase indicator as input feature vs. hard architecture split.

## SMART Goal

**Specific:** Implement three neural evaluator architectures (monolithic, phase-conditioned, phase-embedded) in Rust, train them on minimax-generated expert positions, and measure per-phase accuracy and playing strength within the existing autoresearch benchmark harness.

**Measurable:** (1) Per-phase classification accuracy against the minimax oracle at depth 4+; (2) Win-rate of neural-eval + alpha-beta vs. the handcrafted-eval minimax bot over 1000 matches; (3) Sample efficiency ratio (training positions needed to reach 90% oracle agreement) for each architecture; (4) Phase-boundary accuracy delta (accuracy drop within ±2 moves of a phase transition).

**Achievable:** NMM's state space is small enough for exhaustive position sampling. The existing Rust autoresearch harness provides benchmarking infrastructure. Neural network inference in Rust via `tch-rs` (libtorch bindings) or `candle` (pure Rust) is straightforward for small MLPs. Training data generation reuses the existing minimax bot. Target: 2-layer MLP with ~10K parameters, trainable in minutes on CPU.

**Relevant:** Directly extends the project's bot progression roadmap (step between minimax and full AlphaZero). Answers a transferable question about phase-conditioned learning that applies to any game with explicit phase structure.

**Time-bound:** 4 weeks. Week 1: position generator + training data pipeline. Week 2: three architectures + training loop. Week 3: evaluation harness integration + playing-strength benchmarks. Week 4: analysis, ablations, writeup.

## Constraints

- **Compute:** Single machine, no GPU required (CPU-only training of small MLPs on ~1M positions). GPU optional for faster iteration.
- **Dependencies:** Existing Rust codebase (`olympiad_game_nine_mens_morris`, `olympiad_bots`, `olympiad_autoresearch`). Neural network library: `candle` (pure Rust, no Python dependency) or `tch-rs`.
- **Data:** Self-generated via the existing minimax bot. No external dataset needed. Retrograde table (Stahlhacke) available as optional ground-truth oracle for endgame positions.
- **Language:** Rust (matching the existing codebase). Training loop can optionally use Python for prototyping, but final evaluator must be callable from the Rust bot framework.

## Benchmark

- **Name:** NMM Autoresearch Benchmark (project-internal).
- **Source:** `crates/olympiad/autoresearch/` harness with `programs/nine-mens-morris.md` configuration.
- **Metrics:** (1) Win-rate vs. random baseline over 200+ matches; (2) Win-rate vs. handcrafted minimax (depth 2) over 1000 matches; (3) Per-phase oracle agreement (% of positions where the neural eval's top-ranked move matches the minimax oracle's).
- **Current SOTA in this harness:** Minimax alpha-beta depth 2 with handcrafted evaluator (`mills*200 + stones*100 + potential_mills*10`). Win-rate vs. random: expected ~95%+. The greedy mill bot achieves 0.885 vs. random.
- **External reference:** Gasser (1996) solved the game — perfect play from any position is known. Stahlhacke's retrograde tables provide move-level ground truth for endgame positions. No published neural-evaluator benchmark for NMM exists (which is part of the novelty).

## Success Criteria

A publishable result requires demonstrating at least one of:
1. **Phase-conditioned architectures learn significantly faster** (≥30% fewer training samples to reach 90% oracle agreement) than monolithic architectures — with statistical significance across multiple training seeds.
2. **Measurable phase-boundary accuracy gap** in the monolithic model that the conditioned models avoid — i.e., the monolithic evaluator systematically misjudges positions near phase transitions.
3. **Playing-strength improvement** — the phase-conditioned neural evaluator, when plugged into the alpha-beta framework, achieves a higher win-rate against the handcrafted evaluator than the monolithic neural evaluator does.

A negative result (phase conditioning doesn't help at NMM's scale) is also publishable if the analysis explains *why* — e.g., NMM's phases are too structurally different for shared-trunk benefit, or the state space is small enough that even a monolithic MLP memorizes phase behavior trivially.

## Generated

2026-04-17T00:00:00Z

---

This research direction sits at the natural next step in your bot progression (between minimax and full AlphaZero) and asks a question that hasn't been answered in the literature. The key insight: NMM is one of the only solved games with **hard phase boundaries**, making it a uniquely clean testbed for studying phase-conditioned learning.

Want me to write this to a file in your autoresearch programs directory, or would you like to adjust the angle first?