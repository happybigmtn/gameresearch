# Chapter 1 — Nine Men's Morris

*The first game we worked through, and still the one we know best.*

---

## The rules

The game is roughly 2000 years old. It's played on a 24-point board —
three concentric squares connected at their midpoints. Each player has
nine stones. The game has three phases:

1. **Placement.** Players alternate placing stones on empty points,
   one at a time, until both have placed all nine. Whenever a player
   aligns three of their own stones along one of the board's sixteen
   *mill lines*, they "form a mill" — and immediately remove one of
   the opponent's stones from the board.
2. **Movement.** Once all stones are placed, players alternate sliding
   a stone to an adjacent empty point along the board's edges. Forming
   a mill still lets you capture.
3. **Flying.** When a player is down to exactly three stones, they can
   move any stone to any empty point — "flying" — giving them a
   desperate chance to close mills.

You lose if you are reduced to two stones, or if you have no legal
move on your turn. The game was *solved* by Ralph Gasser in 1996 via
retrograde analysis: assuming perfect play from both sides, the game
ends in a draw. But against any imperfect opponent, a good player
wins decisively.

So there is a ceiling — beating the solved tablebase is impossible by
definition — and there is a floor — not playing randomly. The
interesting engineering is in-between. Now let's look at three bots,
in order of sophistication, and what each of them teaches us.

---

## 1. The RandomBot — a proper floor

Our first bot is not an insult; it is a *control group*. It walks the
list of legal moves and picks one uniformly at random. The seed is
deterministic, so the same match played twice gives the same result —
this matters for reproducibility, so when we improve a bot we can tell
whether the improvement is real or we just got lucky in the benchmark.

Random vs. random over 200 matches:
- **106 / 93 / 1** (candidate / baseline / draws). Win-rate **0.53**.

That's very close to the symmetric 0.5. The small bias (0.03) is the
first-move advantage — Nine Men's Morris gives placer A slightly more
initiative than placer B during the placement phase. This is the
baseline against which every future bot is measured. Anything that
beats 0.53 is, to some degree, non-random. The keep-threshold in
`program.md` is 0.55, which is comfortably above it.

---

## 2. The GreedyMillBot — one step of look-ahead

The simplest thing above random is: don't be random. Ask, of every
legal move, "if I made this move, how many of my own mills would be
on the board right after?" Then pick the move that maximises that count.

In code it's a loop. For each legal `action`, we clone the session
state, apply the action (the plugin tells us immediately if it would
be illegal, in which case we discard it), and count our mills in the
resulting position. We also subtract the opponent's mill count so that
in the less common case where *our* move lets the opponent close a
mill, we prefer not to. That's the whole heuristic.

Why does this work? Because NMM's economy is dominated by mills.
Closing a mill captures an opponent stone, and losing stones is how
you lose the match. A player who *ever closes a mill when they can*
and *never willingly gifts one* is already playing at a level the
random opponent almost cannot touch.

GreedyMillBot vs. RandomBot over 200 matches:
- **177 / 1 / 22**. Win-rate **0.885**. Decision: **Kept**.

Nearly an eighteen-to-one record. The one baseline win is a stalemate
edge case in the NMM plugin — we'll come back to it. The twenty-two
draws come from a subtle failure mode: in the late flying phase,
greedy has no look-ahead, so it can shuffle a stone back and forth
forever while the game's stalemate counter forces a termination. The
next bot fixes this.

---

## 3. The MinimaxBot — real look-ahead with alpha-beta

Greedy asks "what happens if I move here?" Minimax asks "what happens
if I move here, and then the opponent plays their best reply, and then
I play my best reply to that?" This is *search*. You recursively expand
the game tree to some depth, you evaluate the leaf positions with a
static heuristic, and you propagate the values back up — your turns
trying to *maximise* the score, the opponent's turns trying to
*minimise* it.

The classical elaboration is **alpha-beta pruning**: as you walk the
tree, you keep running bounds on the best score each player can
guarantee so far (alpha for max, beta for min). If you ever discover
that the current branch can't improve the running best for your side
— because the opponent has a reply that caps it below what you can
already secure elsewhere — you stop exploring that branch. You don't
*miss* the best move; you just skip branches that provably can't be it.

