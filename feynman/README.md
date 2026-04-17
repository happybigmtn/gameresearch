# The gameresearch lectures

*A plain-language account of what we're building, why, and how each
piece actually works. Read the setup chapter first; then any per-game
chapter stands alone.*

## Reading order

| # | File | What's in it |
|---|------|--------------|
| 0 | [00-the-setup.md](00-the-setup.md) | The premise, the two layers of tooling, the Rust plugin trait, and a scope survey of all thirty games. Start here. |
| 1 | [01-nine-mens-morris.md](01-nine-mens-morris.md) | Rules of Nine Men's Morris. Random → Greedy → Minimax. Why depth-2 minimax gets 0.940 vs random with zero losses. |
| 2 | [02-bagchal.md](02-bagchal.md) | Rules of Bagchal (Nepalese tiger-vs-goat, S2 mirror variant). Random collapses to 0.455; Greedy hits 1.000. The baseline-too-weak pathology, and why self-play is the new benchmark. |
| 3 | [03-big-small.md](03-big-small.md) | Rules of Big Small (2-player deterministic card counting). Random self-play produces 100% draws. The baseline-too-quiet pathology, and the general lesson about baselines that discriminate. |
| 4 | [04-equation-pyramid.md](04-equation-pyramid.md) | Rules of Equation Pyramid (2-player arithmetic race, 10 rounds, pick 3 cells to hit a target). Random vs. random hangs the harness — a structural benchmark limitation and what to do about it. |

## Living document

Each chapter is updated when that game's bot gets a new iteration. The
setup chapter's "progression table" in section 4 is the cross-game
leaderboard. New chapters are added as new games are scaffolded; the
per-game writing style is modelled on the Feynman lectures (plain
language, analogy-first, reasoning-visible) and should stay consistent
across chapters.

## Reproducing any result in this document

Every benchmark line in every chapter comes from the `olympiad-
autoresearch` CLI in the [autonomy](https://github.com/happybigmtn/autonomy)
repo. To reproduce:

```bash
cd ~/Coding/autonomy
cargo run --release -p olympiad-autoresearch -- \
  --program crates/olympiad/autoresearch/programs/<game>.md \
  --candidate <random|greedy|minimax> \
  [--baseline <random|greedy|minimax>] \
  --journal /tmp/<game>-<candidate>.jsonl
```

The numeric results should match byte-for-byte; the benchmark is
deterministic per seed.
