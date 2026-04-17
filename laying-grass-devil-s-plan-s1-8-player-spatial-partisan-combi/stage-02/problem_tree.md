[thinking] The user is asking me to decompose a research problem into sub-questions. This is a legitimate research planning task — they want me to analyze the "Laying Grass" game from Devil's Plan and produce a structured markdown document with sub-questions, priority ranking, and risks.
           
           This is a text generation task — I should produce the research decomposition as requested. Let me also check if there's existing work in this repo related to this game, since the research goal mentions "Rust game engine (already in this repo)."
           
           Let me look at the codebase first to understand what already exists, so my decomposition is grounded in the actual project state.
Let me first check what already exists in the repo for this game, so the decomposition is grounded in reality.

[thinking] Now I have a thorough understanding of the existing codebase. Let me also check the research report and GDD for additional context.
Now let me check the research report for the authoritative rules and the existing autoresearch harness.

[thinking] Now I have a thorough understanding of the codebase. Let me also check for the existing research goal format and the knowledge base structure.
Now let me check for existing research goals and the knowledge base question format.

[thinking] Now I have a comprehensive understanding of the project. Let me now produce the research decomposition document. Let me look at a few more things to understand the Day 5 section specifically.
Let me read the Day 5 section of the research report for the authoritative rules.

[thinking] Now I have a comprehensive understanding. Let me produce the research decomposition document. The user's research goal document is very detailed, so I need to decompose it into prioritized sub-questions.
           
           Let me think about the key sub-questions:
           
           1. **Formalization & Rule Correctness** — The game needs to be precisely formalized. Is the existing Rust implementation a correct model? How does the 20×20 reduction affect game dynamics vs. the original 30×30?
           
           2. **CGT Properties of Connectivity-Constrained Spatial Games** — Can classical CGT tools (temperature, thermography) be adapted for games with a global connectivity constraint? What happens to decomposition? Can positions be characterized?
           
           
           3. **Multi-player Extension of CGT** — Standard CGT is 2-player. What are the theoretical barriers to 8-player analysis? How do coalitions form implicitly through spatial proximity?
           
           4. **Power...
Now I have a thorough understanding of both the research goal and the existing codebase. Here's the decomposition:

---

# Research Decomposition: Laying Grass — Connectivity-Constrained Spatial Partisan Games

## Source

- **Research Goal:** `docs/kb/questions/topic_init-rc-20260417-155038-a70119.md`
- **Authoritative Rules:** `research/devils_plan_s1_game_design_report.md` §Day 5 (lines 167–182)
- **Game Spec:** `specs/160426-olympiad-games/dp-laying-grass.md`
- **Implementation:** `crates/olympiad/games/laying-grass/src/lib.rs` (1,923 lines, all 6 spec tests pass)
- **Autoresearch Harness:** `crates/olympiad/autoresearch/` (benchmark + journal + program.md loop)
- **Existing Bot Infra:** `crates/olympiad/bots/` (NMM greedy bot at 0.885 vs random — current branch)

---

## Sub-questions

### SQ1: Can local CGT decomposition work under a global connectivity constraint?

**The core theoretical barrier.** Classical CGT decomposes a game position into independent sub-positions and computes Sprague-Grundy values or surreal number sums. Laying Grass's hard connectivity constraint (every placed tile must extend a contiguous region) means a move in one board region can affect legality of moves in a distant region — the frontier is a single connected boundary, not independent local battles.

**What to investigate:**
- On small boards (8×8, 2-player), enumerate all legal endgame positions with ≤12 empty cells. For each, attempt a "local game" decomposition by identifying connected components of the empty frontier and checking whether moves in one component affect the other's legal-move set.
- Quantify: what fraction of positions admit independent sub-game decomposition? Hypothesis: very few, because the connectivity constraint propagates across the entire boundary.
- If decomposition fails globally, does it hold *conditionally* — e.g., when two players' territories are separated by ≥2 empty rows, do the sub-boards become independent until they collide?
- Compare with Go's "settled groups" analysis — does a similar concept of "locally settled territory" exist in Laying Grass?

**Output:** Empirical decomposability fraction on 8×8 solved positions; conditional-independence theorem or counterexample.

### SQ2: What is the temperature structure and first-mover advantage in small-board Laying Grass?

