# Known Failure Modes & Anti-Patterns

## Architecture-Level Anti-Patterns

### 1. The Monolith Trap

**Symptom**: One module contains 60%+ of the codebase. Only 1-2 agents
can work at a time regardless of team size.

**Why it happens**: The architect doesn't decompose deeply enough, or
the domain has a single core entity everything revolves around.

**Fix**: Decompose by feature/capability, not by entity. Even if everything
touches "orders," the order creation flow, order listing flow, and order
fulfillment flow can be separate modules with separate file manifests.

### 2. Dependency Chain of Death

**Symptom**: DAG has 6+ tiers with only 1-2 modules per tier. Agents
wait sequentially for each other, eliminating parallelism benefits.

**Why it happens**: Feature modules depend on each other instead of on
shared abstractions. Module A needs Module B's types, Module B needs
Module C's types, creating a chain.

**Fix**: Extract ALL shared types into the foundation module. Feature
modules should depend only on foundation, never on each other. If two
features must interact, define the interaction contract in foundation.

### 3. The Missing Foundation

**Symptom**: Feature agents start immediately and each creates their own
type definitions, database utilities, and helper functions. Integration
fails because every module has different conventions.

**Why it happens**: Skipping Tier 0 to "save time."

**Fix**: Always build foundation first. The 15 minutes spent on shared
types, schemas, and utilities saves hours of integration rework.

### 4. Implicit Contract Syndrome

**Symptom**: Module A exports `getUser()` returning `{name, email}`.
Module B expects `fetchUser()` returning `{username, emailAddress}`.
Integration fails on every boundary.

**Why it happens**: No explicit contracts defined before parallel work begins.
Each agent invents its own interface.

**Fix**: Define ALL cross-module contracts in the foundation module with
exact type signatures, method names, error formats, and mock data.

### 5. The God Config File

**Symptom**: Every module adds entries to the same configuration file
(routes, middleware, environment variables). Merge conflicts on every
integration.

**Why it happens**: Centralized registration patterns (express route
registration, middleware chains) where all modules write to the same file.

**Fix**: Each module exports its configuration fragment. A dedicated
integration module composes them. Use convention-based discovery
(file-system routing) where possible.

### 6. Over-Parallelization

**Symptom**: 8 agents spawned for a project that only has 3 truly
independent work streams. Most agents idle-wait or duplicate work.
Token costs explode.

**Why it happens**: "More agents = faster" assumption.

**Fix**: 3-5 agents is the sweet spot. Width is determined by the DAG,
not by team size. If the DAG only has 3 modules in its widest tier,
3 agents is the maximum useful parallelism.

Anthropic's Agent Teams documentation: "Three focused teammates often
outperform five scattered ones."

## Implementation-Level Anti-Patterns

### 7. Cross-Module File Editing

**Symptom**: Agent A modifies a file in Agent B's module. Merge conflicts
and unexpected behavior at integration.

**Why it happens**: Weak CLAUDE.md instructions, missing file ownership
matrix, or a genuine architectural oversight where a file should be
shared but isn't.

**Fix**: Enforce strict file ownership in CLAUDE.md. Include the file
ownership matrix in the architecture plan. If a file needs to be
touched by multiple agents, move it to foundation.

### 8. Direct Inter-Module Imports

**Symptom**: `import { helper } from '../../modules/auth/utils/helper'`
appears in the products module. Auth module changes break products.

**Why it happens**: Agent takes the shortest path to resolve an import
instead of going through the contract layer.

**Fix**: CLAUDE.md rule: "NEVER import from another feature module directly.
All cross-module imports go through src/shared/." Contract layer enforces
stable interfaces.

### 9. Test Coupling

**Symptom**: Auth module tests import and execute products module code.
Both modules must be complete before either can run tests.

**Why it happens**: Tests use real implementations instead of mocks.

**Fix**: All tests use mock data from `src/shared/mocks/`. Mocks are part
of the foundation module and defined alongside contracts.

### 10. Schema Drift

**Symptom**: After Tier 0, an agent adds a column to a database table
to support its feature. Other agents' queries break because they don't
know about the new column.

