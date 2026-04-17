[thinking] The user is asking me to decompose a research problem about "Equation High-Low" from Devil's Plan Season 1 into prioritized sub-questions. This is a legitimate research planning request about game theory.
           
           Let me first check the project context to understand what already exists - the game implementation, autoresearch infrastructure, etc.
Let me check the existing project context before decomposing this.

[thinking] Now I have comprehensive context. Let me create the research decomposition document the user requested. They want:
           
           1. Source
           2. Sub-questions (at least 4, prioritized)
           3. Priority Ranking
           4. Risks
           
           Let me think about this carefully. The game is Equation High-Low - a construct-then-bet game with:
           - Expression construction (combinatorial)
           - Poker-style betting (imperfect information)
           - High/Low/Swing direction choice (meta-game)
           - 7 players with elimination
           
           The research intersects: CFR for imperfect-information games, combinatorial optimization for expression construction, and multi-player game theory.
           
           
           I'm identifying the core research questions: how to enumerate and abstract the expression space into strategically-equivalent classes for tractable analysis, how to formalize the mixed simultaneous-sequential game structure in extensive form, and how to adapt CFR algorithms when hand ...
`★ Insight ─────────────────────────────────────`
The existing codebase gives us a strong foundation: a **complete EHL game engine** (expression parsing via Shunting Yard, 9-phase state machine, settlement with tiebreaks), a **generic autoresearch harness** (program YAML + LLM-iterated bot code + deterministic benchmarking), and **robopoker** as a reference MCCFR implementation. The key research gap is bridging these — robopoker assumes exogenous dealt hands, but EHL's hands are endogenously constructed.
`─────────────────────────────────────────────────`

---

# Research Decomposition: Equation High-Low (Devil's Plan S1)

## Source

- **Game rules authority:** `research/devils_plan_s1_game_design_report.md` S1 Day 6 Main
- **Game engine:** `crates/olympiad/games/equation-high-low/src/lib.rs` — complete Rust implementation with expression parsing, 9-phase state machine, betting, settlement, tiebreaks
- **Game spec:** `specs/160426-olympiad-games/dp-equation-high-low.md` — state types, constants, edge cases
- **Autoresearch harness:** `crates/olympiad/autoresearch/` — program loader, benchmark runner, journal (JSONL), used for NMM bot iteration
- **CFR reference:** `vendor/robopoker/` — full MCCFR solver for NLHE (external sampling, clustering, EMD, subgame solving)
- **Bot pattern reference:** `crates/olympiad/bots/nine-mens-morris/` — RandomBot + GreedyMillBot + MinimaxBot progression
- **Domain:** Imperfect-information multi-player games with endogenous hand construction; CFR; combinatorial game theory

## Sub-questions

### SQ-1: Expression Space Structure & Abstraction

**Question:** Given a hand of N cards (numbers + operators) and 3 starting operators (+, -, ÷), how large is the space of valid expressions, and can it be partitioned into strategically-equivalent abstraction classes?

**Why this matters:** This is the foundation everything else builds on. In poker, hand abstraction clusters dealt hands by equity distribution (EMD distance). In EHL, we need to cluster hands by their *achievable result set* — the range of integer values a player can produce from their cards. Two hands with identical achievable result sets are strategically equivalent regardless of which specific cards they hold.

**Decomposition:**
- Enumerate all valid expressions for representative hand sizes (3-6 cards) using the existing Shunting Yard parser
- Compute the "result spectrum" for each possible card combination: `{(result, metal_tiebreak) | valid expression}`
- Measure spectrum overlap across hands to define equivalence classes
- Estimate game tree branching factor at the construction node

**Output:** A function `achievable_results(hand: &Hand) -> Vec<(i64, MetalTiebreak)>` and empirical data on how many equivalence classes exist for simplified card sets.

---

### SQ-2: Extensive-Form Game Formalization

**Question:** How do you represent the construct-then-bet-then-declare structure as an extensive-form game suitable for CFR, given that construction and direction declaration are *simultaneous* while betting is *sequential*?

**Why this matters:** CFR operates on extensive-form game trees with information sets. EHL's structure is unusual: it interleaves simultaneous moves (construction, direction choice) with sequential moves (betting rounds). Standard CFR handles imperfect information via information sets, but simultaneous moves require either converting to a normal-form subgame or using a "nature samples order" trick.