**Temperature** in CGT measures how urgently a position demands a move. "Hot" games have large temperature (both players benefit from moving); "cold" games are near-zugzwang. The research goal proposes measuring temperature on 8×8 2-player boards — this is tractable but requires adapting temperature computation to a game that doesn't cleanly decompose (see SQ1).

**What to investigate:**
- Implement alpha-beta search with connectivity-aware move pruning for 8×8 2-player Laying Grass. The existing Rust engine (`lib.rs`) implements all placement validation (`validate_placement`, line 584) — this is the move generator.
- Compute exact game values for endgame positions. Measure first-mover advantage: % of positions won by P1 under optimal play, as a function of board fill ratio.
- Track how temperature evolves through a game: is there a phase transition between "expansion" (high temperature, both players want to grab area) and "consolidation" (low temperature, defensive play)?
- The scoring function is based on largest-square, not total area. Does this create zugzwang-like situations where a player would prefer to *not* place (because placing a tile could block their own square)?

**Output:** First-mover advantage curve, temperature distribution across game phases, zugzwang frequency measurement.

### SQ3: How do power cells (steal/stone/tile-swap) alter game-theoretic equilibrium?

**Controlled disruption analysis.** The 4 special types create discontinuities in the combinatorial structure:
- **Stone** (6 cells): a blocking move that creates a persistent obstacle — changes the topology of available space
- **Steal** (4 cells): transfers ownership of an enemy tile — breaks the opponent's connectivity potentially
- **Tile-swap coupon** (8 cells): choice among next 5 tiles instead of forced-take — information advantage
- **Key** (8 cells): currency for stone removal and patching — resource conversion

**What to investigate:**
- Run matched experiments: train RL agents (or run exact search on 8×8) with and without power cells. Measure territory distribution shift, largest-square outcomes, and win-rate variance.
- Does the *steal* mechanic create a fundamentally different game class? Stealing a border tile can disconnect an opponent's territory — this is a "topology-altering move" not present in standard placement games. Is there an analogous concept in the Hackenbush/Nim literature?
- Are stone placements analogous to "komaster" threats in Go's CGT analysis — forcing moves that shift the initiative?
- The `place_cells` function (line 631) already handles special-cell triggering via adjacency — measure trigger rates empirically under different strategies.

**Output:** Win-rate and territory-shape statistics with/without power cells; characterization of steal as topology-disrupting vs. value-adding.

### SQ4: What board representations enable GNN-based evaluation for connectivity-constrained territory games?

**Representation design for learned evaluation.** The research goal proposes GNN-based state encoders. The key question is: what graph structure should the GNN operate over?

**Candidate representations:**
1. **Grid graph** (20×20 = 400 nodes, 4-connected): simple but doesn't encode connectivity constraint explicitly. The GNN must *learn* that contiguity matters.
2. **Territory boundary graph**: nodes are frontier cells (empty cells adjacent to any territory); edges connect adjacent frontier cells. Compact but loses interior information.
3. **Dual graph**: one node per contiguous territory region plus one per "contested frontier." Edges represent adjacency relationships. Encodes the connectivity constraint structurally.
4. **Augmented grid with connectivity features**: standard grid graph but each node carries features including {owner, distance-to-own-boundary, component-id, local-degree-of-freedom-count}.

**What to investigate:**
- On the 20×20 board (400 cells), representation (1) is small enough for message-passing GNNs but may need many layers to propagate connectivity information across the board.
- The existing `largest_square_side` function (line 514) uses brute-force O(n^3) search — for learned evaluation, can a GNN predict largest-achievable-square more efficiently?
- Test whether GNN evaluation quality correlates with layer count (proxy for information propagation distance — connectivity requires long-range communication).
- Compare with the AlphaZero approach for Go (CNN on raw grid) vs. KataGo's ownership predictions — what features matter?

**Output:** Representation benchmark comparing value-prediction accuracy and policy quality across the 4 candidates; scaling analysis for 8×8 → 15×15 → 20×20.

### SQ5: Do RL agents trained via self-play discover CGT-consistent strategies?

**The RL-CGT bridge.** The research goal's most ambitious measurement: Spearman ρ ≥ 0.4 between CGT temperature estimates and agent move preferences.

