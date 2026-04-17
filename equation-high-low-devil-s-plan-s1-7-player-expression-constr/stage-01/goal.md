[thinking] The user is asking me to create a research goal document about "Equation High-Low" from Devil's Plan Season 1. This is a legitimate research planning request about a game theory topic. Let me think about this carefully and generate a well-structured research goal.
           
           Let me first understand the game:
           - Equation High-Low is from the Korean TV show "Devil's Plan" Season 1
           - It's a 7-player game involving expression construction with mathematical operators
           - Players construct mathematical expressions using +, -, ÷, √, and multiplier operators
           - There's a poker-like betting component
           - Players choose "high" or "low" direction
           
           
           - The game blends imperfect information and strategic betting with the combinatorial challenge of building expressions
           
           What makes this particularly interesting is how it merges three distinct game theory domains: imperfect-information games like poker, combinatorial...
This is a legitimate and interesting research planning request about the Equation High-Low game. Let me first check the project context to understand how this fits into your existing work.

`★ Insight ─────────────────────────────────────`
Equation High-Low is fascinating from a game theory perspective because it's a **"construct-then-bet"** game — unlike poker where hands are dealt randomly, players *build* their hands. This makes the hand distribution endogenous (strategy-dependent), which fundamentally changes the equilibrium structure and makes standard CFR assumptions more complex.
`─────────────────────────────────────────────────`

[thinking] Now let me look at the autoresearch program for nine-mens-morris to understand the pattern, and check if there's already an Equation High-Low game or any devil's plan content.
Let me check the existing autoresearch program structure and any Devil's Plan content.

[thinking] There's already an Equation High-Low spec and game implementation. Let me read both to understand the current state.
There's already an Equation High-Low game spec and implementation. Let me read them to ground the research goal properly.

[thinking] Now I have a thorough understanding of the project context. Let me also check the existing research docs and the robopoker CFR vendor to understand what CFR infrastructure exists.
Let me check the existing CFR/poker infrastructure and research docs.

[thinking] Now I have a comprehensive understanding of:
           
           1. **The game**: Equation High-Low is a 7-player poker-like game with expression construction, high/low/swing direction choice, and betting rounds
           2. **The existing implementation**: A complete game engine exists in Rust with full rules, betting, expression evaluation, and settlement
           3. **The existing CFR infrastructure**: robopoker provides a full MCCFR solver for NLHE with abstractions, external sampling, etc.
           4. **The autoresearch pattern**: The Nine Men's Morris program shows the autoresearch pattern — iterating on a bot with clear search directions
           
           
           5. **The novel research angle**: What makes this game uniquely interesting is that players construct their own expressions rather than receiving dealt hands, creating an endogenous hand distribution that's a strategic choice rather than random chance. The expression construction phase has...
Now I have deep context. Here's the research goal:

`★ Insight ─────────────────────────────────────`
**Equation High-Low is a fundamentally different beast from poker for CFR.** In poker, hands are dealt randomly (exogenous chance). In EHL, players *construct* their hand values through expression building (endogenous strategic choice). This breaks a core CFR assumption: that the "chance player" is nature, not a strategic agent. The construction phase creates a combinatorial action space (all valid expressions from cards) interleaved with betting, making information abstraction much harder — you can't just cluster "dealt hands" because hands are built, not dealt.

The robopoker vendor already provides MCCFR infrastructure for NLHE, and the Olympiad game engine has a complete EHL implementation with expression parsing, betting, and settlement. This gives you a working testbed.
`─────────────────────────────────────────────────`

---

Here's the SMART research goal document. This is a design decision point — the novel angle frames the research. Before I write this to a file, I want to walk you through the angle and see if you want to shape it.

---

# Research Goal: Equation High-Low (Devil's Plan S1)

## Topic

Imperfect-information multi-player games with **endogenous hand construction** — where players strategically build their hand values through combinatorial expression optimization, interleaved with poker-style betting and a dual-objective (high/low/swing) commitment phase. Domain intersection of game theory, combinatorial game theory, and reinforcement learning.

