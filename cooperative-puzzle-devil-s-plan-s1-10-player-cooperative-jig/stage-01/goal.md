[thinking] The user is asking me to create a research goal document about a specific topic: Cooperative Puzzle from Devil's Plan S1 — a 10-player cooperative jigsaw on a rotating platform, with focus on timing, turn-handoff optimization, scheduling, and cooperative puzzle-solving algorithms.
           
           This is a legitimate research planning request. I should generate a well-structured SMART research goal in markdown as requested. This doesn't require tool use — it's a text generation task where I'm drafting a research plan based on the specified topic and constraints.
           
           Let me think carefully about the novel angle here. The Devil's Plan is a Korean reality TV show where players engage in various strategic games. 
           
           The cooperative jigsaw puzzle game on a rotating platform creates an interesting research problem because it combines multi-agent scheduling under physical constraints with combinatorial puzzle-s...
# Research Goal: Cooperative Puzzle Assembly Under Cyclic Access Constraints

## Topic
Cooperative multi-agent jigsaw assembly on a rotating platform — scheduling, turn-handoff optimization, and information-sharing strategies for 10-player cooperative puzzle solving under periodic physical access constraints. Inspired by the Devil's Plan S1 cooperative jigsaw game.

## Novel Angle

**Standard multi-agent cooperative assembly** assumes agents have concurrent or on-demand access to the shared workspace. The rotating platform fundamentally changes the problem structure: each agent gets a **fixed-duration access window** in a **cyclic schedule**, creating a periodic cooperative assembly problem where:

1. **Cyclic access asymmetry**: Unlike standard task allocation, agents cannot act on demand. The platform rotation imposes a deterministic but rigid temporal structure — each agent's action window is bounded, and idle time between turns is wasted.

2. **Information decay at handoff boundaries**: When an agent's window ends, their partial knowledge of puzzle state (which pieces were tried, which regions look promising) must transfer to the next agent. This is distinct from standard MARL communication — it's a **structured handoff under time pressure**, closer to shift-change problems in manufacturing or surgical team handoffs than to typical multi-agent messaging.

3. **Joint scheduling + assembly ordering**: The combinatorial puzzle has placement dependencies (edge pieces first, region clustering). The rotation schedule determines *who* acts *when*. The novel optimization is over the **joint space of assembly strategy and agent-turn allocation** — should skilled agents get longer windows? Should the rotation order adapt based on puzzle progress?

**Why timely**: Recent MARL work has increasingly addressed **heterogeneous agents with constrained communication** and **temporally-extended cooperative tasks**. Work on cooperative assembly in robotics (multi-arm manipulation, warehouse coordination) has grown significantly, but the **cyclic fixed-schedule** constraint pattern remains underexplored. Most cooperative assembly research assumes flexible scheduling or simultaneous access. The cyclic constraint creates a distinct optimization landscape that maps to real-world shift-based teamwork (factory lines, relay-style operations, surgical teams).

**How this differs from standard approaches**: Standard cooperative puzzle solving treats the puzzle as a pure combinatorial problem. Standard MARL treats communication as a learnable channel. This work treats the **temporal access pattern as a first-class constraint** that fundamentally shapes both the assembly strategy and the communication protocol. The rotation schedule is not just a nuisance — it's an optimizable parameter.

## Scope

A single paper covering:
- Formal model of cyclic-access cooperative assembly (the "Rotating Table" problem)
- Comparison of handoff strategies: blind (no communication), state-summary, priority-queue transfer
- Scheduling variants: fixed equal rotation vs. adaptive window allocation vs. skill-weighted rotation
- Evaluation on synthetic jigsaw instances (grid-based with dependency structure) at 10-agent scale

Out of scope: physical robot implementation, visual piece recognition, natural language communication.

## Benchmark

- **Name**: No standard benchmark exists for this specific problem formulation. We define "RotatingTable-v0," a parameterized synthetic environment.
- **Environment**: Grid-based jigsaw (8x8 to 16x16) with piece-placement dependencies (edges → borders → interior). 10 agents with cyclic access windows. Configurable rotation period, communication bandwidth, and agent skill heterogeneity.
- **Metrics**:
  - **Completion rate**: Fraction of puzzle assembled within total time budget
  - **Time-to-completion**: Wall-clock turns to full assembly (lower is better)
  - **Handoff efficiency**: Useful information transferred / total handoff bandwidth
  - **Scheduling regret**: Gap between achieved completion time and oracle-optimal schedule
- **Baselines / Current SOTA**: No direct SOTA exists. Baselines will be: (a) round-robin with no communication, (b) round-robin with full-state transfer, (c) greedy adaptive scheduling, (d) independent MARL (MAPPO/QMIX adapted to cyclic access). The contribution is demonstrating that joint schedule-communication optimization outperforms these natural baselines.
- **Source**: Custom Gymnasium-compatible environment, to be released with the paper.

## SMART Goal

**Specific**: Design and evaluate a joint scheduling-and-handoff optimization framework for 10-agent cooperative jigsaw assembly under cyclic platform rotation constraints, demonstrating that adaptive rotation schedules with structured handoff protocols outperform fixed-schedule baselines.

**Measurable**: Achieve ≥20% improvement in time-to-completion over the best fixed-schedule baseline across three puzzle difficulty levels (64, 144, 256 pieces), and ≥15% improvement in completion rate under tight time budgets (budget = 2× oracle-optimal).

**Achievable**: The synthetic environment is lightweight (grid-based, no vision). Training MAPPO/QMIX variants at 10-agent scale on grid puzzles is feasible on a single GPU within hours. The main novelty is the problem formulation and scheduling optimization, not scale.

**Relevant**: Advances cooperative MARL under structured temporal constraints — a gap between fully-concurrent cooperation and single-agent sequential planning. Applicable to shift-based teamwork, relay manufacturing, and multi-robot assembly with access constraints.

**Time-bound**: 8 weeks — environment implementation (2 weeks), baseline experiments (2 weeks), adaptive scheduling + handoff optimization (3 weeks), paper writing (1 week, overlapping).

## Constraints

- **Compute**: Single GPU (RTX-class), training runs capped at ~4 hours each
- **Tools**: PyTorch, PettingZoo/Gymnasium for multi-agent env, Stable-Baselines3 or EPyMARL for MARL algorithms
- **Data**: Synthetic jigsaw instances (procedurally generated, no external dataset needed)
- **Agent count**: Fixed at 10 to match the Devil's Plan game; ablations at 4 and 8

## Success Criteria

A publishable result requires:
1. **Formal problem definition** that clearly distinguishes cyclic-access cooperative assembly from existing multi-agent assembly formulations
2. **Empirical evidence** that the cyclic access constraint creates qualitatively different optimal strategies vs. free-access cooperation (not just slower versions of the same strategies)
3. **Demonstrated benefit** of joint schedule-handoff optimization over independent optimization of each
4. **Ablation studies** showing which components matter: rotation adaptivity, handoff protocol richness, agent heterogeneity
5. **Target venue**: Workshop paper at NeurIPS/ICML cooperative AI or game-theoretic MARL workshops, or full paper at AAMAS/CoRL

---

**Generated**: 2026-04-17T00:00:00Z