# Adaptive Re-Planning Strategy

Live execution rarely matches the plan perfectly. Research shows that
coordination failures account for 36.94% of all multi-agent system failures
(MAST taxonomy), and rigid plans that don't adapt to runtime conditions
amplify these failures. Adaptive re-planning builds resilience into the
execution by defining triggers, decision procedures, and recovery circuits.

## Re-Planning Philosophy

The key insight from Anthropic's 2026 Agentic Coding Trends Report and the
HuggingFace implementation guide: agents should "reduce scope" to ship the
smallest safe increment rather than grinding on failing approaches. This
principle applies at every level -- task, wave, and execution-wide.

From Steve Yegge's Gas Town model: use "molecules" (chains of sequenced
atomic tasks persisting across crashes via git backing) so that progress
survives agent failures. Every commit is a recovery point.

From Cursor's architecture: Judge agents determine whether to continue at
each cycle end, providing an automatic re-assessment mechanism.

## Trigger Taxonomy

### Level 1: Task-Level Triggers

| Trigger | Detection | Automatic Action |
|---------|-----------|-----------------|
| **Task timeout** | No commits for >20 min | Commit progress, spawn fresh agent with remaining tasks |
| **Task too complex** | Single task running >25 min | Split task into sub-tasks, spawn fresh agents |
| **Agent degradation** | Agent running >35 min total (Zylos: failure rate quadruples past this) | Checkpoint commit, kill session, spawn fresh agent |
| **Test loop** | Agent re-runs failing test >5 times | Re-prompt with error, suggest alternative approach |
| **Context exhaustion** | Output quality degrades (repetitive, incomplete) | Commit progress, kill session, spawn fresh |
| **Scope creep** | Agent modifies files outside manifest | Revert unauthorized changes, re-prompt with constraints |

**Task-level re-planning procedure:**
1. Commit whatever progress exists (`git add -A && git commit -m "WIP: [task] partial progress"`)
2. Assess: what's done, what's remaining, why did it fail?
3. If remaining work is small: re-prompt current agent with error context
4. If remaining work is large: split into 2-3 sub-tasks
5. Spawn fresh agent(s) with updated prompts pointing to committed progress
6. Continue monitoring

### Level 2: Module-Level Triggers

| Trigger | Detection | Automatic Action |
|---------|-----------|-----------------|
| **Multiple task failures** | >2 tasks fail in same module | Halt module, diagnose systemic issue |
| **Contract mismatch** | Module tests pass but contract tests fail | File CONTRACT_CHANGE_REQUEST |
| **Boundary violation** | Agent modified files outside module | Revert violations, re-prompt with explicit constraints |
| **Agent crash** | Session dies or disconnects | Check git log for progress, respawn |

**Module-level re-planning procedure:**
1. Halt the module's agent
2. Review all committed code and test results
3. Identify root cause:
   - If implementation approach is wrong: update spawn prompt, respawn
   - If contract is insufficient: file CONTRACT_CHANGE_REQUEST
   - If task decomposition is wrong: re-decompose, update remaining tasks
4. Respawn with updated context
5. Do NOT restart completed, passing tasks

### Level 3: Wave-Level Triggers

| Trigger | Detection | Automatic Action |
|---------|-----------|-----------------|
| **Wave timeout** | Wall clock > 2x estimated wave duration | Pause, assess all agents |
| **Budget exceeded** | Token spend > 150% of wave budget ceiling | Pause, prioritize remaining work |
| **Gate failure** | Wave exit gate fails after all agents complete | Diagnose which module(s) broke |
| **Cascading failure** | >2 agents fail in the same wave | HALT all agents immediately |

**Wave-level re-planning procedure:**
1. Halt all agents in the wave
2. Commit all progress across all agents
3. Assess the damage:
   - Which modules completed successfully? (preserve these)
   - Which modules failed? (diagnose root causes)
   - Is the architecture plan still valid?
4. Decision:
   - If 1-2 modules failed: fix and respawn only those
   - If most modules failed: systemic issue -- check foundation
   - If foundation is the issue: rebuild foundation, restart the wave
5. Update the execution plan with revised estimates and prompts
6. Resume execution from the last stable checkpoint

### Level 4: Execution-Level Triggers

| Trigger | Detection | Automatic Action |
|---------|-----------|-----------------|
| **Total budget exceeded** | Cumulative spend hits hard ceiling | HALT all, scope reduction required |
| **Architecture mismatch** | Multiple contract change requests | HALT, update architecture plan |
| **PRD contradiction** | Implementation reveals PRD impossibility | HALT, update PRD |
| **Environment failure** | CI/CD, dependencies, or infra broken | HALT, fix environment first |

