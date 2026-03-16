# Implementation Anti-Patterns

Known failure modes when executing multi-agent implementation plans. These are
drawn from Anthropic's engineering case studies, academic research (MAST taxonomy,
CodeCRDT, Agyn), practitioner experience, and the architecture skill's
anti-patterns translated to the implementation context.

## Planning-Level Anti-Patterns

### 1. The Monolithic Task

**What:** A single task covers an entire module ("implement the auth service").

**Why it fails:** Tasks >20 minutes exhaust the agent's effective context window.
Output quality degrades as conversation length grows. The agent makes locally
sensible but globally inconsistent decisions across the long task.

**Fix:** Decompose every module into 3-8 tasks of 5-15 minutes each. Each task
produces a testable, committable unit.

---

### 2. The Missing Foundation

**What:** Feature tasks start before shared types, schemas, and utilities are
fully defined and verified.

**Why it fails:** Each feature agent invents its own type definitions, mock data,
and utility functions. At merge time, everything conflicts.

**Fix:** Tier 0 tasks must complete and pass their verification gate before any
Tier 1 task begins. No exceptions.

---

### 3. The Implicit Dependency

**What:** Tasks don't specify their dependencies, so the scheduler doesn't know
which tasks must complete before others.

**Why it fails:** Agents start tasks before their inputs are ready. They either
fail immediately (file not found) or silently produce wrong output (using stale
or invented data instead of the real dependency).

**Fix:** Every task has an explicit `Depends on` field listing task IDs. The
execution schedule respects all dependency edges.

---

### 4. The Over-Parallel Plan

**What:** Plan schedules 8-10 agents for a project that only has 4-5 independent
work streams.

**Why it fails:** Research consistently shows performance saturates at 4-5 parallel
coding agents. Beyond that, coordination overhead (merge conflicts, contract
mismatches, duplicate work) outweighs time savings. Multi-agent systems consume
~15x more tokens than single-agent, and cost scales linearly with agent count.

**Fix:** Match agent count to the DAG's actual maximum tier width. 3-5 agents is
the sweet spot for most projects. If you have 10 modules, group related modules
into 4-5 agent lanes.

---

### 5. The Vague Spawn Prompt

**What:** Spawn prompt says "implement the auth module" without specifying files,
contracts, tests, or constraints.

**Why it fails:** Anthropic's research found that vague delegation was the primary
cause of early multi-agent system failures. Agents duplicate work, leave gaps,
drift outside their scope, and invent their own interfaces.

**Fix:** Every spawn prompt includes 7 sections: role, architecture context,
interface contracts, file manifest, task sequence, constraints, success criteria.
See spawn-prompts.md.

---

### 6. The Skipped Refactoring

**What:** Implementation plan has no refactoring tasks between implementation tasks.

**Why it fails:** AI-generated code accumulates technical debt faster than human
code. Inconsistent patterns emerge across task boundaries. Duplicated logic
appears independently in different tasks. Later tasks struggle because they
build on increasingly messy foundations.

**Fix:** Insert a refactoring task after every 3-4 implementation tasks. The
refactoring task reviews recent code, removes duplication, ensures consistent
patterns, and verifies the full test suite passes.

---

### 7. The Context Dump

**What:** Spawn prompt includes the entire architecture plan, the full PRD, and
the complete implementation plan -- giving the agent maximum context.

**Why it fails:** ETH Zurich research found that overly detailed context files
REDUCE task success rates and INCREASE cost by 20%+. Agents discover file
structures effectively on their own. Excessive context wastes tokens and
dilutes the important information.

**Fix:** Include only the information relevant to this agent's tasks: its module's
boundary map entry, its specific interface contracts, its file manifest, and
CLAUDE.md conventions. Nothing more.

---

## Execution-Level Anti-Patterns

### 8. The Unfrozen Foundation

**What:** Feature agents modify files in `src/shared/` after Tier 0 is complete.

**Why it fails:** Any modification to shared types or utilities affects all modules.
If Agent A1 modifies a shared type to fit its needs, Agent A2's code may break
when it's merged. The integration agent then faces cascading failures.

**Fix:** CLAUDE.md and spawn prompts must explicitly state that `src/shared/` is
FROZEN and read-only after Tier 0. Agents needing foundation changes must file
a CONTRACT_CHANGE_REQUEST.

---

### 9. The Cross-Module Import

**What:** A feature agent imports from another feature module's directory
(`src/modules/auth/` imports from `src/modules/users/`).

**Why it fails:** Creates implicit dependencies between modules that should be
independent. At merge time, if the imported module's internal structure changed,
the import breaks. Violates the architecture's module boundary design.

**Fix:** All cross-module communication goes through shared contracts in
`src/shared/types/`. If two modules need to interact, the interaction must be
mediated by the integration layer.

---

