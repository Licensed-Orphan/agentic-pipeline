# CLAUDE.md Configuration for Multi-Agent Projects

## Why CLAUDE.md Matters for Agent Swarms

CLAUDE.md is loaded automatically by every Claude Code session -- including
every teammate in Agent Teams and every subagent. It is the single most
important mechanism for constraining agent behavior and preventing
architectural drift across a multi-agent swarm.

From Anthropic's Agent Teams documentation: "Teammates read CLAUDE.md files
from their working directory. Use this to provide project-specific guidance
to all teammates."

When 5 agents work in parallel, each making locally sensible decisions,
CLAUDE.md is what keeps them globally consistent.

## CLAUDE.md Template for Multi-Agent Projects

```markdown
# CLAUDE.md

## Project Overview
[Project name] - [One sentence description]

## Architecture
This project follows the architecture plan in `ARCHITECTURE.md`.
Read it before making any changes.

## Tech Stack
- Language: [language] v[version]
- Framework: [framework] v[version]
- Database: [database] via [ORM]
- Testing: [test runner]
- Package manager: [npm/yarn/pnpm/bun]

## Commands
- Install: `[command]`
- Dev server: `[command]`
- Build: `[command]`
- Test all: `[command]`
- Test specific module: `[command] -- --grep "[module]"`
- Lint: `[command]`
- Type check: `[command]`

## Conventions

### Naming
- Files: [kebab-case/camelCase/PascalCase]
- Variables: [camelCase/snake_case]
- Types/Interfaces: [PascalCase]
- Constants: [SCREAMING_SNAKE_CASE]
- Test files: `[name].test.ts` alongside source files

### Code Style
- [Key style rules -- max line length, import ordering, etc.]
- No `any` type -- use `unknown` and narrow
- All async functions must have try/catch with specific error handling
- All API responses use the standard error format: `{error: {code, message}}`
- Use early returns over nested conditionals (max 3 levels deep)

### Imports
- Shared types: `import { Type } from '@/shared/types'`
- Shared utilities: `import { util } from '@/shared/utils'`
- Module internals: relative imports within the module
- NEVER import directly between feature modules -- use contracts

## Module Ownership Rules

### File Ownership
Every file belongs to exactly one module. Check ARCHITECTURE.md Section 2.3
(File Ownership Matrix) before creating or modifying any file.

- `src/shared/` -- Owned by foundation module. READ-ONLY for all others.
- `src/modules/[name]/` -- Owned exclusively by that module's agent.
- `src/app.ts`, `src/routes/` -- Owned by integration module.

### Rules
- NEVER modify files outside your assigned module
- NEVER import directly from another feature module's internals
- ALWAYS import shared types from `src/shared/types/`
- ALWAYS use mock data from `src/shared/mocks/` for testing
- If you need a type or utility that doesn't exist in shared/,
  STOP and request it through a CONTRACT_CHANGE_REQUEST.md

## Interface Contracts

All cross-module interfaces are defined in `src/shared/contracts/`.
These are FROZEN after the foundation module completes.

- Read your module's contract before implementing
- Implement exactly the interface specified -- no additions, no omissions
- Use the provided mock data for testing
- If a contract is wrong or insufficient, create CONTRACT_CHANGE_REQUEST.md

## Testing Requirements

### Every Module Must Have
- Unit tests for all public functions (happy path + error cases)
- Contract compliance tests verifying interface implementation
- Export verification tests confirming all public APIs

### Before Reporting Completion
Run these commands and ensure ALL pass:
1. `npx tsc --noEmit` (type check)
2. `npm test -- --grep "[your-module]"` (module tests)
3. `npm run lint` (linting)

## DO NOT
- Add new dependencies to package.json (foundation owns this)
- Modify database schema after Tier 0 (schema is frozen)
- Create files outside your module's file manifest
- Use `eval()`, `exec()`, or dynamic code execution
- Commit secrets, API keys, or credentials
- Leave TODO/FIXME comments as implementation substitutes
- Skip failing tests
- Use `any` type
- Import from other feature modules directly

## Error Handling
- All API endpoints: return `{error: {code: string, message: string}}`
- All async operations: wrap in try/catch with specific error types
- All database queries: use parameterized queries (never string interpolation)
- All user input: validate at API boundary before processing

## Security
- Sanitize all user input (XSS prevention)
- Use parameterized queries only (SQL injection prevention)
- Never log sensitive data (passwords, tokens, PII)
- Set secure, httpOnly, sameSite flags on cookies
```

## Per-Module CLAUDE.md Extensions

For large projects, place additional CLAUDE.md files in module directories:

```
src/
  CLAUDE.md                    # Project-wide rules (loaded by all agents)
  modules/
    auth/
      CLAUDE.md                # Auth-specific rules (loaded by auth agent)
    products/
      CLAUDE.md                # Products-specific rules
```

Module-specific CLAUDE.md contains:
- Module-specific conventions
- Domain-specific validation rules
- Module-specific test patterns

## Key Configuration Patterns

### For Agent Teams

When using Agent Teams, the lead should include in its coordination:

```
All teammates: read CLAUDE.md and ARCHITECTURE.md before starting any work.
Your module assignment is in ARCHITECTURE.md Section [X].
Your interface contracts are in src/shared/contracts/.
Do NOT modify any file outside your assigned module.
Report completion only after all verification commands pass.
```

### For Subagents with Worktrees

When spawning subagents, reference CLAUDE.md in the spawn prompt:

```
Follow all rules in CLAUDE.md.
Your specific module assignment: [module name]
Your file manifest: [list or reference to ARCHITECTURE.md section]
Verification commands: [list]
```

### For Manual Worktree Sessions

Each worktree contains the same CLAUDE.md (it's in the repo), so rules
apply automatically. Start each session with:

```
Read CLAUDE.md and ARCHITECTURE.md.
I am implementing the [module name] module.
My file manifest is in ARCHITECTURE.md Section [X].
```

## Anti-Patterns in CLAUDE.md

**Too vague**: "Write clean code" -- means nothing to an agent.
Fix: "Functions must not exceed 50 lines. Max 3 levels of nesting."

**Too permissive**: "Feel free to add utilities as needed."
Fix: "DO NOT create files outside your module's file manifest."

**Missing error patterns**: Not specifying error format.
Fix: "All errors: `{error: {code: string, message: string}}`"

**No import rules**: Not specifying how modules import from each other.
Fix: "NEVER import directly between feature modules. Use contracts."

**No verification commands**: Agent doesn't know how to self-check.
Fix: Include exact commands with exact flags.
