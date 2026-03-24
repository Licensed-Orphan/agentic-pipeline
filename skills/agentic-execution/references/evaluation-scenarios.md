# Evaluation Scenarios

Three test scenarios for validating the agentic-execution skill.

---

## Scenario 1: Happy Path — Small Project Execution Plan

**Input:** An implementation plan for a 3-module CRUD API (auth, products,
orders) with ~15 tasks across 3 waves, using Subagents with Worktrees.

**Expected behavior:**
- Selects Subagents with Worktrees runtime (simple parallel, no cross-talk)
- Generates 4-5 specialist agent definitions (Foundation Builder, 2-3
  Feature Implementers, Integration Wirer)
- Uses Opus for foundation, Sonnet for features
- Produces 3-wave dispatch plan with copy-paste-ready Agent tool calls
- Includes pre-flight script, gate scripts for each wave, merge sequence
- Budget projection in the $40-60 range with 50% contingency
- Execution plan is written to EXECUTION.md

**Validate:**
- [ ] All dispatch commands are complete (no "[adapt this]" placeholders)
- [ ] Every wave has pre-conditions, dispatch commands, monitoring, and exit gate
- [ ] Gate scripts are executable bash with concrete test commands
- [ ] Budget table includes per-wave ceiling and early-stopping criteria
- [ ] Completion checklist maps back to PRD acceptance criteria

---

## Scenario 2: Edge Case — No Implementation Plan Provided

**Input:** User says "generate an execution plan" but only provides a PRD
and architecture plan — no implementation plan exists.

**Expected behavior:**
- Detects the missing implementation plan in Step 1 (Consume)
- Does NOT proceed to generate an incomplete execution plan
- Advises the user to run `agentic-implementation` first
- Explains what the implementation plan provides that the execution plan needs
  (spawn prompts, task decomposition, verification gates)

**Validate:**
- [ ] Skill halts gracefully rather than generating a broken plan
- [ ] Recommends the correct upstream skill by name
- [ ] Does not hallucinate spawn prompts or task breakdowns

---

## Scenario 3: Failure Case — Project Too Small for Multi-Agent

**Input:** An implementation plan with 6 tasks, all tightly coupled (each
task depends on the previous one's output), estimated at 25 minutes total.

**Expected behavior:**
- Recognizes that <10 tasks + tightly coupled + <30 min total = single-agent
  is better (per Constraints and the 45% threshold)
- Recommends single-agent execution instead of generating a multi-agent plan
- Explains why: coordination overhead would exceed parallelism gains,
  sequential dependencies prevent meaningful parallel dispatch
- Offers to generate a simplified single-agent execution checklist instead

**Validate:**
- [ ] Skill does NOT generate a full multi-agent execution plan
- [ ] Cites the "Know when NOT to swarm" principle or Constraints
- [ ] Provides a constructive alternative (not just a refusal)
