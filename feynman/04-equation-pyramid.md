# Chapter 4 — Equation Pyramid

*The fourth game, a 2-player arithmetic race, and the first one where
our benchmark harness has run into a mechanism-level limitation.*

---

## The rules

Equation Pyramid is a 2-player game over **ten rounds**. Each round
is a race to build an arithmetic expression that hits a target.

- At the start of each round, **ten pyramid cells** are revealed.
  Each cell has a letter (A–J), an operator (+, −, ×, ÷), and a
  signed operand. Example cells: `[A +3]`, `[B -7]`, `[C *2]`.
- A **target number** is announced for the round.
- The **first player to buzz** earns the right to declare. They pick
  **three distinct cells** from the ten. The three cells' operators
  and operands are applied to a running value left-to-right: start
  at 0, then `op0 value0`, then `op1 value1`, then `op2 value2`.
  If the resulting number equals the target, the declarer scores;
  if it doesn't, the opponent scores.
- After ten rounds the higher score wins.

Two structural things make Equation Pyramid unusual:

1. **Declaration is committing.** You buzz first (claiming the slot)
   *before* you have to say which triple you meant. If you buzz
   without a solution in hand, you're gambling.
2. **The search space is tiny.** There are exactly C(10, 3) = **120
   possible triples**, and each one's value is a closed-form integer
   given the round's cells. A computer can enumerate all 120 in a
   microsecond and list every valid answer before buzzing.

So the *optimal* player is a near-trivial pre-computation: at round
start, evaluate all 120 triples, keep those matching the target,
buzz if any match was found, declare one of them. The game's
difficulty for humans is purely arithmetic memory under time
pressure — a constraint that doesn't apply to bots. We expect the
correct bot to win every round it buzzes first on, and the bots to
split who-buzzes-first evenly; therefore the correct bot should
approach 100% win rate against any non-card-counting opponent.

---

## 1. The enumerator

Following the template, we add an `enumerate_legal_moves` helper in
the game plugin. For Equation Pyramid the enumeration is small:

- When buzzing is legal and the agent hasn't already buzzed, return
  one action: `buzz` with a null payload.
- When the agent is the active declarer (they buzzed first this
  round and now owe a triple), return 120 actions, one per
  ascending 3-card subset of `0..=9`, each with payload
  `{"triple": [a, b, c]}`.

The canonical ordering (ascending `a < b < c`) matters: it makes
the enumeration deterministic across seeds so tiebreakers in the
bot's move-selection are reproducible.

We also expose `seat_score_diff(state, seat)` — a score-differential
view for future bots that want to evaluate positions mid-match.

---

## 2. The RandomBot baseline — and the benchmark limitation

RandomBot for Equation Pyramid always buzzes when it can, then picks
a random triple from the 120. The chance a random triple hits the
target is small — probably 1–10 triples out of 120 evaluate to the
target in a typical round, so the random declarer is usually
*wrong*, which means the opponent scores.

This is where we ran into a **benchmark-harness limitation** for
the first time. The 2000-tick ceiling on a single match is normally
generous — NMM takes ~30 turns, Bagchal takes ~50 turns, Big Small
takes ~18 rounds × a couple of turns each. Equation Pyramid, in
contrast, has what we could call a *rejection-heavy* structure:
each round may involve many buzz-declare-wrong sequences before
somebody (by luck) declares correctly. With two random players both
always buzzing and almost always declaring wrong, a single round
can easily require dozens of micro-interactions. Ten rounds per
match × dozens of interactions per round × 200 matches blew past
both the 2000-tick ceiling and our patience (the 20-match probe
ran for six minutes without completing).

The scaffold is in, the enumerator is correct, the bot compiles
and its unit tests pass — but we have **no reliable benchmark
number yet** for random self-play in Equation Pyramid. That's
legitimate data: the random baseline failing to terminate tells us
the random-vs-random experiment is the wrong experiment to run
first.

Two concrete fixes, recorded in the program.md for future work:

1. **Bound the benchmark by matches rather than ticks.** The
   harness currently caps per-match ticks. Raise
   `max_ticks_per_match` for Equation Pyramid to ~20,000, or fix
   the match-stall detection to count *decisive turns* rather than
   raw ticks.
2. **Skip the random baseline.** Write the arithmetic-solver bot
   directly. For a game where the optimal strategy is trivially
   precomputable, "does anything beat random?" is the wrong
   question — the interesting question is "how close to 100% can
   we get?" Measure the arithmetic bot against a slower variant
   of itself.

---

## 3. The progression so far

| Iteration | Technique | Win-rate vs random | Status |
|-----------|-----------|-------------------|--------|
| v1 scaffold | RandomBot (buzz + uniform-random triple) | — | Benchmark doesn't terminate; baseline rejected |

What's next:

1. **Arithmetic-solver bot.** At round start, iterate the 120
   triples, evaluate each, keep those matching the target. Buzz
   only if we have a solution in hand. Declare a specific match
   deterministically (e.g., the triple with the smallest sum of
   indices to break ties).
2. **Benchmark ceiling fix.** Raise `max_ticks_per_match` for
   Equation Pyramid, or write a game-specific match driver that
   knows buzz-reject cycles don't count against the stall budget.
3. **Self-play benchmark.** Once the arithmetic-solver is in, run
   it against itself — both players will always find solutions, so
   the winner is determined by who buzzes first. Expect 0.50
   by-symmetry, with seat-0's structural buzz-priority pulling it
   toward 0.55–0.60 on our harness.

There's also a broader lesson sneaking out of this chapter:
**benchmark structure has to match game structure**. NMM, Bagchal,
and Big Small are all games where a match has a natural turn-count
upper bound. Equation Pyramid's match-length is *unbounded under
pathological play* — exactly the play that random produces. The
harness-side fix is to count progress rather than raw ticks; the
bot-side fix is to play something sensible. Both are appropriate
at different points in the iteration loop.

---

*Reproduce the hang with:*

```bash
cargo run --release -p olympiad-autoresearch -- \
  --program crates/olympiad/autoresearch/programs/equation-pyramid.md \
  --candidate random
```

*(Expect no output for minutes before Ctrl-C is needed. Don't say I
didn't warn you.)*

*Back to [the setup](00-the-setup.md).*
