# Progress Tracking & Recovery

How to monitor implementation progress across parallel agents and recover from
failures. Multi-agent systems fail silently -- without active tracking, you
discover problems at integration time when they are most expensive to fix.

## Progress Tracking Mechanisms

### Task Status Board

Maintain a task status file in the project root that serves as the single source
of truth for implementation progress:

```markdown
<!-- IMPLEMENTATION_STATUS.md -->
# Implementation Status

Last updated: [timestamp]

## Wave 1: Foundation
| Task | Agent | Status | Verification | Notes |
|------|-------|--------|-------------|-------|
| T0.1 | A0 | Done | PASS | Types exported |
| T0.2 | A0 | Done | PASS | Schema created |
| T0.3 | A0 | In Progress | - | Working on utils |
| T0.R1 | A0 | Not Started | - | |

## Wave 2: Features
| Task | Agent | Status | Verification | Notes |
|------|-------|--------|-------------|-------|
| T1.1 | A1 | Not Started | - | Blocked: Tier 0 Gate |
| T2.1 | A2 | Not Started | - | Blocked: Tier 0 Gate |

## Gates
| Gate | Status | Timestamp |
|------|--------|-----------|
| Tier 0 | PENDING | - |
| Tier 1 | PENDING | - |
| Final | PENDING | - |
```

**Update rules:**
- Agents update their own task status upon completion
- The lead agent (or human) updates gate status
- Status transitions: `Not Started` -> `In Progress` -> `Testing` -> `Done` or `Failed`
- Failed tasks include a brief note explaining the failure

### Feature List File

For larger projects, maintain a structured feature list that maps directly to
PRD requirements:

```json
{
  "features": [
    {
      "id": "F1",
      "name": "User Authentication",
      "prd_phase": "Phase 2",
      "tasks": ["T1.1", "T1.2", "T1.3", "T1.R1"],
      "status": "in_progress",
      "tests_passing": 8,
      "tests_total": 12,
      "agent": "A1"
    },
    {
      "id": "F2",
      "name": "Content Management",
      "prd_phase": "Phase 3",
      "tasks": ["T2.1", "T2.2", "T2.R1"],
      "status": "not_started",
      "tests_passing": 0,
      "tests_total": 0,
      "agent": "A2"
    }
  ],
  "last_updated": "2026-03-05T12:00:00Z"
}
```

### Git Commit Strategy

Agents should commit after every task completion with meaningful messages:

```
feat(shared): T0.1 - Add shared type definitions

- Created User, Session, Content types in src/shared/types/
- All types exported from index.ts
- Type tests pass (5/5)
- TypeCheck: PASS, Lint: PASS
```

**Why this matters:**
- Commits provide a recovery point if an agent's context is exhausted
- Commit messages document what was actually built (vs. what was planned)
- A new agent can be spawned and pointed at the commit history to resume work

### Per-Agent Progress Files

For Agent Teams or complex implementations, each agent maintains a progress file:

```markdown
<!-- .claude/progress/agent-a1.md -->
# Agent A1 Progress: Auth Module

## Completed
- T1.1: Auth service skeleton (PASS)
  - Files: src/modules/auth/index.ts, src/modules/auth/auth.service.ts
  - Tests: 5/5 passing

## In Progress
- T1.2: Login/logout implementation
  - Started: [timestamp]
  - Status: Writing tests

## Remaining
- T1.3: Session management
- T1.R1: Module refactoring

## Issues
- None currently
```

## Recovery Procedures

### Task Failure Recovery

**Symptom:** Agent reports a task as Done but tests fail, or agent gets stuck
and cannot make tests pass.

**Recovery steps:**
1. **Diagnose:** Read the test output. Is it a logic error, contract mismatch, or environment issue?
2. **If logic error:** Re-prompt the agent with the test failure output and ask it to fix
3. **If contract mismatch:** The foundation contract doesn't match what the agent expected. Fix in foundation (requires re-running Tier 0 gate) or adjust the agent's implementation
4. **If the agent is stuck (>20 min on a single task):**
   - Commit whatever progress exists
   - Kill the session
   - Split the task into smaller sub-tasks
   - Spawn a fresh agent with the sub-tasks and a pointer to the committed progress
