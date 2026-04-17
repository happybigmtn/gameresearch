# gameresearch

Game-theory literature reviews + iterative bot development, powering
`crates/olympiad/*` in the [`autonomy`](https://github.com/happybigmtn/autonomy)
repo.

Two layers cooperate:

| Layer           | Tool                                   | What it produces                                       |
|-----------------|----------------------------------------|--------------------------------------------------------|
| Research (lit)  | `research` CLI + AutoResearchClaw      | `goal.md`, `problem_tree.md`, `sources.json` per game  |
| Iteration (bot) | `olympiad-autoresearch` Rust crate     | Journalled win-rate history of candidate bots per game |

The research layer feeds the iteration layer: each game's
`stage-03/sources.json` becomes the bibliography cited by that game's
`programs/<game>.md` inside `autonomy`.

> **Directory name note:** this folder was intended to be `gameresearch/`
> but the `research` CLI currently writes to `~/Coding/gamedesign/`. Rename
> is a tidy-up — update `RC_RUNS_ROOT` in `~/.local/bin/research` after the
> sweep completes and `mv` the folder.

---

## 1. Research layer — `research` CLI

Wraps [AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw)'s
first three pipeline stages (TOPIC_INIT → PROBLEM_DECOMPOSE →
SEARCH_STRATEGY) and routes the output here. Stages 4-6 are skipped
deliberately: the OpenAlex citation-ranked fallback pollutes results with
top-cited papers unrelated to game theory. Stage 3 already emits the
Opus-4.7-synthesised bibliography we actually want.

### Install (one-time)

```bash
# AutoResearchClaw Python venv
python3 -m venv ~/.local/share/researchclaw/.venv
~/.local/share/researchclaw/.venv/bin/pip install -e '~/.local/share/researchclaw[anthropic]'

# ACP bridge to Claude Code (so Opus 4.7 does the synthesis)
npm install -g acpx --ignore-scripts
```

Config lives at `~/.config/researchclaw/config.yaml` and sets
`provider: acp` + `primary_model: claude-opus-4-7`, reusing your existing
`claude` CLI auth (no separate API key needed).

### Usage

```bash
# One-off research run
research --prompt "Nine Men's Morris optimal strategy and solver algorithms"

# Output lands in ~/Coding/gamedesign/<slugified-prompt>/
#   stage-01/goal.md          — scoped research question
#   stage-02/problem_tree.md  — structured sub-questions
#   stage-03/sources.json     — curated bibliography (the gold)

# Deeper dive through the full lit collection (slower, noisier)
research --prompt "..." --to-stage KNOWLEDGE_EXTRACT
```

### Behaviour

- **Smart resume.** Re-running with the same prompt short-circuits if
  `checkpoint.json` shows the target stage was already completed; prints
  the existing results path and exits. If earlier stages crashed, it
  resumes from the last checkpoint.
- **Deterministic slug.** Output directory is
  `~/Coding/gamedesign/<slug-of-first-60-chars>/` — predictable enough to
  reference from `programs/<game>.md`.
- **Stops at Stage 3 by default.** Override via `--to-stage
  KNOWLEDGE_EXTRACT` if you want the noisy web-fetch layer too.

### Sweeping every Devil's Plan game

```bash
~/Coding/gamedesign/_sweep.sh    # runs all 31 games, ~2h wall time
tail -f ~/Coding/gamedesign/_sweep.log
```

The sweep array in `_sweep.sh` hand-writes each prompt with game name,
player count, and 1-2 defining mechanics — Opus 4.7 uses those hints to
pull the right solver literature.

---

## 2. Iteration layer — `olympiad-autoresearch`

Rust-native karpathy/autoresearch-style loop for game bots. Lives in
`crates/olympiad/autoresearch/` inside `autonomy`. A
human-authored `program.md` instructs an LLM agent (Opus 4.7 via ACP) to
iterate on a single `bot.rs` file, measured head-to-head against a
baseline via the existing `olympiad-games` plugin registry. Kept candidates
are journalled; regressions are reverted.

### Crate layout

```
crates/olympiad/
├── bots/                       # shared Bot trait
│   ├── src/lib.rs
│   └── nine-mens-morris/       # per-game bot crate (the file Opus edits)
│       └── src/lib.rs
└── autoresearch/
    ├── src/
    │   ├── lib.rs              # AutoresearchConfig, Error
    │   ├── benchmark.rs        # run_benchmark + MatchOutcome
    │   ├── journal.rs          # append-only JSONL of iterations
    │   ├── program.rs          # program.md loader (YAML front-matter + MD)
    │   └── bin/
    │       └── olympiad_autoresearch.rs   # CLI
    └── programs/
        └── nine-mens-morris.md # human-authored iteration instructions
```

### Running a benchmark iteration

```bash
cd ~/Coding/autonomy
cargo run -p olympiad-autoresearch -- \
  --program crates/olympiad/autoresearch/programs/nine-mens-morris.md \
  --journal .reports/autoresearch/nmm.jsonl
```

The draft CLI currently exercises program loading, the `Bot` trait, and
the journal — it doesn't yet play full matches (the random baseline
submits null payloads, which game plugins with payload schemas reject).
The first LLM iteration's job is to fix that: see
`programs/nine-mens-morris.md`.

