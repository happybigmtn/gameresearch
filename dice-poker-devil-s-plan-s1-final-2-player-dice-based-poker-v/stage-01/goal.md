[thinking] The user is asking me to create a research goal document about Dice Poker game theory. This is a legitimate research planning request. Let me think about this carefully and produce a well-structured SMART research goal.
           
           Let me think about what I know about this topic:
           
           1. **Dice Poker / Devil's Plan**: The Devil's Plan is a Korean reality TV show where contestants play various strategy games. The Season 1 final featured a dice poker variant - a 2-player game combining dice rolling with poker-style hand ranking and betting mechanics.
           
           2. **Related games**: Poker Dice, Yacht/Yahtzee, Liar's Dice - these are all dice-based games with poker-like elements.
              
           3. **Game theory context**: Poker has been extensively studied with Nash equilibria solutions for heads-up limit Texas Hold'em, and Liar's Dice has been analyzed in imperfect information game solving literature. The standard approac...
# Research Goal: Dice Poker Nash Equilibrium

## Topic
Game-theoretic analysis of 2-player dice poker variants — specifically the class of games combining independent stochastic hand generation (dice rolls), optional re-roll mechanics, and poker-style betting with imperfect information. Motivated by the Devil's Plan S1 finale variant but generalizable to Poker Dice and related games.

## Novel Angle

**The independence gap in imperfect-information game solving.** Card poker has been the dominant testbed for imperfect-information game solving (CFR family, DeepStack, Pluribus, ReBeL). A critical but under-examined structural difference exists between card poker and dice poker: in card poker, hands are **negatively correlated** (cards dealt to me cannot appear in your hand), enabling Bayesian narrowing of opponent ranges during betting. In dice poker, hands are **stochastically independent** — my roll tells me nothing about yours.

This independence fundamentally changes the information structure of the betting game:
- **Bluffing calculus differs**: In card poker, holding a strong hand removes strong hands from the opponent's range, making bluffs against you more likely. In dice poker, hand strength is uncorrelated, so the bluff/value ratio in equilibrium should differ structurally.
- **Re-roll as a signaling/concealment mechanism**: Dice poker variants typically allow 1-2 re-rolls before betting. The number of dice re-rolled is often observable, creating a **pre-bet signaling game** with no card poker analogue. How many dice to re-roll becomes a strategic decision balancing hand improvement against information leakage.
- **Tractable but unstudied**: The game tree is small enough (5 dice, 6 faces, ~7,776 outcomes per player, 1-2 betting rounds) to solve exactly via linear programming or tabular CFR, yet no published equilibrium analysis exists for this game class.

