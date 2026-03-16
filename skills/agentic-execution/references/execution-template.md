# Execution Plan Template

Use this template to generate execution plans for live multi-agent swarm
dispatch. Adapt section depth based on project size and orchestration runtime.

---

```markdown
# Execution Plan: [Project Name]

Generated from: [PRD] + [Architecture Plan] + [Implementation Plan]
Date: [current date]
Orchestration Runtime: [Subagents with Worktrees | Agent Teams | Manual Worktrees]

## 1. Execution Overview

### Pipeline Summary

| Document | File | Status |
|----------|------|--------|
| PRD | [filename] | Validated |
| Architecture Plan | [filename] | Validated |
| Implementation Plan | [filename] | Validated |

### Execution Metrics

| Metric | Value |
|--------|-------|
| Total tasks | [N] |
| Execution waves | [N] |
| Max parallel agents | [N] |
| Specialist roles defined | [N] |
| Critical path | [N tasks, ~N min] |
| Estimated wall-clock time | [N min] |
| Total token budget | ~[N]M tokens |
| Estimated cost | ~$[N] |
| Gate checkpoints | [N] |

### Orchestration Runtime Decision

**Selected runtime:** [Subagents with Worktrees / Agent Teams / Manual Worktrees]

| Factor | Assessment | Implication |
|--------|-----------|-------------|
| Inter-agent communication | [Yes/No] | [Teams vs Subagents] |
| File overlap risk | [Low/Med/High] | [Worktree isolation required?] |
| Task count | [N] | [Overhead threshold] |
| Budget sensitivity | [Low/Med/High] | [Model tier optimization] |
| Operator experience | [Beginner/Intermediate/Expert] | [Manual vs automated dispatch] |

### Specialist Agent Roster

| Agent ID | Specialist Role | Model | Wave(s) | Task Count | Est. Tokens | Est. Cost |
|----------|----------------|-------|---------|------------|-------------|-----------|
| S0 | Foundation Builder | claude-opus-4-6 | 1 | [N] | [N]K | $[N] |
| S1 | [Module] Implementer | claude-sonnet-4-6 | 2 | [N] | [N]K | $[N] |
| S2 | [Module] Implementer | claude-sonnet-4-6 | 2 | [N] | [N]K | $[N] |
| S3 | [Module] Implementer | claude-sonnet-4-6 | 2 | [N] | [N]K | $[N] |
| S4 | Integration Wirer | claude-sonnet-4-6 | 3 | [N] | [N]K | $[N] |
| S5 | Merge Conductor | claude-opus-4-6 | 2→3 | [N] | [N]K | $[N] |
| S6 | Test Hardener | claude-sonnet-4-6 | 4 | [N] | [N]K | $[N] |
| **Total** | | | | **[N]** | **[N]K** | **$[N]** |

---

## 2. Pre-Flight Checklist

Run these checks before dispatching any agents. Every check must pass.

### Environment Verification

```bash
#!/bin/bash
set -e
echo "=== Pre-Flight Checklist ==="

# 1. Git state
echo "1. Verifying clean git state..."
git status --porcelain | grep -q . && echo "FAIL: Uncommitted changes" && exit 1
echo "   PASS: Clean working tree"

# 2. On correct branch
echo "2. Verifying branch..."
BRANCH=$(git branch --show-current)
echo "   Current branch: $BRANCH"
[[ "$BRANCH" == "main" || "$BRANCH" == "master" ]] || echo "   WARNING: Not on main/master"

# 3. Dependencies installed
echo "3. Verifying dependencies..."
[command to verify dependencies, e.g., npm ci or pip install -r requirements.txt]
echo "   PASS: Dependencies installed"

# 4. Build works
echo "4. Verifying build..."
[build command]
echo "   PASS: Build succeeds"

# 5. Tests pass
echo "5. Verifying existing tests..."
[test command]
echo "   PASS: Tests pass"

