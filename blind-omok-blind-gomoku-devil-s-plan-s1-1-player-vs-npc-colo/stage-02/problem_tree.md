[thinking] The user is asking me to decompose a research problem about Blind Omok/Blind Gomoku into prioritized sub-questions. They've provided a detailed research goal document and want me to generate a structured markdown output with sections: Source, Sub-questions, Priority Ranking, and Risks.
           
           This is a legitimate research planning task. Let me analyze the research goal and decompose it into meaningful sub-questions. I'll do this as text output since this is a research strategy/planning task that doesn't require tool use.
           
           Let me think about the key dimensions of this research problem:
           
           1. **Game environment design** - How to properly formalize Blind Omok as a game-theoretic object
           
           
           2. **Memory management as strategy** - How to model and allocate finite memory
           3. **Agent architectures** - What agent designs work for memory-bounded play
           4. **Gomoku solver theory** - What existing Gomoku theo...
## Source

Research goal document: *Blind Omok / Memory-Bounded Imperfect-Recall Gomoku* (2026-04-17). Derived from the color-hidden Gomoku variant in *Devil's Plan* Season 1. Core premise: standard imperfect-information game AI assumes perfect recall; Blind Omok violates this by hiding jointly-created public state, making memory a finite strategic resource.

---

## Sub-questions

### SQ1 — Game Formalization
**How should Blind Omok be formalized as a game-theoretic object, and what existing framework (POMDP, imperfect-recall extensive-form game, decDec-POMDP) best captures the structure of "forgotten public state"?**

This is foundational. The choice of formalism determines which solution concepts apply, which algorithms are candidates, and how the information-set explosion from self-forgetting is handled. Key sub-issues:
- Is the color-hiding mechanism best modeled as nature moves (revealing color only at placement time then removing the signal), or as an observation function with a decaying memory buffer?
- Does the asymmetric variant (NPC has perfect info, player is memory-bounded) reduce to a single-agent POMDP with a particular belief-space structure?
- What is the branching factor of the belief space as a function of memory budget K and stones-on-board N?

### SQ2 — Memory Allocation Strategy
**Given a fixed memory budget of K slots on a board with N > K stones, what principles govern optimal allocation of memory to stone positions — and is this problem separable from move selection?**

This is the novel strategic dimension the research goal identifies. Sub-issues:
- Can memory allocation be decomposed into a threat-relevance scoring function (which positions matter most to remember) that is updated each turn?
- Is there a formal connection to attention mechanisms — i.e., is "choosing what to remember" equivalent to a hard-attention bottleneck over board state?
- Does optimal memory allocation depend primarily on **own** threats (tracking your attack lines) or **opponent** threats (tracking their defense/attack), and does this shift across game phases (opening → midgame → endgame)?
- What is the information-theoretic minimum memory needed to play at X% of perfect-recall performance? (Lower bound analysis.)

### SQ3 — Agent Architecture Design
**What agent architecture best exploits structured, finite memory for adversarial board-game play — heuristic belief tracking, learned recurrent memory, or explicit external memory?**

This directly addresses the three-agent comparison in the scope. Sub-issues:
- For the **heuristic agent**: which Gomoku threat taxonomy (open-4, broken-3, etc.) best defines the priority ordering for memory slots? Does the standard threat-space search hierarchy transfer directly?
- For the **learned agent**: what observation encoding enables the RL agent to learn memory management implicitly? Does it need explicit "write/read/forget" actions on a memory buffer (NTM-style), or can a recurrent hidden state learn this end-to-end?
- For the **RL training regime**: is self-play feasible given the asymmetry (agent is memory-bounded, opponent is not), or must the agent train against a fixed opponent curriculum?
- What is the right action space factorization — joint (place stone, update memory) or sequential (place stone, then decide memory update)?

### SQ4 — Gomoku Theory Transfer
**Which results from standard Gomoku solver theory (threat-space search, proof-number search, known opening theory) remain valid under imperfect recall, and which break?**

Standard Gomoku on 15×15 with opening restrictions is solved (first player wins). On 19×19, strong engines use threat-space search (Allis 1994) and proof-number search. Under Blind Omok:
- Threat-space search requires knowing which stones are yours — how does uncertainty in stone ownership degrade threat detection accuracy?
- Can a **probabilistic threat-space search** be defined, where each threat pattern has a confidence weight based on memory reliability?
- Do known opening sequences (e.g., indirect openings in Renju) remain effective when the player may lose track of their own stones?
- At what memory budget does the first-player advantage (if any) disappear?

### SQ5 — Evaluation Methodology
**What metrics and experimental protocols validly measure the quality of memory-bounded play, beyond raw win rate?**

Win rate alone conflates memory strategy quality with move-selection quality. Sub-issues:
- How to isolate memory management quality: compare agents with identical move-selection but different memory strategies?
- Is **belief accuracy** (% of stones correctly tracked) the right proxy, or should it be weighted by tactical relevance (tracking a stone in a live-4 matters more than one in a dead position)?
- How to construct the **memory-performance frontier**: sweep K, but also vary opponent strength (minimax depth 2/4/6/random) to see if the frontier shape is opponent-dependent?
- What board sizes (9×9, 15×15, 19×19) are needed to separate memory pressure from game complexity?