**Why now?** The explosion of interest in imperfect-information game solving has focused almost exclusively on card-based games. Recent work on using CFR variants for smaller games, and the growing interest in "solving" games from competitive TV formats (e.g., The Genius, Devil's Plan), creates both technical tools and audience for this work. The structural comparison between independent-hand vs. correlated-hand poker is a clean theoretical contribution that existing work has not addressed.

**How this differs from standard approaches:**
- Not a "solve poker better" paper — it's about how a structural change (independence) reshapes equilibrium
- Not a pure RL/self-play paper — the game is small enough for exact solutions, enabling precise characterization rather than approximate policies
- Not a Liar's Dice paper — Liar's Dice involves sequential claiming and calling, not hand ranking with betting rounds

## Scope

Formally define a parameterized 2-player dice poker game (number of dice, faces, re-rolls, betting structure). Compute exact Nash equilibria for the Devil's Plan variant and natural parameter settings. Characterize how equilibrium bluffing frequencies, re-roll strategies, and exploitability change compared to equivalent-sized card poker games with correlated deals.

## SMART Goal

**Specific:** Compute and analyze the Nash equilibrium for 2-player, 5-dice poker with 1 re-roll round and 1 betting round (fold/call/raise), comparing equilibrium properties (bluff frequency, expected value, re-roll signaling) against an isomorphic card poker game with identical hand rankings but correlated (deck-based) deals.

**Measurable:** Deliver (1) exact Nash equilibrium strategy tables for both game variants, (2) exploitability < 10⁻⁶ (measured as epsilon in epsilon-Nash), (3) quantitative comparison of bluff frequencies and EV across ≥3 parameter settings (varying re-roll visibility, bet sizing, number of dice).

**Achievable:** Game tree size is ~60M nodes for the full 5-dice variant (7,776² × betting actions), solvable via sequence-form LP or tabular CFR on a single machine in hours. Smaller variants (3-4 dice) solvable in minutes for rapid iteration.

**Relevant:** Contributes to imperfect-information game theory by isolating the effect of hand correlation on equilibrium structure — a dimension absent from existing poker-solving literature.

**Time-bound:** 4 weeks. Week 1: formal game model + solver implementation. Week 2: equilibrium computation across parameter grid. Week 3: comparative analysis (dice vs. card variants) + re-roll signaling analysis. Week 4: writing + figures.

## Constraints

- **Compute:** Single machine, no GPU required. Sequence-form LP via SciPy/CVXPY or tabular CFR in Rust/Python. Largest variant (~60M game tree nodes) fits in <16GB RAM.
- **Tools:** Custom game tree generator, CFR or LP solver, standard scientific Python stack for analysis/visualization. No external APIs or proprietary software needed.
- **Data access:** No external data needed — the game is fully specified by its rules. The Devil's Plan variant rules are publicly documented from show transcripts and fan analysis.
- **No human subjects:** Pure computational game theory.

## Benchmark

- **Name:** No standard benchmark exists for dice poker equilibrium. This work establishes the first.
- **Evaluation metrics:**
  - **Exploitability** (epsilon): distance from Nash equilibrium, measured as max gain from best-response deviation. Target: ε < 10⁻⁶.
  - **Convergence rate**: iterations to reach target exploitability (for CFR-based solver).
  - **Bluff frequency ratio**: fraction of bets made with below-median hands at equilibrium, compared across game variants.
  - **Re-roll information value**: difference in exploitability between observable vs. hidden re-roll variants, quantifying the signaling cost.
- **Current SOTA:** No published equilibrium solutions exist for dice poker with betting. Poker Dice hand probabilities are well-known (combinatorics), but strategic analysis with betting rounds is absent. Liar's Dice has been studied (e.g., as a benchmark in the Annual Computer Poker Competition), but its sequential-claiming structure is fundamentally different. The closest related work is on Kuhn Poker and Leduc Poker as small imperfect-information game benchmarks — this work would establish dice poker as a complementary benchmark with independent hand structure.
- **Baseline:** Random play and simple heuristic strategies (e.g., "always call," "bet only with top-25% hands") serve as sanity-check baselines. The primary comparison is between exact equilibria of the dice variant vs. the card variant.

## Success Criteria

A publishable result requires:

1. **Exact equilibrium characterization** for at least the core 5-dice, 1-re-roll, 1-bet-round variant, with exploitability proof (ε < 10⁻⁶)
2. **Quantitative demonstration** that hand independence (dice) produces structurally different equilibrium strategies than hand correlation (cards) — specifically, measurably different bluff frequencies, fold thresholds, or EV distributions
3. **Re-roll signaling analysis** showing the strategic cost of observable re-rolls (how much exploitability changes when the number of re-rolled dice is hidden vs. revealed)
4. **Clean theoretical framing** of independent-hand vs. correlated-hand imperfect-information games as a general dimension of game structure, with dice poker and card poker as concrete instances
5. **Open-source solver** and equilibrium tables for reproducibility

Target venues: AAAI, AAMAS, or Games and Economic Behavior (if the theoretical contribution is strong enough). Workshop-tier: NeurIPS workshop on strategic reasoning, or AAAI workshop on game theory.

## Generated
2026-04-17T00:00:00Z