# 6. Required files exist
echo "6. Verifying pipeline documents..."
[[ -f "[PRD file]" ]] || (echo "FAIL: PRD not found" && exit 1)
[[ -f "[Architecture file]" ]] || (echo "FAIL: Architecture plan not found" && exit 1)
[[ -f "[Implementation file]" ]] || (echo "FAIL: Implementation plan not found" && exit 1)
echo "   PASS: All pipeline documents present"

# 7. Agent definitions exist
echo "7. Verifying agent definitions..."
[[ -d ".claude/agents/" ]] || (echo "FAIL: .claude/agents/ not found" && exit 1)
for agent in [list agent definition files]; do
  [[ -f ".claude/agents/$agent" ]] || (echo "FAIL: $agent not found" && exit 1)
done
echo "   PASS: All agent definitions present"

# 8. CLAUDE.md configured
echo "8. Verifying CLAUDE.md..."
[[ -f "CLAUDE.md" ]] || (echo "FAIL: CLAUDE.md not found" && exit 1)
echo "   PASS: CLAUDE.md present"

echo "=== Pre-Flight: ALL CHECKS PASSED ==="
echo "Ready to dispatch Wave 1."
```

### Manual Pre-Flight Checks

- [ ] Sufficient API credits / token budget available
- [ ] Terminal sessions available for agent count ([N] terminals needed for Manual Worktrees)
- [ ] No active PRs or merges in progress on the repository
- [ ] Team notified that multi-agent execution is starting (if shared repo)
- [ ] Implementation plan reviewed and approved by user

---

## 3. Specialist Agent Definitions

### .claude/agents/foundation-builder.md

```markdown
---
name: foundation-builder
description: Builds shared types, schemas, utilities, and mock data
tools: Read, Edit, Write, Bash, Grep, Glob
model: opus
isolation: worktree
---

