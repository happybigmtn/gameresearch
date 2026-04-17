# Chapter 0 — The setup

*What we're building, why, and what the machinery looks like before
any particular game gets involved.*

---

## 1. What are we actually trying to do?

There is a Korean reality competition show, *The Devil's Plan*, in
which roughly a dozen clever people play roughly thirty games over a
week for a cash prize. The games range from classical board games
(Nine Men's Morris, Bagchal, a 3-dimensional 4-player tic-tac-toe) to
social-deduction games (the Virus Game with hidden factions), to
rule-construction games (Rules Race), to betting games with imperfect
information (Equation High-Low). Some of the games have been studied
for centuries. Some were invented for the show.

What we want is, for every one of these thirty games, a computer program
that plays it well. By "well" we mean concrete: when we pit it against
a random-move opponent over a few hundred matches, it wins far more
often than half the time — and as we iterate, it keeps improving.

That's the target. The rest of this document is about *how* we get
there, and why each piece of the contraption exists. We'll build up the
whole machinery here, and then each game gets its own chapter where we
walk through its rules and the progression of bots for it.

---

## 2. Two kinds of work, two layers of tooling

There's a natural tension in building game bots: you want to know what
clever people have already figured out about the game (so you don't
reinvent a solved game from scratch), *and* you want to be able to
rapidly iterate on code (so that you can measure whether your latest
idea actually helps).

Those are genuinely two different activities. Reading research is one
kind of work — cerebral, slow, involves libraries and citations. Trying
to build a better bot is another — mechanical, fast, involves running
code and checking numbers. We've built separate tooling for each.

**The research layer** is a command called `research`. You give it a
topic like *"Nine Men's Morris optimal strategy and solver
algorithms"*, and it returns — about four minutes later — a small pile
of files:

- `goal.md` — a precisely scoped research question.
- `problem_tree.md` — a structured decomposition of sub-questions.
- `sources.json` — a curated bibliography: the paper from 1996 where
  Gasser at ETH Zürich *solved* Nine Men's Morris (showing that
  perfect play is a draw), the URL to Stahlhacke's public perfect-play
  database, a pointer at AlphaZero for the "but what if you want to
  learn it rather than compute it" angle, and a few fallbacks.

Under the hood, `research` is a thin wrapper around a Python library
called AutoResearchClaw, which runs Claude Opus 4.7 through a
twenty-three-stage pipeline that ends in a publishable research paper.
We stop the pipeline at stage three. The reason is *very* important,
so let me explain it carefully.

Stage three (search strategy) is when the language model synthesises
what it knows, enumerates the relevant prior art, and produces a
bibliography. Because Opus 4.7 has read a great deal of game-theory
literature, this list is essentially correct: you get Gasser, you get
Schaeffer's "Checkers is Solved" for its retrograde-analysis technique,
you get AlphaZero, you get the right textbook. Stage four then goes
out to the academic web (OpenAlex, Semantic Scholar) and tries to
fetch those papers for real — and here the wheels come off. Those APIs
don't index classical combinatorial game theory well, and their fallback
when a query returns too few results is *popular papers in anything*.
So stage four turns a bibliography about Nine Men's Morris into a
bibliography about cancer statistics, molecular docking, and 6G wireless
communications, because those are the most-cited papers containing the
words "optimal" or "strategy".

We tested this. The stage-six knowledge cards for Nine Men's Morris
came back full of oncology references. Opus-4.7's synthesised
bibliography (stage three) was correct on every source. So the right
answer was to stop the pipeline at the point where the AI's own
knowledge is the signal, and skip the point where unreliable web search
becomes the noise.

**The iteration layer** is a Rust crate called `olympiad-autoresearch`.
It's inspired by, and deliberately structurally similar to, Andrej
Karpathy's `autoresearch` repository — but adapted for game bots
instead of language-model training. The idea is simple:

- There is one file that the AI agent is allowed to edit. This is
  `bot.rs`, the game-playing policy.
- There is one file that the human edits. This is `program.md`, which
  points at the bot, at the game's rules spec, at the research
  bibliography, and at a keep-threshold — the win-rate above which a
  candidate is accepted.
- There is a benchmark. The candidate bot plays two hundred matches
  against a baseline; if its win-rate exceeds the threshold, the change
  is kept, otherwise reverted.
- Every iteration is journalled (append-only JSON lines), so you have
  a permanent record of what was tried and whether it worked.

That's the whole shape of the iteration layer. Karpathy runs it against
a language model that trains itself; we run it against a game plugin
that plays itself. Otherwise it's the same pattern.

---

## 3. The rules live in Rust plugins