**What to investigate:**
- Train MAPPO/IPPO agents on 15×15 4-player boards using the PettingZoo-wrapped Rust engine. The autoresearch harness already supports benchmark loops (`BenchmarkConfig`, `run_benchmark` in the autoresearch crate) — extend it for RL self-play.
- On the 8×8 2-player solved positions from SQ2, evaluate whether the RL agent's move ranking correlates with exact game values.
- Specific behaviors to test: (a) Do agents play "hot" positions first? (b) Do agents avoid zugzwang? (c) Do agents build toward squares (the scoring objective) or maximize raw territory? (d) Do agents learn to exploit power cells strategically or ignore them?
- The NMM greedy bot (0.885 vs random) establishes the baseline development pattern. For Laying Grass, define heuristic baselines: greedy-area (maximize cells placed), greedy-square (maximize largest-square), greedy-connectivity (maximize frontier length).

**Output:** Correlation metric (Spearman ρ), behavioral analysis of trained agents, win-rate vs heuristic baselines.

### SQ6: Does the 8-player free-for-all produce emergent coalition dynamics through spatial proximity?

**Multi-agent social structure.** In the show, alliances mattered little (tradeable coupons were the only cooperation lever). But in a computational setting with learning agents, spatial proximity creates implicit coordination or conflict.