[Paste the full foundation builder agent definition from the implementation
plan's spawn prompts, adapted with specialist-specific constraints]
```

### .claude/agents/feature-implementer.md

```markdown
---
name: feature-implementer
description: Implements a single feature module from the architecture plan
tools: Read, Edit, Write, Bash, Grep, Glob
model: sonnet
isolation: worktree
permissionMode: acceptEdits
---

[Paste the feature implementer agent template]
```

### .claude/agents/integration-wirer.md

```markdown
---
name: integration-wirer
description: Wires all modules together with routes, middleware, and E2E tests
tools: Read, Edit, Write, Bash, Grep, Glob
model: sonnet
isolation: worktree
---

[Paste the integration agent template]
```

### .claude/agents/merge-conductor.md

```markdown
---
name: merge-conductor
description: Manages sequential branch merges with test-after-each verification
tools: Read, Bash, Grep, Glob
model: opus
---

[Merge conductor definition -- read-only except for git operations]
```

[Include additional specialist agents as needed: test-hardener, security-reviewer,
performance-auditor, recovery-agent]

---

## 4. Wave Dispatch Plan

### Wave 0: Pre-Execution Setup

**Purpose:** Initialize project structure, CLAUDE.md, and agent configurations.
**Dispatcher:** Lead (human or orchestrator)
**Duration:** ~5 min

```bash
# Step 1: Create agent definition files
mkdir -p .claude/agents
# [Commands to write each agent definition file]

# Step 2: Create/update CLAUDE.md with multi-agent instructions
# [Command to write CLAUDE.md]

# Step 3: Create gate verification scripts
mkdir -p scripts
# [Commands to write each gate script]

# Step 4: Initialize progress tracking
# [Command to create IMPLEMENTATION_STATUS.md]

# Step 5: Create initial commit
git add .claude/agents/ scripts/ CLAUDE.md IMPLEMENTATION_STATUS.md
git commit -m "chore: initialize multi-agent execution environment"
```

**Exit gate:** All setup files committed, pre-flight checklist passes.

---

### Wave 1: Foundation

**Purpose:** Build shared types, schemas, utilities, and mock data.
**Agent:** S0 (Foundation Builder)
**Model:** claude-opus-4-6
**Tasks:** [T0.1, T0.2, ..., T0.R1]
**Estimated duration:** ~[N] min
**Token budget:** [N]K tokens (~$[N])
**Timeout:** [N] min (escalate if exceeded)

#### Pre-conditions
- [ ] Pre-flight checklist passed
- [ ] Wave 0 setup committed
- [ ] Clean git state on main branch

#### Dispatch Command

**For Subagents with Worktrees:**
```
Use the Agent tool:
  description: "Foundation Builder"
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: |
    [Paste the complete foundation builder spawn prompt from the
    implementation plan, including all 7 sections: role, architecture
    context, interface contracts, file manifest, task sequence,
    constraints, success criteria]
```

**For Agent Teams:**
```
Create an agent team for the foundation phase.
Spawn one teammate: "Foundation Builder" with this prompt:
[Paste foundation builder spawn prompt]
Require plan approval before any file changes.
```

**For Manual Worktrees:**
```bash
# Create worktree
git worktree add ../[project]-foundation -b tier-0/foundation

# Launch Claude in the worktree
cd ../[project]-foundation
claude --agent foundation-builder

# Paste this as the initial prompt:
# [Paste foundation builder spawn prompt]
```

#### Monitoring (During Execution)

Check progress every ~10 minutes:
```bash
# For worktree-based execution:
cd ../[project]-foundation
git log --oneline -5   # Check commits
cat IMPLEMENTATION_STATUS.md 2>/dev/null   # Check task status

# For subagent execution:
# Check the Agent tool result when it returns
```

**Health indicators:**
- Commits appearing every 5-10 minutes = healthy
- No commits for >15 minutes = investigate
- Type errors in git log messages = foundation quality issue

#### Exit Gate

```bash
#!/bin/bash
# scripts/gate-wave-1.sh
set -e
echo "=== Wave 1 Exit Gate: Foundation ==="

echo "1. Type checking..."
npx tsc --noEmit || exit 1

echo "2. Foundation tests..."
npm test -- --testPathPattern="shared" --ci || exit 1

echo "3. Linting..."
npx eslint src/shared/ --max-warnings=0 || exit 1

echo "4. Export verification..."
# [Verify all shared exports are importable]

echo "5. Mock data validation..."
# [Verify mock data matches type definitions]

echo "=== Wave 1 Gate: PASSED ==="
echo "Foundation is FROZEN. Proceeding to Wave 2."
```

**Gate criteria:**
- [ ] All Tier 0 tasks marked Done
- [ ] Type check: 0 errors
- [ ] All foundation tests pass
- [ ] No lint warnings
- [ ] All shared exports importable
- [ ] Mock data validates against types
- [ ] Foundation committed to main (or foundation branch ready for merge)

**On gate failure:**
1. Read the failure output
2. If fixable: re-prompt the foundation agent with the error
3. If agent context exhausted: spawn Recovery Agent with error context
4. If architectural: HALT -- update architecture plan before continuing

**Post-gate actions:**
- Merge foundation branch to main (if not already on main)
- Update IMPLEMENTATION_STATUS.md: Wave 1 = PASSED
- Announce: "Foundation frozen. Starting Wave 2."

---

### Wave 2: Feature Implementation (Parallel)

**Purpose:** Implement all feature modules in parallel.
**Agents:** S1, S2, S3, ... (one per module)
**Model:** claude-sonnet-4-6 (per agent)
**Tasks:** [T1.1-T1.R1], [T2.1-T2.R1], [T3.1-T3.R1]
**Estimated duration:** ~[N] min (wall clock, parallel)
**Token budget:** [N]K tokens per agent (~$[N] total)
**Timeout:** [N] min per agent (escalate if exceeded)

#### Pre-conditions
- [ ] Wave 1 exit gate PASSED
- [ ] Foundation merged to main
- [ ] Clean git state

#### Dispatch Commands

**For Subagents with Worktrees (parallel dispatch):**
```
Launch all feature agents simultaneously using parallel Agent tool calls:

Agent 1:
  description: "[Module A] Implementer"
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: |
    [Module A spawn prompt from implementation plan]

Agent 2:
  description: "[Module B] Implementer"
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: |
    [Module B spawn prompt from implementation plan]

Agent 3:
  description: "[Module C] Implementer"
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: |
    [Module C spawn prompt from implementation plan]
```

**For Agent Teams:**
```
Create an agent team for feature implementation.

Spawn teammates:
1. "[Module A] Developer" - [Module A spawn prompt]
2. "[Module B] Developer" - [Module B spawn prompt]
3. "[Module C] Developer" - [Module C spawn prompt]

Create tasks with dependencies:
- All feature tasks depend on "Foundation Complete" (already done)
- Tasks within each module are sequential
- No cross-module dependencies in this wave

Use delegate mode (Shift+Tab) to ensure lead coordinates, not implements.
```

**For Manual Worktrees:**
```bash
# Create worktrees (run all at once)
git worktree add ../[project]-[module-a] -b tier-1/[module-a]
git worktree add ../[project]-[module-b] -b tier-1/[module-b]
git worktree add ../[project]-[module-c] -b tier-1/[module-c]

# Launch Claude in each (separate terminal windows/tabs)
# Terminal 1:
cd ../[project]-[module-a] && claude --agent feature-implementer
# Paste: [Module A spawn prompt]

# Terminal 2:
cd ../[project]-[module-b] && claude --agent feature-implementer
# Paste: [Module B spawn prompt]

# Terminal 3:
cd ../[project]-[module-c] && claude --agent feature-implementer
# Paste: [Module C spawn prompt]
```

#### Monitoring (During Execution)

Check progress every ~10 minutes for each agent:
```bash
# Quick status check across all worktrees
for dir in ../[project]-[module-a] ../[project]-[module-b] ../[project]-[module-c]; do
  echo "=== $(basename $dir) ==="
  cd "$dir"
  git log --oneline -3
  echo "---"
done
```

**Health indicators per agent:**
- Regular commits (every 5-15 min) = healthy
- Tests mentioned in commit messages = TDD being followed
- No commits for >20 min = investigate (context exhaustion or stuck)
- Commits touching files outside manifest = boundary violation, intervene

**Live Dashboard (update IMPLEMENTATION_STATUS.md):**
```
Wave 2 Progress:
  S1 [Module A]: [In Progress / Testing / Done] - T1.2 of T1.4
  S2 [Module B]: [In Progress / Testing / Done] - T2.1 of T2.3
  S3 [Module C]: [In Progress / Testing / Done] - T3.3 of T3.5
  Longest lane: S3 (~15 min remaining)
```

#### Exit Gate

**Per-agent gate (each agent runs before reporting done):**
```bash
#!/bin/bash
# scripts/gate-module.sh [module-name]
set -e
MODULE=$1
echo "=== Module Gate: $MODULE ==="
npx tsc --noEmit
npm test -- --testPathPattern="modules/$MODULE" --ci
npx eslint "src/modules/$MODULE/" --max-warnings=0
echo "=== Module Gate: $MODULE PASSED ==="
```

**Wave-level gate (after all agents complete, before merge):**
```bash
#!/bin/bash
# scripts/gate-wave-2.sh
set -e
echo "=== Wave 2 Exit Gate: Features ==="

# Verify each agent branch independently
for branch in tier-1/[module-a] tier-1/[module-b] tier-1/[module-c]; do
  echo "Checking $branch..."
  git checkout "$branch"
  npx tsc --noEmit
  npm test --ci
  echo "$branch: PASSED"
done

git checkout main
echo "=== Wave 2 Gate: ALL MODULES PASSED ==="
echo "Proceeding to merge sequence."
```

**On gate failure:**
1. Identify which module(s) failed
2. If single task failure: re-prompt the agent with error output
3. If module-wide failure: spawn Recovery Agent targeting that module
4. If contract mismatch: file CONTRACT_CHANGE_REQUEST, update foundation
5. Do NOT proceed to merge until all modules pass independently

---

### Wave 2→3 Transition: Merge Sequence

**Purpose:** Merge all feature branches into main sequentially.
**Agent:** S5 (Merge Conductor) or manual execution
**Duration:** ~[N] min

#### Merge Commands

```bash
#!/bin/bash
# scripts/merge-sequence.sh
set -e
echo "=== Sequential Merge with Test-After-Each ==="

# Merge order (dependency order from implementation plan):
BRANCHES=("tier-1/[module-a]" "tier-1/[module-b]" "tier-1/[module-c]")

for branch in "${BRANCHES[@]}"; do
  echo "--- Merging $branch ---"
  git merge "$branch" --no-edit

  echo "Running full test suite..."
  npm test --ci || {
    echo "FAIL: Tests failed after merging $branch"
    echo "Reverting merge..."
    git revert --no-commit HEAD
    git checkout -- .
    echo "ACTION REQUIRED: Fix $branch and retry"
    exit 1
  }

  echo "Type checking..."
  npx tsc --noEmit || {
    echo "FAIL: Type errors after merging $branch"
    git revert --no-commit HEAD
    git checkout -- .
    exit 1
  }

  echo "$branch: MERGED AND VERIFIED"
done

echo "Building..."
npm run build || exit 1

echo "=== All Merges Complete and Verified ==="
```

**Rollback procedure:**
```bash
# If a merge fails:
git reset --hard HEAD~1    # Undo the merge
# Fix the issue in the feature branch
# Re-attempt: git merge [branch] --no-edit
```

**Post-merge actions:**
- Clean up worktrees: `git worktree remove ../[project]-[module-*]`
- Update IMPLEMENTATION_STATUS.md: Wave 2 = MERGED
- Verify all tests still pass on main

---

### Wave 3: Integration

**Purpose:** Wire all modules together, compose routes, write integration and E2E tests.
**Agent:** S4 (Integration Wirer)
**Model:** claude-sonnet-4-6 (or opus for complex integration)
**Tasks:** [T4.1, T4.2, T4.3, T4.R1]
**Estimated duration:** ~[N] min
**Token budget:** [N]K tokens (~$[N])

#### Pre-conditions
- [ ] Wave 2 merge sequence completed
- [ ] All tests pass on main after merge
- [ ] Build succeeds on main

#### Dispatch Command

[Same dispatch structure as Wave 1, using integration-wirer agent definition
and integration spawn prompt from implementation plan]

#### Exit Gate

```bash
#!/bin/bash
# scripts/gate-wave-3.sh (Final Gate)
set -e
echo "=== Final Verification Gate ==="

echo "1. Type check..."
npx tsc --noEmit

echo "2. Full test suite..."
npm test --ci

echo "3. E2E tests..."
npm run test:e2e --ci

echo "4. Production build..."
npm run build

echo "5. Lint..."
npx eslint . --max-warnings=0

echo "=== FINAL GATE: PASSED ==="
```

---

### Wave 4: Hardening (Optional)

**Purpose:** Additional quality passes -- test coverage, security review, performance audit.
**Agents:** S6 (Test Hardener), S7 (Security Reviewer), S8 (Performance Auditor)
**Dispatch:** Parallel (each agent reviews different concerns independently)

[Same dispatch structure, using specialist agent definitions]

---

## 5. Live Monitoring Dashboard

### Progress Tracking File

Create `IMPLEMENTATION_STATUS.md` in project root:

```markdown
# Execution Status Dashboard

## Current State
- **Active Wave:** [N]
- **Start Time:** [timestamp]
- **Elapsed:** [N] min
- **Budget Used:** ~[N]K tokens ($[N])
- **Budget Remaining:** ~[N]K tokens ($[N])

## Wave Status
| Wave | Status | Start | End | Duration | Agents |
|------|--------|-------|-----|----------|--------|
| 0: Setup | DONE | [time] | [time] | [N] min | 0 |
| 1: Foundation | [IN PROGRESS/DONE] | [time] | [time] | [N] min | 1 |
| 2: Features | [PENDING/IN PROGRESS/DONE] | [time] | [time] | [N] min | [N] |
| 2→3: Merge | [PENDING/IN PROGRESS/DONE] | [time] | [time] | [N] min | 1 |
| 3: Integration | [PENDING/IN PROGRESS/DONE] | [time] | [time] | [N] min | 1 |

## Agent Status
| Agent | Role | Status | Current Task | Last Commit | Health |
|-------|------|--------|-------------|-------------|--------|
| S0 | Foundation | [Active/Done/Failed] | T0.3 | [hash] | OK |
| S1 | [Module A] | [Active/Done/Failed] | T1.2 | [hash] | OK |
| S2 | [Module B] | [Active/Done/Failed] | T2.1 | [hash] | OK |

## Gate Log
| Gate | Status | Timestamp | Notes |
|------|--------|-----------|-------|
| Wave 1 Exit | [PASS/FAIL] | [time] | |
| Wave 2 Exit | [PENDING] | | |
| Merge Sequence | [PENDING] | | |
| Final Gate | [PENDING] | | |

## Issues & Recoveries
| Issue | Wave | Agent | Resolution | Status |
|-------|------|-------|-----------|--------|
| [Description] | [N] | [SN] | [Action taken] | [Resolved/Open] |
```

### Monitoring Commands

```bash
# Quick health check
scripts/health-check.sh

# Detailed status (updates IMPLEMENTATION_STATUS.md)
scripts/update-status.sh

# Cost estimate
scripts/check-budget.sh
```

---

## 6. Adaptive Re-Planning Triggers

### Automatic Triggers

| Trigger | Condition | Action |
|---------|-----------|--------|
| Agent stuck | No commits for >20 min | Spawn Recovery Agent |
| Task too large | Single task >25 min | Split task, spawn fresh agent |
| Gate failure | Any gate check fails | Diagnose, fix, re-gate |
| Budget exceeded | Wave cost > 150% of projection | Pause, assess, reduce scope |
| Cascading failure | >2 agents fail in same wave | HALT all agents, reassess |
| Contract mismatch | Integration tests fail on types | Freeze, update foundation, restart affected |
| Context exhaustion | Agent output quality degrades | Commit progress, spawn fresh agent |

### Re-Planning Procedure

When a trigger fires:
1. **Capture state** -- commit all progress, update status dashboard
2. **Diagnose** -- identify root cause (use Recovery Agent if needed)
3. **Decide** -- fix-in-place, respawn, split, or halt-and-replan
4. **Execute** -- apply the decision with updated spawn prompts
5. **Verify** -- run the relevant gate before resuming normal execution
6. **Document** -- update the Issues & Recoveries log

---

## 7. Budget & Resource Plan

### Cost Projection

| Wave | Agents | Model | Est. Tokens | Est. Cost | Ceiling |
|------|--------|-------|-------------|-----------|---------|
| 0: Setup | 0 | - | 0 | $0 | - |
| 1: Foundation | 1 | Opus | [N]K | $[N] | $[N] |
| 2: Features | [N] | Sonnet | [N]K | $[N] | $[N] |
| 2→3: Merge | 1 | Opus | [N]K | $[N] | $[N] |
| 3: Integration | 1 | Sonnet | [N]K | $[N] | $[N] |
| 4: Hardening | [N] | Sonnet | [N]K | $[N] | $[N] |
| **Total** | | | **[N]K** | **$[N]** | **$[N]** |

### Early-Stopping Criteria

| Condition | Action |
|-----------|--------|
| Total spend reaches 80% of ceiling | Alert operator, assess remaining work |
| Total spend reaches 100% of ceiling | HALT all agents, prioritize remaining work |
| Single agent exceeds 150% of its budget | Kill agent, assess if task needs splitting |
| Wall-clock time exceeds 2x estimate | Pause, identify bottleneck, consider scope reduction |

### Scope Reduction Priority

If budget forces scope reduction, cut in this order (least important first):
1. Wave 4 hardening (defer to manual later)
2. Non-critical feature modules (defer to next execution)
3. Integration E2E tests (keep unit + integration, defer E2E)
4. Refactoring tasks within modules (accept tech debt temporarily)
5. NEVER cut: foundation, core feature modules, verification gates

---

## 8. Completion Checklist

### Technical Verification
- [ ] Final gate script passes (type check, tests, build, lint)
- [ ] All PRD acceptance criteria verified
- [ ] All implementation plan tasks marked Done
- [ ] No TODO comments or placeholder implementations
- [ ] All worktrees cleaned up
- [ ] All feature branches deleted (merged to main)

### PRD Acceptance Criteria Mapping

| PRD Criterion | Test/Verification | Status |
|--------------|-------------------|--------|
| [Criterion 1 from PRD] | [Test name or manual check] | [PASS/FAIL] |
| [Criterion 2 from PRD] | [Test name or manual check] | [PASS/FAIL] |

### Post-Execution Cleanup

```bash
# Remove worktrees
git worktree list | grep -v "bare" | while read dir branch rest; do
  git worktree remove "$dir" 2>/dev/null
done

# Remove merged branches
git branch --merged main | grep -v "main\|master" | xargs -r git branch -d

# Final commit
git add IMPLEMENTATION_STATUS.md
git commit -m "chore: execution complete, status updated"
```

---

## 9. Post-Execution Retrospective

### What Worked
- [What went well during execution]

### What Didn't Work
- [What failed or was slower than expected]

### Metrics
| Metric | Projected | Actual | Delta |
|--------|-----------|--------|-------|
| Wall-clock time | [N] min | [N] min | [+/-N] min |
| Total tokens | [N]K | [N]K | [+/-N]K |
| Total cost | $[N] | $[N] | [+/-$N] |
| Gate failures | 0 | [N] | [+N] |
| Re-plans needed | 0 | [N] | [+N] |
| Agent respawns | 0 | [N] | [+N] |

### Lessons for Next Execution
- [Actionable improvements for the next run]

---

## Appendix A: Gate Verification Scripts

[List all gate scripts with full content for copy-paste into scripts/ directory]

## Appendix B: Full Specialist Agent Definitions

[List all .claude/agents/ files with full content]

## Appendix C: Quick Reference Card

| Action | Command |
|--------|---------|
| Start execution | `bash scripts/preflight.sh` |
| Dispatch Wave 1 | [command] |
| Check Wave 1 progress | [command] |
| Gate Wave 1 | `bash scripts/gate-wave-1.sh` |
| Dispatch Wave 2 | [command] |
| Check all agents | [command] |
| Gate Wave 2 | `bash scripts/gate-wave-2.sh` |
| Run merge sequence | `bash scripts/merge-sequence.sh` |
| Dispatch Wave 3 | [command] |
| Final gate | `bash scripts/gate-final.sh` |
| Cleanup | `bash scripts/cleanup.sh` |
```

---

## Template Sizing Guide

| Project Size | Waves | Agents | Specialists | Est. Plan Length |
|:-------------|:------|:-------|:-----------|:----------------|
| Small (3-5 modules) | 3 | 2-3 | 4-5 | 3-5 pages |
| Medium (5-10 modules) | 3-4 | 3-5 | 6-8 | 5-10 pages |
| Large (10-15 modules) | 4-5 | 4-6 | 8-10 | 10-15 pages |

**Rules of thumb:**
- Waves = Tiers in the DAG + 1 (setup) + 1 (optional hardening)
- Max parallel agents = DAG's maximum tier width (never exceed 6)
- Budget should include 50% contingency for re-planning
- Every wave needs ~5 min gate overhead in the timeline
- First execution will take 1.5-2x projected time (learning curve)