Two wrinkles are worth walking through, because they're where a
textbook minimax and an NMM minimax diverge.

**Wrinkle one: whose turn is it?**  Classical minimax assumes players
strictly alternate — depth 0 is me, depth 1 is you, depth 2 is me.
NMM breaks this. When you close a mill, *you* get another move (the
capture), so two plies in a row are from the same player. If you
assume alternation, the search miscounts whose score it's optimising
at each level.

The fix is to ask the plugin, at every node, "whose turn is it now?"
via a helper called `seat_to_move`. That's just looking at which seat
has non-empty legal actions. Then we branch as max or min based on
that seat's identity, not based on the depth parity. NMM's same-player
continuations become same-level extensions of the search, which is
what we want.

**Wrinkle two: what do the leaves evaluate to?**  At depth zero — when
we've looked ahead as many plies as our budget allows — we need a
static score for the position. Our evaluator, from our perspective, is:

```
(our_mills     - opponent_mills)     × 200
+ (our_stones   - opponent_stones)   × 100
+ (our_threats  - opponent_threats)  × 10
```

A *threat* is a mill line with two of our stones and one empty point
— one move from closing. The weights aren't tuned; they're argued
from domain: a fully-formed mill captures an opponent stone per cycle
and is worth roughly two stones of long-run value; a stone is direct
material; a threat is a cheap tiebreaker between otherwise-equivalent
positions.

MinimaxBot depth-2 vs. RandomBot over 200 matches:
- **188 / 0 / 12**. Win-rate **0.940**. Decision: **Kept**.

Zero losses. The look-ahead closes every tactical mistake greedy
could make. The twelve draws are all in the flying-phase endgame where
the evaluator can't distinguish between positions that are functionally
equal (same material, same mills) and the stalemate counter fires.

Why depth 2 and not 3? Because the game tree branches quickly —
placement phase has up to 24 options, and we're cloning a heavy
JSON-backed state at every node. Depth 3 searches roughly fifteen
times more positions per move than depth 2. On the laptop I'm typing
this on, depth 3 made one match take roughly a second; two hundred
matches would have taken ten minutes or more. Depth 2 is plenty to
demonstrate that look-ahead helps dramatically, and the depth is a
simple dial — `MinimaxBot { depth: 3 }` is one line away when you
have more wall-clock to burn.

---

## 4. The progression so far

| Iteration | Technique | Win-rate vs random | Losses | Draws |
|-----------|-----------|-------------------:|-------:|------:|
| v1        | RandomBot (self-play symmetry) | 0.530 | 93 | 1 |
| v2        | GreedyMillBot (depth-1)        | 0.885 | 1  | 22 |
| v3        | MinimaxBot (depth-2, α-β)      | 0.940 | 0  | 12 |

Each step is more expensive to compute but meaningfully stronger.
What's still on the table, in order of what we'd try next:

- **Move ordering.** Alpha-beta prunes more aggressively if you search
  the likely-best moves first. The greedy score itself is a good
  heuristic for ordering. Adding this should let depth 3 run in the
  time depth 2 takes now.
- **Quiescence search.** When the leaf position has a pending capture,
  extend the search past the nominal depth until no mill-closing move
  is available. This stops us from stopping the search in the middle
  of a trade and mis-evaluating.
- **Stalemate-aware evaluation.** Add a term for *progress toward
  stalemate* (increment of the stalemate counter) so the bot prefers
  moves that don't lock in a forced draw when ahead in material.
- **Endgame tablebase.** Once the total stone count is ≤ 6, look up
  the position in Stahlhacke's Mühle database and play the
  retrograde-optimal move. This turns the endgame from a heuristic
  into perfect play.

Each of these is another commit on trunk, with a journalled benchmark
to tell us whether it helped.

---

*Reproduce any row with:*

```bash
cargo run --release -p olympiad-autoresearch -- \
  --program crates/olympiad/autoresearch/programs/nine-mens-morris.md \
  --candidate <random|greedy|minimax>
```

*Next: [Chapter 2 — Bagchal](02-bagchal.md).*
