# Chapter 2 — Bagchal

*The second game we worked through. A Nepalese hunt game with an
asymmetry that the show's Season 2 producers made extra-strange.*

---

## The rules

Bagchal is a classical Nepalese board game played on a 5×5 point
grid with 25 intersections. The edges between intersections form a
graph that includes all horizontal and vertical lines plus a
half-set of diagonals, giving the board its characteristic
"patterned cells" look. The game is asymmetric: one player controls
**four tigers** starting at the corners; the other player holds
**twenty goats** off-board at game start.

Play proceeds in two phases:

1. **Goat placement.** The goat player places one goat per turn on
   any empty point. Tigers alternate turns with placements by
   *sliding* one of their tigers along a board edge to an adjacent
   empty point, or *jumping* over an adjacent goat — collinearly,
   to a still-empty landing — which captures that goat.
2. **Goat movement.** Once all twenty goats are placed, the goat
   player moves by sliding a goat to an adjacent empty point.
   Tigers continue to slide or jump.

Tigers win if they capture five goats. Goats win if every tiger is
immobilised — every tiger has zero legal slides or jumps from its
current position.

**The Devil's Plan S2 variant** adds a mirror twist. Two Bagchal
sessions run in parallel on two boards, and each of the two players
is simultaneously the goat player on one board *and* the tiger
player on the other:

- Seat 0 plays goats on the **left** board, tigers on the **right**.
- Seat 1 plays tigers on the left, goats on the right.

A match is won by aggregate across the two boards. Which board moves
first on each round is dictated by the plugin schedule (left board
first in our implementation), and that small detail turns out to
matter for the baseline.

---

## 1. The enumerator

The interesting work, per the template, is writing
`enumerate_legal_moves(state, agent) -> Vec<AgentAction>`. Four
action kinds must be covered:

- `place_goat { board, point }` — the goat player, while goats
  remain off-board.
- `move_goat { board, from, to }` — the goat player, once all
  placed.
- `move_tiger { board, from, to }` — the tiger player, sliding.
- `jump_tiger { board, from, over, to }` — the tiger player, jumping
  over an adjacent goat to a collinear empty landing.

The first three are straightforward. *Jump* has a subtlety: we need
to know whether the candidate landing `to` is *collinear* with
`from` and `over` — a tiger cannot jump from one corner, over the
centre, to the opposite corner unless there's a straight-line edge
all the way through. The way we detect collinearity is to represent
each of the 25 points by its `(x, y)` on a uniform grid, compute the
vector `over - from`, step from `over` by that same vector, and check
whether there's a board-graph edge from `over` to the landing. If
either the vector step falls off the grid or there's no edge, the
jump isn't legal and we skip it.

This is roughly 150 lines of code, all of it in the Bagchal plugin
crate where the private state and graph helpers are accessible.
Exposing just `enumerate_legal_moves` as pub lets the bot crate stay
small and ignorant of the rules.

---

## 2. The RandomBot baseline and the seat-bias finding

Bagchal random vs. random over 200 matches:

- **91 / 109 / 0**. Win-rate **0.455** from seat-0's perspective.

Two things about this result. First, there are no draws — Bagchal's
stalemate rule fires on a turn counter, and random play in 200
matches never hit it. Second, seat-0 is losing more than winning,
because in the S2 two-board mirror, seat-0 plays goats on the left
and tigers on the right, while seat-1 plays tigers on the left and
goats on the right — but the left board moves first each round,
which gives the first-mover role (which is a goat placement on the
left) a structural disadvantage against four tigers already on the
board. The randomness is symmetric; the game's initial conditions
are not.

This is exactly the kind of finding the baseline is supposed to
reveal. Any future Bagchal bot needs to be measured *against* this
~0.455 — not the abstract 0.5 — because that's the true "no-skill"
equilibrium in this particular variant.

---

## 3. Greedy Bagchal — and an instructive "too clean" result

The natural first heuristic: for each candidate move, clone the
state, apply the action, then score the post-move position as
`material_delta * 100 - opponent_mobility_after`. Material is the
captured-goats differential across both boards (weighted by which
role you play on which side — the plugin exposes this as
`seat_material_score`); opponent mobility is the total legal-move
count for the opposite seat, which tiger-play wants to drive down.