5. **If the task is fundamentally wrong:** Discard the worktree, update the implementation plan, respawn

### Agent Failure Recovery

**Symptom:** Agent session dies, disconnects, or becomes unresponsive.

**Recovery steps:**
1. Check if the agent committed any progress (look at git log in its worktree)
2. If progress exists: spawn a new agent, point it at the remaining tasks
3. If no progress: spawn a new agent with the original spawn prompt
4. If repeated failures: the task may be too complex -- split it

### Context Window Exhaustion

**Symptom:** Agent output quality degrades (incomplete implementations, ignored
constraints, repetitive patterns).

**Prevention:**
- Size tasks to 5-15 minutes (well within context limits)
- Each agent spawns fresh per task group (not reused across waves)
- Keep spawn prompts focused (no unnecessary context)

**Recovery:**
1. Have the agent commit its current progress
2. Kill the session
3. Spawn a fresh agent with remaining tasks
4. Point the new agent at the committed code for context

### Merge Conflict Recovery

**Symptom:** `git merge` fails with conflicts.

**Recovery steps:**
1. `git merge --abort` (don't try to resolve blindly)
2. Identify conflicting files
3. **If file is in one agent's manifest:** The other agent violated file ownership. Rebuild the offending agent's branch without the violation.
4. **If file is shared (package.json, config):** Manually merge the changes
5. **If multiple files conflict:** This indicates an architecture boundary issue. Reassess the module boundaries before proceeding.

### Cascading Failure Recovery

**Symptom:** Multiple agents fail, or integration tests fail across the board.

**This is the most serious failure mode.** Steps:
1. **Halt all running agents** -- don't let them continue producing work on a broken foundation
2. **Assess the scope:** Is it a foundation issue (affects everything) or a localized issue (affects one dependency chain)?
3. **If foundation issue:** Rebuild foundation, re-run Tier 0 gate, then respawn all feature agents
4. **If localized issue:** Fix the specific module, re-merge, re-test
5. **If architectural issue:** Update the architecture plan (module boundaries or contracts were wrong), then regenerate the affected sections of the implementation plan

### Contract Change Recovery

**Symptom:** A Tier 1+ agent discovers that a foundation contract doesn't support
a needed use case.

**Recovery steps:**
1. Agent creates `CONTRACT_CHANGE_REQUEST.md`:
   ```markdown
   # Contract Change Request
   Requested by: Agent A1 (auth module)
   Contract: UserType in src/shared/types/user.ts
   Change: Add optional `lastLoginAt: Date` field
   Reason: Login tracking requires timestamp
   Impact: All modules consuming UserType must handle the new optional field
   ```
2. Lead reviews and approves/rejects
3. If approved:
   - Pause all agents consuming the affected contract
   - Update the foundation contract
   - Re-run Tier 0 gate
   - Notify affected agents of the change
   - Resume paused agents
4. If rejected: The requesting agent must find an alternative approach

## Monitoring Cadence

### For Subagent Execution
- Check task status after each agent completes (automatic -- results return to main session)
- Run gate verification scripts at wave boundaries
- Monitor git branches for unexpected file changes

### For Agent Teams
- Use `TeammateIdle` hooks to monitor agent progress
- Lead agent should periodically check task status and verify actual work output
  (agents may mark tasks Done prematurely)
- Use `TaskCompleted` hooks to trigger verification

### For Manual Worktree Sessions
- Periodically check each worktree's git log for progress
- Run per-module gate scripts in each worktree before attempting merge
- Maintain the task status board manually

## When to Abort and Replan

Sometimes the implementation plan itself is wrong. Signs that replanning is needed:

- **3+ tasks fail in the same module** -- the decomposition is wrong
- **Contract change requests keep coming** -- the architecture plan's contracts are insufficient
- **Merge conflicts on every branch** -- module boundaries are wrong
- **Foundation keeps getting modified** -- it wasn't frozen properly or it's incomplete
- **Estimated time exceeded by 2x** -- tasks were undersized or the architecture is more complex than planned

When replanning:
1. Commit all progress
2. Document what worked and what failed
3. Update the architecture plan if needed
4. Regenerate the implementation plan for remaining work
5. Do NOT re-implement completed, passing modules
