[thinking] The user is asking me to decompose a research problem about the "Scale Game" from Devil's Plan Season 1 into prioritized sub-questions. This is a legitimate research planning request — they've provided a detailed research goal document and want me to break it down into sub-questions with priorities and risks.
           
           This is a text-generation task (research decomposition), not a code task. I can respond directly without tools. Let me analyze the research goal and produce a well-structured decomposition.
           
           Let me think about the key sub-questions that need to be answered:
           
           1. **Formalization**: How do you formally model this as a multi-agent POMDP? What's the state space, action space, observation function, transition dynamics?
           
           
           2. **Information-theoretic bounds**: What's the query complexity gap between isolated rooms vs. centralized weighing? This is the core theoretical contribution.
           3. **...
# Scale Game — Research Decomposition

## Source

Research goal document: "Scale Game — Multi-Agent Weighing Puzzles" (2026-04-17). Combines classical weighing puzzle theory (Dyson 1946, information-theoretic bounds) with multi-agent game theory (Bayesian persuasion, social deduction games) and multi-agent POMDPs. Target: AAMAS-tier venue.

---

## Sub-questions

### SQ1: Formal Model — What is the correct POMDP formulation?

**What needs answering:** Define the state space (weight assignments over 7 players from integer range [1..W]), action space (which objects to place on each room's balance, plus communication actions), observation function (balance outcomes visible only to room occupants), and transition dynamics (room reassignment between rounds). Key design choice: is communication a separate action type (cheap talk) or is it embedded in player movement (carrying beliefs)?

**Why it matters:** Everything downstream — bounds, equilibrium analysis, RL training — depends on getting the formalization right. A model that's too abstract loses the combinatorial structure; one that's too concrete won't generalize.

**Open tension:** Standard multi-agent POMDPs assume simultaneous moves. The Scale Game has sequential weighings within a round (one balance per room, players take turns choosing what to weigh). This may require an extensive-form game representation layered on the POMDP, not a flat Dec-POMDP.

---

### SQ2: Information-Theoretic Bounds — How much does room isolation cost?

**What needs answering:** In the centralized case (single agent, all balances, all observations), classical results give O(log₃ N) weighings to identify N possible weight assignments. Under 3-room isolation with k weighings per room per round:

- **Lower bound:** What's the minimum total weighings (across all rooms) needed to uniquely determine all weights, assuming honest reporting and lossless inter-room communication?
- **Upper bound:** Construct an explicit weighing strategy achieving this.
- **Isolation gap:** Ratio of isolated-to-centralized query complexity. Is it constant, logarithmic, or worse in the number of players?

**Why it matters:** This is the cleanest theoretical contribution — a provable result about how spatial isolation degrades inference, independent of strategic behavior. It's also the most publishable component in isolation.

**Key insight to pursue:** Each room's balance partitions the weight space into 3 outcomes (left heavy, balanced, right heavy). With 3 rooms operating independently, you get 3³ = 27 outcomes per round vs. 3 for a single centralized weighing. But the outcomes are *correlated* (same underlying weights), so the information gain isn't simply multiplicative. The gap depends on how much each room's weighings overlap in the weight combinations they test.

---

### SQ3: Equilibrium Characterization — When does deception dominate honesty?

**What needs answering:** Model inter-room communication as cheap talk in a signaling game. A player moving from Room A to Room B reports balance outcomes from A. Under what conditions is truthful reporting a (Bayesian) Nash equilibrium?

Specific sub-sub-questions:
- **Cooperative baseline:** If all players are fully cooperative (shared utility), is honest reporting + optimal weighing strategy efficient? (This reduces to SQ2.)
- **Competitive threshold:** As the competitive component of utility increases (individual ranking matters more), at what point does a player gain by misreporting? Is there a clean phase transition parameterized by the cooperation-competition ratio?
- **Deception detection:** Can a Bayesian receiver detect lies by cross-referencing a report against their own room's observations? If so, deception has a cost — what's the equilibrium deception rate?

**Why it matters:** This is the novel game-theoretic contribution. The claim that "multi-agent strategic weighing puzzles are unstudied" stands or falls on whether the strategic layer produces non-trivial equilibria that differ from the cooperative solution.

---

### SQ4: Belief Representation — How should agents track nested uncertainty?

**What needs answering:** An agent must maintain a joint belief over: (a) the true weight assignment (combinatorial), (b) what each other player knows (epistemic), and (c) whether each other player is truthful (trust). The full belief space is intractable (exponential in players × weight range). What approximations preserve enough structure for good decisions?

Candidate approaches:
- **Factored beliefs:** Assume independence between weight uncertainty and trust uncertainty. Loses correlations but scales.
- **Particle filtering:** Sample-based representation of the joint belief. Handles correlations but variance grows with state space.
- **Constraint propagation with soft trust:** Maintain a constraint set (hard deductions from own observations) plus a probabilistic layer (soft constraints from reported observations, weighted by trust). This is domain-specific but potentially most efficient.

**Why it matters:** This determines whether the RL agents in SQ5 can actually learn, and whether the Bayesian baselines in experiments are computationally feasible. A bad belief representation makes the entire computational agenda intractable for the 7-player instance.

---

### SQ5: Agent Design — Can self-play RL discover non-obvious strategies?

**What needs answering:** Train agents via self-play on the ScaleGameSim. Compare:
- **Naive constraint propagation:** Honest, greedy weighing (maximize information gain per weighing), trust all reports.
- **Bayesian-rational:** Optimal weighing + Bayesian belief tracking with trust modeling (from SQ4).
- **Learned policy (RL):** End-to-end policy trained via PPO/MAPPO on the POMDP.

Key questions: Does RL discover strategies that qualitatively differ from Bayesian-rational? (E.g., strategic weighing choices that reveal information about *other players' strategies* rather than directly about weights. Or coordinated deception patterns.) Do learned strategies match human play patterns from the show?

**Why it matters:** Behavioral results ground the theory. If RL agents converge to honest play, that supports the "truthful equilibrium" hypothesis. If they learn to deceive, that validates the game-theoretic novelty claim. Either way, the comparison to human strategies from the show provides a qualitative sanity check.

---

### SQ6: Simulator Design — What fidelity is needed?

**What needs answering:** ScaleGameSim must encode: room assignments, balance mechanics (2-arm comparison yielding {L, R, =}), player movement between rounds, communication protocol, and scoring. Design decisions:
- **Parameterization:** Fixed at (7, 3, [1..W]) or general (n, k, [1..W])? General is more work but makes the contribution stronger.
- **Communication model:** Free-form (any message) or structured (report a set of balance outcomes)? Free-form is realistic but makes the action space huge; structured is tractable.
- **Timing:** Simultaneous or sequential weighings within a round?

**Why it matters:** The simulator is infrastructure, not a contribution in itself — but wrong design choices here waste weeks. Build for the 7-player instance first; parameterize only what's needed for ablations.

---

## Priority Ranking

| Rank | Sub-question | Rationale |
|------|-------------|-----------|
| **1** | SQ1: Formal Model | Everything depends on it. Blocks SQ2–SQ6. Budget: week 1. |
| **2** | SQ6: Simulator | Needed for all computational experiments. Co-develop with SQ1 in weeks 1–2. |
| **3** | SQ2: Info-Theoretic Bounds | Cleanest standalone contribution. Can be pursued analytically in parallel with simulator work. Weeks 2–4. |
| **4** | SQ4: Belief Representation | Gates the feasibility of SQ3 and SQ5. Must be resolved before RL training. Weeks 3–4. |
| **5** | SQ3: Equilibrium Characterization | Core game-theoretic novelty. Requires SQ1 + SQ4. Analytical + computational. Weeks 4–6. |
| **6** | SQ5: RL Agents | Strongest empirical result but most dependent on prior work. Weeks 5–7. |

**Critical path:** SQ1 → SQ6 → SQ4 → SQ5, with SQ2 and SQ3 running in parallel on the analytical track.

**Minimum publishable unit:** SQ1 + SQ2 + SQ3 (formalization + bounds + equilibrium characterization). The RL experiments (SQ5) strengthen the paper but aren't strictly necessary if the theoretical results are tight.

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Model misspecification.** The POMDP formulation doesn't capture a key game mechanic (e.g., timing of player movement, partial observability of *who* is weighing *what*), requiring rework after week 2. | Medium | High | Re-watch show episodes for edge cases during formalization. Validate model against 2–3 specific game scenarios before building simulator. |
| **Bounds are trivial.** The isolation gap turns out to be a simple constant factor (e.g., exactly 3× because 3 rooms), yielding no interesting structure. | Medium | High | If the gap is trivial under honest play, pivot to characterizing the gap *under adversarial reporting* — this is harder and more interesting. The strategic layer is where the novelty lives. |
| **Belief space intractability.** Even the 7-player instance has too many states for exact Bayesian inference, making SQ3 and SQ5 require heavy approximation that undermines the results. | Medium | Medium | Bound the weight range tightly (e.g., [1..10] gives 10⁷ ≈ 10M states — large but feasible with constraint pruning). Use the structure: most weight assignments are ruled out after 2–3 weighings, so the effective state space shrinks fast. |
| **RL doesn't converge.** Multi-agent RL in POMDPs is notoriously unstable. Self-play may cycle or converge to degenerate strategies. | High | Medium | Use MAPPO (known to work in multi-agent settings). Start with 2-player 2-room simplified instance to validate training pipeline. If full RL fails, fall back to best-response dynamics against fixed Bayesian opponents — less general but more stable. |
| **Lack of novelty recognition.** Reviewers see this as "just another social deduction game" or "just a weighing puzzle variant" without appreciating the intersection. | Medium | Medium | Frame the contribution around the *problem class* (adversarial distributed constraint satisfaction), not the specific game. Connect explicitly to distributed computing theory (consensus under Byzantine faults) as an additional framing that signals depth. |
| **Scope creep into general (n, k).** Pursuing generalization beyond 7-player instance consumes time without strengthening the core results. | Medium | Low | Hard-scope the paper to the specific instance. State generalization as future work. Only parameterize the simulator enough for ablations (vary W, vary trust level), not arbitrary (n, k). |