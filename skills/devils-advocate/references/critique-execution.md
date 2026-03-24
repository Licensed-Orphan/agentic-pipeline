# Critique Module: Execution Plan Stage

## Pre-Mortem Prompt

"The rollout followed this execution plan precisely and caused a major incident.
Budget overran by 300%, two agents produced conflicting outputs that corrupted the
main branch, and the team had to revert everything. What was the failure mode?"

## Dimension Checklist

### 1. Budget Projection Realism
- Check for contingency buffer: 15-25% for known unknowns is standard. Zero padding is S1.
- Verify component-level breakdown: does the sum of per-wave costs equal the total?
  Common error: budgets that account for task execution but not verification, retries,
  or coordination overhead
- Model worst case: what happens if the most expensive task costs 3x more than projected?
- Check model tier assignments: are expensive models (Opus) reserved for tasks requiring
  complex reasoning? Are cheaper models used for mechanical tasks?

### 2. Gate Script Correctness
- Check each gate handles all exit codes correctly
- Verify positive test: would this gate pass with known-good input?
- Verify negative test: would this gate fail with known-bad input? Gates that never
  fail are useless
- Check edge cases: empty input, malformed input, enormous input
- Verify idempotency: running the gate twice produces the same result
- Check timeout behavior: does the gate handle its own timeout or can it hang indefinitely?

### 3. Agent Assignment Optimization
- Verify each agent has the tools required by its assigned task
- Check model appropriateness: Opus for lead/foundation/critical path, Sonnet for
  parallel feature work, Haiku for exploration/research
- Verify context budget: will agent context window accommodate spawn prompt +
  expected tool output + generated code?
- Check agent count: 3-5 parallel coding agents is the sweet spot. More degrades performance.
- Verify 5-6 tasks per agent to keep productive without excessive context switching

### 4. Timeout and Escalation Adequacy
- Verify every task and gate has an explicit timeout -- missing timeouts are always S1
- Check calibration: timeout should be 3-5x expected duration. Too short = false failures,
  too long = wasted resources
- Verify escalation path for each failure type: retry → alert → human intervention
- Check escalation triggers are unambiguous ("after 3 retries" is clear; "when appropriate" is not)

### 5. Merge Strategy Safety
- Check for file conflict risk: if multiple agents modify files in the same directory,
  what is the conflict resolution strategy?
- Verify branching model: is it compatible with the parallelization strategy?
- Check integration order: does it respect dependencies?
- Verify rollback granularity: can individual agent contributions be reverted?
- Check CI validation: is there a full test suite run after EACH merge?
- Verify merge is sequential with test-after-each (not parallel merge)

### 6. Adaptive Re-Planning
- Verify triggers exist for: task splitting, agent replacement, wave restructuring,
  scope reduction, contract evolution, cascading failure
- Check circuit breaker: is there a defined halt condition?
- Verify the plan specifies what happens when a wave gate fails

## Deterministic Pre-Checks

- Verify the execution plan references an implementation plan and its artifacts
- Check that every wave has: pre-conditions, dispatch commands, monitoring hooks,
  exit gate, timeout, budget ceiling
- Verify .claude/agents/ specialist definitions are specified for each role
- Check that dispatch commands are copy-paste ready (exact CLI syntax)
- Verify pre-flight checklist exists
- Count total waves and agents -- flag if >5 parallel agents
