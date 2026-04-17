[thinking] The user is asking me to create a research goal document about Blind Omok/Blind Gomoku from Devil's Plan. This is a legitimate research planning request. Let me think about this carefully and generate a well-structured research goal.
           
           Let me think about what Blind Omok is:
           - From the Korean TV show "Devil's Plan" Season 1
           - It's a variant of Gomoku (5-in-a-row) played on a 19x19 board
           - The key twist: stones are placed face-down, so players must memorize which stones belong to whom
           - Players can't see the colors of placed stones — they need to remember
           
           
           - This creates a game with imperfect information where memory becomes a critical strategic resource
           
           Now I'm considering what makes this variant interesting from a research perspective. Standard Gomoku is well-studied with known solutions for smaller boards and strong AI players, but Blind Gomoku introduces something unique: the hidde...
# Research Goal: Blind Omok / Memory-Bounded Imperfect-Recall Gomoku

## Topic

Blind Omok (Blind Gomoku), as featured in *Devil's Plan* Season 1 — a color-hidden variant of 5-in-a-row on a 19×19 board where the player must memorize stone ownership without visual feedback. The intersection of Gomoku solver theory, imperfect-information game play, and cognitively-bounded agents.

## Novel Angle

**Memory as a finite, allocable resource in imperfect-recall combinatorial games.**

Standard Gomoku is solved or near-solved on small boards, and strong heuristic players exist for 15×15 and 19×19. Imperfect-information game AI (Pluribus, ReBeL, Student of Games) has advanced rapidly — but these methods universally assume **perfect recall**: the agent remembers all past observations. Blind Omok violates this assumption in a structurally distinctive way. Unlike poker (where hidden info is the opponent's private cards) or Kriegspiel chess (where hidden info is opponent piece positions), Blind Omok hides **jointly-created public state** — both players see stones placed but neither sees colors. The information *was* available at placement time but must be retained through memory alone.

This creates a problem that existing imperfect-info game frameworks don't address well:

1. **Perfect-recall CFR/MCTS is inapplicable** because the information set structure explodes when the agent must model its own forgetting. Information Set MCTS and counterfactual regret minimization both assume the agent can reconstruct which information set it's in from the action history — but a memory-bounded agent cannot.

2. **The memory allocation problem is strategic.** With N stones on the board and memory capacity for M < N color labels, the agent must decide *which* positions to track. This is not uniform — stones near active threat patterns matter more. No existing work formulates this as an explicit resource-allocation sub-problem within game search.

3. **Timely connection to bounded rationality in AI.** There's growing interest in agents that operate under cognitive constraints (resource-rational analysis, meta-learned memory in RL, attention bottlenecks). Blind Omok provides a clean, well-defined testbed where memory pressure is the *defining* constraint, unlike Atari/board games where memory is incidental.

**Why now:** The recent wave of memory-augmented architectures (state-space models, recurrent alternatives to transformers, external memory networks) provides new tools for building agents with *structured*, *finite* memory — but these have mostly been evaluated on language and navigation tasks, not adversarial board games with combinatorial structure.

**How this differs from standard approaches:**
- Standard Gomoku AI: assumes perfect information → not applicable
- Standard imperfect-info game AI: assumes perfect recall → not applicable
- Kriegspiel/dark chess: hidden info is opponent-private → different structure
- This work: hidden info is *forgotten public state*, memory is a strategic resource

## Scope

Design and evaluate memory-management strategies for a Blind Omok agent on a 19×19 board, comparing:

1. A **belief-state agent** with bounded memory (stores color beliefs for top-K positions, selected by threat-relevance heuristic)
2. A **learned memory agent** (small recurrent network or external memory that learns what to remember via RL)
3. Baselines: perfect-recall oracle, random-recall, FIFO-recall

The study focuses on the **1-player-vs-NPC formulation** (as in Devil's Plan): one player is memory-constrained, the opponent plays with full information. This asymmetry simplifies the game-theoretic analysis while preserving the core challenge.

## SMART Goal

**Specific:** Implement a Blind Omok environment and three agent architectures (heuristic belief-state, learned memory via RL, and oracle baseline), then measure win rate and memory efficiency across memory budgets (K = 10, 20, 40, 80, all).

**Measurable:** Report win rate vs. a fixed-strength Gomoku opponent (e.g., minimax depth-4 with standard threat evaluation) across memory budgets. Plot the **memory-performance frontier**: win rate as a function of bits stored.

**Achievable:** The game environment is straightforward to implement. The heuristic agent requires no training. The RL agent can be trained with PPO on a single GPU in hours (small observation/action space: 19×19 board + K memory slots). Minimax opponent with alpha-beta pruning at depth 4 runs in milliseconds per move.

**Relevant:** Contributes to imperfect-information game theory (imperfect recall), bounded rationality, and memory-augmented RL. The Blind Omok setting is novel in the literature.

**Time-bound:** Environment + baselines: 1 week. RL agent training and experiments: 1 week. Analysis and writeup: 1 week. Total: 3 weeks.

## Benchmark

- **Name:** Blind Omok Memory Challenge (custom, no existing benchmark)
- **Source:** Self-implemented environment based on standard Gomoku rules + color-hiding mechanism from Devil's Plan S1
- **Board:** 19×19, standard Gomoku win condition (5-in-a-row)
- **Metrics:**
  - **Win rate** vs. fixed opponent (minimax-4) over 1000 games, per memory budget K
  - **Memory efficiency:** win rate normalized by memory slots used (wins per bit)
  - **Belief accuracy:** fraction of stone colors correctly tracked at each turn
  - **Critical error rate:** frequency of misremembering a stone involved in an active threat (4-in-a-row or open-3)
- **Current SOTA:** None. No prior work specifically studies Blind Omok or memory-bounded Gomoku. Standard Gomoku SOTA (perfect information) achieves near-perfect play on 15×15; on 19×19, strong heuristic engines exist but no formal solution. This work establishes the first baseline for the memory-constrained variant.

## Constraints

- **Compute:** Single GPU (RTX-class), training budget ~8 GPU-hours
- **Tools:** Rust (game engine, minimax opponent — aligns with project codebase), Python/PyTorch (RL agent training)
- **Data:** Self-play generated; no external dataset required
- **Scope limitation:** 1-player-vs-NPC only (symmetric 2-player imperfect-recall is future work)
- **Board size:** 19×19 (matching Devil's Plan), with ablations on 9×9 and 15×15

## Success Criteria

A publishable result would demonstrate:

1. **The memory-performance frontier is non-trivial.** Showing that strategic memory allocation (choosing *what* to remember) significantly outperforms naive strategies (FIFO, random) at the same memory budget — i.e., memory management is a meaningful strategic dimension.

2. **Learned memory outperforms heuristic memory.** The RL agent discovers memory strategies that are not obvious from Gomoku threat analysis alone (e.g., remembering opponent stones in surprising positions, or deliberately "forgetting" to free capacity for higher-priority tracking).

3. **Characterization of the forgetting cliff.** Identifying the critical memory threshold below which win rate collapses, and showing this threshold depends on opponent strength and game phase.

4. **Transferable insight.** The memory allocation framework generalizes beyond Gomoku — the formulation of "which public-state features to track under bounded recall" applies to any imperfect-recall game.

Target venue: Workshop paper at NeurIPS (Games track), AAAI (Game Playing), or CoG (IEEE Conference on Games).

## Generated

2026-04-17T00:00:00Z