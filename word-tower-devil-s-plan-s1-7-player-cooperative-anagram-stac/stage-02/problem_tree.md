[thinking] The user is asking me to decompose a research problem about Word Tower from Devil's Plan S1 into prioritized sub-questions. They've provided a detailed research goal document and want me to produce a structured markdown document with sections: Source, Sub-questions, Priority Ranking, and Risks.
           
           This is a legitimate text-generation/research-planning task. I don't need tools for this — it's analytical writing based on the provided context. Let me produce a high-quality decomposition.
           
           Let me think carefully about the research problem:
           
           Word Tower is a cooperative game where 7 players pool letter cubes to build stacked words. 
           
           The research goal is to build an AI solver that maximizes tower height using LLM-guided constraint search. I'm identifying the core technical challenges: how to formally represent the problem as either an MDP, CSP, or hybrid approach; how to generate valid word c...
# Research Decomposition: Cooperative Word Tower Optimization

## Source

**Origin document:** Research Goal — *Cooperative Word Tower Optimization*
**Domain:** Cooperative combinatorial optimization × constrained NLP
**Game:** Word Tower (Devil's Plan S1E3) — 7-player cooperative anagram stacking with cube budget

---

## Sub-questions

### SQ-1: Problem Formalization — What is the right formal model?

**Core question:** Should Word Tower be modeled as a constrained MDP, a constraint-satisfaction problem (CSP) with sequential structure, or a cooperative planning problem with factored state?

**Why it matters:** The formalization determines the entire solver architecture. An MDP framing enables RL extensions but may overfit to sequential decision-making when the real structure is closer to a resource-allocation CSP. A CSP framing captures constraints cleanly but may miss the sequential depletion dynamics.

**Sub-components:**
- Define the state space: joint inventory (7 player multisets) × placed-word sequence × remaining layer requirements
- Define the action space: (word, allocation-map) pairs where allocation-map assigns each letter in the word to a specific player's inventory
- Characterize branching factor: for a 5-letter word with 7 players, the number of valid allocation-maps can be combinatorial — is this tractable?
- Identify whether the problem is closer to cooperative multi-agent planning (Dec-POMDP family) or centralized planning with distributed resources

**Deliverable:** Formal problem definition with complexity analysis (NP-hardness proof or reduction to known problem).

---

### SQ-2: Word Proposal — How to efficiently generate feasible candidate words from pooled inventories?

**Core question:** Given 7 players' inventories (a distributed multiset of letter cubes), how do you efficiently enumerate or sample valid English words that can be constructed from the union of inventories?

**Why it matters:** This is the inner loop of any solver. Naive dictionary enumeration (check all 280K words against the pooled inventory) is feasible per-layer but becomes expensive when multiplied by look-ahead depth. The LLM-guided approach promises to focus search on high-quality candidates, but the quality of that focus is an empirical question.

**Sub-components:**
- **Trie-based enumeration:** Build a trie from the dictionary, prune branches where the pooled inventory lacks required letters. What is the empirical speedup over brute-force?
- **LLM-as-proposal-distribution:** Prompt a small LLM with the available letters and ask for valid words. How does proposal quality (precision, recall, diversity) compare to trie enumeration?
- **Hybrid approach:** Use trie enumeration for short words (≤6 letters) where it's fast, LLM proposals for longer words where the search space is sparser. Where is the crossover point?
- **Minimum length constraints:** How do per-layer minimum-length requirements interact with inventory depletion? At what layer does feasibility typically collapse?

**Deliverable:** Candidate generation module with latency and quality benchmarks (proposals/second, fraction of valid words, coverage of optimal words).

---

### SQ-3: Resource Allocation — How to optimally assign cubes across players for a chosen word?

**Core question:** Given a target word and 7 player inventories, how should cube contributions be allocated to minimize damage to future feasibility?

**Why it matters:** The same word can be constructed from different player allocations, and the choice affects what each player can contribute to future layers. This is a **multi-agent bin-packing subproblem** embedded inside each layer's decision. A greedy allocation (take from whichever player has the most of that letter) may be far from optimal when future layers are considered.

**Sub-components:**
- **Myopic allocation:** Assign each letter to the player with the largest surplus of that letter. Baseline; expected to be decent but suboptimal.
- **Entropy-preserving allocation:** Allocate to maximize the entropy (diversity) of remaining inventories across all players. Preserves optionality.
- **Look-ahead allocation:** Evaluate allocation choices by simulating future layers. Computational cost vs. quality tradeoff.
- **Allocation as auxiliary optimization:** For a fixed word sequence, is the allocation problem independently solvable (e.g., as a min-cost flow)?

**Deliverable:** Allocation strategies with ablation showing impact on tower height when word sequence is held constant.

---

### SQ-4: Look-Ahead Planning — How to evaluate the long-term impact of placing a word now?

**Core question:** Given the cascading constraint structure (spent cubes are gone forever), how deep must the solver look ahead to avoid the horizon effect, and what heuristics make deep look-ahead tractable?

**Why it matters:** This is the central algorithmic challenge identified in the research goal. Greedy solvers will place long words early and run out of rare letters. But exhaustive look-ahead is exponential. The practical question is: what is the minimum look-ahead depth that captures most of the value, and what pruning heuristics keep it tractable?

**Sub-components:**
- **Horizon effect characterization:** On synthetic instances, how often does the greedy solver's tower height fall short of optimal? By how many layers? This quantifies the cost of myopia.
- **Beam search with inventory heuristic:** Use a heuristic (e.g., maximum number of feasible words from remaining inventory) to prune beam search. What beam width × look-ahead depth is sufficient?
- **Constraint propagation as pruning:** After tentatively placing a word, propagate constraints (remaining inventory → feasible word set) to detect dead-ends early. How effective is arc consistency here?
- **LLM as value estimator:** Can the LLM estimate "how many more words can you make from these letters" as a look-ahead heuristic? Calibration and reliability.

**Deliverable:** Look-ahead module with scalability curves (tower height vs. look-ahead depth vs. wall-clock time).

---

### SQ-5: Benchmark Design — How to generate representative and challenging game instances?

**Core question:** What distribution over cube allocations produces instances that are (a) solvable (non-trivial tower possible), (b) challenging (greedy is suboptimal), and (c) representative of the actual game's difficulty?

**Why it matters:** Without a well-designed benchmark, results may be artifacts of easy or degenerate instances. The 5 reconstructed broadcast instances are too few for statistical significance; the synthetic generator must produce diverse, meaningful instances.

**Sub-components:**
- **Letter distribution:** English bigram-weighted vs. uniform vs. Scrabble-tile-frequency. Which produces the most interesting (non-trivial, non-degenerate) instances?
- **Total cube count parameterization:** Too few cubes → trivially short towers; too many → trivially solvable. What is the "interesting" range?
- **Difficulty estimation:** Can you predict instance difficulty (optimal tower height) without solving it? Useful for stratified evaluation.
- **Validation against broadcast data:** Do synthetic instances match the qualitative difficulty of the 5 reconstructed Devil's Plan instances?

**Deliverable:** Open-source `wordtower-bench` generator with difficulty-stratified instance sets.

---

### SQ-6: LLM Integration Architecture — How to couple the neural and symbolic components efficiently?

**Core question:** What is the right interface between the LLM (word knowledge, feasibility intuition) and the classical search (constraint propagation, beam search, allocation optimization)?

**Why it matters:** Tight coupling (LLM called at every search node) may be too slow. Loose coupling (LLM generates a fixed candidate set upfront) may miss context-dependent opportunities. The architecture choice determines both quality and latency.

**Sub-components:**
- **Proposal caching:** Generate LLM proposals once per unique inventory state, cache aggressively. Hit rate?
- **Iterative refinement:** Classical search identifies "stuck" states, queries LLM for creative word proposals targeting specific leftover letters. How often does this rescue otherwise-dead search branches?
- **No-LLM ablation:** How much of the solver's performance comes from the LLM vs. the search? If trie enumeration + good search matches LLM-guided search, the LLM contribution is marginal.

**Deliverable:** Architecture comparison (tight vs. loose vs. iterative coupling) with latency-quality Pareto curves.

---

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0** | SQ-1: Formalization | Everything else depends on the formal model. Without it, solver design is ad hoc. This is also the paper's first intellectual contribution. |
| **P0** | SQ-5: Benchmark | No evaluation without a benchmark. Must be built early so all experiments use it. Co-developed with formalization. |
| **P1** | SQ-2: Word Proposal | The inner loop — determines what moves are available. Must be fast and high-recall. Blocks solver development. |
| **P1** | SQ-4: Look-Ahead Planning | The core algorithmic novelty. Directly determines whether the solver beats baselines. But depends on SQ-2 for candidate generation. |
| **P2** | SQ-3: Resource Allocation | Important for quality but secondary — a decent heuristic (entropy-preserving) may suffice for the core paper. Full optimization is an enhancement. |
| **P2** | SQ-6: LLM Integration | The "novel angle" claim depends on this, but the solver can work with trie enumeration alone. LLM integration is the differentiator, not the foundation. |

**Execution order:** SQ-1 and SQ-5 in parallel (week 1–2) → SQ-2 (week 2–3) → SQ-4 (week 3–4) → SQ-3 and SQ-6 in parallel (week 4–5) → integration and experiments (week 5–6).

---

## Risks

### R1: The LLM contribution is marginal (HIGH probability, HIGH impact)

**Threat:** Trie-based dictionary enumeration over 280K words with inventory filtering may be fast enough and complete enough that adding an LLM to propose words provides negligible improvement. The paper's novel angle collapses.

**Mitigation:** Design the benchmark to include large inventory sizes (50–70 cubes) where trie enumeration produces thousands of candidates and the LLM's ability to *rank* or *prioritize* words becomes valuable even if enumeration is tractable. Also position the LLM's value as being in **look-ahead estimation** (SQ-4), not just proposal generation.

**Contingency:** If ablation shows LLM adds <5% tower height, reframe the contribution as the **formalization + benchmark + classical solver**, with LLM integration as a negative/null result (still publishable if the benchmark is strong).

### R2: Combinatorial explosion in allocation (MEDIUM probability, MEDIUM impact)

**Threat:** For longer words (8+ letters) with 7 players, the number of valid allocation-maps may be intractable, making per-word allocation optimization a bottleneck.

**Mitigation:** Prove that allocation for a fixed word reduces to min-cost flow or assignment problem (polynomial). If not, use greedy allocation (surplus-first) as the default and show it's within a bounded factor of optimal.

### R3: Benchmark instances are degenerate (MEDIUM probability, HIGH impact)

**Threat:** Randomly generated cube distributions may be either trivially solvable (tower height = total cubes / avg word length, no interesting choices) or trivially infeasible (too many Q/Z/X, almost no words possible).

**Mitigation:** Pilot the generator early (week 1). Filter instances: require that greedy achieves ≥3 layers AND exhaustive search achieves ≥greedy+2 layers (proving look-ahead matters). Tune letter distributions against the 5 broadcast instances for calibration.

### R4: Reconstructed broadcast instances are unreliable (LOW probability, MEDIUM impact)

**Threat:** Reconstructing exact cube distributions from broadcast footage may be error-prone (cubes not fully visible, editing cuts). The 5 "ground-truth" instances may be inaccurate.

**Mitigation:** Treat broadcast instances as **qualitative validation only** — check that solver behavior matches the game's qualitative dynamics (early layers easy, late layers hard, rare-letter management matters). Do not use them for quantitative claims. All statistical results come from synthetic instances.

### R5: Scope creep into multi-agent communication (LOW probability, MEDIUM impact)

**Threat:** Reviewers may expect the cooperative aspect to involve actual multi-agent communication protocols (negotiation, information sharing), not centralized planning over distributed inventories.

**Mitigation:** Frame explicitly as **centralized cooperative planning** — the solver has full observability of all inventories. Position Dec-POMDP / communication learning as future work. Justify centralized framing: in the actual game, players discuss openly, so full observability is realistic.

### R6: No existing SOTA makes claims hard to position (MEDIUM probability, LOW impact)

**Threat:** Without an existing SOTA to beat, the "20% improvement over baselines" claim is self-referential — you're beating your own baselines.

**Mitigation:** Ensure baselines are strong and well-motivated (greedy = natural human strategy; exhaustive = theoretical ceiling for small instances). Compare against existing **general-purpose cooperative planners** (e.g., MCTS-based cooperative planning from Hanabi literature) adapted to this domain. This provides an external reference point.

---

`★ Insight ─────────────────────────────────────`
- **The "is the LLM even useful?" risk (R1) is the most important one to address early.** Run a quick pilot in week 1: take 20 instances, compare trie enumeration vs. LLM proposal for candidate word quality. If the LLM isn't clearly better at *something* (ranking, rare-word discovery, look-ahead estimation), the paper's framing needs to shift before you're committed.
- **Allocation (SQ-3) is likely a min-cost flow problem** — each letter in the word is a "demand node," each player's inventory of that letter is a "supply node," and the cost is some function of remaining inventory diversity. This reduction, if it works, makes allocation polynomial and removes R2 entirely. Worth attempting in the formalization phase.
- **The benchmark is arguably the most durable contribution.** Solvers improve; clean problem formalizations and benchmarks persist. Investing heavily in `wordtower-bench` quality pays off even if the solver results are modest.
`─────────────────────────────────────────────────`