**Why it happens**: No "schema is frozen" rule, or the rule exists but
isn't enforced.

**Fix**: Database schema is owned exclusively by foundation module.
After Tier 0, schema changes require a CONTRACT_CHANGE_REQUEST and
foundation update before any agent can proceed.

### 11. Agent Working on Wrong Module

**Symptom**: Agent spawned for "auth" starts implementing "user management"
because it seems related.

**Why it happens**: Vague spawn prompt, missing file manifest, agent's
training data associates auth with user management.

**Fix**: Spawn prompts must include exact file manifest. CLAUDE.md must
include "ONLY create/modify files listed in your module's file manifest."

### 12. Convention Divergence

**Symptom**: Module A uses camelCase, Module B uses snake_case. Module A
returns `{error: "..."}`, Module B returns `{message: "..."}`.

**Why it happens**: Each agent follows different conventions from its
training data. CLAUDE.md doesn't specify conventions explicitly enough.

**Fix**: CLAUDE.md must include exact naming conventions, error formats,
import patterns, and code style rules. Be specific: "Files: kebab-case.
Variables: camelCase. Types: PascalCase. Error format: {error: {code, message}}."

## Coordination-Level Anti-Patterns

### 13. Lead Does the Work

**Symptom**: In Agent Teams, the lead implements features itself instead
of delegating to teammates.

**Why it happens**: Lead determines it can do the work faster than
coordinating teammates.

**Fix**: Tell the lead: "Wait for your teammates to complete their tasks
before proceeding. Your role is coordination, not implementation."
Use Shift+Tab (Delegate Mode) to restrict the lead to coordination only.

### 14. Big Bang Integration

**Symptom**: All agents complete their work, then everything is merged
at once. Integration fails spectacularly with dozens of errors.

**Why it happens**: No incremental integration checkpoints defined.

**Fix**: Define integration checkpoints at each DAG tier boundary.
Merge branches one at a time, running tests after each merge.
Catch problems while they're small and attributable.

### 15. Stale Task Status

**Symptom**: Agent Teams task list shows tasks as "in progress" when
they're actually done, blocking dependent tasks.

**Why it happens**: Known limitation of Agent Teams -- teammates
sometimes fail to mark tasks as completed.

**Fix**: Lead should periodically check task status and manually
update if needed. Include instructions in lead's prompt: "Verify
task completion by checking the actual work output, not just the
task status."

### 16. Context Window Exhaustion

**Symptom**: Agent starts strong but produces increasingly poor output
as conversation gets long. Forgets constraints from CLAUDE.md.

**Why it happens**: Single agent handling too much work without
context compaction or session restart.

**Fix**: Size tasks to 5-15 minutes of work. Use subagents for
focused tasks to preserve main context. Enable auto-compaction.
For Agent Teams, each teammate has its own 1M token context window.

### 17. Duplicate Work

**Symptom**: Two agents implement the same utility function, or
both create a shared type that should be in foundation.

**Why it happens**: From Anthropic's research: "vague task delegation
resulted in duplicate work across agents."

**Fix**: Explicit file manifests for every module. Foundation module
owns all shared code. File ownership matrix prevents overlap.

## Recovery Playbook

When things go wrong, follow this decision tree:

```
Agent produced bad output?
├── Discard worktree branch, respawn with better prompt
│
Merge conflict?
├── Single file: Indicates boundary violation, fix ownership
├── Many files: Architectural problem, revisit module boundaries
│
Tests fail after merge?
├── Identify which module's code causes failure
├── Revert that merge, fix the module, re-merge
│
Contract mismatch at integration?
├── Determine which side is "correct" per the contract
├── Fix the non-compliant module
├── If contract is wrong, update foundation and notify all agents
│
Agent stuck / not making progress?
├── Check context window usage (may need compaction)
├── Provide more specific instructions
├── Split the task into smaller pieces
│
Token budget exceeded?
├── Reduce team size
├── Use Haiku for exploration, Sonnet for implementation
├── Reduce task scope per agent
```
