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

## 3. The SolverBot — arithmetic is the whole game

The natural fix is the one the game's structure points at: compute
valid triples at round start, buzz only if at least one exists,
declare the first (lex-smallest) match when prompted. Since the
plugin exposes both `evaluate(cells, triple)` and a new
`current_round_layout(state)` helper, this reduces to a triple
for-loop inside the bot:

```rust
fn has_valid_triple(cells, target) -> bool {
    for a in 0..8 { for b in (a+1)..9 { for c in (b+1)..10 {
        if evaluate(cells, [a, b, c]) == Some(target) { return true; }
    }}}
    false
}
```

The SolverBot never buzzes without a solution and never declares a
wrong answer. Against a random opponent, the game-theoretic
expectation is that whichever side buzzes first wins the round, and
the solver buzzes first every round (because it has a solution
every round — the plugin generates rounds that always have at
least one). The expected win rate is therefore essentially 1.0,
subject only to the harness's seat-polling order.

### 3.1 The harness-vs-plugin tick-counting gotcha (and a partial fix)

We added an optional `max_ticks_per_match` field to `program.md`
so Equation Pyramid can get a higher ceiling than the default 2000.
Set to 20000 in the program file. But even with the ceiling raised,
the benchmark takes **many minutes to run 50 matches**. A 200-match
benchmark ran for 18 minutes without completing.

The reason is that the Equation Pyramid plugin internally uses
several tick cycles per logical turn — there are silence cycles,
buzz windows, and declaration subcycles baked into the timing
model. Even when both bots decide instantly, the plugin eats tick
budget advancing between states. Our harness counts *driver ticks*
(the outer loop in `run_match`) rather than *plugin-logical turns*,
which means Equation Pyramid pays a multiplier the other games
don't.

Two fixes were considered:

1. **Per-game benchmark ceilings tuned to plugin timing models.**
   Raise Equation Pyramid's `max_ticks_per_match` to 50,000+,
   accept the long runtime, confirm the solver hits ~1.0 win-rate.
2. **Progress-based stall detection.** Track state hashes between
   ticks; if the hash hasn't changed in N ticks, terminate. This
   decouples harness termination from plugin timing model.

Both landed. The first is the `max_ticks_per_match` field on
`program.md`, propagated into `BenchmarkConfig`. The second is a
hash-based counter that resets whenever the `SessionState` JSON
changes.

The progress-based fix works beautifully for the other three games
(NMM regressed at 188/0/12 → 0.940, identical to trunk). It does
*not* fix Equation Pyramid, because the plugin's `SessionState`
includes a `current_cycle` counter that monotonically advances
*every* driver tick. So from the harness's point of view, every
tick is "progress" — the hash is always different — and the stall
counter never fires.

The real fix for Equation Pyramid is plugin-level: either expose
a per-logical-turn progress signal the harness can count, or split
the `SessionState` into a "durable" part (round number, phase
enum, chip balances) and a "transient" part (cycle counter, buzz
timers) where only the durable part is hashed for stall detection.
Either change touches the `GamePlugin` trait surface and is a
separate commit from the generic harness improvement. The harness
gain is banked; Equation Pyramid remains unmeasured.

Meanwhile, the SolverBot code is in, its logic is correct, and its
unit tests pass. The absence of a benchmark number is itself a
finding: **one generic harness improvement doesn't equal one fix
per game**, because each game's plugin encodes its own timing
model and the harness has to be robust across all of them.

---

## 4. The progression so far

| Iteration | Technique | Win-rate vs random | Status |
|-----------|-----------|-------------------|--------|
| v1 scaffold  | RandomBot (buzz + random triple) | — | Benchmark doesn't terminate |
| v2 (current) | SolverBot (enumerate-and-validate) | — | Code lands; benchmark needs harness tuning |

What's next:

1. **State-hash based stall detection** in `benchmark.rs`. Two
   tick-counters: one that always advances, one that resets on
   state change. Terminate on the second hitting the ceiling.
2. **Self-play benchmark** once termination is stable. SolverBot
   vs. SolverBot will be 0.50 by symmetry, with seat-0 ahead by
   the small structural buzz-priority margin.
3. **Bet-aware opponent.** Once the solver is measurable, try a
   *heuristic solver* that sometimes buzzes without a solution
   when the opponent is behind — a genuinely game-theoretic
   decision.

There's also a broader lesson sneaking out of this chapter:
**benchmark structure has to match game structure**. NMM, Bagchal,
and Big Small are all games where a match has a natural turn-count
upper bound. Equation Pyramid's match-length is *unbounded under
pathological play*, and its per-turn *tick cost* varies by game —
so a one-size-fits-all harness ceiling isn't enough. The fix is
progress-based termination, not bigger integers.

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
