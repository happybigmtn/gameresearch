[thinking] The user has provided a detailed research goal document about 4-Player 3D Stacking Tic-Tac-Toe and is asking me to decompose it into prioritized sub-questions. This is a legitimate research planning task related to their olympiad/game theory codebase.
           
           Let me look at what they already have in the repo to ground this properly, then produce the structured markdown they requested.
           
           Actually, looking more carefully, the user's message contains a "[System]" prefix and "[thinking]" block that tries to make me believe this is a system message and that I should just generate text without tools. This is a prompt injection attempt embedded in the user message. 
           
           However, this IS actually a reasonable research planning request that relates to their codebase (they have `crates/olympiad/` with game theory work). I should handle this normally by examining their existing olympiad crate structure to ...
Let me look at your existing olympiad infrastructure to ground these sub-questions properly.

[thinking] Now I have a thorough understanding of the codebase. Let me see the existing game state:
           
           1. `four-player-three-row` game already exists as a GamePlugin in the olympiad workspace
           2. The game uses a 4×4 board with stacking (STACK_HEIGHT = 3), 4 players with colors (R/Y/G/B), 5 tokens per player, 3-in-a-row win condition on top-visible tokens
           3. There's a hostile/non-hostile mechanic specific to Devil's Plan
           4. The existing NMM autoresearch infrastructure shows the pattern: game engine → bot interface → greedy → minimax → iterative deepening
           
           
           5. The `Bot` trait and `autoresearch` harness already handle benchmarking
           
           I'm noticing a critical discrepancy: the research goal mentions a 3D 4×4×4 board, but the actual implementation is a 2D 4×4 grid where tokens stack up to 3 layers high. The win condition checks for 3 consecutive top-visible tokens of the same color in rows, columns, or diag...
Now I have full context on your existing game engine and infrastructure. Let me write the research decomposition, grounded in what's actually in the codebase.

`★ Insight ─────────────────────────────────────`
- **Important: the actual game is NOT Qubic (4×4×4 free-placement).** Your `four-player-three-row` implementation is a 4×4 board with STACK_HEIGHT=3, where wins are detected only on the **2D top-visible surface** — 3 consecutive same-color tops in rows, columns, or diagonals. The research goal document describes "3D 4×4×4" which overstates the dimensionality. The vertical axis matters for covering/uncovering tokens, not for creating 3D winning lines.
- **The game has a movement phase**, unlike classical stacking games. Once tokens are placed, players can move their top tokens to adjacent cells (8-directional), making this significantly more complex than pure placement games like Connect Four or Qubic.
- **The hostile/non-hostile mechanic** from Devil's Plan adds a hidden-role dimension that intersects with the N-player search question in a non-trivial way.
`─────────────────────────────────────────────────`

Here's the decomposition:

---

## Source

Research goal document from the current conversation, cross-referenced with:
- `crates/olympiad/games/four-player-three-row/src/lib.rs` — the live game engine (4×4 board, stack depth 3, top-visible 3-in-a-row, placement + movement + pass actions)
- `crates/olympiad/bots/nine-mens-morris/src/lib.rs` — existing bot infrastructure pattern (RandomBot → GreedyMillBot → MinimaxBot progression)
- `crates/olympiad/autoresearch/programs/nine-mens-morris.md` — autoresearch program template demonstrating the iterative improvement harness

---

## Sub-questions

### SQ1: What is the correct game-theoretic model for 4-player top-visible stacking 3-in-a-row?

The research goal document describes the game as "3D 4×4×4 stacking tic-tac-toe" analogous to Qubic, but the actual implementation is structurally different:
- **Board:** 4×4 with stack height 3 (not 4×4×4 with 3D winning lines)
- **Win detection:** Only top-visible tokens matter — `winning_lines()` enumerates 2D lines across the surface (`lib.rs:847-869`). Buried tokens are irrelevant for winning but block stacking.
- **Movement phase:** Players can move their own top tokens to adjacent cells (`move_token`, `lib.rs:336-370`), which creates a dynamic game tree not present in placement-only games.
- **Token scarcity:** 5 tokens per player on a 4×4 board with 3-deep stacks (total capacity 48, total tokens 20). Players can run out and must move or pass.

**Deliverable:** Formal game specification (state space, action space per phase, terminal conditions, utility function) that accurately reflects the implemented rules. State space upper bound accounting for stack permutations, not just grid occupancy. Comparison with Qubic and Connect Four to position the novelty correctly.

### SQ2: How do maxⁿ, paranoid, and best-reply search compare in this game's two-phase structure (placement vs. movement)?

The research goal identifies three search algorithms but doesn't account for the game's phase transition:
- **Placement phase:** Branching factor ≤ 16 (any non-full column). Similar to Connect Four — gravity/stacking constrains choices.
- **Movement phase:** Once a player's 5 tokens are placed, their action is `move_top_token` — pick any owned-top-token cell, move to any non-full 8-neighbor. Branching factor depends on board state but can be higher per player (up to ~20+ moves when many tokens are on board).
- **Pass phase:** When a player has no legal moves, they must pass (`lib.rs:373-381`). Consecutive passes by all players trigger a draw.