Before we can iterate on bots we need a rigorous definition of each
game. Fortunately, the `autonomy` repository we're building in already
has that. Under `crates/olympiad/games/` there are thirty crates, one
per game, each implementing a `GamePlugin` trait that looks roughly
like this:

```rust
pub trait GamePlugin {
    fn game_id(&self) -> &'static str;
    fn setup(&self, seed: u64, roster: &[AgentId]) -> SessionState;
    fn available_actions(&self, state: &SessionState, agent: AgentId) -> Vec<ActionSpec>;
    fn apply_action(&self, state: &mut SessionState, agent: AgentId, action: AgentAction) -> ActionResult;
    fn tick(&self, state: &mut SessionState, cycle: CycleIndex) -> Vec<SessionEvent>;
    fn is_complete(&self, state: &SessionState) -> bool;
    fn final_key_deltas(&self, state: &SessionState) -> Vec<(AgentId, i64)>;
    // ... plus renderers for human-readable output
}
```

Each game plugin knows how to set up a fresh match from a random seed,
knows what actions are legal from any given state, knows how to apply
an action (and whether it's rejected as illegal), knows when the match
is over, and knows who won.

This is good. A bot's job, in this framework, is to choose one
`AgentAction` per turn. Everything else — the board, the rules, the
terminal conditions, the scoring — is the plugin's problem.

The tension is that `available_actions` returns *categories* of actions
with payload schemas, not enumerated moves. For Nine Men's Morris in
the placement phase, it returns *one* action spec saying "you may submit
an action of kind `place_stone` with a payload of the form `{point:
u8}`" — which tells you the move type but leaves the target point up
to you. So the bot itself has to pick which point, and the plugin only
discovers whether your point was legal when you submit it. That's a
reasonable design for generic game servers, but it means the bot can't
simply pick a random element from `available_actions` — it has to know
the game's geometry.

Our answer was to add, per game, a helper function on the plugin that
enumerates *every* concrete legal move. For Nine Men's Morris this is
`enumerate_legal_moves(state, agent) -> Vec<AgentAction>`; it walks
the 24-point board, knows the phase (placement vs. movement vs. flying
vs. settled), knows the adjacency graph for movement, and produces
concrete `{point: 12}` or `{from: 7, to: 11}` actions ready to submit.
Bots call that and pick from what comes back. Writing that enumerator
is the first step of adding any game to the framework.

---

## 4. Cross-game progression table

The table below is the running scoreboard across all scaffolded games.
Each row is one benchmark, reproducible from the `--program` /
`--candidate` / `--baseline` arguments shown.

| Game | Candidate | Baseline | Matches | W–L–D | Win-rate |
|------|-----------|----------|---------|-------|---------:|
| NMM       | Random    | Random | 200 | 106 / 93 / 1   | 0.530 |
| NMM       | Greedy    | Random | 200 | 177 / 1 / 22   | 0.885 |
| NMM       | Minimax-2 | Random | 200 | 188 / 0 / 12   | 0.940 |
| Bagchal   | Random    | Random | 200 |  91 / 109 / 0  | 0.455 |
| Bagchal   | Greedy    | Random | 200 | 200 / 0 / 0    | 1.000 |
| Bagchal   | Greedy    | Greedy | 200 | 108 / 92 / 0   | 0.540 |
| Big Small | Random    | Random | 200 |   0 / 0 / 200  | 0.000 |

Per-game chapters give the rules and the context that turn each row
into a story.

---

## 5. Extending to all thirty games — what's tractable, what's hard

Nine Men's Morris is, by the standards of the Devil's Plan corpus,
easy. Two players, perfect information, a small action space, known
solver. Of the thirty games, maybe six fall into that bucket — roughly,
the classical board games at the end of each season (NMM, Bagchal,
4-Player 3-in-a-Row, Hexagon, Wall Go, Equation Pyramid). For each of
those, the exact same template works: expose an `enumerate_legal_moves`
on the plugin, write a RandomBot baseline, write a GreedyXBot with a
domain-appropriate heuristic, write a MinimaxBot with an evaluator
weighted by what matters in that game. Different details, same recipe.

The other twenty-four games are progressively harder. Let me group
them honestly, because "30 games" is misleading; the difficulty curve
is steep.

### 5.1 Stochastic perfect-information games (~5)

Triple Dice, Dice Poker, Big Small, parts of Scale Game. Here the
state includes random draws, so the search has to handle *chance
nodes* (you average over possible outcomes) in addition to max/min.
Classical technique: *expectiminimax*. Same shape as minimax, with a
third node type. More computationally expensive but conceptually a
small extension.

### 5.2 Imperfect-information adversarial games (~6)

Equation High-Low, Sniper Hold'em, Doubt and Bet, Secret Number, and
parts of Time Auction. Here your opponent sees things you can't —
cards, numbers, bids — and optimal play requires reasoning about what
they could have. Minimax doesn't directly apply. The technique is
**Counterfactual Regret Minimisation (CFR)** and its variants (MCCFR,
DeepCFR); it's the technology behind the best poker bots. Implementing
a full CFR is substantially more work than a minimax — you're solving
for an *equilibrium* rather than *evaluating positions*. Feasible but
a multi-week project per game, not a single-commit iteration.

### 5.3 Hidden-role / social-deduction games (~4)

The Virus Game, Corrupt Police, Unknown, parts of Halloween Monster.
These games have hidden player types (terrorist / civilian / neutral)
and rely on inferring who-is-who from behaviour. Game theory here is
about **belief updating and communication equilibria**, and the state
of the art is mostly neural — train an agent via self-play that learns
both a policy and an implicit belief model. Very open research area;
even the top systems (e.g., Cicero for Diplomacy) required massive
engineering effort.

### 5.4 Mechanism / rule-construction games (~4)

Rules Race, Zoo, parts of Laying Grass. These have a *rule-composition
sub-game* where players design interacting rules that then resolve
mechanically. Optimal play involves reasoning about the commitment
structure of the chosen rule set. Classical game theory (commitment
games, mechanism design) applies at the meta-level; the execution
sub-game is tractable but the meta-game is where the interesting
decisions live. Good heuristics are more practical than full solutions.

### 5.5 Cooperative / real-time puzzle games (~5)

Cooperative Puzzle, Word Tower, Fragments of Memory, Montage,
Equation Pyramid (sort of). These aren't adversarial in the standard
sense — players optimise jointly against a time/error budget. The
techniques are closer to *scheduling* and *constraint-satisfaction*
than to minimax. Much easier to build good bots here because there's
no opponent model; just optimise the objective.

### What we can realistically do

For any game in group 5.1 (stochastic perfect-info), our NMM recipe
extends naturally. For group 5.5 (cooperative), a different but
equally tractable recipe applies. For groups 5.2–5.4, each game is
its own multi-week project, often requiring a different architectural
decision (CFR vs. deep RL vs. mechanism design).

So when we extend this framework to "all thirty games", the honest
progress curve is:

- A **baseline random bot** for each game (cheap, mostly mechanical —
  requires writing each game's `enumerate_legal_moves`).
- A **greedy heuristic bot** for the perfect-information subset.
- A **look-ahead bot** (minimax or expectiminimax) for the perfect-
  or partial-perfect-information subset.
- A **CFR or neural policy** for the imperfect-information games, on a
  much longer timeline and probably with different tooling (likely
  using `candle` or `tch` for Rust-native tensor work).

The per-game chapters update as each game lands. For now, Nine Men's
Morris, Bagchal, and Big Small are the worked-through examples, and
they together establish the template every other game follows.

---

## 6. Why this shape is the right shape

A final reflection before we dive into game chapters. Why split the
work into a research layer, an iteration layer, and per-game plugins?
Why not just write one big thing?

Because these three layers evolve at wildly different speeds. The
research layer evolves once per game (you do it, you're done). The
plugin layer evolves when the rules are discovered or refined (rare;
Devil's Plan rule sets are fixed). The bot layer evolves every
iteration — dozens of times per game.

Keeping them separate means you don't touch research code when
you're iterating on a bot, you don't touch plugin code when you're
tuning an evaluator, and you don't touch bot code when you're
scaffolding a new game. Each layer has a distinct signature — a
function call, a trait implementation, a file format — and the
interfaces between them are narrow.

And because each layer has its own benchmark — the research layer's
"did the bibliography cover the right references," the plugin's "did
100 matches against a known opponent preserve win rates," the bot's
"is the candidate's win rate above threshold" — you can always tell
*which* layer regressed if something breaks. That's an enormous win
for debugging: "something's wrong" is a much worse starting point
than "stage three's synthesiser is wrong."

This is also what compounding looks like. Each piece of code you
write here makes the next piece of code easier to write. The
research wrapper makes future games' bibliographies free. The plugin
trait makes future games' rules self-describing. The iteration
harness makes future bots' benchmarks free. The journal makes future
sessions' "what did we already try" questions cheap to answer.

When we're done, we won't have thirty independent one-offs. We'll have
one framework with thirty instantiations, and adding the thirty-first
will be easier than adding the first.

---

*Next: [Chapter 1 — Nine Men's Morris](01-nine-mens-morris.md).*
