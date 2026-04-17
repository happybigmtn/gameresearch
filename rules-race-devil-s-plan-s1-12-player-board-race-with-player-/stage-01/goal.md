[thinking] The user is asking me to create a research goal document about "Rules Race" from Devil's Plan S1. This is a text-generation task about game theory research planning. Let me first understand the context better by looking at the project.
           
           Let me check if there's relevant context in the project files first, since this mentions a "research-cli" project and references the GDD.md file that's been modified.
Let me first understand the project context before drafting this research goal.

Good, there's substantial context. Let me read the Rules Race spec and the existing research.

Now let me read the research report and any existing research work on Rules Race specifically.

Now I have thorough context on the Rules Race game mechanics and the project structure. Let me also check what the `docs/kb` research cache already contains for this specific run.

Good — the existing `docs/kb` artifacts are from a *different* research run (Cooperative Puzzle), not Rules Race. Let me now read the implementation code to understand how far the rule evaluation engine is built.

I now have a comprehensive understanding of the codebase, the game mechanics, and the existing research pipeline. Let me produce the research goal document.

`★ Insight ─────────────────────────────────────`
- **Rules Race is a "commitment game"** — players lock in one immutable personal rule at setup time without seeing opponents' rules, then the system evaluates all 12+1 rules every state change. This creates a one-shot mechanism design problem where the rule grammar is the strategy space.
- **The rule grammar is intentionally compositional** — rules trigger on predicates (movement, resource gain) and produce effects (movement, resource change) that can trigger *other* rules, creating cascade chains. The spec even has a `MAX_EVAL_ITERATIONS = 1000` cap and a Court referral for overflow — the designers recognized the Turing-complete flavor.
- **The 8-alliance exploit from the show** was a resource loop: personal rules triggered on ticket-discard composed with a Group Rule that converted tickets into tickets, generating unbounded resources per turn. The losing alliance wrote rules that triggered on movement predicates that never fired because the winners' rules operated in resource-space. This is a coordination failure in predicate-space, not in strategy-space.
`─────────────────────────────────────────────────`

Here's the SMART research goal:

---

# Research Goal: Self-Referential Rule Composition and Resource-Loop Equilibria in Commitment Games

## Topic

Player-authored rule systems in competitive board games where rules compose like programs — specifically the "Rules Race" game (Devil's Plan S1), a 12-player board race with a setup phase where each player commits to one immutable personal rule from a fixed grammar, followed by a race phase where all rules evaluate on every state change with cascading triggers. The interplay between one-shot commitment (rule authoring under uncertainty), self-referential rule composition (rules whose effects trigger other rules' predicates), and the resulting resource-loop dynamics (unbounded ticket/movement generation from circular trigger chains).

## Novel Angle

**Standard game theory** treats strategy spaces as given. **Standard mechanism design** treats rule systems as authored by the designer, not the players. Rules Race inverts this: **players are mechanism designers** who each contribute one production rule to a shared reactive system, then become players within the mechanism they collectively built. This creates a two-layer game:

1. **Layer 1 (Setup)**: A one-shot simultaneous commitment game over a combinatorial grammar of condition→action rules. Each player selects one rule from ~11×10×10×9 = 9,900 valid (subject×verb)→(subject×verb) combinations (after parse-time constraints). The strategy space is the rule grammar itself.

2. **Layer 2 (Race)**: A stochastic board game where the 12 personal rules + 1 mutable Group Rule compose into a reactive system evaluated at every state change. The emergent dynamics depend on which *predicate classes* the collective rule-set triggers on — a rule that triggers on "movement" is useless if no other rule produces movement as a side-effect.

**What's unexplored**: The existing game theory literature covers mechanism design (Myerson, Maskin), commitment games (Schelling, Crawford-Sobel), and reactive rule systems (term rewriting, production systems). But the **intersection** — where heterogeneous self-interested agents each contribute one production rule to a shared reactive system, and the Nash equilibrium of the commitment game depends on the fixed-point behavior of the resulting rule composition — is not well-studied. Specifically:

- **Predicate-class coordination failure**: The show demonstrated that the decisive strategic error was not in choosing a "weak" rule but in choosing a rule whose trigger predicate belonged to a class (movement events) that no allied rule's action produced. The 8-alliance won because their rules formed a closed resource loop in ticket-space; the 4-alliance's rules were syntactically valid but semantically inert. This is a coordination problem over *predicate classes*, not over individual strategies — closer to language coordination games than to standard Nash equilibria.

