# Dependency DAG Construction & Parallelization Strategy

## Why a DAG?

The dependency DAG (Directed Acyclic Graph) is the execution plan for your agent
swarm. It answers: what can be built in parallel, what must wait, and what is the
fastest possible path to completion.

Without a DAG, agents either:
- Work sequentially (wasting parallelization opportunity)
- Work in parallel on dependent tasks (producing integration failures)

## DAG Terminology

- **Node**: A module or work unit that an agent will implement
- **Edge**: A dependency -- "B depends on A" means A must complete before B starts
- **Tier**: A set of nodes with no dependencies on each other (can run in parallel)
- **Critical path**: The longest chain of sequential dependencies
- **Width**: The maximum number of nodes in any single tier (max parallel agents)

## DAG Construction Process

### Step 1: List All Modules

Start with every module from the boundary map:

```
foundation (schemas, types, shared utilities)
auth (authentication module)
products (product catalog)
orders (order management)
notifications (notification system)
ui-shell (layout, navigation, routing)
integration (route composition, final wiring)
```

### Step 2: Map Dependencies

For each module, ask: "What must exist before this module can be built?"

```
foundation       -> (none)
auth             -> foundation
products         -> foundation
orders           -> foundation, products (needs product types)
notifications    -> foundation
ui-shell         -> foundation
integration      -> auth, products, orders, notifications, ui-shell
```

### Step 3: Assign Tiers

Tier 0 has no dependencies. Tier N depends only on tiers 0 through N-1.

```
Tier 0: foundation
Tier 1: auth, products, notifications, ui-shell    (4 parallel agents)
Tier 2: orders                                      (depends on products)
Tier 3: integration                                 (depends on all)
```

### Step 4: Optimize for Width

The wider the tiers, the more agents can work simultaneously. Look for
opportunities to reduce dependencies:

**Before optimization:**
```
Tier 0: foundation
Tier 1: auth, products
Tier 2: orders (depends on products), notifications (depends on auth)
Tier 3: ui-shell (depends on orders)
Tier 4: integration
```
Critical path: 5 tiers

**After optimization** (extract shared types into foundation):
```
Tier 0: foundation (including product types and auth types)
Tier 1: auth, products, orders, notifications, ui-shell  (5 parallel agents)
Tier 2: integration
```
Critical path: 3 tiers

The key insight: move type definitions and interfaces into the foundation tier
to break dependency chains between feature modules.

### Step 5: Identify the Critical Path

The critical path determines the minimum possible build time. Calculate it by
finding the longest chain:

```
foundation (15 min) -> orders (20 min) -> integration (10 min) = 45 min
```

If `orders` is on the critical path, consider:
- Splitting it into `orders-core` (Tier 1) and `orders-checkout` (Tier 2)
- Moving its dependencies (product types) into foundation
- Assigning a more capable model (Opus) to critical-path modules

### Step 6: Define Synchronization Points

Where tiers meet, agents must synchronize. Define what "complete" means:

```markdown
## Synchronization: Tier 0 -> Tier 1

**Gate criteria (ALL must pass):**
- [ ] All database migrations run successfully
- [ ] All shared types compile without errors (npx tsc --noEmit)
- [ ] All utility function tests pass
- [ ] Foundation module linting passes
- [ ] Schema exports are importable from expected paths

**Artifacts available to Tier 1 agents:**
- `src/shared/db/schema.ts` (database schema)
- `src/shared/types/index.ts` (all shared type exports)
- `src/shared/utils/index.ts` (all utility exports)
- `src/shared/db/client.ts` (database client instance)
```

## DAG Visualization Format

Use ASCII art for the architecture plan (machine-readable and diffs cleanly):

```
                    ┌─────────────┐
                    │ Foundation   │  Tier 0
                    │ (1 agent)   │
                    └──────┬──────┘
           ┌───────┬───────┼───────┬──────────┐
           v       v       v       v          v
      ┌────────┐ ┌──────┐ ┌─────┐ ┌────────┐ ┌────────┐
      │ Auth   │ │Prods │ │Notif│ │UI Shell│ │Search  │  Tier 1
      │(1 agt) │ │(1agt)│ │(1ag)│ │(1 agt) │ │(1 agt) │
      └────┬───┘ └──┬───┘ └──┬──┘ └───┬────┘ └───┬────┘
           │        │        │        │           │
           v        v        v        v           v
      ┌──────────────────────────────────────────────┐
      │              Integration                      │  Tier 2
      │              (1 agent)                        │
      └──────────────────────────────────────────────┘
```

Also include a table format for easy reference:

| Tier | Modules | Agents | Dependencies | Est. Time |
|------|---------|--------|-------------|-----------|
| 0 | foundation | 1 | none | 15 min |
| 1 | auth, products, notifications, ui-shell, search | 5 | Tier 0 | 20 min |
| 2 | integration | 1 | Tier 0, 1 | 10 min |
| **Total** | **7 modules** | **5 max parallel** | | **~45 min** |

## Advanced DAG Patterns

### Staggered Start

Not all Tier 1 modules need the entire foundation. If `auth` only needs the
user schema and `products` only needs the product schema, they can start as
soon as their specific dependencies are ready:

```
foundation-schemas (5 min) -> auth (can start immediately)
foundation-utils (10 min)  -> products (must wait for utils)
```

### Pipeline Parallelism

For modules with sub-phases (API then UI), start the next module's API while
the current module builds its UI:

```
Module A API -> Module A UI
Module B API -> Module B UI  (starts after Module A API, not Module A UI)
```

### Integration Agent

For large projects, dedicate one agent solely to integration work:
- Composing routes from all feature modules
- Wiring up cross-cutting middleware
- Running the full integration test suite
- Resolving any interface mismatches

This agent starts idle and activates when Tier 1 completes.

## Common DAG Mistakes

**False dependencies**: Module B doesn't actually need Module A's implementation,
just its types. Move types to foundation to break the dependency.

**Integration big bang**: Waiting until everything is done to integrate. Instead,
integrate incrementally -- each tier should produce a runnable system.

**Ignoring the critical path**: Optimizing wide tiers while the critical path
remains long. Focus optimization on the longest sequential chain.

**Over-parallelization**: More agents doesn't always mean faster. Coordination
overhead increases with team size. 3-5 agents is the sweet spot for most projects.
Beyond that, task management and merge complexity dominate.

**Circular dependencies in the DAG**: If A depends on B and B depends on A,
the DAG is broken. Extract the shared dependency into a lower tier.
