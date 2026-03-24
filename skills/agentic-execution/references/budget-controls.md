# Budget & Resource Controls

Multi-agent execution is expensive. Research consistently shows that multi-agent
systems consume ~15x more tokens than single-agent approaches (implementation
anti-patterns research). With 3-5 agents running in parallel, costs scale
linearly with agent count and superlinearly with coordination overhead.

Budget controls are first-class constraints in execution plans -- not
afterthoughts. Every wave has a ceiling, every agent has a budget, and
early-stopping triggers prevent runaway spend.

## Cost Model

### Token Cost Estimation

**Per-agent cost factors:**
- Input tokens: spawn prompt + files read + conversation history
- Output tokens: code written + test output + explanations
- Tool use overhead: each tool call adds ~200-500 tokens

**Relative cost by model (ratios remain stable across pricing changes):**

| Model | Relative Input Cost | Relative Output Cost | Typical session tokens |
|-------|--------------------|--------------------|----------------------|
| claude-opus-4-6 | 5x Sonnet | 5x Sonnet | 100-300K |
| claude-sonnet-4-6 | 1x (baseline) | 1x (baseline) | 80-200K |
| claude-haiku-4-5 | ~0.25x Sonnet | ~0.25x Sonnet | 50-100K |

Check current absolute pricing at your API provider's pricing page.

**Per-agent session cost (typical):**

| Agent Role | Model | Typical Tokens | Est. Cost |
|-----------|-------|---------------|-----------|
| Foundation Builder | Opus | 200-300K | $15-23 |
| Feature Implementer | Sonnet | 100-200K | $2-3 |
| Integration Wirer | Sonnet | 150-250K | $3-4 |
| Merge Conductor | Opus | 50-100K | $4-8 |
| Test Hardener | Sonnet | 80-150K | $1-2 |
| Security Reviewer | Sonnet | 50-100K | $1-2 |
| Recovery Agent | Opus | 50-150K | $4-12 |

### Project Cost Projection

**Small project (3-5 modules, 3 waves, 2-3 agents):**
- Wave 1 (Foundation, Opus): ~$15-20
- Wave 2 (Features, 2 Sonnet): ~$4-6
- Wave 2→3 (Merge, Opus): ~$4-8
- Wave 3 (Integration, Sonnet): ~$3-4
- Contingency (50%): ~$13-19
- **Total: $39-57**

**Medium project (5-10 modules, 4 waves, 4-5 agents):**
- Wave 1 (Foundation, Opus): ~$20-25
- Wave 2 (Features, 4 Sonnet): ~$8-12
- Wave 2→3 (Merge, Opus): ~$6-10
- Wave 3 (Integration, Sonnet/Opus): ~$5-8
- Wave 4 (Hardening, 3 Sonnet): ~$3-6
- Contingency (50%): ~$21-30
- **Total: $63-91**

**Large project (10-15 modules, 5 waves, 5-6 agents):**
- Wave 1 (Foundation, Opus): ~$25-30
- Wave 2 (Features, 5 Sonnet): ~$10-15
- Wave 2→3 (Merge, Opus): ~$8-12
- Wave 3 (Integration, Opus): ~$15-20
- Wave 4 (Hardening, 3 Sonnet): ~$3-6
- Recovery agents (est. 2-3 respawns): ~$12-24
- Contingency (30%): ~$22-32
- **Total: $95-139**

### Cost Optimization Strategies

1. **Model stratification** -- use the cheapest model that meets the quality bar:
   - Opus: lead, foundation, critical path, recovery, complex integration
   - Sonnet: feature implementation, test writing, standard integration
   - Haiku: exploration, research, code review classification

2. **Spawn prompt efficiency** -- include only relevant context:
   - DON'T include the full architecture plan (ETH Zurich: 20%+ cost increase)
   - DO include the specific module's boundary map, contracts, and file manifest
   - DON'T include information about other agents' work
   - DO include CLAUDE.md conventions (shared, not duplicated)

3. **Task sizing** -- right-size tasks to minimize wasted context:
   - 5-15 min tasks use context efficiently
   - >20 min tasks waste context on history that could be compacted
   - <5 min tasks waste tokens on spawn overhead

4. **Fresh agents** -- spawn fresh agents at wave boundaries:
   - Don't reuse an agent across waves (context pollution)
   - Each fresh spawn starts with a clean context window
   - The spawn prompt is more efficient than accumulated conversation

5. **Early termination** -- cut agents that are spinning:
   - Set timeouts per agent (see wave-dispatch.md)
   - Kill and respawn rather than letting agents grind
   - "Reduce scope" is cheaper than "try harder"

