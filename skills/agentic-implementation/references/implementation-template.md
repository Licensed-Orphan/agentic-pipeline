# Implementation Plan Template

Use this template to generate the implementation plan document. Adapt section
depth based on project size: small projects (3-5 modules) need less detail;
large projects (10-15 modules) need full specification.

---

```markdown
# Implementation Plan: [Project Name]

Generated from: [PRD filename] + [Architecture Plan filename]
Date: [current date]
Orchestration Model: [Subagents with Worktree Isolation | Agent Teams | Manual Worktrees]

## 1. Implementation Overview

### Summary

| Metric | Value |
|--------|-------|
| Total tasks | [N] |
| Foundation tasks (Tier 0) | [N] |
| Feature tasks (Tiers 1-N) | [N] |
| Integration tasks | [N] |
| Refactoring tasks | [N] |
| Execution waves | [N] |
| Max parallel agents | [N] |
| Critical path length | [N tasks, ~N min] |
| Estimated wall-clock time | [N min] (with [N]-agent parallelism) |
| Estimated sequential time | [N min] (single agent, for comparison) |
| Parallelism speedup | [N]x |

### Orchestration Decision

**Model selected:** [Subagents with Worktree Isolation / Agent Teams / Manual Worktrees]

**Rationale:** [Why this model was chosen based on the project characteristics]

| Factor | Assessment |
|--------|-----------|
| Inter-agent communication needed? | [Yes/No -- if Yes, Agent Teams] |
| File overlap risk? | [Low/Med/High -- if High, worktree isolation required] |
| Task count | [N -- if >15, consider Agent Teams for coordination] |
| Complexity | [Low/Med/High -- High favors Opus lead + Sonnet workers] |

### Agent Roster

| Agent ID | Role | Model | Assigned Modules | Task Count |
|----------|------|-------|-----------------|------------|
| A0 | Lead / Foundation Builder | claude-opus-4-6 | shared, foundation | [N] |
| A1 | Feature Implementer | claude-sonnet-4-6 | [module-name] | [N] |
| A2 | Feature Implementer | claude-sonnet-4-6 | [module-name] | [N] |
| A3 | Feature Implementer | claude-sonnet-4-6 | [module-name] | [N] |
| A4 | Integration Agent | claude-sonnet-4-6 | integration, e2e | [N] |

### PRD-to-Architecture-to-Task Traceability

| PRD Phase | Architecture Module(s) | DAG Tier | Task IDs | Agent(s) |
|-----------|----------------------|----------|----------|----------|
| Phase 1: Foundation | shared, database | Tier 0 | T0.1-T0.N | A0 |
| Phase 2: [Name] | [module] | Tier 1 | T1.1-T1.N | A1 |
| Phase 3: [Name] | [module] | Tier 1 | T2.1-T2.N | A2 |
| ... | ... | ... | ... | ... |

---

## 2. Task Breakdown

### Tier 0: Foundation

> **Execution:** Sequential, single agent (A0)
> **Gate:** All Tier 0 tasks must complete and pass verification before any Tier 1 task begins.
> **Rationale:** Foundation defines shared types, schemas, and utilities that all other modules depend on. Errors here cascade everywhere.

#### T0.1: [Task Name]

| Field | Value |
|-------|-------|
| **Module** | shared |
| **Agent** | A0 |
| **Objective** | [One sentence: what this task produces] |
| **Complexity** | S / M / L |
| **Depends on** | None (first task) |
| **Blocks** | T0.2, T0.3, ... |

**Files to create:**
- `src/shared/types/index.ts` -- Shared type definitions
- `src/shared/types/[domain].ts` -- Domain-specific types

**Files to read (context only):**
- `package.json` -- Verify dependencies
- `tsconfig.json` -- Verify TypeScript config

**Interface contracts to produce:**
- `UserType`, `SessionType`, ... (from architecture plan Section 4)

**Test-first requirements:**
1. Write `src/shared/types/__tests__/[domain].test.ts` first
2. Tests verify: type exports exist, required fields present, constraints enforced
3. Run: `npm test -- --testPathPattern="shared/types"` -- must pass

**Success criteria:**
- [ ] All shared types exported from `src/shared/types/index.ts`
- [ ] Type tests pass with 100% coverage of exported types
- [ ] `npx tsc --noEmit` succeeds with no errors
- [ ] No `any` types used

**DO NOT:**
- Add runtime dependencies
- Create files outside `src/shared/types/`
- Import from any module outside `src/shared/`

---

#### T0.2: [Next Foundation Task]
[Same structure as T0.1]

---

#### T0.R1: Foundation Refactoring

| Field | Value |
|-------|-------|
| **Module** | shared |
| **Agent** | A0 |
| **Objective** | Review and clean up all Tier 0 code before feature work begins |
| **Depends on** | All T0.* tasks |

**Refactoring checklist:**
- [ ] Remove duplicated type definitions
- [ ] Ensure consistent naming conventions per CLAUDE.md
- [ ] Verify all exports are intentional and documented
- [ ] Run full lint + type check + test suite
- [ ] Commit with clean history

---

### TIER 0 VERIFICATION GATE

```bash
# Run before advancing to Tier 1
npx tsc --noEmit                           # Type check
npm test -- --testPathPattern="shared"      # Foundation tests
npx eslint src/shared/ --max-warnings=0    # Lint
echo "Tier 0 gate: PASS/FAIL"
```

**Gate criteria:**
- [ ] All Tier 0 tasks marked Done
- [ ] All type exports verified against interface contracts
- [ ] All tests pass (0 failures)
- [ ] No lint warnings
- [ ] Type check clean
- [ ] Foundation code committed to main/foundation branch

**Artifacts released to feature agents:**
- `src/shared/types/` -- Frozen, read-only for all feature agents
- `src/shared/utils/` -- Frozen, read-only for all feature agents
- `src/shared/mocks/` -- Mock data for isolated testing
- `src/db/schema.ts` -- Database schema, frozen

---

### Tier 1: Feature Implementation (Parallel)

> **Execution:** Parallel across agents A1, A2, A3 (each in own worktree)
> **Gate:** All Tier 1 tasks in a module must complete before that module's integration tasks begin.

#### Agent A1 Lane: [Module Name]

##### T1.1: [Task Name]

| Field | Value |
|-------|-------|
| **Module** | [module-name] |
| **Agent** | A1 |
| **Objective** | [One sentence] |
| **Complexity** | S / M / L |
| **Depends on** | Tier 0 Gate |
| **Blocks** | T1.2 |

**Files to create:**
- `src/modules/[name]/[file].ts`
- `src/modules/[name]/__tests__/[file].test.ts`

**Files to read (context only):**
- `src/shared/types/index.ts` -- Shared types (DO NOT MODIFY)
- `src/shared/mocks/[name].ts` -- Mock data for testing

**Interface contracts to implement:**
- `[ContractName]` from `src/shared/types/contracts.ts`

**Test-first requirements:**
1. Write `src/modules/[name]/__tests__/[file].test.ts` first
2. Tests cover: happy path, validation errors, edge cases (empty input, max values)
3. Use mock data from `src/shared/mocks/` -- do NOT create new mocks
4. Run: `npm test -- --testPathPattern="modules/[name]"` -- must pass

**Success criteria:**
- [ ] Tests written and failing (red)
- [ ] Implementation passes all tests (green)
- [ ] Contract compliance verified (exports match interface)
- [ ] No files modified outside `src/modules/[name]/`

**DO NOT:**
- Modify any file in `src/shared/` (frozen after Tier 0)
- Import from other feature modules (`src/modules/[other]/`)
- Add new dependencies to `package.json`
- Create database tables or modify schema

---

[Repeat for each task in each agent lane]

---

#### Agent A1 Lane: Module Refactoring

##### T1.R1: [Module] Refactoring

[Same refactoring task structure as T0.R1, scoped to this module]

---

### TIER 1 VERIFICATION GATE

```bash
# Per-module verification (run by each agent before reporting done)
npx tsc --noEmit
npm test -- --testPathPattern="modules/[name]"
npx eslint src/modules/[name]/ --max-warnings=0

