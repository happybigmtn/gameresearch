[thinking] The user is asking me to create a research goal document about the "Scale Game" from Devil's Plan Season 1. This is a legitimate research planning request. Let me think about this carefully and generate a well-structured research goal.
           
           The Scale Game from Devil's Plan S1 is a fascinating multi-player deduction game involving:
           - 7 players in 3 isolated rooms
           - 2-arm balances (comparison scales)
           - Hidden integer weights
           - Players must deduce weights through strategic weighing
           
           Let me think about what would be novel here. This sits at the intersection of:
           1. Weighing puzzles (classic combinatorial optimization)
           
           
           2. Multi-agent information games (game theory)
           3. Imperfect information with isolated communication
           
           The classic weighing puzzle literature assumes a single decision-maker, but the Scale Game introduces multiple agents with information asymmetry, strategic communication where p...
# Research Goal: Scale Game — Multi-Agent Weighing Puzzles

## Topic

Multi-agent weight-deduction under information isolation: the "Scale Game" from Devil's Plan S1 as a formal game-theoretic model combining classical weighing puzzles with strategic interaction, imperfect information, and distributed constraint propagation across isolated player groups.

## Novel Angle

**Classical weighing puzzles are solved — multi-agent strategic weighing puzzles are not.**

The counterfeit coin / weight-deduction literature (dating to Dyson 1946, extended through information-theoretic bounds by Shapiro, Söderberg, etc.) assumes a **single rational agent** with full access to all balance outcomes. The Scale Game breaks this assumption in three ways that, combined, create an unstudied problem class:

1. **Information isolation with partial observation fusion.** Players are split across 3 rooms, each room has its own balance, and players only observe their own room's weighings. Deduction requires fusing partial constraint sets across rooms — but the only communication channel is indirect (players move between rooms in later rounds, carrying beliefs not raw data). This creates a *distributed constraint satisfaction problem with lossy message passing*, structurally different from centralized weighing.

2. **Adversarial-cooperative hybrid incentives.** Players share some goals (collective survival) but compete individually (ranking). This means information sharing is strategic — a player may reveal truthful balance outcomes, selectively omit, or actively deceive. The weighing puzzle becomes a *signaling game* where the informativeness of each weighing depends on who controls the scale and who observes.

3. **Bayesian inference under strategic noise.** When fusing reports from other rooms, a rational agent must maintain beliefs not just over hidden weights but over **other players' truthfulness**. This is a nested belief problem (I believe player X believes player Y's weight is...) layered on top of combinatorial constraint propagation — a structure that doesn't appear in standard weighing puzzles or standard Bayesian games.

**Why timely:** Recent work on information design in games (Bayesian persuasion, Kamenica & Gentzkow line), LLM-based game agents (Diplomacy, Werewolf, Avalon), and multi-agent POMDP solvers creates both the theoretical tools and the evaluation methodology to formalize this class of problem. The Devil's Plan show provides a concrete, well-defined instance that grounds the formalization.

**What's different from standard approaches:** Standard weighing puzzle work optimizes query complexity for a single agent. Standard social deduction game work (Werewolf/Mafia) lacks the combinatorial constraint structure. This work sits at the intersection — constraint propagation under adversarial partial observability — which neither community has addressed.

## Scope

Formalize the Scale Game as a multi-agent POMDP with combinatorial state (weight assignments), define the information structure (room isolation, balance observations, inter-room communication), and:

1. Derive information-theoretic bounds on weight deduction under room isolation (how many weighings per room are needed vs. the centralized case?)
2. Implement and compare agent strategies: (a) honest constraint propagation, (b) Bayesian belief-tracking with trust modeling, (c) a learned policy via self-play RL
3. Characterize equilibrium behavior: when is truthful reporting a Nash equilibrium? When does strategic deception dominate?

Single paper scope — the formalization + bounds + computational experiments on the specific 7-player, 3-room, integer-weight instance.

## Benchmark

- **Name:** No standard benchmark exists. We construct **ScaleGameSim** — a parameterized simulator for the Scale Game (n players, k rooms, weight range [1..W], r rounds of weighing + communication).
- **Source:** Custom implementation (the show's rules are fully specified and deterministic aside from initial weight assignment).
- **Metrics:**
  - *Deduction accuracy*: fraction of games where an agent correctly identifies all/own weights
  - *Query efficiency*: number of weighings to reach ≥95% posterior confidence
  - *Exploitability*: how much a deceptive agent gains over honest baseline (expected rank improvement)
  - *Information-theoretic gap*: ratio of weighings needed under isolation vs. centralized oracle
- **Current SOTA:** None — this problem formulation is new. Closest reference points are optimal strategies for classical n-coin weighing (log₃ query complexity) and social deduction game agents (e.g., DeepRole for Avalon achieves ~62% win rate; LLM agents in Werewolf variants).

## SMART Goal

**Specific:** Formalize the Scale Game as a multi-agent POMDP, implement a simulator, derive isolation-vs-centralized query complexity bounds, and train self-play RL agents that outperform honest-Bayesian baselines in deduction accuracy and competitive ranking.

**Measurable:** (1) Formal model with proven bounds published. (2) RL agents achieve ≥15% higher deduction accuracy than naive constraint propagation under adversarial conditions. (3) Characterize at least one non-trivial equilibrium (truthful or deceptive) with proof or computational verification.

**Achievable:** The game state is small (7 players × integer weights in a bounded range ≈ hundreds of states), making exact Bayesian inference and tabular/small-network RL feasible on a single GPU. The simulator is straightforward to implement in Rust or Python.

**Relevant:** Contributes to combinatorial game theory (new problem class), multi-agent RL (structured partial observability), and Bayesian inference (adversarial constraint fusion). Connects to the growing interest in AI for social deduction / hidden-information games.

**Time-bound:** 8 weeks — formalization and simulator (weeks 1–2), bounds and honest baselines (weeks 3–4), RL agent training and equilibrium analysis (weeks 5–7), writeup (week 8).

## Constraints

- **Compute:** Single GPU (RTX-class), CPU cluster for parallel game simulations. No large-scale pretraining.
- **Tools:** Rust (game engine / simulator, aligns with project codebase), Python (RL training via stable-baselines3 or cleanrl), constraint solver (z3 or custom propagation).
- **Data access:** No external dataset needed — the game is fully synthetic/parameterized. Show episodes provide qualitative validation of human strategies.
- **Scope limitation:** Focus on the 7-player 3-room instance; generalization to arbitrary (n, k) is future work, not required for the paper.

## Success Criteria

A publishable result requires **at least two** of:

1. **Novel formalization** accepted as non-trivial: the multi-agent weighing POMDP model with room isolation is recognized as a genuinely new problem class (not reducible to existing frameworks without loss).
2. **Tight bounds:** The information-theoretic gap between isolated and centralized weighing is characterized (upper and lower bounds within a constant factor for the 7-player instance).
3. **Behavioral result:** Computational or analytical demonstration that strategic deception is (or is not) an equilibrium — with quantified payoff differences. A phase transition (e.g., "deception dominates when trust drops below threshold τ") would be strong.
4. **RL agent performance:** Self-play agents learn non-obvious strategies that qualitatively match or exceed human play patterns observed in the show (validated by comparing strategy traces).

**Target venue:** AAAI, AAMAS, or a game theory workshop (e.g., GAIW). The multi-agent + combinatorial structure fits AAMAS particularly well.

## Generated

2026-04-17T00:00:00Z