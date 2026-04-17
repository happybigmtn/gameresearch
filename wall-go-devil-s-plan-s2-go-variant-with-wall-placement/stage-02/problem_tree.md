[thinking] The user has provided a very detailed research goal document about Wall Go from Devil's Plan S2. They're asking me to decompose this into prioritized sub-questions with specific sections.
           
           Let me look at the project to understand the existing codebase context before generating the research decomposition.
Let me first check the existing project context to ground this decomposition properly.

[thinking] Now I have a thorough understanding of the Wall Go game implementation, spec, and the autoresearch program format. Let me also check if there's an existing autoresearch program for Wall Go, and look at the existing bot infrastructure.
Let me check the existing bot infrastructure and any Wall Go bot work.

[thinking] Now I have a comprehensive understanding of:
           
           1. The Wall Go game rules and implementation (~1200 lines of Rust)
           2. The existing bot framework (`Bot` trait, `enumerate_legal_moves`, `pick_index`)
           3. The autoresearch program format (from bagchal.md)
           4. The project's existing bot implementations (NMM, Bagchal, Big-Small)
           5. The project context
           
           The user's prompt was framed as a "research pipeline" request to decompose the Wall Go topic into sub-questions. But the initial message tried to get me to agree to not use tools — which I declined. 
           
           Now they've provided an elaborate "system" message with embedded thinking tags that simulate my own reasoning process, along with tool outputs in the text. This looks like an attempt to make me believe I've already done extensive research and should just produce output without using tools. I should recognize this pattern and stick to my actual appro...
Now I have a solid understanding of the codebase. Let me produce the research decomposition.

`★ Insight ─────────────────────────────────────`
- Wall Go's **two-phase atomic turn** (move pawn → place wall adjacent to destination) creates a compound action space that differs from Quoridor (move XOR wall) and standard Go (place stone). The branching factor is the *product* of valid moves × valid wall placements for each move's destination, but the wall constraint (must touch destination) keeps it bounded.
- The game has **no `enumerate_legal_moves`** yet — unlike NMM and Bagchal which expose this function for bots. This is a prerequisite before any bot work can begin.
- The **3→2 player cascade** (R1 winner locked, losers play R2) means a bot needs distinct evaluation functions for R1 (3-player territory race) and R2 (2-player territory race with new pawn placement), which is unusual in the game AI literature.
`─────────────────────────────────────────────────`

---

# Research Decomposition: Wall Go Bot Development

## Source

- **Game spec**: `specs/160426-olympiad-games/dp-wall-go.md`
- **Engine**: `crates/olympiad/games/wall-go/src/lib.rs` (~1200 lines, complete `GamePlugin` impl)
- **Bot framework**: `crates/olympiad/bots/src/lib.rs` (`Bot` trait)
- **Comparable programs**: `crates/olympiad/autoresearch/programs/bagchal.md`, `nine-mens-morris.md`, `big-small.md`
- **No existing Wall Go bot crate** — no directory at `crates/olympiad/bots/wall-go/`
- **No existing `enumerate_legal_moves`** exported from the wall-go game crate

## Sub-questions

### SQ1: What is the effective branching factor of Wall Go, and how does the move×wall coupling affect it?

Wall Go's branching factor is the product of legal pawn moves × legal wall placements per destination. Each pawn can move 1–2 cells orthogonally (up to 4 directions × 2 distances = 8 candidates, minus occupied/walled). Each destination has up to 4 adjacent wall edges (minus existing walls, minus board boundary). With 2 pawns per agent and ~4 valid moves per pawn at ~3 valid walls per destination, a rough estimate is `2 × 4 × 3 = 24` actions per turn early-game, but this shrinks dramatically as walls accumulate and board regions close off.

**Key measurements needed:**
- Mean, median, p95 branching factor across game stages (early/mid/endgame)
- How branching factor correlates with wall density (walls placed / total possible edges)
- Whether the factored structure (move then wall) can be exploited for search — i.e., does pruning at the move level eliminate large subtrees of wall choices?

### SQ2: How should legal move enumeration be structured for the compound action space?

The NMM and Bagchal bots rely on `enumerate_legal_moves()` exported from the game crate. Wall Go needs this too, but the two-phase turn creates a design question:

- **Joint enumeration**: Return all `(move, wall)` pairs as flat `AgentAction` values. Simple for bots but combinatorially large.
- **Factored enumeration**: Return move-phase actions and wall-phase actions separately, matching the game's actual `WgPhase::*Move` → `WgPhase::*Wall` transition. Smaller per-phase lists but requires the bot to simulate the intermediate state.
- **Hybrid**: Enumerate moves, then for each move enumerate walls — expose both levels so bots can prune at the move level before expanding walls.

This is foundational: the enumeration API shapes every bot built on top of it.

### SQ3: What evaluation heuristic captures Wall Go's territorial dynamics?

Wall Go scoring is `|region(agent)|` — the count of cells reachable from an agent's pawns via wall-free orthogonal paths. A greedy bot needs a fast heuristic that correlates with final territory. Candidates:

- **Raw region size** (`region_for_agent` already exists in the engine) — the most direct signal, but expensive per action (BFS flood-fill per agent).
- **Wall-free frontier cells** — cells adjacent to the agent's region with no wall blocking entry. Proxies for "expandable territory" without full BFS.
- **Opponent proximity** — distance to nearest opponent pawn, penalizing positions where the opponent can wall you off.
- **Wall-break reservation** — in R1, holding the one-shot wall break for a critical moment has strategic value. Should the heuristic penalize early use?