**Decomposition:**
- Formalize the 9-phase `EqPhase` sequence as a game tree with explicit player/chance nodes
- Handle the Construct phase: model as simultaneous action where each player commits an expression, then reveal is a chance node that resolves private info
- Handle the Reveal phase: simultaneous High/Low/Swing declaration, modeled as a normal-form game embedded in the tree
- Define information sets: what does each player know at each decision point? (own hidden card, all open cards, betting history, but NOT others' expressions or direction choices until reveal)
- Compute information set count for simplified variants (2-3 players, cards 0-5, operators {+, -})

**Output:** Formal game tree specification with information sets, suitable for MCCFR implementation. Estimated info-set count for the simplified variant.

---

### SQ-3: CFR Adaptation for Endogenous Hand Construction

**Question:** How must standard MCCFR be modified when the "hand" is not dealt by nature but constructed by the player, and what convergence guarantees survive?

**Why this matters:** This is the core theoretical contribution. In poker CFR, the chance player deals cards and this chance node is factored out of the regret computation. In EHL, the "chance" at the construction phase is actually a strategic choice — each player solves `argmax/argmin expression_value(hand, target_direction)` subject to bluffing incentives. The regret update rule must account for the fact that changing your construction strategy changes the hand distribution your opponents face.

**Decomposition:**
- Identify which CFR variant is suitable: external sampling MCCFR (as in robopoker) naturally handles large action spaces by sampling; the construction action is just another action node
- The key insight: construction *is* an action, not a chance event. Each possible expression is a distinct action at the construction node. This is a large action space but finite
- Test whether vanilla MCCFR converges when the "hand abstraction" is replaced by "construction abstraction" (clustering construction actions by their result value)
- Compare convergence rate against a baseline where construction is treated as a separate optimization (solve expression optimally, then play poker with the result)

**Output:** Modified MCCFR algorithm with construction-aware regret updates. Convergence proof or empirical convergence curve for the simplified variant.

---

### SQ-4: Direction Meta-Game Equilibrium

**Question:** What are the equilibrium properties of the High/Low/Swing declaration game, and does Swing have positive expected value at equilibrium?

**Why this matters:** The direction choice is the most strategically rich part of EHL. Swing is all-or-nothing — you must beat both the High winner and Low winner. At first glance, Swing seems dominated (too risky). But if most players declare High (chasing values near 20), a Swing player with a moderate expression value (say 8-12) might win Low by default. The meta-game creates a rock-paper-scissors dynamic: too many High players → Low is profitable → too many Low players → High is profitable → Swing exploits the imbalance.

**Decomposition:**
- Solve the direction subgame in isolation: given fixed expression results for all players, what is the Nash equilibrium of the simultaneous {High, Low, Swing} declaration?
- Characterize when Swing is EV-positive: what conditions on the expression value distribution make Swing a viable strategy?
- Analyze the coupled game: how does the direction meta-game influence construction strategy? (If Swing is viable, players might construct "versatile" expressions that can win both High and Low, rather than maximizing for one direction)
- Empirical measurement: in solved equilibria of the simplified variant, what fraction of play is Swing?

**Output:** Analytical or computed Nash equilibrium of the direction subgame. Conditions under which Swing is played at equilibrium.

---

### SQ-5: Bot Progression & Autoresearch Program Design

**Question:** What is the right hierarchy of EHL bots for the autoresearch harness, and what win-rate thresholds should gate each level?

**Why this matters:** The autoresearch harness iterates bots against baselines with threshold-gated acceptance. The NMM program defined a clear progression: random → greedy-mill → minimax → iterative deepening. EHL needs an analogous progression, but the strategy space is richer (expression construction + betting + direction).

**Decomposition:**
- **Level 0 — RandomBot:** Uniform random from legal actions. Baseline.
- **Level 1 — GreedyExpressionBot:** Optimal expression construction (maximize value for High, minimize for Low), always declare High, no bluffing in betting (call/fold based on expression strength). Target: 75%+ vs random.
- **Level 2 — DirectionAwareBot:** Same as L1 but with a simple direction heuristic (declare High if result > 10, Low if < 10, never Swing). Target: 65%+ vs L1.
- **Level 3 — BettingBot:** Add betting strategy (raise with strong expressions, fold with weak). Uses hand strength estimation. Target: 60%+ vs L2.
- **Level 4 — CFR-Lite:** Precomputed blueprint strategy from simplified MCCFR. Target: exploitability < 50 mbb/h.

**Output:** Autoresearch program file `programs/equation-high-low.md` with YAML config and search direction instructions.

---

### SQ-6: Multi-Player Scaling & Elimination Dynamics

**Question:** How do 7-player dynamics and chip-based elimination affect equilibrium strategy, and can 2-player CFR results transfer?

**Why this matters:** Most CFR theory is 2-player zero-sum. EHL is up to 7 players with elimination at 0 chips. Near elimination, chip utility is non-linear (ICM in poker tournaments). A player with 2 chips plays very differently than one with 50 chips. This affects both betting strategy and direction choice.

**Decomposition:**
- Model the chip-utility function: near elimination, survival > chip accumulation
- Analyze whether the 7-player game decomposes into pairwise interactions (it partially does in betting, but direction declaration is fundamentally multi-player)
- Test whether a 2-player equilibrium strategy performs well in the 7-player setting (transfer learning from simplified solver)
- Measure the effect of chip stack asymmetry on direction choice and Swing frequency

**Output:** Empirical analysis of strategy transfer from 2-player to 7-player. ICM-like chip utility curve for EHL.

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0** | SQ-1: Expression Space | Foundation — everything else depends on knowing the combinatorial structure |
| **P0** | SQ-5: Bot Progression | Immediately actionable — creates the autoresearch program, enables iteration today |
| **P1** | SQ-2: Game Formalization | Required before any CFR implementation; informs info-set count and tractability |
| **P1** | SQ-4: Direction Meta-Game | Analytically tractable in isolation; yields publishable insight quickly |
| **P2** | SQ-3: CFR Adaptation | Core theoretical contribution but depends on SQ-1 and SQ-2 |
| **P3** | SQ-6: Multi-Player Scaling | Extension work; meaningful only after 2-player results exist |

**Recommended execution order:** SQ-1 and SQ-5 in parallel → SQ-2 and SQ-4 in parallel → SQ-3 → SQ-6

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Expression space too large for exact enumeration** — with 10 cards and operators, valid expressions could be factorial in card count | Medium | High — blocks SQ-1 and all downstream | Start with 3-5 card hands; use result-value bucketing (only ~40 distinct integer results in [1, 20] matter) to reduce effective space |
| **Simultaneous construction breaks CFR convergence** — MCCFR convergence proofs assume turn-based play; simultaneous construction may require normal-form conversion that explodes the game tree | Medium | High — blocks SQ-3 | Use the standard "nature assigns move order" trick to convert simultaneous moves to sequential with imperfect info; or treat construction as a single normal-form game solved separately |
| **7-player intractability** — even the simplified variant may have too many info sets for MCCFR when scaled to 7 players | High | Medium — limits scope but 2-3 player results are still publishable | Scope the paper to 2-3 players; frame 7-player as future work; use the autoresearch bot progression (SQ-5) for practical 7-player play |
| **Swing is never played at equilibrium** — if Swing is strictly dominated, the direction meta-game collapses to a simpler High/Low split-pot game | Low | Medium — reduces novelty | Early analytical check (SQ-4 in isolation) reveals this before heavy investment; if true, the construct-then-bet contribution still stands |
| **Existing expression parser limitations** — the current Shunting Yard implementation may not enumerate *all* valid expressions for a hand (it validates one expression, doesn't generate all) | Medium | Medium — blocks SQ-1 enumeration | Build an expression enumerator on top of the existing parser; the parser validates, enumerator generates |
| **Autoresearch harness assumes 2-player** — the current benchmark runner plays `candidate vs baseline` in a 2-player match; EHL needs multi-player support | High | Medium — blocks SQ-5 | Extend benchmark to fill remaining seats with copies of the baseline bot; this is a small engineering task |

---

`★ Insight ─────────────────────────────────────`
**The "achievable result set" is the key abstraction.** In poker, hand abstraction clusters hands by equity (probability of winning against random opponent hands). In EHL, the analog is the *result spectrum* — the set of integer values achievable from a given card hand. Two hands with the same result spectrum are strategically identical. This abstraction reduces the combinatorial explosion of expressions to a much smaller space of integer-valued outcomes, making CFR tractable. The number of distinct achievable result sets is likely orders of magnitude smaller than the number of valid expressions.
`─────────────────────────────────────────────────`

Want me to write this as an autoresearch program file (`programs/equation-high-low.md`) to kick off SQ-1 and SQ-5, or would you prefer to start with the expression space enumeration code first?