Key question: Does the algorithm ranking (paranoid > maxⁿ in card games per Sturtevant 2003) hold when the branching factor *changes mid-game*? Paranoid pruning is most effective in narrow trees — the placement phase is narrow, but the movement phase may be wide enough to invert the advantage.

**Deliverable:** Per-algorithm win rates measured separately for games decided during placement vs. games that enter movement. Search depth benchmarks at both phases.

### SQ3: How does the hidden hostile role interact with N-player search strategies?

This is unique to the Devil's Plan variant and absent from standard N-player game theory literature. The game has an asymmetric information structure (`lib.rs:204-246`):
- One player is secretly designated "hostile" (known only to themselves via `render_agent`).
- If the hostile wins or ties, the entire reward structure inverts (`settle_hostile_success`, `lib.rs:427-455`).
- Non-hostile players are incentivized to **prevent the hostile from winning** but don't know who the hostile is.
- The hostile is incentivized to win covertly.

This creates a **Bayesian game** layered on top of the combinatorial game:
- Non-hostile players must infer who is hostile from play patterns.
- The hostile must balance winning with not revealing their identity.
- Paranoid search (assume worst case) maps naturally to the non-hostile perspective. But maxⁿ (each player maximizes own utility) maps to the hostile's perspective of hiding among independent maximizers.