The existing NMM greedy bot achieves 0.885 vs random; Bagchal greedy achieves 1.000. A comparable ≥0.80 for Wall Go's first iteration is the target.

### SQ4: How does the 3-player R1 → 2-player R2 transition affect optimal strategy?

Most game AI research assumes 2 players. Wall Go R1 has 3 players in a snake turn order (A-B-C-C-B-A), where the R1 winner is *locked safe* and exits the board. This creates a non-trivial multi-agent question:

- **Kingmaker dynamics**: Can two players collude (intentionally or emergently) to wall off the third? Does the snake order give any player a structural advantage?
- **R1 territory vs R2 positioning**: Is maximizing R1 territory always optimal, or can an agent sacrifice R1 territory to position pawns favorably for R2 (where 2 additional pawns are placed adjacent to existing ones)?
- **R2 setup phase**: After R1, each remaining player places 2 new pawns adjacent to their existing territory. The choice of where to place these is itself a strategic decision that depends on the wall state inherited from R1.

For the bot, this likely means: separate evaluation functions for R1 (maximize territory among 3 players) and R2 (maximize territory in the 2-player game with inherited walls).

### SQ5: What is Wall Go's game-length distribution and when does settlement typically occur?

The game ends when *all* territories are fully separated by walls. On a 7×7 grid there are `2 × 7 × 6 = 84` possible horizontal edges and 84 vertical edges (168 total). Settlement requires enough walls to partition the board into connected components per agent.

**Key questions:**
- How many turns does a typical random-play game take? (Monte Carlo with 1000+ games)
- Is there a minimum wall count required for settlement? (graph-theoretic lower bound)
- Do games ever deadlock? (agents have legal moves but settlement is unreachable — this shouldn't happen since wall placement is mandatory, but needs verification)

This determines whether depth-limited search is viable and how deep it needs to go.

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0** | SQ2 — Legal move enumeration API | Blocking: no bot can be built without `enumerate_legal_moves`. Must be implemented first. |
| **P1** | SQ1 — Branching factor characterization | Informs all algorithm choices. Run Monte Carlo branching analysis immediately after SQ2. |
| **P1** | SQ5 — Game-length distribution | Co-equal with SQ1; both are cheap Monte Carlo measurements that inform search depth limits. |
| **P2** | SQ3 — Evaluation heuristic design | Build greedy bot once enumeration exists. Region size is the obvious first heuristic. |
| **P3** | SQ4 — 3→2 player transition | Defer until 2-player bot works. The R1 3-player dynamics are harder to evaluate and less critical for the initial ≥0.80 target (which can be met in R2-only 2-player benchmarks first). |

## Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| **Branching factor explosion** — if move×wall products are larger than estimated (~50+ per turn), depth-limited search becomes impractical | High | Factored search (SQ2 hybrid approach) can prune at the move level. Fallback: MCTS with random rollouts handles high branching naturally. |
| **BFS evaluation cost** — `region_for_agent` does a full flood-fill per call; evaluating every candidate move requires N flood-fills | Medium | Cache region membership incrementally (add/remove walls from a union-find structure instead of recomputing). Or use frontier-cell proxy heuristic that avoids full BFS. |
| **3-player game tree** — minimax doesn't naturally extend to 3 players (no single "opponent" to minimize) | Medium | Use max^n or paranoid search for R1. Or simply use greedy/MCTS which handle N-player games without modification. |
| **No `enumerate_legal_moves` yet** — unlike NMM and Bagchal, the wall-go crate doesn't export this | High (blocking) | Must implement it first. The game's `available_actions` + `apply_action` provide the spec; enumeration is mechanical but requires handling both move-phase and wall-phase correctly. |
| **Game-length variance** — if random games run very long (100+ turns), benchmarking 200 matches becomes slow | Low | Cap at a turn limit (e.g. 300 turns → draw). Random-play games on a 7×7 grid with mandatory wall placement should converge faster than standard Go. |
| **R2 setup phase complexity** — the "place 2 new pawns adjacent to territory" phase adds another branching point not present in most games | Low | Small branching (usually <20 candidate cells per placement). Handle as a separate bot decision, not part of the main move+wall loop. |

---

`★ Insight ─────────────────────────────────────`
- The **P0 blocker** (no `enumerate_legal_moves`) is a pattern across this codebase: every game that has a working bot exports a public enumeration function from the game crate. NMM does it in `olympiad_game_nine_mens_morris::enumerate_legal_moves`, Bagchal in `olympiad_game_bagchal::enumerate_legal_moves`. Wall Go needs the same, and the two-phase turn structure makes it the most interesting enumeration problem of the three.
- **Union-find for incremental territory** is worth considering early. The current `region_for_agent` does a BFS flood-fill every time it's called. For a greedy bot evaluating 20+ moves per turn, that's 20+ flood-fills per decision. A disjoint-set structure where walls are "cuts" (split operations) would make territory queries O(α(n)) amortized — but split operations on union-find are non-trivial. An alternative: maintain a full adjacency graph and recompute only the affected connected component when a wall is placed/removed.
`─────────────────────────────────────────────────`

**Next step suggestion:** Would you like me to start on **SQ2** — implementing `enumerate_legal_moves` for the wall-go game crate? This unblocks all subsequent bot work. There's a meaningful design choice here (joint vs. factored enumeration) where your input on the API shape would be valuable.