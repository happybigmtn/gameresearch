[thinking] The user is asking me to decompose a research problem about the Hexagon game from Devil's Plan S1 Final into prioritized sub-questions. They've provided a detailed research goal document and want me to generate a structured markdown document with sections: Source, Sub-questions, Priority Ranking, and Risks.
           
           This is a legitimate research planning task. I can generate this as text output since it's a synthesis/analysis task that doesn't require reading files or running commands. Let me think through the sub-questions carefully.
           
           Actually, let me check if there's an existing research structure in this repo that I should align with. The git status shows an IMPLEMENTATION_PLAN.md and some olympiad-related crates. Let me look at what exists.
Let me check the existing research structure in this repo so the decomposition aligns with your project conventions.

[thinking] Now let me look at the existing hexagon game implementation and the autoresearch harness to understand what's already built.
Let me look at the existing hexagon game implementation and the autoresearch infrastructure.

[thinking] Now I have a thorough understanding of the codebase context:
           
           1. **The Hexagon game is already implemented** in `crates/olympiad/games/hexagon/src/lib.rs` as a Rust game plugin within an Olympiad framework
           2. **The autoresearch harness** exists for iterating on bots — it benchmarks bots against baselines
           3. **The research pipeline** has already been producing knowledge base documents in `docs/kb/`
           4. **The game implementation** shows Hexagon as a buzz-and-declare game where:
              - 19 hex cells have numbers 1-10
              - A target sum is announced each round
              - Players buzz in and declare triples of cells that lie on a line and sum to the target
              - Correct answers earn +1 point, wrong or timeout answers cost -1 point
              - There are 10 rounds total
              - A study phase lets players see the numbers before they're hidden, making it a memory challenge
           
           This is fundamentally different from what...
Let me check the existing problem decomposition format and autoresearch programs.

[thinking] Now let me check the existing NMM bot and program to understand the full pattern, and the existing NMM autoresearch binary.
Now let me look at the existing NMM program and the autoresearch binary to understand the full pipeline pattern.

[thinking] Now I have a comprehensive understanding of the codebase, the research pipeline, and the Hexagon game. Let me look at the Bot trait to understand the interface.
Now I have a thorough picture. Let me look at the Bot trait and existing decomposition files for format conventions.

[thinking] Now I have a thorough understanding of:
           
           1. The Hexagon game implementation in Rust — it's a buzz-and-declare game with:
              - 19 hex cells, numbers 1-10 per cell
              - A target sum is chosen each round (3-30, with ≥3 valid triples)
              - 27 valid collinear triples defined by the hex topology
              - Study phase shows numbers, then they're hidden
              - Players buzz in and declare 3-cell triples that are collinear and sum to target
              - Correct = +1, wrong/timeout = -1
              - Silence for 3 cycles ends the round, 10 rounds total
           
           2. The autoresearch pipeline (program.md → bot → benchmark → journal)
           
           3. The existing problem decomposition format from the knowledge base
           
           4. The Bot trait interface
           
           I need to create a research decomposition grounded in the actual game mechanics. The research goal description drifts from the real implementation — this is fundamentally a memory and pattern-matching ga...
`★ Insight ─────────────────────────────────────`
- **Critical discrepancy:** The research goal describes Hexagon as a "query and reveal" game where players choose which cells to inspect. The actual implementation (`crates/olympiad/games/hexagon/src/lib.rs`) is quite different — it's a **buzz-and-declare** game where all numbers are shown during a study phase, then hidden, and players race to find collinear triples summing to a target. The "memory" is about retaining the full board from the study phase, not about sequential cell queries.
- **The 27 triples are the real structure:** The hex topology defines exactly 27 collinear 3-cell lines. The game generates a target sum and filters those 27 triples to find ones that match. Strategic depth lies in which triples to memorize and declare vs. which to sacrifice when memory is imperfect.
`─────────────────────────────────────────────────`

Here's the decomposition, corrected against the actual game implementation:

---

## Source