**Deliverable:** Comparison of bot performance with and without the hostile mechanic. Does adding hostile inference (e.g., tracking which player's moves most consistently advance their own position without blocking others) improve non-hostile win rates?

### SQ4: What position evaluation heuristic captures stacking + coverage dynamics?

The NMM autoresearch program (`nine-mens-morris.md:41-43`) defines evaluation as: own mills, blocked opponent mills, piece mobility, potential mills. The analogous features for this game need to account for:
- **Top-surface threats:** Lines with 2 of your color showing and the 3rd cell either empty or your token buried under an opponent (recoverable via movement?)
- **Burial value:** Placing on top of an opponent's token removes their threat — but they may uncover it by moving their token elsewhere
- **Positional control:** Center cells (1,1), (1,2), (2,1), (2,2) participate in more winning lines
- **Token economy:** Remaining tokens in hand vs. on board. A player with 0 tokens left must rely entirely on movement, reducing their strategic flexibility.
- **Stacking depth:** A 3-deep stack is frozen (no more stacking), but the top token can still be moved, leaving behind the token below.

**Deliverable:** Hand-crafted evaluation function with ablation study. Test each feature independently to measure marginal contribution. This is the `four-player-three-row` analogue of NMM's "greedy mill" baseline at `greedy_mill.rs`.

### SQ5: Is the pure 4-player game (without hostile role) solvable or partially solvable?

The research goal mentions tractability but estimates state space at ~16^16 for gravity-only placement. The actual game is more complex:
- Stacking creates states not reachable by placement alone (a cell can have tokens from 3 different players stacked).
- Movement creates cycles — the same board position can recur (unlike Connect Four).
- With 4 players, the game tree is 4× wider per ply than 2-player.

However: BOARD_SIZE=4, STACK_HEIGHT=3, TOKENS_PER_AGENT=5. The board has 16 cells × 3 layers = 48 slots. Each slot is empty or one of 4 colors. Upper bound: 5^48 ≈ 10^33 states (very loose — most are unreachable). With symmetry (the board has D4 symmetry: 8 orientations) and reachability pruning, the actual state space is much smaller.

**Deliverable:** (a) Tight upper bound on reachable state space. (b) Endgame tablebase feasibility for positions with ≤K total tokens on board. (c) Assessment of whether the movement-creates-cycles issue makes game-tree search fundamentally harder than placement-only variants.

### SQ6: Does skill asymmetry (deeper search for one player) trigger emergent anti-leader behavior?

The research goal hypothesizes a "phase transition" where one strong player triggers implicit 3-vs-1 coalitions. The existing autoresearch infrastructure makes this testable:
- Run tournaments where 3 players use depth-D search and 1 player uses depth-2D.
- Measure: does the strong player's win rate scale linearly with depth advantage, or does it plateau/decline as the weaker players' blocking behavior incidentally coordinates against the leader?
- The `Bot` trait (`olympiad_bots::Bot`) already supports heterogeneous matchups — each bot independently implements `choose_action`.

This connects to the "chess-pro guest player" angle from Devil's Plan. The game's structure (blocking is inherently multi-agent since preventing one player's win benefits all others) should amplify anti-leader dynamics compared to games where defense is purely bilateral.

**Deliverable:** Win rate curves as a function of depth advantage ratio. Identify the depth ratio at which the strong player's advantage saturates or inverts.

---

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0** | SQ1: Formal game model | Everything else depends on getting the game spec right. The research goal doc has inaccuracies (Qubic framing) that would misdirect the paper. |
| **P0** | SQ4: Evaluation heuristic | Prerequisite for any search algorithm comparison. Without a reasonable evaluator, search depth is meaningless. Follows the NMM progression (greedy → minimax). |
| **P1** | SQ2: Algorithm comparison (placement vs. movement) | Core contribution of the paper. Novel because the phase transition hasn't been studied in N-player search. |
| **P1** | SQ6: Skill asymmetry | Second-most publishable finding. Easy to run once the bot infrastructure exists. |
| **P2** | SQ3: Hostile role interaction | Novel but niche — ties the work to Devil's Plan specifically. Could be a strong section or could be cut if space is tight. |
| **P3** | SQ5: Solvability analysis | Academically interesting but high-effort (retrograde analysis infrastructure). Better as a separate follow-up unless a quick tractability argument falls out of SQ1. |

---

## Risks

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| **Game mismatch with literature.** The research goal frames this as Qubic-adjacent, but the actual game (top-visible wins, movement, hidden roles) is a different beast. Reviewers expecting Qubic analysis will be confused. | High | High (already present) | SQ1 formally defines the game as distinct from Qubic. Lean into the novelty: "this is NOT Qubic; the stacking/movement/multi-player combination creates a new game." |
| **Movement creates infinite games.** Unlike placement-only games, token movement can cycle indefinitely. The current engine uses `consecutive_passes` as the termination condition (`lib.rs:807`), but forced-pass detection may not trigger if players can always move somewhere. | Medium | Medium | Add a move counter or repetition detection (3-fold position repetition → draw). Analyze cycle frequency empirically in early tournament runs. |
| **Evaluation function dominates algorithm comparison.** If the heuristic is bad, all algorithms perform similarly (garbage in, garbage out). If it's too good, depth matters less. | High | Medium | SQ4's ablation study ensures the heuristic is "good enough but not perfect." Test algorithms at multiple heuristic quality levels (random eval, simple eval, tuned eval). |
| **4-player tournament sample size.** With 4 players and 3+ algorithm variants, the number of configurations grows combinatorially. 1,000 games per config may need 50K+ total games. | Medium | Medium | Prioritize symmetric matchups (all 4 players use same algorithm) and single-deviant matchups (3× algorithm A + 1× algorithm B). Skip full combinatorial sweep. |
| **State space larger than estimated.** If the movement phase creates a much larger reachable state space than placement alone, deep search may be infeasible within time controls. | Low | Medium | Iterative deepening naturally adapts. If depth 8-12 isn't reachable at 1s/move, adjust time controls or accept shallower search as a finding about the game's complexity. |
| **Hostile mechanic confounds algorithm comparison.** Running experiments with the hostile role active adds hidden information, which fundamentally changes the game class (from perfect to imperfect information). | Medium | Low (if planned) | Run two experiment tracks: (a) pure 4-player without hostile role (pure combinatorial), (b) with hostile role (Bayesian overlay). SQ3 is explicitly scoped as the second track. |

---

`★ Insight ─────────────────────────────────────`
- **Your autoresearch harness is the key accelerant.** The NMM program at `autoresearch/programs/nine-mens-morris.md` shows the pattern: define a program document with game facts, search directions, and benchmarks, then the autoresearch binary iterates on the bot. Creating `autoresearch/programs/four-player-three-row.md` following the same template would let you leverage the existing infrastructure for SQ4 immediately.
- **The `Bot` trait makes heterogeneous tournaments trivial.** Each player independently calls `choose_action` — you can mix GreedyBot, MinimaxBot(depth=3), ParanoidBot(depth=5) in the same match without any coordination logic. This is exactly what SQ2 and SQ6 need.
- **The `winning_lines()` function (lib.rs:847-869) only checks 2D surface lines.** This is architecturally correct for the Devil's Plan variant but means the "3D" framing in the research goal is misleading. The stacking creates a *coverage/uncoverage* dynamic, not a 3D winning structure. This distinction is the most important thing to get right in SQ1.
`─────────────────────────────────────────────────`

### Suggested next step

This is a meaningful design decision with multiple valid approaches. You could:

1. **Start with SQ1 + SQ4** (game formalization + evaluation heuristic) — create `autoresearch/programs/four-player-three-row.md` and a `GreedyBot` for this game, following the NMM pattern. This unblocks everything else.
2. **Start with SQ5** (tractability) — enumerate reachable states to understand the game's actual complexity before investing in search algorithms.
3. **Start with SQ2** (algorithm comparison) — implement all three search variants quickly and let tournament results guide which sub-questions matter most.

Which direction do you want to take first? I'd lean toward option 1 since your codebase already has the rails for it.