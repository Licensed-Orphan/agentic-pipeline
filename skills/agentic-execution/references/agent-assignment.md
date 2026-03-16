# Agent Assignment & Orchestration Patterns

## Choosing Your Orchestration Model

Claude Code offers three levels of multi-agent coordination. Choose based on
your project's complexity, the need for inter-agent communication, and your
comfort with the tooling.

### Option 1: Subagents with Worktree Isolation

**Best for**: Projects where agents work independently and report results back.

```
Main Claude Session (Lead/Architect)
  ├── Subagent A (worktree: feature/auth)
  ├── Subagent B (worktree: feature/products)
  ├── Subagent C (worktree: feature/notifications)
  └── Integration (main branch, after all merge)
```

**How it works:**
- Main session spawns subagents with `isolation: worktree`
- Each subagent gets an isolated copy of the repository
- Subagents report results back to the main session
- Main session reviews and merges branches

**Configuration**: Define custom agents in `.claude/agents/`:

```yaml
---
name: feature-implementer
description: Implements a feature module from an architecture plan
tools: Read, Edit, Write, Bash, Grep, Glob
model: sonnet
isolation: worktree
permissionMode: acceptEdits
---

You are a feature implementation agent. You receive a module specification
from an architecture plan and implement it completely.

Your workflow:
1. Read the ARCHITECTURE.md to understand the full system design
2. Read the specific module assignment provided in your spawn prompt
3. Read all interface contracts your module depends on
4. Implement the module following the file manifest exactly
5. Write tests for all public interfaces
6. Run tests and fix any failures
7. Run linting and type checking
8. Report completion with a summary of files created/modified

Rules:
- ONLY create/modify files listed in your module's file manifest
- NEVER modify files owned by other modules
- Use mock data from contracts for testing -- do not depend on other modules
- Follow all conventions from CLAUDE.md
- If you discover a contract needs changing, STOP and report it
```

**Pros**: Simple, each agent is independent, worktree prevents conflicts
**Cons**: Agents can't communicate with each other during work

### Option 2: Agent Teams (Experimental)

**Best for**: Complex projects requiring inter-agent communication and
shared task coordination.

```
Team Lead (Claude Code session)
  ├── Teammate: Foundation Builder
  ├── Teammate: Auth Implementer
  ├── Teammate: Products Implementer
  ├── Teammate: UI Shell Builder
  └── Shared Task List with dependency tracking
```

**How it works:**
- Lead creates team, spawns teammates, coordinates work
- Teammates share a task list with dependency tracking
- Teammates communicate directly via mailbox
- Lead synthesizes results and handles integration

**Configuration**: Enable Agent Teams:
```json
// settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Spawn with specific context**:
```
Create an agent team to implement the architecture plan in ARCHITECTURE.md.

Spawn teammates:
1. "Foundation Builder" - implements Tier 0 (shared types, schemas, utils)
2. "Auth Developer" - implements the auth module (Tier 1)
3. "Products Developer" - implements the products module (Tier 1)
4. "UI Developer" - implements the UI shell (Tier 1)

Require plan approval before any teammate makes changes.
Create tasks with dependencies matching the DAG tiers.
Foundation tasks must complete before feature tasks unblock.
```

**Pros**: Agents can communicate, shared task tracking, lead coordinates
**Cons**: Experimental, higher token cost, no session resumption

### Option 3: Manual Worktree Sessions

**Best for**: Maximum control, experienced users, large projects.

```bash
# Create worktrees
git worktree add ../project-auth feature/auth
git worktree add ../project-products feature/products
git worktree add ../project-ui feature/ui-shell

# Launch Claude in each (separate terminal windows)
cd ../project-auth && claude
cd ../project-products && claude
cd ../project-ui && claude