## Novel Angle

### What has NOT been well-studied

Standard CFR and its variants (MCCFR, Deep CFR, ReBeL, Student of Games) assume the "hand" is dealt by an exogenous chance node — nature draws cards, players observe private information, then act. The entire abstraction pipeline (hand clustering, EMD metrics, isomorphic equivalence classes) depends on this assumption.

**Equation High-Low violates this assumption.** In EHL:

1. **Hands are constructed, not dealt.** Players receive cards, but the *value* of their hand depends on how they arrange those cards into a mathematical expression. Two players with identical cards can produce different results. The "chance node" is replaced by a **simultaneous construction phase** where each player solves a combinatorial optimization problem (best expression from cards + operators) under strategic constraints (targeting high ≤20 or low ≥1).

2. **The construction strategy is private information.** Opponents observe open cards but not the hidden card or the chosen expression. This creates a novel type of information asymmetry: not just "what cards do they hold?" but "what did they *do* with those cards?"

3. **Direction choice creates a meta-game absent in poker.** The High/Low/Swing declaration after construction is a strategic commitment that splits the pot. Swing is an all-or-nothing bet that you can beat *both* sides. This creates Nash equilibrium structures fundamentally different from standard split-pot poker variants (Hi-Lo Omaha) because the construction phase lets players *aim* for specific result ranges.

4. **Multi-player (up to 7) with elimination.** Most CFR results are 2-player zero-sum. The 7-player structure with chip-based elimination creates a non-zero-sum dynamic where coalition behavior and chip-pressure are first-order effects.

### Why this is timely

- **CFR scaling to richer game classes is an active frontier.** Recent work (2022-2025) has pushed CFR into larger state spaces (Student of Games) and combined it with search (ReBeL), but the underlying game structure has remained poker-like: deal → bet → reveal. Games with construction phases (Scrabble-like, expression-building) have been studied in the perfect-information combinatorial optimization literature but NOT through the lens of imperfect-information equilibrium computation.

- **LLM-driven game agents need new benchmarks.** The rise of LLM-based game agents (GPT-4 playing Diplomacy, etc.) has created demand for games that test *both* mathematical reasoning (expression construction) and social reasoning (betting/bluffing). EHL is a natural benchmark for this.

- **The game engine already exists.** The Olympiad `dp.equation_high_low` plugin provides a complete, tested Rust implementation with expression parsing, betting rounds, settlement, and rendering — a ready-made environment for algorithmic research.

### How this differs from standard approaches

| Dimension | Standard Poker CFR | EHL Research |
|---|---|---|
| Hand origin | Exogenous chance node | Endogenous construction |
| Private info | Card identity | Card identity + expression choice |
| Action space (construction) | N/A | Combinatorial (all valid expressions) |
| Objective | Maximize chips | Dual-objective (high vs. low) with swing meta-game |
| Players | 2 (most results) | 2-7 with elimination |
| Abstraction basis | Card equity distributions | Expression value distributions conditioned on direction |

## Scope

Focused enough for a single paper: **CFR for games with endogenous hand construction.**

- Formalize the "construct-then-bet-then-commit" game class as an extensive-form game with simultaneous moves
- Develop an information abstraction for EHL that clusters hands by their *expression value distribution* (what results are achievable from a given set of cards) rather than by dealt cards
- Implement a simplified EHL variant (2-3 players, reduced card set, single hand) and solve it with MCCFR
- Measure exploitability and compare strategy profiles against baselines (random, greedy-expression, optimal-expression-no-bluff)
- Characterize the equilibrium properties: does the direction meta-game have a mixed-strategy Nash equilibrium? What fraction of the equilibrium strategy involves Swing?

## SMART Goal

**Specific:** Develop and validate a CFR-based solver for a simplified Equation High-Low variant that handles endogenous hand construction, and characterize the resulting equilibrium strategies.