### The `program.md` format

Each program file has YAML front-matter followed by Markdown
instructions passed verbatim to the LLM agent.

```markdown
---
game_id: dp.nine_mens_morris
bot_source_path: crates/olympiad/bots/nine-mens-morris/src/lib.rs
spec_path: specs/160426-olympiad-games/dp-nine-mens-morris.md
research_sources_path: ~/Coding/gamedesign/nine-men-s-morris-.../stage-03/sources.json
baseline_bot_id: dp.nine_mens_morris.random.v1
keep_threshold: 0.55
matches_per_benchmark: 200
---

# Research context, search directions, non-goals — free-form markdown
```

Front-matter fields:

| Field                      | Purpose                                                      |
|----------------------------|--------------------------------------------------------------|
| `game_id`                  | matches `GamePlugin::game_id()` in `olympiad-games`          |
| `bot_source_path`          | the one file the LLM edits, relative to workspace root       |
| `spec_path`                | game rules spec — LLM reads for mechanics                    |
| `research_sources_path`    | bibliography from the research layer (optional)              |
| `baseline_bot_id`          | opponent to race against; from `Bot::bot_id()`               |
| `keep_threshold`           | accept candidate only if win-rate ≥ this                     |
| `matches_per_benchmark`    | head-to-head matches each candidate plays                    |

### Journal format

Every benchmark appends one JSON line to the journal file:

```json
{
  "timestamp": "2026-04-17T14:49:00Z",
  "game_id": "dp.nine_mens_morris",
  "candidate_bot_id": "dp.nine_mens_morris.alpha_beta.v3",
  "baseline_bot_id": "dp.nine_mens_morris.random.v1",
  "matches_played": 200,
  "candidate_wins": 142,
  "baseline_wins": 50,
  "draws": 8,
  "candidate_win_rate": 0.71,
  "decision": "kept",
  "note": "alpha-beta depth 3 with mill+mobility evaluator"
}
```

The journal is self-describing — replay with `jq`, load in any
notebook, or feed to the next LLM iteration as its memory of what
worked.

---

## End-to-end workflow for a new game

1. **Research.** `research --prompt "Game X, <player count>, <key mechanic>"`.
   Wait ~4 min. Confirm `stage-03/sources.json` looks domain-relevant.
2. **Add bot crate.** `crates/olympiad/bots/<game>/` mirroring
   `nine-mens-morris` — baseline `RandomBot`, registered in workspace
   `Cargo.toml`.
3. **Author `programs/<game>.md`.** Copy the NMM template, point
   `research_sources_path` at your fresh `stage-03/sources.json`, fill in
   the "game facts" and "search direction" sections from your read of
   that sources list.
4. **Iterate.** `cargo run -p olympiad-autoresearch -- --program
   programs/<game>.md --journal .reports/autoresearch/<game>.jsonl`.
   Inspect the journal, bump `keep_threshold`, run again.

## References

- Source for the pattern: [karpathy/autoresearch](https://github.com/karpathy/autoresearch)
- Research tool: [aiming-lab/AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw)
- ACP bridge: [openclaw/acpx](https://github.com/openclaw/acpx)
- Autonomy PR introducing the Rust harness: `feat/olympiad-autoresearch`