# Each session reads ARCHITECTURE.md and works on its assignment
```

**Pros**: Full control, can use different models per session, no coordination overhead
**Cons**: Manual merge management, no inter-agent communication

## Agent Role Definitions

### Lead/Architect Agent

**Responsibility**: Coordination, integration, quality gates
**Model**: Opus (highest capability for architectural decisions)
**Tasks**:
- Parse the PRD and architecture plan
- Assign modules to worker agents
- Review worker output at integration checkpoints
- Resolve contract conflicts
- Perform final integration
- Run full test suite

### Foundation Builder Agent

**Responsibility**: Tier 0 -- shared types, schemas, utilities
**Model**: Sonnet (good balance of capability and speed)
**Tasks**:
- Create database schema and migrations
- Define all shared types and interfaces
- Build utility functions
- Create mock data for all contracts
- Set up project configuration
- Verify all exports are importable

### Feature Implementer Agent

**Responsibility**: One feature module per agent
**Model**: Sonnet
**Tasks**:
- Read module assignment from architecture plan
- Implement all files in the module's file manifest
- Write unit tests using contract mocks
- Run tests and verify passing
- Report completion

### Integration Agent

**Responsibility**: Final tier -- wiring, route composition, e2e tests
**Model**: Sonnet or Opus (complex cross-module reasoning)
**Tasks**:
- Compose routes from all feature modules
- Wire up cross-cutting middleware
- Run integration test suite
- Verify all contracts are satisfied
- Fix any interface mismatches

### Reviewer Agent (Optional)

**Responsibility**: Quality validation after implementation
**Model**: Sonnet
**Tasks**:
- Review code for consistency across modules
- Verify contract compliance
- Check for security vulnerabilities
- Validate naming conventions match CLAUDE.md
- Flag potential integration issues

## Assignment Matrix Template

Map every module to an agent assignment:

```markdown
## Agent Assignments

| Module | Agent Role | Model | Tier | Dependencies | Est. Tasks | Est. Time |
|--------|-----------|-------|------|-------------|-----------|-----------|
| foundation | Foundation Builder | sonnet | 0 | none | 8 | 15 min |
| auth | Feature Implementer A | sonnet | 1 | foundation | 6 | 12 min |
| products | Feature Implementer B | sonnet | 1 | foundation | 7 | 15 min |
| orders | Feature Implementer C | sonnet | 2 | foundation, products | 6 | 12 min |
| ui-shell | Feature Implementer D | sonnet | 1 | foundation | 5 | 10 min |
| integration | Integration Agent | opus | 3 | all | 4 | 10 min |

**Max parallel agents**: 4 (Tier 1)
**Critical path**: foundation -> orders -> integration = 37 min
**Total with parallelism**: ~47 min (vs ~64 min sequential)
```

## Model Selection Guidelines

| Agent Role | Recommended Model | Rationale |
|-----------|------------------|-----------|
| Lead/Architect | Opus | Complex reasoning, architectural decisions |
| Foundation Builder | Sonnet | Reliable, fast enough for type definitions |
| Feature Implementer | Sonnet | Good balance for standard feature work |
| Integration Agent | Sonnet/Opus | Cross-module reasoning complexity |
| Reviewer | Sonnet | Code review is well-suited to Sonnet |
| Explorer/Researcher | Haiku | Fast, cheap, good for read-only exploration |

From Anthropic's multi-agent research: using Opus as lead with Sonnet workers
"outperformed single-agent Opus by 90.2%." Similarly, OpenAI's Codex uses
different reasoning levels for different agent roles.

## Task Sizing Rules

**Too small** (< 5 min): Coordination overhead dominates. Merge into adjacent tasks.
**Just right** (5-15 min): Self-contained, produces a clear deliverable.
**Too large** (> 20 min): Agent works too long without check-in. Split into sub-tasks.

**Target**: 5-6 tasks per agent keeps everyone productive without excessive
context switching.

Each task must specify:
- **Objective**: One sentence of what it delivers
- **Files to create/modify**: Exact file paths
- **Files to read (not modify)**: Dependencies from other modules
- **Success criteria**: How to verify completion
- **Constraints**: What NOT to do

## Spawn Prompt Best Practices

When spawning agents, provide comprehensive context. Agents don't inherit
conversation history, only CLAUDE.md and project context.

**Include in every spawn prompt:**
1. Reference to ARCHITECTURE.md for full system understanding
2. Specific module assignment with file manifest
3. Interface contracts the module depends on
4. Interface contracts the module must produce
5. Test expectations and verification commands
6. Explicit constraints (DO NOT CHANGE files, naming conventions)
7. Success criteria for the task

**Example spawn prompt:**
```
You are implementing the auth module from ARCHITECTURE.md.

Read ARCHITECTURE.md Section 3.2 for your module specification.
Your file manifest is in Section 3.2.1.
Your interface contracts are defined in src/shared/contracts/auth.contract.ts.

Implementation requirements:
- Create all files listed in the auth module file manifest
- Implement the AuthServiceContract interface
- Write tests using mock data from src/shared/mocks/
- All tests must pass: npm test -- --grep "auth"
- Type checking must pass: npx tsc --noEmit
- Linting must pass: npm run lint

DO NOT modify any file outside your module's file manifest.
DO NOT modify any file in src/shared/ (foundation is frozen).
```
