# Step-by-Step Execution Runbook Template

The execution runbook is the operator's playbook. An operator (human or lead agent)
follows it step-by-step to drive the swarm from start to completion. Every action
is explicit -- no improvisation required.

## Runbook Structure

The runbook follows a strict sequence:
1. Pre-flight
2. For each wave: Dispatch -> Monitor -> Gate -> Transition
3. Post-execution

## Runbook Template

```markdown
# Execution Runbook: [Project Name]

Execution Plan: [EXECUTION.md]
Started: [timestamp]
Operator: [name]
Runtime: [Subagents / Agent Teams / Manual Worktrees]

---

## Step 1: Pre-Flight (5 min)

### 1.1 Run pre-flight checklist
```bash
bash scripts/preflight.sh
```
Expected: "ALL CHECKS PASSED"

If any check fails: FIX before continuing. Do not dispatch agents on a
broken baseline.

### 1.2 Verify budget
- API credits available: $[N]+ needed
- Terminals available: [N] (for Manual Worktrees)

### 1.3 Create execution log
```bash
echo "# Execution Log" > EXECUTION_LOG.md
echo "Started: $(date)" >> EXECUTION_LOG.md
echo "---" >> EXECUTION_LOG.md
git add EXECUTION_LOG.md && git commit -m "chore: start execution"
```

---

## Step 2: Wave 0 - Setup (~5 min)

### 2.1 Create agent definitions
```bash
mkdir -p .claude/agents
# [Commands to create each agent definition file]
```

### 2.2 Create gate scripts
```bash
mkdir -p scripts
# [Commands to create each gate script and mark executable]
chmod +x scripts/*.sh
```

### 2.3 Initialize progress tracking
```bash
# [Command to create IMPLEMENTATION_STATUS.md]
```

### 2.4 Commit setup
```bash
git add .claude/agents/ scripts/ IMPLEMENTATION_STATUS.md
git commit -m "chore: initialize execution environment"
```

### 2.5 Log
```bash
echo "Wave 0: Setup COMPLETE $(date)" >> EXECUTION_LOG.md
```

---

## Step 3: Wave 1 - Foundation (~[N] min)

### 3.1 Dispatch Foundation Builder
```
[Exact dispatch command for the selected runtime]
```

### 3.2 Monitor (every 10 min)
```bash
# Check commits
git log --oneline -5 [branch-or-worktree]

# Check status file
cat IMPLEMENTATION_STATUS.md 2>/dev/null
```

**If no commits for >15 min:** Check agent status, consider timeout.
**If type errors in commits:** Foundation quality issue, investigate.

### 3.3 Gate: Foundation Complete
When agent reports completion:
```bash
bash scripts/gate-wave-1.sh
```

Expected: "Wave 1 Gate: PASSED"

**If gate fails:**
1. Read error output
2. Attempt fix: re-prompt agent with error
3. If agent context exhausted: spawn Recovery Agent
4. If architectural: HALT and update architecture plan

### 3.4 Post-gate: Freeze Foundation
```bash
# Merge foundation branch (if worktree)
git merge [foundation-branch] --no-edit

# Update status
echo "Wave 1: Foundation COMPLETE $(date)" >> EXECUTION_LOG.md
echo "Foundation FROZEN -- all src/shared/ files are read-only" >> EXECUTION_LOG.md
```

---

## Step 4: Wave 2 - Feature Implementation (~[N] min)

### 4.1 Dispatch Feature Agents (Parallel)
```
[Exact parallel dispatch commands for selected runtime]
```

**Dispatched agents:**
- S1: [Module A] on branch tier-1/[module-a]
- S2: [Module B] on branch tier-1/[module-b]
- S3: [Module C] on branch tier-1/[module-c]

### 4.2 Monitor (every 10 min)
```bash
# Quick status of all agents
bash scripts/check-all-agents.sh
```

**Health check for each agent:**
- Recent commits? (every 5-15 min = healthy)
- Tests in commit messages? (TDD being followed)
- Files outside manifest? (boundary violation -- intervene)

### 4.3 Handle Stragglers
If one agent is significantly behind:
1. Check if it's stuck (no commits) vs. slow (working on large task)
2. If stuck >20 min: commit progress, spawn fresh agent for remaining tasks
3. If slow: let it continue, other agents wait at the gate
4. If consistently failing: split the remaining tasks, spawn recovery agent

### 4.4 Per-Module Gates
As each agent completes:
```bash
bash scripts/gate-module.sh [module-name]
```

### 4.5 Wave 2 Gate
After ALL agents complete and pass per-module gates:
```bash
bash scripts/gate-wave-2.sh
```

### 4.6 Log
```bash
echo "Wave 2: Features COMPLETE $(date)" >> EXECUTION_LOG.md
echo "  S1 [Module A]: PASSED" >> EXECUTION_LOG.md
echo "  S2 [Module B]: PASSED" >> EXECUTION_LOG.md
echo "  S3 [Module C]: PASSED" >> EXECUTION_LOG.md
```

---

## Step 5: Merge Sequence (~[N] min)

### 5.1 Run sequential merge
```bash
bash scripts/merge-sequence.sh
```

**If merge conflict on branch X:**
1. Abort: `git merge --abort`
2. Diagnose: which files conflict?
3. If same file, same section: boundary violation in architecture
4. If package.json: manual merge, deduplicate dependencies
5. Fix in the feature branch, re-attempt merge
6. If stuck: spawn Recovery Agent with conflict details

### 5.2 Verify merged state
```bash
npm test --ci
npx tsc --noEmit
npm run build
```

### 5.3 Cleanup worktrees
```bash
bash scripts/cleanup-worktrees.sh
```

### 5.4 Log
```bash
echo "Merge Sequence: COMPLETE $(date)" >> EXECUTION_LOG.md
```

---

## Step 6: Wave 3 - Integration (~[N] min)

### 6.1 Dispatch Integration Wirer
```
[Exact dispatch command]
```

### 6.2 Monitor (every 10 min)
Check for commits, verify route composition progress.

### 6.3 Final Gate
```bash
bash scripts/gate-final.sh
```

Expected: "FINAL GATE: PASSED"

**If final gate fails:**
1. Identify failing test category (unit, integration, E2E)
2. If integration test: check module wiring
3. If E2E test: check full request flow
4. Spawn Recovery Agent if needed

---

## Step 7: Wave 4 - Hardening (Optional, ~[N] min)

### 7.1 Dispatch review agents (parallel)
```
[Dispatch Test Hardener, Security Reviewer, Performance Auditor in parallel]
```

### 7.2 Review findings
Collect reports from all reviewers. Prioritize:
1. CRITICAL security findings -> fix immediately
2. Performance issues affecting user experience -> fix
3. Test coverage gaps for error paths -> fix
4. Everything else -> document for future sprint

### 7.3 Apply fixes
If fixes needed, dispatch a Recovery Agent for each fix.
Re-run final gate after each fix.

---

## Step 8: Completion

### 8.1 Verify all acceptance criteria
```
[Map each PRD criterion to its test or manual verification]
```

### 8.2 Final cleanup
```bash
bash scripts/cleanup.sh
```

### 8.3 Update status
```bash
echo "=== EXECUTION COMPLETE ===" >> EXECUTION_LOG.md
echo "Completed: $(date)" >> EXECUTION_LOG.md
echo "All acceptance criteria verified." >> EXECUTION_LOG.md
git add . && git commit -m "chore: execution complete"
```

### 8.4 Fill retrospective
Update the Post-Execution Retrospective section of EXECUTION.md with:
- Actual vs. projected time
- Actual vs. projected cost
- Gate failures encountered
- Re-planning events
- Lessons learned
```

---

## Runbook Conventions

### Status Markers
Use consistent status markers throughout the log:
- `COMPLETE` -- step finished successfully
- `PASSED` -- gate verification passed
- `FAILED` -- gate or step failed (include details)
- `SKIPPED` -- step intentionally skipped (include reason)
- `HALTED` -- execution stopped (include reason and recovery plan)

### Timing
- Log timestamps for every major event (wave start/end, gate results)
- Track elapsed time vs. projected time
- If elapsed > 1.5x projected: assess whether to continue or re-plan

### Escalation Chain
1. Re-prompt agent with error context (cheapest)
2. Spawn Recovery Agent (moderate cost)
3. Manual diagnosis and fix (human time)
4. Halt and re-plan (most expensive, last resort)
