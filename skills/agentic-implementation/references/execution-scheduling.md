# Execution Scheduling

How to convert a task list into a time-sequenced execution schedule that maximizes
parallel throughput while respecting dependencies.

## Core Concepts

### Execution Waves

A wave is a group of tasks that can execute simultaneously. Waves are separated
by synchronization gates. Within a wave, agents work independently.

```
Wave 1: [Foundation tasks, sequential, 1 agent]
  |-- GATE: Tier 0 verification
Wave 2: [Feature tasks, parallel, 3-5 agents]
  |-- GATE: Per-module + integration verification
Wave 3: [Integration tasks, sequential or limited parallel, 1-2 agents]
  |-- GATE: Final verification
```

### Agent Lanes

An agent lane is the sequence of tasks assigned to a single agent. Within a lane,
tasks execute sequentially. Across lanes, tasks execute in parallel.

```
Lane A0: T0.1 -> T0.2 -> T0.3 -> T0.R1 [Foundation]
Lane A1: T1.1 -> T1.2 -> T1.3 -> T1.R1 [Module A]
Lane A2: T2.1 -> T2.2 -> T2.R1         [Module B]
Lane A3: T3.1 -> T3.2 -> T3.3 -> T3.R1 [Module C]
Lane A4: T4.1 -> T4.2 -> T4.R1         [Integration]
```

### The Critical Path

The critical path is the longest sequential chain from start to finish. It
determines the minimum possible wall-clock time regardless of how many agents
you add.

```
Critical Path = Foundation + Longest Feature Lane + Integration
Example: 30 min + 45 min + 20 min = 95 min minimum
```

Optimization: Shorten the critical path by splitting the longest feature lane
or moving work to shorter lanes.

## Building the Schedule

### Step 1: Group Tasks by Wave

**Wave 1 (Foundation):**
- All Tier 0 tasks
- Must execute sequentially by a single agent
- Rationale: Foundation defines shared contracts; parallel foundation work
  creates inconsistent contracts

**Wave 2 (Features):**
- All Tier 1+ feature tasks
- Execute in parallel across agents (one lane per module)
- Each lane is internally sequential
- Staggered starts are possible if specific dependencies are met before the
  full Tier 0 gate (advanced optimization -- use cautiously)

**Wave 3 (Integration):**
- All integration and wiring tasks
- Execute after all feature lanes complete (or after specific lanes if
  integration tasks depend on only a subset of modules)
- May run limited parallelism (e.g., E2E tests while documentation generates)

### Step 2: Assign Agent Lanes

**Assignment principles:**

1. **One agent per module** -- this is the default. Each module's file manifest
   belongs exclusively to one agent.

2. **Group small modules** -- if a module has only 1-2 tasks, combine it with
   a related module into one lane (same agent handles both sequentially).

