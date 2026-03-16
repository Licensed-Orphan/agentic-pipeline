# Execution Anti-Patterns

Known failure modes specific to the EXECUTION phase of multi-agent swarms.
These are distinct from planning-level and architecture-level anti-patterns
(covered in the implementation and architecture skills). These failures occur
during live dispatch, monitoring, and completion.

**Critical research context:** The MAST taxonomy (UC Berkeley, 2025) analyzed
1,600+ multi-agent traces and found that **79% of failures originate from
specification and coordination issues**, not model limitations. The execution
plan's quality -- spawn prompt precision, gate rigor, recovery circuits -- is
the primary determinant of success. Additionally, Google/MIT (2025) found that
uncoordinated agents amplify errors **17.2x**, while centralized orchestration
reduces this to **4.4x**. Structured wave dispatch with gates is not optional.

## Dispatch Anti-Patterns

### 1. The Blind Launch

**What:** Dispatching agents without running the pre-flight checklist.

**Why it fails:** Agents start working on a broken baseline -- dependencies
aren't installed, tests already failing, uncommitted changes pollute worktrees.
Every agent wastes tokens discovering the same environmental issues.

**Fix:** ALWAYS run the pre-flight script before dispatching any agent. The
5 minutes spent on pre-flight saves hours of debugging failed agents.

---

### 2. The Big Bang Dispatch

**What:** Dispatching ALL agents (foundation + features + integration) at once
to "save time."

**Why it fails:** Feature agents start before the foundation exists. Each agent
invents its own types and conventions. Integration fails on every boundary.
Exactly the "Missing Foundation" problem from the architecture anti-patterns,
but caused by impatient dispatch rather than bad planning.

**Fix:** Waves are mandatory. Foundation completes and gates before features
start. Features complete and merge before integration starts. No exceptions.

---

### 3. The Fire-and-Forget

**What:** Dispatching agents and not monitoring them during execution.

**Why it fails:** An agent gets stuck 5 minutes in, but nobody notices for
45 minutes. Another agent scope-creeps into a different module's files.
A third exhausts its context and produces garbage output. All discovered
only at gate time, when fixing is most expensive.

**Fix:** Monitor every 10 minutes. Check git logs for recent commits. Look
for signs of stuckness, scope creep, or quality degradation. Early
intervention is 10x cheaper than post-gate diagnosis.

---

### 4. The Prompt Abbreviator

**What:** Shortening spawn prompts to "save tokens" -- removing the interface
contracts, file manifests, or constraints sections.

**Why it fails:** Anthropic research: "Vague delegation was the primary cause
of early multi-agent system failures." Agents without explicit contracts invent
their own interfaces. Agents without file manifests create duplicate files.
Agents without constraints refactor everything in sight. The tokens "saved"
on the prompt are dwarfed by the tokens wasted on rework.

**Fix:** Use the full 7-section spawn prompt from the implementation plan.
The spawn prompt IS the agent's contract. Cutting it saves pennies and costs
dollars.

---

### 5. The Model Extravagance

**What:** Using Opus for every agent, including simple feature implementation
tasks that Sonnet handles well.

**Why it fails:** Multi-agent execution already costs 15x more than single-agent.
Using Opus for all agents makes it 3-5x more expensive on top of that, with
marginal quality improvement on standard implementation tasks. Budget ceiling
hit before execution completes.

**Fix:** Model stratification: Opus for lead/foundation/critical path/recovery.
Sonnet for feature implementation, testing, standard integration. Haiku for
exploration and research. Anthropic: "Opus lead with Sonnet workers outperformed
single-agent Opus by 90.2%."

---

## Monitoring Anti-Patterns

### 6. The Status Illusion

**What:** Trusting agent self-reported task status without verifying actual work.

**Why it fails:** Agents sometimes mark tasks as "Done" without running
verification commands (the "Premature Done" anti-pattern from implementation).
In Agent Teams, teammates sometimes fail to update task status at all. The
status dashboard shows "all green" while the actual code has failures.

**Fix:** Verify actual work output at every checkpoint:
- Check git log (are there real commits?)
- Run gate scripts (do tests actually pass?)
- Spot-check file manifests (are the right files created?)
Never advance a wave based on status alone -- always run the gate script.

---

### 7. The Sunk Cost Spiral

**What:** Continuing to re-prompt a failing agent instead of killing it and
respawning fresh.

**Why it fails:** Each re-prompt adds to the agent's context, making it
less likely to succeed (context pollution). After 3-4 failed attempts, the
conversation history is so polluted that the agent keeps making the same
mistakes. Meanwhile, tokens accumulate rapidly.

**Fix:** Follow the "two-strike rule": if an agent fails the same task twice
after re-prompting, kill the session and spawn a completely fresh agent with:
1. The original spawn prompt
2. A note about what was tried and why it failed
3. A pointer to committed progress (if any)

---

### 8. The Absent Operator

**What:** Walking away from execution and coming back hours later.

**Why it fails:** Without monitoring, failures compound. A stuck agent wastes
its full timeout before being noticed. A budget breach isn't caught until
the ceiling is hit. A cascading failure propagates across all agents. The
execution becomes a token furnace with no one to pull the plug.

**Fix:** Set up monitoring cadence:
- Check every 10 min during active waves
- Set terminal alerts for long-running agents
- Use background agent dispatch with notification on completion
- For long executions, use Agent Teams (lead provides automated monitoring)

---

## Merge Anti-Patterns

### 9. The Parallel Merge

**What:** Merging all feature branches into main simultaneously (big bang merge).

**Why it fails:** When multiple merges happen at once, it's impossible to
determine which merge introduced a test failure. Debugging requires reverting
all merges and re-applying one by one anyway -- so sequential merge with
test-after-each is strictly superior.