# Cross-module integration (run by integration agent after all Tier 1 merges)
npm test                                    # Full test suite
npm run build                               # Build verification
```

**Gate criteria:**
- [ ] All Tier 1 tasks marked Done
- [ ] Per-module tests pass in isolation
- [ ] Sequential merge completed (one branch at a time, test after each)
- [ ] Full test suite passes after all merges
- [ ] Build succeeds
- [ ] No lint warnings

---

### Tier 2: Integration

> **Execution:** Sequential or limited parallel, by integration agent (A4)

[Same task structure for integration tasks: API wiring, route setup, middleware
chains, E2E tests, etc.]

---

### FINAL VERIFICATION GATE

```bash
# Full system verification
npx tsc --noEmit                          # Type check
npm test                                   # Full test suite
npm run build                              # Production build
npm run test:e2e                           # End-to-end tests
npx eslint . --max-warnings=0             # Full lint
```

**Gate criteria:**
- [ ] All tasks across all tiers marked Done
- [ ] Full test suite passes (unit + integration + E2E)
- [ ] Production build succeeds
- [ ] No lint warnings, no type errors
- [ ] All PRD acceptance criteria verified
- [ ] Manual exploratory testing completed (if applicable)

---

## 3. Execution Schedule

### Wave Diagram

```
Wave 1 (Sequential):  [A0: T0.1] -> [A0: T0.2] -> ... -> [A0: T0.R1]
                       |
                       v  TIER 0 GATE
                       |