This is one-move look-ahead, the same as NMM's greedy. In code:

```rust
let material_before = seat_material_score(state, own_seat);
let mut best_score = i32::MIN;
for action in legal {
    let mut probe = state.clone();
    plugin.apply_action(&mut probe, agent, action)?;
    let delta = seat_material_score(&probe, own_seat) - material_before;
    let opp_mob = seat_mobility(&probe, opp_seat);
    let score = delta * 100 - opp_mob;
    // keep max, tiebreak via splitmix
}
```

GreedyBot vs. RandomBot over 200 matches:

- **200 / 0 / 0.** Win-rate **1.0000**. Decision: **Kept**.

Not 70%. Not 80%. A *perfect* record. Random loses every match.

Why so extreme, when NMM greedy vs. random was "only" 0.885? Because
Bagchal punishes worse behaviour more sharply:

1. A random goat doesn't block tiger jumps. A greedy tiger jumps
   every turn one is available, capturing one goat per jump. Once
   five goats have been captured on a board, that board is over —
   and because both sides of the mirror are going at once, the
   greedy player wraps up both boards in roughly ten tiger turns.
2. A random tiger doesn't avoid being pinned. A greedy goat *is*
   pushing down opponent mobility (the `-opp_mob` term), so it
   piles pressure on the random tigers and sometimes immobilises
   them.

So the 1.000 number isn't a "we've solved Bagchal against random"
statement. It's "Bagchal is a game where random play fails
catastrophically in this specific sense — no stone threatened
anywhere, no coordinated pressure." The interesting benchmark
against which to measure future Bagchal bots isn't random any
more; it's **greedy vs. greedy**, which will be close to 0.5 by
symmetry, and any candidate that breaks that symmetry has done
something genuinely new.

---

## 4. Greedy self-play — the new benchmark

Once greedy saturates against random, we switch to greedy-on-greedy
via the CLI's `--baseline` flag:

```bash
cargo run --release -p olympiad-autoresearch -- \
  --program crates/olympiad/autoresearch/programs/bagchal.md \
  --candidate greedy --baseline greedy
```

200 matches:

- **108 / 92 / 0**. Win-rate **0.54** from candidate's perspective.

0.54 is the expected near-symmetric result: seat-alternation washes
out first-mover bias, leaving only the splitmix seed interactions.
This becomes the **new** baseline. A Bagchal candidate is meaningfully
new only if its greedy-vs-candidate win-rate is clearly above 0.5.

There's a general lesson hiding here, worth stating plainly: **the
baseline must be strong enough to discriminate.** Random was a good
baseline when the game was complex enough that many bots fail
badly; once it stops differentiating between our candidates, we
switch to the previous best as the new baseline. The CLI's
`--baseline` flag makes this one keystroke away.

---

## 5. The progression so far

| Iteration | Candidate | Baseline | Win-rate | Losses |
|-----------|-----------|----------|---------:|-------:|
| v1        | RandomBot | RandomBot | 0.455 | (baseline) |
| v2        | GreedyBot | RandomBot | 1.000 | 0 |
| v2 self   | GreedyBot | GreedyBot | 0.540 | 92 |

What's next:

1. **Minimax depth-2** with the greedy score as a move-ordering
   heuristic. Goat-side play is harder than tiger-side because "good
   goat play" is largely about blocking future tiger jumps, which
   requires reasoning about threats rather than immediate material.
2. **Endgame pattern detection.** Classical Bagchal has known draw
   and winning configurations for both sides (e.g. the tigers-in-a-
   row deadlock); encoding a handful as terminal shortcuts lets the
   minimax stop searching fruitless branches.
3. **Cross-board coordination.** In the S2 mirror variant, the
   seat's two roles are independent in the rules but coupled in the
   score. A better evaluator weights urgency on each board
   differently — if I'm winning left and losing right, the right
   board should get more of my attention.

Each is one more commit on trunk with a journalled benchmark.

---

*Reproduce any row with the bagchal program file:*

```bash
cargo run --release -p olympiad-autoresearch -- \
  --program crates/olympiad/autoresearch/programs/bagchal.md \
  --candidate <random|greedy> [--baseline <random|greedy>]
```

*Next: [Chapter 3 — Big Small](03-big-small.md).*
