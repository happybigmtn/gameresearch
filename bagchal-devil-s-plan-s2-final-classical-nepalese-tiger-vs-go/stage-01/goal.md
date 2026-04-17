[thinking] The user is asking me to create a research goal document about Bagchal (a classical Nepalese tiger-vs-goat board game) for a research pipeline. This is a legitimate research planning task. Let me look at what exists in the codebase first to understand the context better, since there's already a Bagchal crate in this project.
           
           Let me explore the existing Bagchal implementation and any research-related files to make an informed research plan.
Let me first understand what already exists in this project for Bagchal and any research infrastructure.

Let me also check the existing bot, the board topology, and the research report for more context.

[thinking] Now I have a thorough understanding of the codebase. Let me analyze what we have:
           
           1. **Bagchal implementation** - A full Rust implementation of Bagchal as a game plugin with:
              - 5x5 board (25 intersections, 56 edges)
              - Standard rules: 4 tigers at corners, 20 goats to place, tigers capture by jumping
              - **Devil's Plan S2 variant**: Two-board mirror structure where each player plays goats on one board and tigers on the other. First board to resolve wins.
              - Win conditions: Tigers win by capturing 5 goats, goats win by immobilizing all tigers or 50-move stalemate
           
           2. **Existing bots**:
              - `RandomBot` - uniform random over legal moves
              - `GreedyBot` - depth-1 greedy using material differential * 100 - opponent mobility
           
           3. **Board topology**: 5x5 grid with orthogonal + diagonal edges on alternating cells (standard Bagchal pattern), generating 56 edges total
           
           Now I'm thinking...
Let me check the prior research attempt cache for context on what failed.

[thinking] Good, there's a literature cache with results from prior research attempts. Let me also check if there's any existing research goal or config file for the research-cli pipeline.
Now I have a thorough understanding of the codebase and the specific Bagchal variant. Let me produce the research goal.