3. **Balance lane lengths** -- if one lane has 8 tasks and another has 2,
   consider whether some tasks from the long lane can be split to the short one
   (only if there's no file ownership conflict).

4. **Foundation agent can become a feature agent** -- after Tier 0 completes,
   A0 can take on a feature lane (if using subagents, spawn a new agent).

5. **Integration agent stays fresh** -- the integration agent should ideally be
   a fresh spawn, not a reused agent with a polluted context window.

**Agent count selection:**

| Task Count | Recommended Agents | Rationale |
|:-----------|:------------------|:----------|
| 5-10 | 2 (1 foundation + 1 feature) | Minimal coordination overhead |
| 10-20 | 3 (1 foundation + 2 feature) | Good parallelism, manageable |
| 20-35 | 4 (1 foundation + 3 feature) | Sweet spot for most projects |
| 35-50 | 5 (1 foundation + 3 feature + 1 integration) | Near optimal |
| 50-75 | 5-6 (1 foundation + 4 feature + 1 integration) | Max recommended |

**Never exceed 6 coding agents** -- research consistently shows diminishing
returns beyond 4-5 agents, and coordination failures increase sharply.

### Step 3: Calculate Timing

Estimate per-task time:
- Small (S): ~5 min
- Medium (M): ~10 min
- Large (L): ~15 min
- Refactoring (R): ~10 min

Calculate per-lane time by summing tasks in each lane.

Calculate wall-clock time:
```
Wall clock = Wave 1 time + max(Wave 2 lane times) + Wave 3 time + Gate overhead
Gate overhead ≈ 5 min per gate (merge + test execution)
```

### Step 4: Optimize

**Optimization strategies (in priority order):**

1. **Shorten the critical path** -- if one feature lane is significantly longer
   than others, split its largest task or move a task to a shorter lane.

2. **Parallelize integration** -- if integration tasks are independent (E2E tests
   vs. deployment config), run them in parallel.

3. **Staggered starts** -- if a feature module depends only on shared types
   (not shared utilities or schema), it can start as soon as types are done,
   before the full Tier 0 gate. Mark these as "early start" tasks.

4. **Pipeline overlap** -- while the last feature module finishes, start
   integration tasks that don't depend on it.

**Do NOT optimize by:**
- Parallelizing foundation tasks (creates inconsistent contracts)
- Running more than 5-6 agents (coordination overhead exceeds gains)
- Skipping refactoring tasks (saves time now, costs more later)
- Removing verification gates (catching errors early is always cheaper)

## Advanced Scheduling Patterns

### Staggered Start Pattern

When some feature modules depend only on specific foundation outputs:

```
A0: [T0.1: types] -> [T0.2: schema] -> [T0.3: utils] -> [T0.R1: refactor]
                |
                v (early start: types only)
A1:          [T1.1] -> [T1.2] -> [T1.R1]
                         |
                         v (waits for schema too)
A2:                   [T2.1] -> [T2.2] -> [T2.R1]
```

Use this only when:
- The architecture plan explicitly marks which foundation artifacts each module needs
- The early-start module truly has no dependency on later foundation tasks
- The risk of foundation changes breaking the early module is accepted

### Pipeline Overlap Pattern

When integration can start before all features finish:

```
A1: [T1.1] -> [T1.2] -> [T1.R1]
A2: [T2.1] -> [T2.2] -> [T2.R1]
A3: [T3.1] -> [T3.2] -> [T3.R1] (longest lane)
A4:                         [T4.1: wire A1+A2] -> [T4.2: wire A3] -> [T4.R1]
                            ^ starts when A1 and A2 complete
```

### Single-Agent Fallback

For small projects or tightly coupled modules where parallelism would create
more overhead than it saves:

```
A0: [T0.1] -> [T0.2] -> [T1.1] -> [T1.2] -> [T2.1] -> [T2.2] -> [Integration]
```

Use when: <10 tasks total, heavy file overlap between modules, or a single
session can complete the work in <30 minutes.

## Schedule Visualization

Always include both a wave diagram (ASCII art) and a timeline table in the
implementation plan. The wave diagram shows parallel structure; the timeline
table shows estimated durations.

### Wave Diagram Format

```
WAVE 1 (Foundation) ──────────────────────────────────────
  A0: [T0.1 types (S)] -> [T0.2 schema (M)] -> [T0.3 utils (S)] -> [T0.R1 (M)]
  ─── GATE: Tier 0 verification (~5 min) ───

WAVE 2 (Features) ───────────────────────────────────────
  A1: [T1.1 auth svc (M)] -> [T1.2 auth logic (L)] -> [T1.R1 (M)]
  A2: [T2.1 user svc (M)] -> [T2.2 user logic (M)] -> [T2.R1 (M)]
  A3: [T3.1 content svc (M)] -> [T3.2 content logic (L)] -> [T3.3 search (M)] -> [T3.R1 (M)]
  ─── GATE: Tier 1 verification + merge (~10 min) ───

WAVE 3 (Integration) ────────────────────────────────────
  A4: [T4.1 routes (M)] -> [T4.2 middleware (M)] -> [T4.3 E2E (L)] -> [T4.R1 (M)]
  ─── GATE: Final verification (~5 min) ───
```

### Timeline Table Format

| Wave | Lane | Tasks | Task Times | Lane Total | Wave Wall Clock |
|:-----|:-----|:------|:-----------|:-----------|:---------------|
| 1 | A0 | T0.1-T0.R1 | 5+10+5+10 | 30 min | 30 min |
| 2 | A1 | T1.1-T1.R1 | 10+15+10 | 35 min | 45 min (max) |
| 2 | A2 | T2.1-T2.R1 | 10+10+10 | 30 min | |
| 2 | A3 | T3.1-T3.R1 | 10+15+10+10 | 45 min | |
| 3 | A4 | T4.1-T4.R1 | 10+10+15+10 | 45 min | 45 min |
| | | | **Gates** | 20 min | |
| | | | **Total** | | **~140 min** |
