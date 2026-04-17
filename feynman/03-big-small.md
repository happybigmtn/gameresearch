# Chapter 3 — Big Small

*The third game, and the one where random play breaks the benchmark
in a different way than Bagchal did.*

---

## The rules

Big Small is played between **two players** over **eighteen rounds**.
It's completely deterministic once the dealer shuffle seed is fixed.

- Each player has a **private deck of cards 0 through 10** (eleven
  cards total), used once each across the match.
- The dealer has **two decks, A and B, each 1 through 9** (nine cards
  each), shuffled from the session seed. Both players can see the
  shuffle since it's derivable from the seed.
- Each round, the roles rotate: one player is the **duel** role, the
  other is the **bet** role.

**What happens in a round:**

1. The dealer flips one card from deck A *or* deck B (the schedule is
   fixed by the session seed, visible to both players).
2. The duel-role player submits one card from their remaining private
   deck. Their reward in chips is the absolute difference between
   their card and the dealer's — so submitting a 10 against a dealer
   1 earns nine chips, while submitting a 5 against a dealer 5 earns
   zero.
3. The bet-role player stakes `amount` chips on `big` or `small`
   (whether the dealer's *next* flip will be higher or lower than the
   current one). If correct, the bet player gains `amount` chips; if
   wrong, loses `amount`.
4. Roles swap. Round counter increments.

After 18 rounds, the chip leader wins. Because the dealer's schedule
and pool are common knowledge, Big Small is in principle **fully
solvable at game start** by a card-counting bot — you can plan all
eighteen turns at once.

---

## 1. The enumerator

The plugin exposes two action kinds:

- `submit_duel_card { card }` — the duel player picks any card still
  in their private deck.
- `bet { big_or_small, amount }` — the bet player picks a direction
  and an amount.

Our enumerator is small:

- For `submit_duel_card`: one action per remaining card in the duel
  player's deck (so up to 11 options, shrinking as cards are used).
- For `bet`: **two actions**, `{big, 0}` and `{small, 0}` — binary
  direction at zero stake. A smarter bot would vary the amount; the
  baseline treats bets as always-zero, which turns out to be the
  interesting failure mode below.

Deliberately leaving out the amount dimension from the enumerator is
itself a design choice. A random bot that rolls a random amount every
turn mostly generates noise; a random bot that stakes zero generates
a very specific kind of signal we want to study. We can always add
amount randomisation later if the zero-stake finding becomes
uninteresting.

---

## 2. The RandomBot baseline — and 100% draws

Random vs. random over 200 matches:

- **0 / 0 / 200.** Win-rate **0.0000** (all draws). Decision:
  Reverted.

Three hundred decisions, no decisive matches. Why?

Because amount=0 bets never move chips — the bet reward is `amount ×
direction_correct`, which is always zero. The only chip flow is from
duel rewards, and with random card picks on both sides, the expected
duel reward is symmetric. Over 18 rounds, two random zero-info
players land on *exactly equal* chip totals surprisingly often. In
our seed space, they hit that tie in every one of the 200 matches.

This is different from Bagchal's "random loses every match" finding.
Here the lesson is: **the baseline is too quiet to measure against.**
Anything we compare to it will either draw too (Reverted) or win a
bit (trivially Kept) without telling us whether the win is because we
played well or because we happened to break the tie.

---

## 3. The general lesson

Two game-specific pathologies surfaced in the first three games:

- **Bagchal**: the baseline was *too weak*. Greedy beat it 100% of
  the time, so we couldn't measure relative improvements among
  non-random candidates. The fix was to promote greedy to baseline
  via the `--baseline` flag.
- **Big Small**: the baseline was *too quiet*. Random-zero-stake
  play draws almost every match, so there's no signal to measure
  against. The fix is to noise the baseline up — make RandomBot
  stake non-zero amounts occasionally so chip totals diverge.

Both pathologies come from the same place, said differently: **a
baseline is only useful when it produces differentiated outcomes.**
Random fails that test in Bagchal because it loses too completely,
and in Big Small because it draws too completely. The right fix per
game depends on what's wrong: Bagchal needs a stronger baseline
(greedy becomes the new benchmark); Big Small needs a *noisier*
baseline (random with non-zero stakes, so that single matches have
richer chip trajectories).

This generalises. Every time we add a game, the first question isn't
"how good is the best bot we can write?" — it's "does the random
baseline produce a distribution of outcomes that can discriminate
between candidates?" If not, the baseline itself is the first thing
to fix.

---

## 4. The progression so far

| Iteration | Candidate | Baseline | Matches | W–L–D | Win-rate |
|-----------|-----------|----------|---------|-------|---------:|
| v1        | Random    | Random   | 200 | 0 / 0 / 200 | 0.000 |

What's next:

1. **Bet-aware RandomBot**. Randomise bet amount between 0 and 3 so
   chip trajectories differ. This is a one-line change to the
   enumerator and produces the "discriminating baseline" that lets
   real candidates be measured.
2. **Card-counting greedy.** At game start, derive the full dealer
   schedule from the seed. Precompute, per round, the expected dealer
   card under each remaining deck. Greedy duel-play: pick the card
   in your hand that gives maximum `|card - known_next_dealer|`.
   Greedy bet-play: stake up to the chip lead when the next dealer
   flip's direction is unambiguous.
3. **Minimax is less useful here** than in NMM because the state
   space is so narrow once you've card-counted. Depth-1 with the
   right evaluator is essentially optimal. The bigger open problem
   is variance when the dealer schedule favours one side; that's
   partly a seed issue, partly a real property of the game.

---

*Reproduce the draw-pathology result with:*

```bash
cargo run --release -p olympiad-autoresearch -- \
  --program crates/olympiad/autoresearch/programs/big-small.md \
  --candidate random
```

*Back to [the setup](00-the-setup.md).*