**Fix:** Sequential merge with test-after-each is MANDATORY. Merge one branch,
run the full test suite, proceed only on success. See merge-integration.md
in the implementation skill references.

---

### 10. The Merge-Then-Fix

**What:** Merging a branch that fails tests, then trying to fix the failures
on main.

**Why it fails:** Fixing on main means the failure is in the shared branch.
Other branches can't merge cleanly against a broken main. The fix might
also break other modules. The debugging scope expands from "this one module"
to "the entire codebase."

**Fix:** If a merge introduces test failures:
1. Revert the merge immediately (`git revert --no-commit HEAD`)
2. Fix in the feature branch (not on main)
3. Verify the fix passes on the feature branch
4. Re-attempt the merge

---

### 11. The Worktree Graveyard

**What:** Not cleaning up worktrees after execution, leaving dozens of
stale worktrees with uncommitted changes.

**Why it fails:** Stale worktrees hold git locks, consume disk space, and
create confusion about which branch is current. Future executions may
accidentally use a stale worktree with polluted state.

**Fix:** Include `scripts/cleanup-worktrees.sh` in the execution plan. Run
it after every merge sequence. Verify with `git worktree list` that only
the main worktree remains.

---

## Recovery Anti-Patterns

### 12. The Blind Respawn

**What:** Respawning a failed agent with the exact same prompt, expecting
different results.

**Why it fails:** If the prompt didn't work the first time, it won't work
the second time (unless the failure was transient). The agent will make the
same mistakes, waste the same tokens, and hit the same wall.

**Fix:** When respawning after failure:
1. Diagnose WHY the agent failed (not just that it failed)
2. Update the spawn prompt to address the root cause:
   - If task was too complex: split it
   - If contract was wrong: fix the contract
   - If context was missing: add it to the prompt
   - If approach was wrong: suggest an alternative in the prompt
3. Include a note about the previous failure: "A previous agent attempted
   this task and failed because [reason]. Avoid this approach."

---

### 13. The Recovery Cascade

**What:** Spawning recovery agents to fix recovery agents. A fix introduces
a new problem, which gets its own recovery agent, which introduces another
problem.

**Why it fails:** Each layer of recovery adds cost and complexity. After
2-3 layers, the problem is usually architectural -- no amount of tactical
fixing will help.

**Fix:** Two-layer recovery limit:
1. First failure: Re-prompt or spawn Recovery Agent
2. Second failure: Spawn Recovery Agent with additional context
3. Third failure: HALT. The problem is systemic. Reassess the
   architecture plan, not the agent prompts.

---

### 14. The Ignored Budget Breach

**What:** Execution continues past the budget ceiling because "we're almost
done."

**Why it fails:** "Almost done" in multi-agent execution is deceptive. The
remaining 20% of work often takes 50% of the budget. Merge failures,
integration issues, and edge cases compound costs. The final bill is 2-3x
the "almost done" estimate.

**Fix:** Hard budget ceilings are HARD. When the ceiling is reached:
1. Commit all progress
2. Assess what's essential vs. deferrable
3. Execute scope reduction
4. If even essential work exceeds budget: HALT and re-plan with a larger
   budget or smaller scope

---

## Execution Decision Anti-Patterns

### 15. The Premature Optimization

**What:** Using advanced dispatch patterns (staggered starts, pipeline overlap)
on the first execution.

**Why it fails:** Advanced patterns add complexity and failure modes. If you
haven't executed a successful sequential wave dispatch yet, adding staggered
starts creates debugging nightmares when dependencies aren't met.

**Fix:** Start with the simplest dispatch pattern:
1. First execution: sequential waves, simple dispatch
2. Second execution: add staggered starts if justified by timeline pressure
3. Third execution: consider pipeline overlap if dependency structure supports it

---

### 16. The Over-Specialized Swarm

**What:** Creating 10+ specialist roles for a small project.

**Why it fails:** Each specialist adds coordination overhead. A project with
5 modules doesn't need separate Security Reviewer, Performance Auditor, Test
Hardener, Documentation Writer, and Code Style Enforcer. The coordination
cost of 10 agents exceeds the value of specialization.

**Fix:** Match specialist count to project complexity:
- Small (3-5 modules): 4-5 specialists (Foundation, 2 Feature, Integration)
- Medium (5-10 modules): 6-8 specialists (+ Merge Conductor, Test Hardener)
- Large (10-15 modules): 8-10 specialists (+ Security, Performance, Recovery)

Never create more specialists than there are meaningful boundaries in the work.

---

## Recovery Decision Tree for Execution Failures

```
Execution failure detected
├── Single agent stuck/failed
│   ├── First failure → Re-prompt with error context
│   ├── Second failure → Spawn fresh Recovery Agent
│   └── Third failure → HALT module, reassess task decomposition
│
├── Gate failure (tests don't pass)
│   ├── Single module → Fix in feature branch, re-merge
│   ├── Multiple modules → Check foundation contracts
│   └── All tests → Build is broken, check for circular deps
│
├── Merge failure
│   ├── Package.json → Manual merge, deduplicate
│   ├── Single file → Boundary violation, fix ownership
│   └── Multiple files → Architecture issue, update plan
│
├── Budget breach
│   ├── Single agent → Kill agent, split task
│   ├── Wave breach → Scope reduction
│   └── Total breach → HALT all, re-plan
│
└── Cascading failure (>2 agents)
    ├── Foundation issue → Fix foundation, restart wave
    ├── Architecture issue → HALT, update architecture
    └── PRD issue → HALT, update PRD, full re-plan
```
