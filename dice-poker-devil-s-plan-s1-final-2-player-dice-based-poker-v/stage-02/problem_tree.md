[thinking] The user is asking me to decompose a research problem about Dice Poker game theory into prioritized sub-questions. This is a legitimate research planning request - they've provided a detailed research goal document and want me to produce a structured decomposition with Source, Sub-questions, Priority Ranking, and Risks sections.
           
           This is a text-generation task that doesn't require tools. Let me produce a well-structured research decomposition based on the detailed goal document they provided.
# Research Decomposition: Dice Poker Nash Equilibrium

## Source

**Research Goal:** Game-theoretic analysis of 2-player dice poker variants — exact Nash equilibrium computation and comparative analysis of independent-hand (dice) vs. correlated-hand (card) imperfect-information games, with re-roll signaling as a novel strategic dimension.

**Motivation:** The independence gap in imperfect-information game solving is a clean, unstudied theoretical contribution. Card poker dominates the literature (CFR, DeepStack, Pluribus), but no published equilibrium analysis exists for dice poker with betting despite tractable game tree sizes (~60M nodes for 5-dice).

---

## Sub-questions

### SQ1: Game Formalization — What is the correct extensive-form representation of parameterized 2-player dice poker?

Define the game tree precisely: initial roll (5d6), observable/hidden re-roll decision (which dice, how many), second roll, then a betting round (fold/call/raise with fixed bet sizing). Key modeling choices:
- Is re-roll count observable but not which dice? Or fully observable? Or hidden?
- How many betting rounds, and what bet structure (pot-limit, fixed-limit, no-limit)?
- How are ties resolved (split pot, or re-deal)?
- What is the hand ranking system (standard poker ranks mapped to dice, or Yacht-style categories)?

This must be parameterized so SQ2-SQ4 can sweep over variants. The formal model also determines game tree size and solver feasibility.

### SQ2: Equilibrium Computation — What are the exact Nash equilibria for the core dice poker variant and natural parameter settings?

Compute Nash equilibria via sequence-form LP (exact) or tabular CFR (approximate with convergence guarantee). Key questions:
- Is the equilibrium unique, or do multiple equilibria exist? If multiple, what is the equilibrium selection story?
- What does the equilibrium re-roll strategy look like? (e.g., always re-roll the worst dice, or sometimes hold a mediocre hand to avoid signaling?)
- What are the equilibrium betting thresholds — which hands bet, call, or fold?
- What is the game value (expected payoff to the first mover under equilibrium play)?
- How does exploitability scale with solver iterations (convergence profile)?

### SQ3: Independence vs. Correlation — How does hand independence (dice) structurally change equilibrium compared to hand correlation (cards)?

Construct an isomorphic card poker game: same hand ranking, same betting structure, but hands are dealt from a shared deck (so my hand constrains yours). Then compare:
- **Bluff frequency**: Is bluffing more or less prevalent when hands are independent? Hypothesis: independence increases bluffing because holding a strong hand doesn't reduce the opponent's probability of being strong.
- **Fold thresholds**: Do players fold more or less often?
- **EV distribution**: Is the game more volatile (higher variance) under independence?
- **Bayesian updating**: In card poker, the betting round updates beliefs about the opponent's range. In dice poker, bets carry signal about hand strength but the prior is unaffected by your own hand. How does this change the information content of a bet?

This is the core theoretical contribution — isolating correlation as a structural variable.

### SQ4: Re-roll Signaling — What is the strategic cost of observable re-rolls, and what is the equilibrium re-roll policy?

The re-roll phase is the novel strategic dimension with no card-poker analogue. Key questions:
- **Information leakage**: How much does observing the number of re-rolled dice narrow the opponent's hand distribution? (Information-theoretic: mutual information between re-roll count and final hand strength.)
- **Signaling game**: Is there a separating equilibrium (strong hands re-roll few, weak hands re-roll many) or a pooling equilibrium (all players re-roll the same number regardless)?
- **Observable vs. hidden**: Compute equilibria for both variants. The difference in game value or exploitability quantifies the "signaling cost" — the EV lost by revealing re-roll information.
- **Partial observability**: What if only the count is visible but not which dice? Does this intermediate case produce qualitatively different equilibria?

### SQ5: Solver Engineering — What is the most efficient solver architecture for this game class?

Practical sub-question enabling SQ2-SQ4:
- Sequence-form LP (exact, polynomial) vs. tabular CFR (iterative, anytime) — which is faster for the ~60M node game tree?
- Can symmetries in dice rolls (permutation equivalence of dice faces within a hand) be exploited to reduce the game tree? E.g., {1,2,3,4,5} and {5,4,3,2,1} are the same hand.
- Memory footprint: can the 5-dice variant fit in 16GB with full strategy tables, or does it require abstraction?
- Is GPU acceleration useful for CFR iterations at this scale, or is it CPU-bound?

### SQ6: Generalization — Does the independence/correlation distinction yield general theoretical principles beyond this specific game?