- **Resource loops as emergent fixed points**: When rule A's action triggers rule B's predicate, and B's action triggers A's predicate, the composition generates an unbounded resource loop (capped only by the iteration bound). These loops are not designed by any single player but emerge from the composition of independently authored rules. Characterizing which rule profiles produce loops, which loops are beneficial vs. harmful, and whether rational agents converge on loop-producing profiles is a novel combinatorial game theory question.

- **Commitment under compositional uncertainty**: Unlike standard Bayesian games where uncertainty is over opponent types/payoffs, here the uncertainty is over how your rule *composes* with unknown rules. A rule that is optimal against one composition may be inert against another. This creates a qualitatively different information structure than standard incomplete-information games.

**Why timely**: Three recent developments create an opportunity:

1. **LLM agents as game players**: With LLM-based agents increasingly used to study strategic behavior in complex games, Rules Race is a natural testbed for studying whether language-model agents discover the predicate-class coordination insight or fall into the same trap as the show's losing alliance. Recent work on LLM game-playing focuses on negotiation and deception; rule-authoring games are unexplored.

2. **Program synthesis meets game theory**: The rule grammar is essentially a domain-specific language. Recent work in program synthesis and neurosymbolic methods has studied how agents learn to write programs; composing programs competitively (where your program interacts with opponents' programs) is a frontier application.

3. **The Autonomy project**: This codebase already implements the full Rules Race rule engine with cascade evaluation, making it possible to run large-scale simulations (millions of 12-player games) to empirically characterize the equilibrium landscape — something that would require months of engineering from scratch.

**How this differs from standard approaches**: Standard combinatorial game theory analyzes games with fixed rules. This analyzes a meta-game where the rules themselves are the strategic variable, and the game's dynamics are an emergent property of the collective rule composition. Standard mechanism design assumes a single designer; here, 12 competing designers each contribute one rule. Standard commitment game theory assumes the commitment space is simple (a number, a partition); here, the commitment space is a grammar of reactive production rules.

## Scope

A single paper covering:

1. **Formal model**: Define the "Commitment Composition Game" (CCG) — a two-layer game where N agents simultaneously commit to one production rule from a shared grammar, then play within the reactive system their rules define. Formalize predicate-class equivalence, rule-composition graphs, and resource-loop characterization.

2. **Equilibrium analysis**: Enumerate the ~9,900-rule strategy space. Classify rules into predicate-class families. Identify which families form resource loops when composed. Compute symmetric Nash equilibria (or prove non-existence) for small N (2-4 players) analytically and for N=12 via simulation.

3. **Simulation study**: Using the existing Rust rule engine, run Monte Carlo simulations of 12-player games with:
   - Random rule selection (baseline)
   - Best-response dynamics (iterated best response over the rule grammar)
   - RL agents trained via self-play on rule selection
   - (Stretch) LLM agents prompted with the rule grammar

4. **Predicate-class coordination analysis**: Measure whether equilibrium rule profiles converge on specific predicate classes, and whether the "predicate-class mismatch" failure mode from the show is a generic attractor or a degenerate case.

Out of scope: extending the rule grammar beyond what's specified; physical/UI implementation; full Olympiad tournament simulation.

## SMART Goal

**Specific**: Formalize the Commitment Composition Game for Rules Race, enumerate the equilibrium landscape of the 12-player rule-commitment game, and characterize the conditions under which self-referential rule compositions produce resource loops that dominate non-loop strategies.

**Measurable**: (a) Complete enumeration of the valid rule space with predicate-class taxonomy; (b) Nash equilibrium characterization for N=2,3,4 (analytical) and N=12 (simulation-based, ε-Nash with ε<0.05 in key payoff); (c) RL self-play convergence curves showing whether agents discover loop-forming rule profiles; (d) statistical comparison (p<0.01) of loop-aware vs. loop-naive strategy performance over ≥10,000 simulated games.

**Achievable**: The rule engine exists in Rust and can evaluate ~100k games/hour on a single core. The strategy space (~9,900 rules) is small enough for exhaustive enumeration but large enough for non-trivial equilibrium structure. RL self-play on discrete combinatorial strategy spaces is well-established. Analytical equilibrium computation for N≤4 is tractable given the rule-space structure.

**Relevant**: Advances the theory of games where players author the rules they play under — relevant to DAO governance, smart contract design, regulatory games, and AI agent coordination in open-ended environments where agents define their own interaction protocols.

**Time-bound**: 8 weeks to submission-ready draft. Week 1-2: formal model + rule enumeration. Week 3-4: equilibrium analysis (small N analytical, large N simulation). Week 5-6: RL self-play experiments. Week 7-8: writing + ablations.

## Benchmark

- **Name**: RulesRace-v0 (custom, built on existing codebase implementation)
- **Source**: `crates/olympiad/games/rules-race/` in this repository — fully implemented rule grammar, cascade evaluation engine, scoring, and game state machine
- **Metrics**:
  - **Final placement** (1st–12th) and key payoff (+3 to -5) per agent per game
  - **Resource loop incidence**: fraction of games where ≥1 cascade exceeds depth 10, 100, or hits the 1000-iteration cap
  - **Predicate-class concentration**: entropy of predicate-class distribution in equilibrium rule profiles (lower = more coordinated)
  - **Strategy dominance**: fraction of games where loop-forming rule profiles outperform non-loop profiles (measured by mean key payoff)
  - **ε-Nash gap**: maximum unilateral deviation gain in the empirical game matrix
- **Current SOTA**: None — this is a novel game formalization. No existing benchmark or published equilibrium results exist for player-authored reactive rule systems. The show itself provides one observed 12-player data point (the 8-alliance loop exploit), which serves as a qualitative validation target but not a quantitative benchmark.

## Constraints

- **Compute**: Single NVIDIA RTX 500 Ada (4 GB VRAM), laptop CPU. Rule engine runs on CPU (Rust, no GPU needed). RL training must use lightweight methods (tabular Q-learning, evolutionary strategies, or small policy networks) — not deep RL at scale.
- **Data**: No external dataset required. All data is generated via simulation using the existing rule engine. The show's single observed game provides qualitative validation only.
- **Tools**: Rust (rule engine, simulation), Python (analysis, plotting, RL training via stable-baselines3 or custom tabular methods). The existing `olympiad-games` crate provides the complete game implementation.
- **Time**: 8 weeks part-time (~20 hours/week).

## Success Criteria

A publishable result requires at least three of:

1. **Formal contribution**: A clean formalization of the Commitment Composition Game that generalizes beyond Rules Race to any game where players author reactive production rules, with well-defined solution concepts (predicate-class Nash equilibrium, loop-dominance).

2. **Enumeration result**: Complete taxonomy of the ~9,900-rule strategy space into predicate-class families, with proof or computational demonstration that the equilibrium structure depends on predicate-class composition rather than individual rule selection.

3. **Equilibrium characterization**: Either (a) analytical Nash equilibria for N≤4 showing qualitative phase transitions between loop-forming and non-loop equilibria, or (b) simulation-based ε-Nash for N=12 showing convergence to loop-dominant profiles.

4. **Empirical validation**: RL self-play agents independently discovering the loop exploit that the show's 8-alliance found, providing computational evidence that the observed human strategy was near-equilibrium rather than accidental.

5. **Negative result**: Evidence that predicate-class coordination failure (the show's 4-alliance mistake) is a robust attractor — i.e., naive agents systematically fall into inert predicate classes, making the loop exploit a dominant strategy against boundedly rational opponents.

**Target venue**: AAAI, AAMAS, or Games and Economic Behavior (if the formal theory is strong enough). The game theory + RL + program composition angle fits AAMAS particularly well.

## Generated

2026-04-17T00:00:00Z

---

`★ Insight ─────────────────────────────────────`
- **The key research insight is the two-layer structure**: Rules Race isn't just a board game — it's a *meta-game* where the strategy space is a programming language. The "predicate-class coordination" concept reframes the show's 8-vs-4 alliance outcome as a language-theoretic phenomenon: the winning coalition's rules formed a closed term-rewriting cycle in resource-predicate space, while the losers' rules were syntactically valid but referentially dangling.
- **The existing Rust engine is a major feasibility enabler**: most game-theory papers that study novel game mechanics spend half their effort building a simulator. Here, the full cascade evaluation engine with overflow detection already exists, which means the research can focus on the analysis rather than the infrastructure.
- **The RL self-play angle has real novelty**: if trained agents independently converge on loop-forming rule profiles, it would be the first demonstration of artificial agents discovering a "mechanism design exploit" through self-play — a qualitatively different finding from standard RL results on fixed-rule games.
`─────────────────────────────────────────────────`