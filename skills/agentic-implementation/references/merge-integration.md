# Merge & Integration Strategy

How parallel agent work converges into a single, working codebase. The merge
strategy is where multi-agent implementations most often fail -- research shows
that integration failures are the primary reason multi-agent systems underperform
single-agent approaches on tightly coupled tasks.

## Merge Principles

1. **Sequential merge with test-after-each** -- merge one branch at a time, run
   the full test suite after each merge. This isolates which merge introduced
   any failure. Never merge all branches at once ("big bang integration").

2. **Foundation merges first** -- the foundation branch is the base that all
   feature branches were built against. It must be on main before any feature
   branch is merged.

3. **Dependency order** -- merge feature branches in order of their DAG
   dependencies. If module B depends on module A's output (even indirectly
   through shared types), merge A first.

4. **Leaves before roots** -- modules with no downstream dependents merge before
   modules that other modules import from. This prevents cascading failures.

5. **Integration branch merges last** -- it depends on all feature modules.

## Merge Sequence

### Phase 1: Foundation Merge

Foundation (Tier 0) should already be on the main branch since it was built
first and gates all other work. If it was built in a separate branch:

```bash
# Merge foundation
git checkout main
git merge foundation-branch --no-edit
npm test -- --testPathPattern="shared" --ci
npx tsc --noEmit
# Must pass before proceeding
```

### Phase 2: Feature Merges (Sequential)

Merge feature branches one at a time. Order by:
1. Modules with no cross-module dependencies (most independent first)
2. Modules that other modules consume (producers before consumers)
3. If no dependency ordering exists, merge the smallest branch first (fewer
   potential conflicts)

```bash
# For each feature branch, in dependency order:
git checkout main
git merge feature-[module-a]-branch --no-edit

# Run FULL test suite (not just the module's tests)
npm test --ci
npx tsc --noEmit
npm run build

# If all pass: continue to next branch
# If any fail: see "Handling Merge Failures" below
```

### Phase 3: Integration Merge

After all feature branches are merged:

```bash
git checkout main
git merge integration-branch --no-edit

# Full verification
npm test --ci                    # Unit + integration tests
npm run test:e2e --ci           # E2E tests
npm run build                    # Production build
npx tsc --noEmit                # Type check
npx eslint . --max-warnings=0  # Lint
```

## Handling Merge Failures

### Git-Level Conflicts

| Conflict Type | Cause | Resolution |
|:-------------|:------|:----------|
| Same file, different sections | Two agents modified different parts of a shared file | **Boundary violation.** This should not happen if file ownership is correct. Identify which agent modified a file outside their manifest. Fix the ownership, rebuild the offending module. |
| Same file, same section | Direct overlap | **Severe boundary violation.** The architecture plan has a flaw in module boundaries. Fix the architecture, rebuild both modules. |
| package.json / lock file | Both agents added dependencies | Manually merge dependency lists. Deduplicate. Run `npm install` and verify. |
| Import path conflicts | Agents used different import styles | Resolve to the convention in CLAUDE.md. Usually auto-resolvable. |

### Test-Level Failures After Merge

| Failure Pattern | Likely Cause | Resolution |
|:---------------|:-------------|:----------|
| Type errors | Contract mismatch: module implements a different shape than what the consumer expects | Compare both sides against the interface contract. Fix the non-compliant side. |
| Test failures in the just-merged module | Merge introduced import resolution issues | Check that the module's imports resolve correctly against main. |
| Test failures in a previously-merged module | The new module's code affects the old module through shared state | Identify the shared state. This indicates a module boundary issue. |
| All tests fail | Build is broken | Check `npm run build` output. Usually a missing export or circular dependency. |

### Merge Failure Recovery

```bash
# If a merge fails tests:
git revert --no-commit HEAD    # Undo the merge (keep changes staged for analysis)
git diff HEAD                  # See what the merge changed
# Diagnose the failure
# Fix in the feature branch
# Re-attempt merge
```

**If the same merge fails twice:** The issue is likely architectural, not
implementational. Review the module boundary and interface contract. Consider
whether the two modules actually have a hidden dependency that the architecture
plan missed.

## Worktree Merge Patterns

### For Subagents with Worktree Isolation

Each subagent works in its own git worktree with its own branch:

```
main (foundation committed here)
├── .claude/worktrees/feature-auth/     (branch: impl/auth)
├── .claude/worktrees/feature-users/    (branch: impl/users)
└── .claude/worktrees/feature-content/  (branch: impl/content)
```

When agents complete, their branches are ready for sequential merge:

```bash
# From the main worktree:
git merge impl/auth
npm test --ci && echo "PASS" || (git merge --abort && echo "FAIL")

git merge impl/users
npm test --ci && echo "PASS" || (git merge --abort && echo "FAIL")

# ... continue for each branch
```

### For Agent Teams

Agent Teams with worktrees handle merging more automatically, but you should
still verify after each merge via hooks or lead agent instructions:

```
Tell the lead: "After each teammate completes and their branch is merged,
run the full test suite before allowing the next merge. Use sequential
merge, not parallel merge."
```

### For Manual Worktree Sessions

Same as subagent pattern, but you manage the worktrees manually:

```bash
# Create worktrees
git worktree add ../project-auth -b impl/auth
git worktree add ../project-users -b impl/users

# Launch Claude in each
cd ../project-auth && claude
cd ../project-users && claude

# After completion, merge sequentially from the main worktree
cd ../project-main
git merge impl/auth && npm test --ci
git merge impl/users && npm test --ci

# Clean up
git worktree remove ../project-auth
git worktree remove ../project-users
```

## Pre-Merge Checklist (Per Agent)

Before reporting a module as complete and ready for merge, the agent must verify:

- [ ] All tasks in the module are Done
- [ ] All tests pass: `npm test -- --testPathPattern="modules/[name]" --ci`
- [ ] Type check clean: `npx tsc --noEmit`
- [ ] Lint clean: `npx eslint src/modules/[name]/ --max-warnings=0`
- [ ] No files outside the module's manifest were created or modified
- [ ] No imports from other feature modules
- [ ] All contract compliance tests pass
- [ ] No TODO comments, no placeholder implementations
- [ ] Meaningful commit messages on the branch
- [ ] Branch is rebased on latest main (if possible without conflicts)

## Integration Testing After All Merges

After all feature branches are merged, before the integration agent begins:

```bash
# Verify the merged state is clean
npm test --ci                         # All unit tests
npx tsc --noEmit                     # Full type check
npm run build                         # Build verification

# Check for common post-merge issues:
# 1. Circular dependencies
npx madge --circular src/             # If available

# 2. Unused exports
# Review src/shared/types/index.ts -- all exports should be consumed

# 3. Missing integration points
# Each module should have at least one consumer
```

## Merge Automation

For implementations using Claude Code's Agent Teams or subagents, the merge
sequence can be partially automated by the lead agent. Include this instruction
in the lead's prompt:

```
After all feature agents report completion:
1. Collect all agent branches
2. For each branch (in dependency order):
   a. Merge into main
   b. Run: npm test --ci && npx tsc --noEmit && npm run build
   c. If pass: continue to next branch
   d. If fail: revert, document the failure, and create a fix task
3. After all merges succeed, spawn the integration agent
```