**What to investigate:**
- In 8-player self-play, do agents that start adjacent (perimeter-based seating, see `perimeter_coords` line 446) develop different interaction patterns than distant agents?
- Does the "enemy tiles must not share faces" rule create natural alliance structure? Two players whose territories are far apart are *not* in conflict, but two whose territories approach the same region are mutually blocking. Does this create a "geographic alliance" effect?
- The research report notes: "Alliance value is marginal — tradeable Tile-swap Coupons are the only real cooperation lever." Is this true computationally, or does implicit spatial cooperation (e.g., two players jointly blocking a third's square attempt) emerge?
- Compare 2-player, 4-player, and 8-player dynamics: at what player count does the game transition from a positional duel to a crowded territorial scramble?

**Output:** Coalition structure analysis, spatial interaction heatmaps, player-count scaling behavior.

---

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0** | SQ2: Temperature & first-mover advantage (8×8 exact) | Foundational — produces the CGT ground truth that SQ1, SQ3, and SQ5 all depend on. Computationally tractable on a single machine. Uses the existing Rust game engine directly. |
| **P1** | SQ1: Decomposability under connectivity constraint | The core theoretical contribution. If positions decompose (even conditionally), standard CGT tools apply and the paper has a clean theorem. If not, the paper pivots to empirical characterization — knowing this early shapes everything. |
| **P2** | SQ5: RL agents discover CGT-consistent strategies | The headline result for AAAI/IJCAI/CoG. Requires SQ2 ground truth. Training on 15×15 is the compute bottleneck (~20 GPU-hours). High reward but depends on P0 and P1 settling the evaluation framework. |
| **P3** | SQ4: GNN representation design | Enables SQ5 but can be explored in parallel on smaller boards. Multiple candidates can be compared cheaply before committing to the expensive 15×15 training runs. |
| **P4** | SQ3: Power cell impact analysis | Strengthens the paper with a "controlled disruption" angle. Can be measured as a side-experiment once the base game is solved (SQ2) and agents are trained (SQ5). Lower priority because the base game without power cells is already novel. |
| **P5** | SQ6: 8-player coalition dynamics | Interesting but speculative. The research report itself notes alliances matter little in this game. Pursue only if SQ2+SQ5 yield strong results and there's time in the 8-week window. |

---

## Risks

### R1: Connectivity constraint makes 8×8 exact analysis intractable
**Severity: High.** The branching factor of Laying Grass is much larger than NMM (tile shapes × rotations × mirror × positions). On 8×8 with 13 tile shapes, 4 rotations, 2 mirror states, and up to ~60 valid anchor positions, a single move could have O(6,000) legal successors. With 2 players over 9 rounds, the game tree is enormous even on 8×8.

**Mitigation:** Focus exact analysis on **endgame positions** (≤12 empty cells, 1–3 rounds remaining), not the full game. Use connectivity-aware alpha-beta pruning — the `validate_placement` function already rejects illegal moves at generation time, and many board states are transpositionally equivalent under symmetry.

### R2: GPU constraint (4 GB VRAM laptop GPU)
**Severity: Medium.** The hardware profile shows an RTX 500 Ada with 4 GB VRAM. GNN-based MAPPO on 15×15 boards with 8 agents may exceed memory. The SMART goal assumed RTX 3090/4090 class.

**Mitigation:** Use the Rust engine for all game simulation (zero GPU cost for environment). Train smaller GNNs (2–3 layers, 64-dim embeddings). Reduce to 10×10 4-player if 15×15 overflows. Consider CPU-only training with smaller networks — CleanRL implementations of PPO are CPU-efficient for environments with fast simulators.

### R3: No existing PettingZoo bridge
**Severity: Medium.** The research goal says "PettingZoo-compatible environment" but the game engine is pure Rust. Building a Python-Rust bridge (PyO3/maturin) and wrapping it in PettingZoo's AEC API is a non-trivial infrastructure task that could consume the first 2 weeks alone.

**Mitigation:** The autoresearch harness already runs head-to-head matches entirely in Rust (`run_benchmark`). Consider training RL bots in pure Rust using `tch-rs` (libtorch bindings) instead of crossing to Python. Alternatively, write a minimal gymnasium env that calls the Rust engine via `pyo3` — the `GamePlugin` trait's `apply_action` + `available_actions` map cleanly to PettingZoo's step/observe API.

### R4: Largest-square scoring creates a degenerate strategy landscape
**Severity: Medium.** The scoring table is coarse: 2×2 through 8×8 maps to only 7 discrete outcomes (-3 to +3). With 8 players on a 20×20 board, most players may converge on 5×5 (neutral) or 6×6 (+1), making the game outcome dominated by the +3 bonus for unique-largest-square rather than by placement quality. This could mean the game-theoretic "interesting" region is narrow.

**Mitigation:** Analyze the scoring distribution early (part of SQ2). If most games collapse to 5×5 vs 5×5 ties, consider supplementary metrics (total territory, frontier control) to measure strategic differentiation beyond the scoring function.

### R5: Novelty claim over-stated — competitive graph coloring literature may already cover this
**Severity: Low-Medium.** The research goal claims this is "underexplored." Competitive/online graph coloring with connectivity constraints has some treatment in the distributed computing literature (e.g., Linial's coloring lower bounds, online interval coloring). If a reviewer identifies this game as a known instance of competitive graph coloring on grid graphs, the novelty claim weakens.

**Mitigation:** Conduct a targeted literature search early: "competitive graph coloring grid" + "territorial games connectivity constraint" + "multiplayer spatial combinatorial games." The research goal's cited references (Cincotti, Li, Calistrate/Albert/Nowakowski) are the right starting points but need expanded search.

### R6: The Rust implementation may diverge from the show's rules
**Severity: Low.** The spec (`dp-laying-grass.md` line 27) already notes "reduce to 20×20 for autonomy deployment." The `tile_queue` drain behavior in `use_tile_swap_coupon` (line 1207) puts remaining tiles at the *front* of the queue after choice removal, not the end — this differs from the show description ("skipped tiles go to end-of-queue"). Minor rule discrepancies accumulate.

**Mitigation:** Validate the Rust implementation against 2–3 hand-transcribed game logs from the show before claiming the environment is correct. The spec already identifies the research report as authoritative; reconcile any differences.

`★ Insight ─────────────────────────────────────`
**Why this decomposition front-loads SQ2 over SQ1:** Temperature analysis on small boards is an *empirical* computation that produces concrete data regardless of whether a clean decomposition theorem exists. SQ1 (decomposability) is the *theoretical* contribution, but it's higher-risk — if decomposition doesn't hold, you need the empirical data from SQ2 to pivot to "here's what the game looks like computationally even without decomposition." Starting with SQ2 de-risks the entire project.

**The tile-swap coupon drain bug (R6):** In the Rust code, `model.tile_queue.drain(..take_count)` followed by `model.tile_queue.extend(front)` puts the unchosen tiles at the *back* of the remaining queue, but the remaining queue was already sliced at `take_count`. The net effect: unchosen tiles from the coupon window go to position `take_count` onward, while the original `take_count+1..` tiles remain in order. This is actually consistent with "skipped tiles go to back" *relative to the non-coupon portion*, but the `tile_queue.extend` call appends to the end of whatever remains — which after the drain is the tiles that were *beyond* the 5-window. So the behavior is: chosen tile is removed, remaining 4 from the window are re-appended after the rest of the queue. This matches the show's rule description. Worth verifying with a trace, though.
`─────────────────────────────────────────────────`