**Measurable:** (1) Exploitability of the computed strategy ≤ 1% of the pot per hand in the simplified variant. (2) Win rate against random baseline ≥ 80% over 10,000 hands. (3) Win rate against greedy-expression baseline ≥ 65%. (4) Characterize the equilibrium High/Low/Swing frequency distribution.

**Achievable:** The simplified 2-3 player variant with reduced card set (numbers 0-5, no metals, 2 operators) has an estimated game tree of ~10^6 information sets — well within MCCFR capacity on a single GPU in hours.

**Relevant:** Advances the theory of CFR in games with endogenous hand construction, a game class that appears in real competition (Devil's Plan, math olympiads) and has no existing equilibrium analysis.

**Time-bound:** 4 weeks to formalization + implementation; 2 weeks to experiments + writing.

## Constraints

- **Compute:** Single GPU (or CPU-only), training time ≤ 8 hours for the simplified variant
- **Tools:** Rust (existing `olympiad-game-equation-high-low` crate + `robopoker` MCCFR infrastructure as reference), Python for analysis/plotting
- **Data:** No external dataset needed — the game is fully self-contained; training is self-play
- **Scope limitation:** Full 7-player EHL with complete card deck is computationally intractable for exact CFR; the paper focuses on a tractable simplified variant and extrapolates insights

## Benchmark

**Name:** EHL-Simplified (custom, introduced by this work)

**Source:** Derived from the `dp.equation_high_low` game spec. 2-3 players, cards 0-5 (no metal tiebreak), operators {+, -, ×}, single hand (no elimination dynamics).

**Metrics:**
- Exploitability (epsilon-Nash gap) in milli-big-blinds per hand (mbb/h)
- Head-to-head win rate vs. baselines (random, greedy-expression, optimal-expression-no-bluff)
- Strategy profile: High/Low/Swing frequency distribution at equilibrium
- Construction strategy: expression value distribution conditioned on direction choice

**Current SOTA:** None. No published equilibrium analysis of Equation High-Low or any construct-then-bet game class exists. The closest related work is:
- Hi-Lo Omaha poker analysis (split-pot games), but these lack the construction phase
- Pluribus/Libratus (multi-player poker CFR), but these assume exogenous hands
- Combinatorial game solving (expression games, Countdown), but these are perfect-information

This work would establish the first baseline.

## Success Criteria

A publishable result would demonstrate:

1. **Formalization:** A clean extensive-form game representation of the construct-then-bet-then-commit game class, showing it is not reducible to standard poker
2. **Algorithm:** An MCCFR variant that handles the simultaneous construction phase, with proven convergence in the simplified variant
3. **Equilibrium insight:** At least one non-obvious property of the equilibrium (e.g., "Swing is played X% of the time at equilibrium despite being all-or-nothing", "optimal construction diverges from max-expression-value by Y% due to bluffing incentives in the betting round", "the hidden card creates Z bits of information advantage")
4. **Baseline comparison:** Clear win-rate advantage over heuristic baselines, demonstrating that the CFR solution captures strategic value from the interaction between construction, betting, and direction choice

Target venue: AAAI, AAMAS, or NeurIPS (Game Theory / RL workshop)

## Generated

2026-04-17T00:00:00Z

---

`★ Insight ─────────────────────────────────────`
**Why the "construct-then-bet" structure matters beyond this game:**
Many real-world strategic situations have this pattern — you prepare something (a business plan, a legal argument, a military formation) and then negotiate/compete with imperfect knowledge of others' preparations. The construction phase makes the game theoretically richer than standard poker because the "hand distribution" at the betting stage is *itself* the result of strategic optimization, creating a nested equilibrium problem. If this CFR approach works, it could generalize to any game where preparation precedes competition.
`─────────────────────────────────────────────────`

Would you like me to write this to a file (e.g., as an autoresearch program like the NMM one), or do you want to adjust the novel angle, scope, or simplification strategy first?