Step back from the specific game:
- Can we characterize a class of "independent-hand betting games" and prove general properties of their equilibria (e.g., higher bluff frequency, different bet-sizing equilibria)?
- Does the result connect to existing theory on common-value vs. private-value auctions (another setting where correlation matters)?
- Are there other game shows or recreational games that fall into this independent-hand class (e.g., certain Mahjong variants, rock-paper-scissors with stakes)?

---

## Priority Ranking

| Priority | Sub-question | Rationale |
|----------|-------------|-----------|
| **P0 — Critical path** | SQ1 (Formalization) | Everything else depends on a precise game definition. Wrong model = wrong results. Do first, validate with Devil's Plan show rules. |
| **P0 — Critical path** | SQ5 (Solver Engineering) | Must have a working solver before any equilibrium analysis. Implement in Week 1 alongside SQ1. |
| **P1 — Core contribution** | SQ2 (Equilibrium Computation) | The primary deliverable. Without exact equilibria, there is no paper. Target Week 2. |
| **P1 — Core contribution** | SQ3 (Independence vs. Correlation) | The novel theoretical angle. This is what differentiates the work from "just solving another small game." Must be addressed for any venue above workshop-tier. Target Week 2-3. |
| **P2 — Strong addition** | SQ4 (Re-roll Signaling) | The second novel dimension. Strengthens the paper significantly but is not strictly required if SQ3 delivers strong results. Target Week 3. |
| **P3 — Stretch goal** | SQ6 (Generalization) | Elevates the paper from "we solved a game" to "we identified a structural principle." Requires clean theoretical framing that may or may not emerge from the empirical results. Target Week 4 if results support it. |

---

## Risks

### Technical Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Game tree too large for exact LP** — 5-dice with full re-roll choices and multi-round betting could exceed memory | Medium | High | Start with 3-dice and 4-dice variants. Use hand-isomorphism compression (sort dice, collapse equivalent hands). Fall back to CFR if LP is infeasible. |
| **Multiple equilibria complicate analysis** — if the game has many Nash equilibria, "the equilibrium strategy" is ill-defined | Medium | Medium | Report the space of equilibria if multiple exist. Use max-entropy or trembling-hand refinement for selection. The comparative analysis (SQ3) may still hold across all equilibria. |
| **Card-poker isomorphism is not clean** — constructing a truly "isomorphic" card game (same hand ranking, same combinatorics, but correlated deals) may require a non-standard deck that feels artificial | Medium | Medium | Use a well-known card variant (e.g., 5-card draw with a stripped deck) and acknowledge the mapping isn't perfect. The key claim is about the direction of the effect, not exact magnitudes. |
| **Re-roll observability modeling ambiguity** — Devil's Plan rules may be ambiguous about what information is revealed during re-rolls | Low | Medium | Define multiple variants (fully observable, count-only observable, hidden) and compare all three. This turns the ambiguity into a feature (parameter sweep). |

### Methodological Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Null result on SQ3** — independence and correlation might not produce meaningfully different equilibria at this game size | Low-Medium | Critical | If bluff frequencies differ by <1%, the core contribution collapses. Mitigate by varying bet sizing (larger bets amplify strategic differences). Also characterize *why* the difference is small if it is — that's still a publishable insight. |
| **"Just another small game" framing** — reviewers may see this as incremental if the theoretical framing (SQ6) doesn't land | Medium | High | Lead with the structural comparison (independence vs. correlation), not with "we solved dice poker." Frame dice poker as a case study of a general principle, not the contribution itself. |
| **Devil's Plan rules under-specified** — fan reconstructions of the rules may be incomplete or incorrect | Medium | Low | The paper should define its own parameterized game inspired by the show variant, not claim to solve the exact show game. This sidesteps rule disputes. |

### Timeline Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Solver debugging eats Week 1-2** — game tree generation bugs are notoriously hard to catch (off-by-one in action indexing, incorrect information set grouping) | High | High | Validate solver on Kuhn Poker (known equilibrium) before running on dice poker. Unit test every component of the game tree builder independently. |
| **Scope creep into Liar's Dice** — reviewers or collaborators may push for Liar's Dice comparison, which has a fundamentally different game structure and would require a second solver | Medium | Medium | Explicitly scope out Liar's Dice in the paper. Mention it in related work as a different game class (sequential claiming vs. hand ranking). |

`★ Insight ─────────────────────────────────────`
**Why independence matters more than it seems:** In Texas Hold'em, card removal effects are a cornerstone of advanced strategy — knowing you hold two aces means there are only two left for opponents. Entire betting strategies are built on this negative correlation. Dice poker strips this away entirely, creating a purer test of betting-as-signaling divorced from Bayesian hand-range narrowing. This makes it a cleaner theoretical object for studying bluffing, even though it's a "simpler" game.

**The re-roll phase is structurally novel:** No card poker variant has an analogue to "I visibly discard 3 of my 5 cards and redraw" before betting begins. The closest is draw poker, but there the discards come from a shared deck (affecting opponent range). In dice poker, re-rolls are independent — the signaling is pure (no card removal confound). This makes re-roll signaling analysis (SQ4) potentially the most original contribution.
`─────────────────────────────────────────────────`