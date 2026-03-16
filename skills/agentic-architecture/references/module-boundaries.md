# Module Boundary Design Methodology

## Why Boundaries Matter for Agents

When multiple agents write code simultaneously, the #1 cause of failure is file
conflicts. Two agents editing the same file creates merge conflicts that are
expensive to resolve and often introduce bugs. Module boundaries exist to make
file overlap impossible by design.

Anthropic's multi-agent research system found that vague task delegation (e.g.,
"research the semiconductor shortage") resulted in duplicate work across agents.
The same applies to code: vague boundaries produce overlapping implementations.

Claude Code Agent Teams documentation is explicit: "Two teammates editing the
same file leads to overwrites. Break the work so each teammate owns a different
set of files."

## The Single-Owner Rule

**Every file in the project belongs to exactly one module.** No exceptions.

This means:
- Two modules never write to the same file
- Shared code lives in an explicit `shared/` or `core/` module
- Configuration files have a single owner (usually the foundation module)
- Test files belong to the module they test
- Migration files belong to the database/schema module

## Boundary Design Process

### Step 1: Identify Natural Seams

Start with the domain model. Natural boundaries appear at:

- **Data domain boundaries** -- Users, Products, Orders are separate domains
- **Technical layer boundaries** -- Database, API, UI are separate layers
- **Feature boundaries** -- Authentication, Search, Notifications are features
- **Infrastructure boundaries** -- Config, Logging, Monitoring are infrastructure

### Step 2: Choose a Primary Decomposition Strategy

Pick ONE primary axis. Mixing axes creates confusion:

**Vertical (Feature) Slicing** -- Best for most applications
```
modules/
  auth/          # Everything for authentication
    api/
    ui/
    types/
    tests/
  products/      # Everything for products
    api/
    ui/
    types/
    tests/
  orders/        # Everything for orders
    api/
    ui/
    types/
    tests/
  shared/        # Cross-cutting utilities
    db/
    types/
    ui-components/
```

Advantages:
- Each feature module is self-contained and independently deployable
- Agent A can build `auth/` while Agent B builds `products/` with zero overlap
- Natural alignment with PRD phases (one phase per feature)
- Maximizes DAG width -- most features can be built in parallel

**Horizontal (Layer) Slicing** -- Best for thin applications or APIs
```
src/
  db/            # All database schemas and migrations
  models/        # All data models and business logic
  api/           # All API endpoints
  ui/            # All UI components
  services/      # All service integrations
  shared/        # Cross-cutting utilities
```

Advantages:
- Clear separation of concerns by technical layer
- Works well when features share heavy infrastructure
- Simpler for pure API services without UI

**Hybrid Slicing** -- For complex applications
```
packages/
  core/          # Shared types, utilities, database
  auth/          # Authentication feature (vertical)
  api/           # API layer for remaining features (horizontal)
  web/           # Web UI (horizontal)
  workers/       # Background processing (horizontal)
```

### Step 3: Define the Shared Foundation

The shared/core module is built FIRST and is READ-ONLY for all other modules
during parallel execution. It contains:

- **Database schema** -- All table definitions and migrations
- **Shared types** -- TypeScript interfaces, API schemas, enums
- **Utility functions** -- Formatters, validators, helpers
- **Configuration** -- Environment variables, feature flags
- **Shared UI components** -- Design system primitives (buttons, inputs, layout)

**Critical rule**: Once the foundation module is complete, it becomes frozen.
Feature modules may only READ from the foundation, never WRITE to it. If a
feature needs a new shared type, it goes through an integration checkpoint.

This mirrors Anthropic's multi-agent research pattern where subagents receive
read-only access to shared resources and store their work in external systems,
passing lightweight references back to the coordinator.

### Step 4: Enumerate Files Per Module

Create an explicit file manifest for each module. This is the agent's contract
for what it owns:

```markdown
## Module: auth
**Owner**: Agent A (Tier 2)
**Worktree Branch**: `feature/auth`
**Files created**:
- `src/modules/auth/api/login.ts`
- `src/modules/auth/api/register.ts`
- `src/modules/auth/api/refresh-token.ts`
- `src/modules/auth/middleware/require-auth.ts`
- `src/modules/auth/types/auth.types.ts`
- `src/modules/auth/tests/login.test.ts`
- `src/modules/auth/tests/register.test.ts`

**Files read (not modified)**:
- `src/shared/db/schema.ts`
- `src/shared/types/user.types.ts`
- `src/shared/utils/validation.ts`

**Interface contracts consumed**:
- `UserRecord` type from `src/shared/types/user.types.ts`
- `db` instance from `src/shared/db/client.ts`

**Interface contracts produced**:
- `AuthMiddleware` exported from `src/modules/auth/middleware/require-auth.ts`
- `AuthService` exported from `src/modules/auth/api/index.ts`
```

### Step 5: Validate No Overlap

Check the file manifest against every other module. If two modules list the
same file under "Files created," the boundaries are wrong. Fix by:

1. Moving the shared file into the foundation module
2. Splitting the file into module-specific versions
3. Creating an interface that both modules implement independently

**Validation matrix:**

| File | Module A | Module B | Module C | Status |
|------|---------|---------|---------|--------|
| `src/shared/types.ts` | READ | READ | READ | OK -- foundation owns |
| `src/auth/api.ts` | CREATE | - | - | OK -- single owner |
| `src/routes/index.ts` | CREATE | CREATE | - | CONFLICT -- needs fix |

## Boundary Size Guidelines

| Project Size | Modules | Files per Module | Agent Assignment |
|-------------|---------|-----------------|-----------------|
| Small (1-3 features) | 3-5 | 5-15 | 1 agent per module |
| Medium (4-8 features) | 5-10 | 10-30 | 1 agent per module |
| Large (8+ features) | 8-15 | 15-50 | 1 agent per module, dedicated integration agent |

**Sizing for Agent Teams**: Claude Code Agent Teams work best with 3-5 teammates.
If your project has 10+ modules, group related modules into agent assignments
(e.g., one agent handles both `auth` and `user-management`).

**Tasks per agent**: Aim for 5-6 tasks per agent. This keeps agents productive
without excessive context switching. If you have 15 independent tasks, 3 agents
is a good starting point.

## Common Boundary Mistakes

**Shared mutable state**: Two modules both write to a `state.ts` file.
Fix: Each module owns its own state; use events or a message bus to sync.

**God module**: One module contains 60%+ of the codebase.
Fix: Decompose by domain or feature. If you can't decompose, it's one agent's job.

**Circular dependencies**: Module A imports from B, B imports from A.
Fix: Extract the shared dependency into the foundation module.

**Implicit contracts**: Module A assumes Module B's database table format.
Fix: Define an explicit interface contract that both modules reference.

**Configuration sprawl**: Every module adds to the same config file.
Fix: One module owns config; others declare their config needs in their own files,
composed at build time.

**Router/index file conflicts**: Multiple feature modules all need to register
routes in a central router file.
Fix: Each module exports its routes; a dedicated integration task composes them.
Alternatively, use convention-based routing (file-system routing) where each
module owns its own route files and the framework auto-discovers them.

**Test fixture conflicts**: Multiple modules write to shared test fixture files.
Fix: Each module owns its own test fixtures. Shared fixtures live in foundation.