**Topic:** Hexagon (Devil's Plan S1 Final) — 2-player 19-hex buzz-and-declare game. Numbers 1-10 are assigned per hex and shown during a study phase, then hidden. A target sum is announced per round. Players buzz in and declare collinear 3-cell triples that sum to the target. Correct = +1, wrong/timeout = -1. 10 rounds. 27 collinear triples per board.

**Basis:** Existing Rust implementation at `crates/olympiad/games/hexagon/src/lib.rs` (995 LOC). The game plugin is registered and has a full test suite. The autoresearch pipeline (`crates/olympiad/autoresearch/`) is operational for Nine Men's Morris and can be extended to Hexagon. No Hexagon bot exists yet — only the game plugin.

**Corrective note:** The research goal describes a "query-and-reveal" information-gathering game. The actual game has a **broadcast study phase** (all numbers visible simultaneously) followed by a **hidden-board race** to declare valid triples from memory. The strategic asymmetry comes from *memory fidelity* and *buzz timing*, not from choosing which cells to query.

---

## Sub-questions

### SQ1 — Triple Enumeration & Target-Sum Combinatorics (Foundational)

**Given a 19-hex board with numbers 1-10 and a target sum T, what is the distribution of valid collinear triples, and which board positions participate in the most solutions?**

The 27 collinear triples are fixed by topology (`HEXAGON_TRIPLES` constant). For a given number assignment and target:
- How many of the 27 triples typically match? (The generator requires ≥3 per round.)
- Which hex positions appear in the most valid triples? The center cell J (index 9) participates in 8 of 27 triples — far more than corner cells (3 triples each). This makes memorizing J's number disproportionately valuable.
- What is the expected "solution density" across the target range [3, 30]?

**Deliverable:** Exhaustive stats over the seed space: per-position participation frequency in valid triples, target-sum distribution, triple overlap rates. This directly informs which cells a bot should prioritize memorizing.

### SQ2 — Optimal Memory Allocation Under Bounded Recall (Core Strategic)

**If a bot can perfectly remember only K of 19 cell values from the study phase, which K cells should it memorize to maximize expected correct declarations?**

This is the central bot-design question. The study phase reveals all 19 values simultaneously for `HEXAGON_STUDY_CYCLES` (currently 1 cycle). A perfect-memory bot stores all 19 values and exhaustively searches for valid triples. A bounded-memory bot must choose which cells to encode.

Sub-components:
- **(a) Topology-weighted priority:** Does memorizing high-degree cells (J has 8 triples, corners have 3) dominate memorizing peripheral cells?
- **(b) Target-conditioned priority:** Given target T, some number ranges are more likely to participate. Should the bot selectively memorize cells whose numbers are in the "productive" range for T?
- **(c) Cluster vs. spread:** Is it better to memorize a connected subgraph of cells (enabling inference about unmemoried neighbors via elimination) or to spread memorized cells across different lines?

**Deliverable:** Analytical framework for memory allocation + empirical win-rate curves for K = 5, 10, 15, 19 against random baseline.

### SQ3 — Buzz Timing & Declaration Race Dynamics (Game-Theoretic)

**When should a bot buzz in vs. wait, and how does the opponent's buzzing behavior change the optimal strategy?**

The buzz system creates a race: both players can buzz simultaneously, resolved by submission counter (FIFO). Buzzing and declaring correctly earns +1; buzzing and failing costs -1. Waiting costs nothing immediately but lets the opponent claim easy triples.

Sub-components:
- **(a) Confidence threshold:** At what probability of correctness should the bot buzz? With +1/-1 payoffs, the break-even is 50%, but stealing triples from the opponent raises the effective value of buzzing early.
- **(b) Remaining-solutions tracking:** The bot can infer how many valid triples remain from the solved count. When few remain, the opponent is more likely to find one — increasing the urgency to buzz.
- **(c) Opponent modeling:** Can the bot infer the opponent's memory quality from their success rate and adapt buzz timing accordingly?

**Deliverable:** Buzz policy parameterized by confidence, remaining solutions, and opponent performance.

### SQ4 — Bot Implementation Pipeline (Engineering)

**What is the concrete sequence of bot implementations needed to go from random baseline to competitive play within the autoresearch pipeline?**

Following the NMM pattern (random → greedy → minimax → tablebase):
1. **Random baseline bot** — uniform random from `{buzz, declare_triple}` with random triple selection
2. **Perfect-memory greedy** — stores all 19 values, always buzzes when a valid undeclared triple is known, picks the first valid triple found
3. **Topology-prioritized greedy** — same as above but searches high-overlap triples first (those sharing cells with future potential triples)
4. **Bounded-memory + confidence-gated** — simulates imperfect memory (configurable K), only buzzes when confidence exceeds threshold

**Deliverable:** Four bot implementations in `crates/olympiad/bots/hexagon/src/`, a `programs/hexagon.md` autoresearch program, and benchmark journal entries showing win-rate progression.

---

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0** | SQ4 (Bot pipeline) | Unblocks benchmarking. Random + perfect-memory greedy bots are prerequisites for measuring anything else. |
| **P1** | SQ1 (Triple combinatorics) | Cheap to compute, directly informs cell-priority heuristics for SQ2. Can run as analysis over the existing `round_layout` function. |
| **P2** | SQ2 (Memory allocation) | The core strategic question, but needs SQ1 stats and SQ4 bots to evaluate empirically. |
| **P3** | SQ3 (Buzz timing) | Only matters once the bot's declaration accuracy is high enough that timing becomes the marginal differentiator. |

---

## Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| **Perfect memory trivializes the game.** With 1 study cycle and 19 cells, a bot that stores all values solves the game exhaustively. The "memory" aspect only matters for human players. | High | Add configurable memory-budget parameter to bots. Benchmark at K < 19 to keep the problem interesting. Alternatively, the autoresearch program can target bot-vs-bot where both have bounded memory. |
| **Research goal misalignment.** The existing `topic_init` document describes a query-and-reveal game that doesn't match the actual Hexagon implementation. Building on that framing will produce irrelevant analysis. | High | Rewrite the research goal to match the actual game mechanics (buzz-and-declare with broadcast study phase). Already flagged above. |
| **Buzz system makes benchmarks noisy.** The FIFO buzz resolution + silence timer creates match dynamics that depend heavily on action ordering within a tick. Small bot changes may produce large win-rate swings. | Medium | Use high match counts (≥500) and report confidence intervals. The existing `BenchmarkConfig` supports this. |
| **Game plugin has no `enumerate_legal_moves` helper.** Unlike NMM which exports `enumerate_legal_moves`, Hexagon's `available_actions` returns schema-level specs (`"buzz"`, `"declare_triple"` with a JSON schema). The bot must construct valid triples itself rather than choosing from an enumerated list. | Medium | The bot needs to decode `HexagonState` from `SessionState` to enumerate valid triples. This requires the bot crate to depend on the game crate's types, or the game crate needs to export an enumeration helper like NMM does. |
| **GPU constraint (4GB RTX 500 Ada).** If SQ2/SQ3 exploration moves to RL training, the laptop GPU is undersized. | Low | Rust-native search bots (greedy, minimax) don't need GPU. RL is explicitly scoped as future work in the NMM program pattern. |