Wave 2 (Parallel):    [A1: T1.1] -> [A1: T1.2] -> [A1: T1.R1]
                      [A2: T2.1] -> [A2: T2.2] -> [A2: T2.R1]
                      [A3: T3.1] -> [A3: T3.2] -> [A3: T3.R1]
                       |
                       v  TIER 1 GATE (sequential merge)
                       |
Wave 3 (Sequential):  [A4: T4.1] -> [A4: T4.2] -> ... -> [A4: T4.R1]
                       |
                       v  FINAL GATE
```

### Timeline Estimate

| Wave | Tasks | Agents | Est. Time | Cumulative |
|------|-------|--------|-----------|------------|
| Wave 1 (Foundation) | [N] | 1 | ~[N] min | ~[N] min |
| Wave 2 (Features) | [N] | [3-5] | ~[N] min | ~[N] min |
| Wave 3 (Integration) | [N] | 1-2 | ~[N] min | ~[N] min |
| **Total** | **[N]** | **[N] max** | | **~[N] min** |

---

## 4. Spawn Prompts

### Agent A0: Foundation Builder

[Full spawn prompt -- see references/spawn-prompts.md for template]

### Agent A1: [Module] Implementer

[Full spawn prompt]

### Agent A2: [Module] Implementer

[Full spawn prompt]

[Continue for all agents]

---

## 5. Merge Strategy

### Merge Order

1. A0 foundation branch (already on main after Tier 0 Gate)
2. A1 branch -> main (run full tests after merge)
3. A2 branch -> main (run full tests after merge)
4. A3 branch -> main (run full tests after merge)
5. A4 integration branch -> main (run full tests + E2E after merge)

### Conflict Resolution

| Conflict Type | Action |
|--------------|--------|
| Same file, different sections | Auto-resolve (git merge) |
| Same file, same section | **Boundary violation** -- fix file ownership |
| Import path conflicts | Integration agent resolves |
| package.json conflicts | Merge dependency lists, deduplicate |
| Test file conflicts | Should not happen (tests are per-module) |

### Rollback Procedure

If a merge introduces test failures:
1. `git revert --no-commit HEAD` (revert the merge)
2. Identify which tests failed and why
3. Fix in the feature branch (respawn agent if needed)
4. Re-attempt merge

---

## 6. Progress Tracking

### Task Status Board

| Task ID | Module | Agent | Status | Verification |
|---------|--------|-------|--------|-------------|
| T0.1 | shared | A0 | Not Started | - |
| T0.2 | shared | A0 | Not Started | - |
| ... | ... | ... | ... | ... |

Status values: `Not Started` | `In Progress` | `Testing` | `Done` | `Failed` | `Blocked`

### Recovery Procedures

| Failure | Detection | Recovery |
|---------|-----------|----------|
| Task failure | Tests fail after implementation | Discard worktree, diagnose, respawn |
| Agent stuck | No progress for >20 min | Kill session, split task, respawn |
| Context exhaustion | Output quality degrades | Commit progress, spawn fresh agent |
| Merge conflict | Git conflict on merge | Identify boundary violation, fix |
| Contract mismatch | Integration tests fail | Fix non-compliant module |
| Cascading failure | Multiple agents fail | Halt all, reassess plan |

---

## 7. Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Foundation contract changes needed | Medium | High | Freeze contracts after Tier 0; use CONTRACT_CHANGE_REQUEST.md |
| Merge conflicts between feature branches | Low | Medium | Strict file ownership; worktree isolation |
| Agent produces low-quality code | Medium | Medium | TDD gates; refactoring cadence; code review agent |
| Context window exhaustion | Medium | Low | Task sizing <15 min; fresh agent per task |
| Integration failures after merge | Medium | High | Sequential merge with test-after-each |
| Scope creep during implementation | Low | Medium | Explicit DO NOT lists in spawn prompts |

---

## Appendix A: CLAUDE.md for This Implementation

[Paste the CLAUDE.md content that all agents should load -- from the architecture
plan's CLAUDE.md recommendations, updated with implementation-specific rules]

## Appendix B: Custom Agent Definitions

[Agent definition files for `.claude/agents/` directory -- one per agent role]

## Appendix C: Verification Scripts

[Bash scripts for each gate that agents can execute directly]
```

---

## Template Sizing Guide

| Project Size | Modules | Tasks | Waves | Agents | Plan Length |
|:-------------|:--------|:------|:------|:-------|:-----------|
| Small | 3-5 | 10-20 | 2-3 | 2-3 | 3-5 pages |
| Medium | 5-10 | 20-40 | 3-4 | 3-5 | 5-10 pages |
| Large | 10-15 | 40-75 | 4-6 | 4-6 | 10-15 pages |

**Rules of thumb:**
- Never exceed 75 tasks total (plan becomes unmanageable)
- 5-6 tasks per agent per wave keeps everyone productive
- Foundation wave should have 3-8 tasks (more for complex shared types)
- Include 1 refactoring task per 3-4 implementation tasks
- Integration wave tasks are typically 50% of feature task count