**Execution-level re-planning procedure:**
1. HALT everything immediately
2. Commit ALL progress across ALL agents and branches
3. Document what worked and what failed in EXECUTION_LOG.md
4. Assess options:
   - **Scope reduction**: Identify minimum viable subset, cut non-essential modules
   - **Architecture revision**: Update module boundaries or contracts
   - **PRD revision**: Update requirements based on discovered constraints
   - **Full restart**: If foundation is broken, restart from scratch (rare)
5. Update upstream documents (PRD, Architecture, Implementation Plan)
6. Generate a new execution plan for the remaining work
7. Resume from the last stable checkpoint

## Recovery Circuits

Recovery circuits are pre-built procedures that fire automatically when
specific conditions are met. They reduce the operator's cognitive load
during crisis moments.

### Circuit 1: Agent Stuck Recovery

```
TRIGGER: Agent has no commits for >20 minutes

ACTION:
1. Check if the agent's session is still alive
2. If alive: read the last few messages for signs of looping
3. If looping: kill session, commit progress
4. Spawn fresh agent with:
   - Remaining tasks only
   - Pointer to committed progress: "Continue from commit [hash]"
   - Additional context about what was attempted
5. Set a new 20-minute timer
```

### Circuit 2: Gate Failure Recovery

```
TRIGGER: Any gate verification script fails

ACTION:
1. Parse the failure output to identify:
   - Which test(s) failed
   - Which file(s) are involved
   - Which module(s) are responsible
2. If single module failure:
   - Spawn Recovery Agent targeting that module
   - Provide the exact error output in the spawn prompt
3. If cross-module failure:
   - Check for contract mismatches
   - If contract issue: file CONTRACT_CHANGE_REQUEST, fix foundation
   - If wiring issue: dispatch Integration Wirer to fix
4. Re-run gate after fix
5. If gate fails again: escalate to operator
```

### Circuit 3: Budget Breach Recovery

```
TRIGGER: Wave spend exceeds 150% of ceiling

ACTION:
1. Pause all agents in the wave
2. Assess: what's complete, what's remaining?
3. For remaining work, determine:
   - Essential (must complete for minimum viable delivery)
   - Desirable (improves quality but not critical)
   - Deferrable (can be done in a future execution)
4. Cut deferrable tasks first, then desirable
5. Resume with reduced scope
6. If even essential tasks exceed budget: HALT and escalate
```

### Circuit 4: Cascading Failure Recovery

```
TRIGGER: >2 agents fail in the same wave

ACTION:
1. HALT all agents immediately
2. This is likely a foundation issue -- diagnose:
   a. Are shared types incorrect or incomplete?
   b. Are mock data fixtures wrong?
   c. Is the database schema missing something?
   d. Is CLAUDE.md misconfigured?
3. If foundation issue:
   a. Fix foundation
   b. Re-run Tier 0 gate
   c. Respawn ALL feature agents (they may have built on broken contracts)
4. If not foundation:
   a. Each failure is independent
   b. Diagnose and fix individually
   c. Respawn only failed agents
5. Update execution plan with new estimates
```

### Circuit 5: Contract Change Request

```
TRIGGER: Agent creates CONTRACT_CHANGE_REQUEST.md

ACTION:
1. Pause ALL agents consuming the affected contract
2. Review the change request:
   - Is the change necessary?
   - What is the blast radius?
   - How many modules are affected?
3. If approved:
   a. Update the foundation contract
   b. Update mock data if affected
   c. Re-run Tier 0 gate
   d. Notify affected agents of the change
   e. Resume paused agents
4. If rejected:
   a. Inform requesting agent to find an alternative
   b. Resume paused agents
5. Log the decision in EXECUTION_LOG.md
```

## Scope Reduction Priority List

When budget or time forces scope reduction, cut in this order
(least important first):

1. **Wave 4 hardening** (defer security/perf review to manual process)
2. **Non-critical modules** (defer to a follow-up execution)
3. **E2E tests** (keep unit + integration, defer E2E)
4. **Refactoring tasks** (accept tech debt temporarily)
5. **Advanced features** within modules (ship core, defer edge cases)
6. **NEVER cut**: foundation, core modules, verification gates

## Decision Framework

When something goes wrong, use this decision tree:

```
Is the failure localized to one agent/task?
├── YES → Re-prompt or spawn Recovery Agent
├── NO → Is it localized to one module?
│   ├── YES → Fix module, respawn agent for that module
│   └── NO → Is it a foundation issue?
│       ├── YES → Fix foundation, re-gate, respawn affected agents
│       └── NO → Is the architecture plan wrong?
│           ├── YES → HALT, update architecture, re-plan
│           └── NO → Is the PRD impossible?
│               ├── YES → HALT, update PRD, re-plan everything
│               └── NO → Environment issue → fix environment, resume
```