`★ Insight ─────────────────────────────────────`
- **The Devil's Plan S2 Bagchal variant is fundamentally different from standard Bagchal.** Standard Bagchal is a single-board asymmetric game (weakly solved as a draw). The S2 variant creates a *symmetric meta-game*: each player simultaneously plays goats on one board and tigers on the other, with alternating turns spent on *either* board. This board-selection dimension doesn't exist in any published Bagchal analysis.
- **The "first board to resolve wins" rule creates a tempo race.** Unlike standard Bagchal where you optimize a single position, here you must balance offense (pushing toward resolution on the board where you have the advantage) against defense (preventing resolution on the board where you're weaker). This is structurally similar to multi-front war games but hasn't been analyzed in the combinatorial game theory literature for Bagchal specifically.
- **The existing codebase gives us a complete experimental platform** — game engine with legal move enumeration, two baseline bots (random and depth-1 greedy), material scoring, and mobility metrics. The state space is tractable (25 nodes per board, two boards, ~10^10 single-board states).
`─────────────────────────────────────────────────`

Here is the SMART research goal:

---

# Research Goal: Dual-Board Bagchal — Board-Selection Strategy in the Devil's Plan S2 Variant

## Topic

Bagchal (बाघ चाल), a classical asymmetric Nepalese board game (4 tigers vs. 20 goats on a 5x5 intersection graph), as instantiated in the Devil's Plan Season 2 (2025) finals variant: a two-board mirror structure where each player plays goats on one board and tigers on the other, with first-to-resolve-any-board determining the winner.

## Novel Angle

**Standard Bagchal is weakly solved** — optimal play from both sides leads to a draw on a single board. Existing computational work (retrograde analysis, alpha-beta search, and limited RL efforts) addresses this single-board setting exclusively. No published work addresses the **dual-board mirror variant** introduced in Devil's Plan S2, which transforms Bagchal in three structurally significant ways:

1. **Board selection as a strategic action dimension.** On each turn, the active player chooses which of the two boards to act on. This creates a meta-game layer absent from single-board Bagchal: when to invest tempo in offense (pushing resolution on a favorable board) vs. defense (preventing resolution on the unfavorable one). This cross-board resource allocation has no analogue in existing Bagchal literature.

2. **Symmetry restoration via role mirroring.** Single-board Bagchal is asymmetric (tiger vs. goat roles differ fundamentally). The dual-board variant restores symmetry at the match level — each player holds one tiger role and one goat role. Whether this eliminates the first-mover advantage present in single-board play is an open question.

3. **Racing dynamics under a first-to-resolve stopping rule.** The game ends when *any* board resolves (tiger captures 5 goats, or all tigers immobilized, or 50-move stalemate). This racing structure is similar to combinatorial game theory's "sum of games" formalism, but with a crucial difference: here the game ends on the *first* component to terminate, not the last. This inverts the standard theory (which usually considers the last game to finish) and creates incentives that diverge from independent per-board optimization.

**Why this is timely:** The Devil's Plan S2 aired in 2025, bringing this variant to cultural prominence. Computational game theory is increasingly addressing game variants from competitive entertainment (poker variants, game show formats). The existing Rust implementation in this codebase provides a validated game engine ready for automated play, requiring only bot development — no environment engineering.

**How this differs from standard approaches:** Rather than re-solving single-board Bagchal (already done) or applying RL to a known game, this work studies how a *structural modification* (dual-board mirror + first-to-resolve) changes the game's strategic landscape. The research contribution is the analysis of the variant itself, not a new algorithm.

## Scope

A single focused paper analyzing the dual-board Bagchal variant through three complementary lenses:

1. **Board-selection policy characterization** — what heuristics govern when to play offense vs. defense? Measure the win-rate impact of board-selection strategies (always-attack, always-defend, material-responsive, mobility-responsive) against baselines.
2. **First-mover advantage quantification** — does the dual-board structure preserve, eliminate, or invert the first-player edge observed in single-board Bagchal? Measure seat-0 vs. seat-1 win rates across bot tiers.
3. **Cross-board coupling strength** — how independent are the two boards in practice? Measure correlation between board states and whether optimal play on one board ignores or responds to the other.

Out of scope: full game-theoretic solution (intractable for the dual-board variant), deep RL training, human study.

## SMART Goal

**Specific:** Implement a family of Bagchal bots with varying board-selection policies (4 fixed heuristics + 1 minimax with cross-board evaluation) and measure their pairwise win rates in the Devil's Plan S2 dual-board variant. Characterize the first-mover advantage and board-selection premium.

**Measurable:**
- Win-rate matrix (5x5 bots, 10,000 games per matchup) with 95% confidence intervals
- First-mover advantage: seat-0 win rate ± CI per matchup tier
- Board-selection premium: win-rate delta between best cross-board-aware bot and best single-board-only bot
- Board independence metric: mutual information between left-board and right-board captured-goats at game end

**Achievable:** The game engine, legal move enumeration, and two baseline bots already exist in the codebase (`crates/olympiad/bots/bagchal/`). The state space per board is ~10^10, and depth-3 minimax with alpha-beta pruning runs in <100ms per move on a single CPU core. 10,000 games per matchup × 25 matchups = 250,000 games, each completing in <500 moves × <100ms ≈ <50s. Total compute: ~3.5 hours on a single core, easily parallelizable.

**Relevant:** Directly supports the project's olympiad game-playing agent infrastructure and extends the existing Bagchal bot pipeline. The dual-board variant is the actual finals format used in the deployed system.

**Time-bound:** 4 weeks to implementation + experimentation + writeup.

## Constraints

- **Compute:** Single machine, multi-core CPU. No GPU required (search-based, not neural).
- **Available tools:** Existing Rust game engine (`olympiad_game_bagchal`), bot framework (`olympiad_bots`), deterministic seeding infrastructure.
- **Data access:** Self-generated via simulation. No external datasets required.
- **Implementation language:** Rust (matching existing codebase).
- **No external dependencies** beyond what's already in the workspace `Cargo.toml`.

## Benchmark

- **Name:** Dual-Board Bagchal Bot Arena (self-constructed)
- **Source:** Generated in-codebase via `olympiad_game_bagchal::BagchalPlugin` + `olympiad_bots::Bot` trait implementations
- **Metrics:**
  - Win rate (primary)
  - Average game length (turns to resolution)
  - Board-selection entropy (how uniformly does the bot spread actions across boards)
  - Material differential at game end
  - First-mover advantage (seat-0 win rate deviation from 50%)
- **Current SOTA on this benchmark:** `GreedyBot` (depth-1, material × 100 − opponent_mobility) vs. `RandomBot`. No published baselines exist for the dual-board variant. The greedy bot's single-board performance and board-agnostic selection policy serve as the internal baseline.
- **No standard external benchmark exists** for this variant. Results will be measured against the internal bot hierarchy and compared to known single-board Bagchal equilibrium outcomes (draw) to assess how the dual-board structure shifts the game balance.

## Success Criteria

A publishable result would demonstrate at least one of:

1. **Board-selection premium is statistically significant** — a cross-board-aware bot beats the best single-board-optimal bot by ≥5% win rate (p < 0.01), proving that board selection is a strategically meaningful dimension that cannot be ignored.
2. **First-mover advantage shifts** — the dual-board variant produces a measurably different first-mover advantage compared to single-board Bagchal (where goats move first and the game is a theoretical draw), with seat-0 win rate deviating from 50% by ≥3% (p < 0.01) or converging to 50% ± 1% across all bot tiers (proving symmetry restoration).
3. **Board coupling is non-trivial** — mutual information between board states exceeds the independence baseline by ≥2 standard deviations, showing that optimal play requires cross-board reasoning, not independent per-board optimization.

Any one of these three would constitute a novel finding about the structural game theory of the dual-board Bagchal variant.

## Generated

2026-04-17T00:00:00Z

---

`★ Insight ─────────────────────────────────────`
- **The "first to resolve" stopping rule inverts combinatorial game theory's standard "sum of games."** In CGT, the value of a sum of games is computed by combining component values — but the standard theory assumes all components are played to completion. Here, the game *stops* when any component resolves, creating incentives to rush one board while stalling the other. This is closer to a "disjunctive sum with early termination" — a less-studied but increasingly relevant formalism.
- **The existing `GreedyBot` is board-agnostic** — it evaluates all legal moves across both boards using the same score function and picks the best. This means it already implicitly does board selection, but without any cross-board awareness (e.g., "I'm losing on the left board, so I should invest all my tempo there to defend"). A board-selection-aware bot would explicitly reason about which board to prioritize.
`─────────────────────────────────────────────────`