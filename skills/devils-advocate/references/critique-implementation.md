# Critique Module: Implementation Plan Stage

## Pre-Mortem Prompt

"The team followed this implementation plan exactly and missed the deadline by 8
weeks. Multiple agents failed, merge conflicts destroyed a week of work, and the
final integration never passed. What went wrong with the plan?"

## Dimension Checklist

### 1. Task Sizing
- Check each task against time-box: can it complete within a single agent execution
  cycle (~15 min for S tasks, ~30 min for M, ~45 min for L)?
- Flag any task touching >5 files as likely too broad
- Verify each task produces a testable, committable unit of work
- Apply INVEST: particularly check Testable (clear pass/fail) and Independent
  (no hidden dependencies on other in-progress tasks)
- Foundation tasks (Tier 0) should be smaller and more numerous -- they gate everything

### 2. Spawn Prompt Quality
- **Context sufficiency:** Does the prompt contain everything the agent needs without
  searching for additional context?
- **Five W's:** What to build, Where to put it, Which interfaces to implement,
  What constraints to follow, What done looks like
- **Negative instructions:** Does the prompt specify what NOT to do? ("Do not modify
  files outside your manifest," "Do not refactor existing code")
- **Verification criteria:** Does the prompt include how the agent verifies its own
  work before reporting completion?
- **Interface signatures:** Does the prompt include exact type signatures and contracts?
- **No conversational references:** Prompts must not reference "what we discussed" or
  assume shared context -- agents start fresh

### 3. Dependency Chain Correctness
- Validate the dependency graph forms a valid DAG (no cycles)
- For each task, list actual inputs and trace each to the producing task -- verify
  the dependency chain reflects this data flow
- Search for false dependencies: "If I removed this edge, would the task still succeed?"
- Search for missing dependencies: "What files/APIs does this task read that are
  produced by another task not listed as a dependency?"
- Check diamond dependencies: when two tasks depend on the same upstream and a
  downstream depends on both, verify the merge point is correct

### 4. Recovery Procedures
- For each task, list plausible failure modes: compilation error, test failure,
  timeout, API rate limit, merge conflict, wrong output format
- Verify each failure mode has a specified recovery action: retry, rollback, skip,
  escalate, manual intervention
- Check rollback capability: can each task's changes be undone cleanly?
- Verify partial success handling: what happens when 3 of 5 files modified correctly?
- Check cascading failure prevention: if task N fails, what happens to N+1, N+2?

### 5. Verification Gate Coverage
- Map every PRD acceptance criterion to at least one verification gate -- flag unmapped criteria
- Classify each gate: Strong (runs tests, checks contracts), Medium (lint, type check),
  Weak (file existence, non-empty output)
- Flag any critical-path task with only Weak gates as S1
- Check for false positives: can a gate pass with incorrect output?
- Check for false negatives: can a gate fail with correct output? (flaky tests)
- Verify gate ordering: cheapest/fastest gates first for fast failure

## Deterministic Pre-Checks

- Verify the plan contains all Output Contract items: Task Breakdown, Execution Schedule,
  Spawn Prompts, Verification Gates, Merge Strategy, Recovery Procedures
- Check that task IDs follow the naming convention (T0.1, T1.3, etc.)
- Verify every task lists its file manifest (files to create/modify, files to read)
- Check that no two tasks in the same wave modify the same file
- Count total tasks -- flag if >50 (context degradation risk)