## Budget Controls in the Execution Plan

### Per-Wave Budget Ceiling

Every wave in the execution plan must specify:
```markdown
### Wave [N]: [Name]
**Budget ceiling:** $[N] (tokens: ~[N]K)
**Agents:** [count] x [model]
**If exceeded:** [Action -- pause and assess / scope reduction / continue with warning]
```

### Per-Agent Token Budget

Each agent's dispatch command should include a budget note:
```
Note: This agent has a budget of ~[N]K tokens (~$[N]).
If the agent is consuming significantly more, consider:
1. Is the task too complex? Split it.
2. Is the agent looping? Kill and respawn.
3. Is the context polluted? Fresh spawn.
```

### Total Execution Budget

Define in the execution plan:
```markdown
## Budget Plan

| Category | Budget | Ceiling | Action if Exceeded |
|----------|--------|---------|-------------------|
| Wave 1 | $[N] | $[N] | Re-assess foundation approach |
| Wave 2 | $[N] | $[N] | Scope reduction on non-essential modules |
| Merge | $[N] | $[N] | Manual merge as fallback |
| Wave 3 | $[N] | $[N] | Reduce E2E test coverage |
| Wave 4 | $[N] | $[N] | Defer to manual review |
| Recovery | $[N] | $[N] | Accept remaining issues |
| **Total** | **$[N]** | **$[N]** | **HALT and re-plan** |
```

### Early-Stopping Decision Matrix

| Spend Level | Action |
|------------|--------|
| 0-50% of budget | Normal execution, no concerns |
| 50-80% of budget | Monitor closely, start identifying deferrable work |
| 80-100% of budget | Alert operator, execute scope reduction plan |
| 100-120% of budget | HALT non-essential agents, complete only critical path |
| >120% of budget | HALT all agents, commit progress, re-plan |

## Token Tracking

### During Execution

For Subagents and Manual Worktrees, track tokens via:
```bash
# Check API usage (if using Anthropic API directly)
# Or monitor through the Claude Code billing dashboard

# Rough estimation from conversation length:
# Each agent message pair ≈ 2-5K tokens (varies widely)
# Each file read ≈ 1-3K tokens
# Each file write ≈ 0.5-2K tokens
```

For Agent Teams, the lead session tracks total team token usage.

### Post-Execution

Record actual costs in the retrospective:
```markdown
### Budget Retrospective

| Wave | Projected | Actual | Delta | Notes |
|------|-----------|--------|-------|-------|
| Wave 1 | $[N] | $[N] | +$[N] | Foundation took longer than expected |
| Wave 2 | $[N] | $[N] | -$[N] | Feature modules were simpler than estimated |
| Recovery | $[N] | $[N] | +$[N] | 2 respawns needed for auth module |
| **Total** | **$[N]** | **$[N]** | **+/-$[N]** | |
```

## When to Use Single-Agent Instead

Multi-agent execution is NOT always better. The Google/MIT study (Dec 2025,
180 configurations) established critical empirical thresholds:

**The 45% Rule:** If a single agent achieves >45% success rate on a
non-decomposable task, adding more agents DEGRADES performance and
increases costs. Multi-agent only helps for parallelizable work.

**Sequential tasks degrade sharply:** Tasks that are inherently sequential
(Step B depends entirely on Step A's output) saw up to 70% performance
DEGRADATION with multi-agent approaches.

**Tool-heavy tasks penalized:** Tasks requiring >10 tool calls suffer
2-6x efficiency penalties when split across agents.

| Condition | Why Single is Better | Research Backing |
|-----------|---------------------|-----------------|
| <10 total tasks | Coordination overhead exceeds parallelism gains | Industry consensus |
| Tightly coupled modules | Parallel execution creates more merge conflicts than time savings | CodeCRDT: 39.4% slowdown on coupled tasks |
| Budget < $20 | Can't afford the multi-agent overhead (15x token cost) | Anthropic |
| Single-file changes | No opportunity for parallelism | - |
| <30 min total work | Setup overhead dominates | - |
| Non-decomposable tasks | Single agent >45% success = more agents hurt | Google/MIT 2025 |
| Deeply sequential chains | Each handoff amplifies errors 17.2x | Google/MIT 2025 |

The implementation plan's template sizing guide (Small projects, 10-20 tasks)
is the threshold below which single-agent execution often outperforms
multi-agent approaches.

**Decision heuristic:** Before generating an execution plan, assess: "Can
this implementation plan's tasks be meaningfully parallelized into 3+
independent work streams?" If yes, proceed with multi-agent. If no,
recommend single-agent execution and explain why.