### SQ6 — Forgetting Dynamics
**Is there a critical memory threshold (phase transition) below which performance collapses, and does this threshold depend on opponent strategy, game phase, or board density?**

The research goal hypothesizes a "forgetting cliff." Sub-issues:
- Is the transition sharp (phase-transition-like) or gradual? Analogy to percolation thresholds in random graphs — when enough stone-ownership labels are lost, threat detection may fail catastrophically rather than degrading gracefully.
- Does the critical threshold shift across game phases? Early game has few stones (memory sufficient) while late game has many (memory saturated) — is mid-game the critical regime?
- Can an agent learn to **induce forgetting pressure on the opponent** — e.g., playing moves that create many dispersed threats requiring the opponent to track more positions?

---

## Priority Ranking

| Rank | Sub-question | Rationale |
|------|-------------|-----------|
| **P0** | SQ1 — Game Formalization | Everything downstream depends on the formal model. Wrong formalism → wrong algorithms → wasted effort. Must be settled first. |
| **P0** | SQ2 — Memory Allocation Strategy | This is the **core novelty claim**. If memory allocation turns out to be trivially solved by a simple heuristic, the research contribution collapses. Must be investigated early to validate the premise. |
| **P1** | SQ4 — Gomoku Theory Transfer | Determines whether existing Gomoku engines can be adapted (saving months) or must be rebuilt. Also identifies which theoretical results can be cited vs. must be re-derived. |
| **P1** | SQ3 — Agent Architecture Design | Depends on SQ1 (formalism constrains architecture) and SQ4 (what Gomoku components to reuse). Second wave of work. |
| **P2** | SQ5 — Evaluation Methodology | Important for paper quality but not a research blocker. Can be refined during experiments. |
| **P2** | SQ6 — Forgetting Dynamics | Empirical investigation that happens naturally during SQ3 experiments. High interest for the paper's story but not prerequisite for anything. |

---

## Risks

| Risk | Severity | Likelihood | Mitigation |
|------|----------|------------|------------|
| **Memory allocation is trivially solved by "track the most recent K stones."** FIFO or recency-based heuristic matches learned agents, eliminating the core contribution. | **Critical** | Medium | Run FIFO baseline *first* (week 1). If it performs within 5% of oracle at K=40, pivot the framing toward characterizing *why* simple strategies suffice (still publishable, different story). |
| **Belief-state explosion.** With N stones and K memory slots, the number of possible belief states is combinatorial. If the POMDP is intractable even approximately, the heuristic agent may be the only viable approach. | **High** | Medium | Use factored belief representation (per-position color probability, not joint distribution). Accept independence assumption as approximation and measure its cost. |
| **RL agent fails to learn meaningful memory strategies.** PPO with a small recurrent net may not discover non-trivial memory allocation in the 8 GPU-hour budget. | **High** | Medium-High | (a) Pre-train on supervised memory task (predict stone colors from game history with bounded context). (b) Use curriculum: start with small boards (9×9) and low N, gradually increase. (c) Add auxiliary reward for belief accuracy to shape learning. |
| **Minimax-4 opponent is too weak or too strong.** If too weak, memory doesn't matter (agent wins regardless). If too strong, agent loses regardless of memory. | **Medium** | Medium | Calibrate by measuring perfect-recall agent win rate vs. minimax at depths 2, 4, 6, 8. Choose the depth where perfect-recall wins 60–80%, leaving room for memory-bounded agents to show differentiation. |
| **No existing literature on imperfect-recall combinatorial games.** The "novel angle" claim is strong, but if related work exists in operations research or cognitive science under different terminology (e.g., "forgetful games," "bounded-recall strategies"), the novelty is undermined. | **Medium** | Medium | Conduct targeted literature search: "imperfect recall extensive form games" (Piccione & Rubinstein 1997, Kaneko & Kline 2008), "absentminded driver problem," "bounded memory in repeated games" (Neyman 1985, Kalai 1990). Check cognitive science for "memory in board games" (de Groot, Chase & Simon on chess chunks). |
| **19×19 board is too large for meaningful experiments within compute budget.** Even minimax-4 on 19×19 may be slow without heavy optimization; RL training on full board may not converge. | **Medium** | Low-Medium | Primary experiments on 9×9 and 15×15; use 19×19 only for final validation runs with the best agent. Implement Gomoku engine in Rust (already in project codebase) for speed. |
| **Scope creep into symmetric 2-player.** Reviewers may demand the symmetric case where *both* players have bounded memory. This requires equilibrium concepts for imperfect-recall games, which are poorly understood. | **Low** | High (at review time) | Explicitly scope out in the paper. Frame asymmetric case as the "Devil's Plan formulation" (faithful to source material). Acknowledge symmetric case as future work with a brief discussion of why it's harder (equilibrium non-existence under imperfect recall, Kuhn 1953). |