### 10. The Test Afterthought

**What:** Tasks are structured as "implement first, write tests after."

**Why it fails:** TDD is the single most important verification strategy for
agentic coding. When agents write tests after implementation, they write tests
that confirm what the code does (which may be wrong) rather than tests that
specify what the code should do.

**Fix:** Every task follows the Red/Green TDD pattern: write failing tests first,
then implement until tests pass.

---

### 11. The Premature Done

**What:** Agent marks a task as Done without running verification commands.

**Why it fails:** Agents sometimes "believe" their implementation is correct
without actually verifying. The error is only discovered at the merge gate,
where it's more expensive to fix.

**Fix:** Spawn prompts must include explicit verification commands that agents
run before reporting Done. Use hooks (`TaskCompleted`) to enforce verification
if using Agent Teams.

---

### 12. The Scope Creeper

**What:** Agent adds features, refactors unrelated code, or improves code
quality in areas outside its file manifest.

**Why it fails:** Modifications outside the file manifest create merge conflicts
and violate module boundaries. The "improvement" may break other agents' work.

**Fix:** Spawn prompts include an explicit DO NOT section listing prohibited
actions. File manifests are enforced -- no files outside the manifest.

---

## Coordination-Level Anti-Patterns

### 13. The Big Bang Merge

**What:** All feature branches are merged into main simultaneously.

**Why it fails:** When multiple merges happen at once, it's impossible to
determine which merge introduced a test failure. Debugging requires reverting
all merges and re-applying one by one anyway.

**Fix:** Sequential merge with test-after-each. Merge one branch, run the full
test suite, proceed only on success.

---

### 14. The Silent Failure

**What:** An agent fails but doesn't report the failure clearly. The lead or
human discovers the problem only at integration time.

**Why it fails:** Late failure discovery is the most expensive kind. The agent
may have been silently producing bad output for multiple tasks.

**Fix:** Every task has explicit verification gates. Progress tracking files
are updated after every task. Failed tasks are immediately flagged with
diagnostic information. Hooks enforce verification on task completion.

---

### 15. The Token Furnace

**What:** Using powerful models (Opus) for every agent, including simple
implementation tasks that Sonnet handles well.

**Why it fails:** Multi-agent systems already consume ~15x more tokens than
single-agent. Using Opus everywhere makes this 3-5x more expensive with
marginal quality improvement on standard implementation tasks.

**Fix:** Opus for the lead/foundation builder. Sonnet for feature implementers.
Haiku for exploration/research. Anthropic research confirms "Opus lead with
Sonnet workers outperformed single-agent Opus by 90.2%."

---

### 16. The Endless Session

**What:** Reusing the same agent session across multiple waves instead of
spawning fresh agents.

**Why it fails:** Context window pollution. After completing Tier 0, the agent's
context contains the full conversation history of foundation work. This dilutes
the context available for Tier 1 work and can cause the agent to confuse
foundation patterns with feature-specific patterns.

**Fix:** Spawn fresh agents at wave boundaries. Each agent starts with a clean
context window containing only its spawn prompt and the information it needs.

---

### 17. The Missing Gate

**What:** Implementation plan has no verification gates between waves. Agents
proceed directly from Tier 0 to Tier 1 to Integration without checkpoints.

**Why it fails:** Errors in earlier tiers propagate uncaught into later tiers,
where they are exponentially more expensive to fix. A broken shared type
definition caught at Tier 0 costs 5 minutes to fix. Caught at integration,
it costs the time to rebuild every module that consumed it.

**Fix:** Verification gates at every wave boundary. No wave starts without the
prior wave's gate passing. Gate scripts are included in the implementation plan
as runnable shell scripts.

---

## Recovery Decision Tree

When something goes wrong during implementation:

```
Problem detected
├── Single task failure
│   ├── Logic error → Re-prompt agent with test output
│   ├── Contract mismatch → File CONTRACT_CHANGE_REQUEST
│   └── Agent stuck → Commit progress, split task, spawn fresh agent
│
├── Module failure (multiple tasks)
│   ├── All tests fail → Foundation contract issue, rebuild module
│   └── Partial tests fail → Fix specific tasks, re-verify module
│
├── Merge conflict
│   ├── Single file → Boundary violation, rebuild offending branch
│   ├── Multiple files → Architecture boundary issue, update plan
│   └── package.json → Manual merge, deduplicate deps
│
├── Integration failure
│   ├── Type errors → Contract mismatch, fix non-compliant module
│   ├── Runtime errors → Wiring issue, trace full call chain
│   └── E2E failures → Missing integration, add wiring tasks
│
└── Cascading failure
    ├── Foundation issue → Halt all, rebuild foundation, respawn all
    ├── Architecture issue → Update architecture plan, regenerate impl plan
    └── Scope issue → PRD was incomplete, update PRD first
```
