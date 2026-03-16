# Wave Dispatch Commands & Patterns

How to transform implementation plan waves into concrete, executable dispatch
commands for each orchestration runtime.

## Dispatch Principles

1. **Every command is copy-paste ready** -- no "adapt this" placeholders
2. **Pre-conditions are verified before dispatch** -- never dispatch blind
3. **Parallel dispatch uses a single message** -- spawn all agents in one action
4. **Each agent receives a self-contained prompt** -- no external references
5. **Timeouts are mandatory** -- agents that run forever waste budget

## Dispatch by Runtime

### Subagents with Worktree Isolation

The simplest and most cost-effective runtime. Each agent is spawned via
the Agent tool with `isolation: worktree`.

**Single agent dispatch (Wave 1 pattern):**
```
Use the Agent tool with:
  description: "[Role Name]"
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: |
    [Complete spawn prompt -- all 7 sections]
```

**Parallel agent dispatch (Wave 2 pattern):**
Spawn all agents in a SINGLE message with multiple Agent tool calls.
This ensures true parallel execution:

```
Call 1: Agent tool
  description: "[Module A] Implementer"
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: "[Module A spawn prompt]"

Call 2: Agent tool
  description: "[Module B] Implementer"
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: "[Module B spawn prompt]"

Call 3: Agent tool
  description: "[Module C] Implementer"
  subagent_type: "general-purpose"
  isolation: "worktree"
  prompt: "[Module C spawn prompt]"
```

**Background dispatch (for long-running agents):**
Use `run_in_background: true` to dispatch agents without blocking.
The system notifies when each agent completes.

```
Agent tool:
  description: "[Role Name]"
  subagent_type: "general-purpose"
  isolation: "worktree"
  run_in_background: true
  prompt: "[Spawn prompt]"
```

### Agent Teams

For complex projects requiring inter-agent communication.

**Team creation and dispatch:**
```
Enable: CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

Prompt to Claude Code lead session:

"Create an agent team to implement [project] from IMPLEMENTATION.md.

Spawn teammates:
1. 'Foundation Builder' with prompt: [foundation spawn prompt excerpt]
2. '[Module A] Developer' with prompt: [module A spawn prompt excerpt]
3. '[Module B] Developer' with prompt: [module B spawn prompt excerpt]

Create tasks with dependencies:
- T0.1-T0.R1: Foundation tasks (sequential, Foundation Builder)
- T1.1-T1.R1: [Module A] tasks (depend on T0.R1, [Module A] Developer)
- T2.1-T2.R1: [Module B] tasks (depend on T0.R1, [Module B] Developer)

Require plan approval before any file changes.
Use delegate mode -- coordinate, do not implement yourself.
After all teammates complete, run the merge sequence in scripts/merge-sequence.sh."
```

**Agent Teams best practices:**
- Use delegate mode (Shift+Tab) to prevent lead from implementing
- Create tasks with explicit dependencies for auto-unblocking
- Keep team size to 3-5 teammates (coordination overhead scales)
- Provide CLAUDE.md context -- teammates load it automatically
- Monitor task status -- teammates sometimes fail to mark tasks done

### Manual Worktree Sessions

For maximum control and mixed-model execution.

**Worktree creation script:**
```bash
#!/bin/bash
# scripts/create-worktrees.sh
set -e

# Ensure we're on main
git checkout main

# Create worktrees for each module
git worktree add ../[project]-foundation -b tier-0/foundation
git worktree add ../[project]-[module-a] -b tier-1/[module-a]
git worktree add ../[project]-[module-b] -b tier-1/[module-b]
git worktree add ../[project]-[module-c] -b tier-1/[module-c]

echo "Worktrees created:"
git worktree list
echo ""
echo "Launch Claude in each worktree in separate terminals:"
echo "  Terminal 1: cd ../[project]-foundation && claude --agent foundation-builder"
echo "  Terminal 2: cd ../[project]-[module-a] && claude --agent feature-implementer"
echo "  Terminal 3: cd ../[project]-[module-b] && claude --agent feature-implementer"
echo "  Terminal 4: cd ../[project]-[module-c] && claude --agent feature-implementer"
```

**Worktree cleanup script:**
```bash
#!/bin/bash
# scripts/cleanup-worktrees.sh
for dir in ../[project]-*; do
  if [ -d "$dir" ]; then
    echo "Removing worktree: $dir"
    git worktree remove "$dir" --force 2>/dev/null || true
  fi
done
git worktree prune
echo "All worktrees cleaned up."
```

## Dispatch Timing Patterns

### Research Wave (Wave 0 -- Optional)

Before implementation begins, dispatch read-only exploration agents to validate
assumptions, surface risks, and build shared context. This is especially
valuable for greenfield projects, unfamiliar codebases, or architectures with
uncertain integration points.

```
[Dispatch 2-3 Haiku/Sonnet research agents in parallel]
  Agent A: "Investigate [hypothesis A] -- read the codebase, report findings"
  Agent B: "Investigate [hypothesis B] -- read docs, report findings"
  Agent C: "Review the architecture plan for risks and gaps"
  -> [All complete, findings synthesized by lead]
  -> [Update architecture/implementation plan if needed]
  -> [Proceed to Wave 1]
```

**When to use:**
- The codebase is unfamiliar or large
- The architecture plan has unverified assumptions
- Debugging with competing hypotheses (agents test different theories in
  parallel and challenge each other's findings -- see Agent Teams pattern)
- Cross-layer changes spanning frontend, backend, and infrastructure

**Agent Teams variant:** Spawn research teammates that can message each other
directly. Use the "competing hypotheses" pattern: each researcher investigates
a different theory and actively tries to disprove the others. The theory that
survives debate is most likely correct.

**Cost:** Low. Research agents are read-only (Haiku model), run in ~5 min each,
and prevent expensive implementation failures downstream.

### Sequential Dispatch (Wave 1)
```
[Dispatch Foundation Builder]
  -> [Wait for completion]
  -> [Run Wave 1 gate]
  -> [Proceed to Wave 2]
```

### Parallel Dispatch (Wave 2)
```
[Dispatch all feature agents simultaneously]
  -> [Monitor all in parallel]
  -> [Wait for all to complete]
  -> [Run per-module gates]
  -> [Run Wave 2 gate]
  -> [Proceed to merge]
```

### Staggered Dispatch (Advanced)
When some feature modules depend only on specific foundation outputs:
```
[Foundation Builder: T0.1 types complete]
  -> [Dispatch Module A early -- only needs types]
  -> [Foundation Builder continues: T0.2 schema, T0.3 utils]
  -> [Foundation complete: dispatch Module B, Module C]
```

Only use staggered dispatch when:
- The architecture plan explicitly marks partial foundation dependencies
- The early-start module truly has no dependency on later foundation tasks
- The implementation plan supports staggered starts

### Pipeline Dispatch (Advanced)
When integration can start before all features complete:
```
[Module A complete + Module B complete]
  -> [Dispatch Integration Wirer for A+B wiring]
  -> [Module C still in progress]
  -> [Module C complete: Integration Wirer adds C wiring]
```

Only use pipeline dispatch when:
- Integration tasks for modules A+B are independent of module C
- The implementation plan explicitly supports partial integration

## Spawn Prompt Embedding Rules

When embedding spawn prompts into dispatch commands:

1. **Include the complete prompt** -- do not reference external files
   (agents don't inherit conversation context)
2. **Use the exact prompt from the implementation plan** -- do not
   paraphrase or abbreviate
3. **Add the runtime context** -- worktree branch name, CLAUDE.md path
4. **Add the verification commands** -- agents must self-verify
5. **Add the reporting requirement** -- agents must report what they did

**Runtime context to prepend:**
```
You are working in a git worktree on branch [branch-name].
Your CLAUDE.md is at the project root -- read it first.
The foundation code is in src/shared/ -- it is FROZEN and read-only.
When you complete all tasks, run the verification commands in your
Success Criteria section and report the results.
```

## Timeout & Escalation

Every dispatched agent must have a timeout:

| Agent Type | Default Timeout | Escalation Action |
|-----------|----------------|-------------------|
| Foundation Builder | 45 min | Check progress, split remaining tasks |
| Feature Implementer | 30 min | Check progress, spawn fresh agent for remaining |
| Integration Wirer | 45 min | Check progress, split integration tasks |
| Merge Conductor | 20 min | Manual intervention on merge conflicts |
| Test Hardener | 20 min | Accept current coverage, defer gaps |
| Recovery Agent | 15 min | Manual diagnosis required |

**Timeout handling:**
1. Check git log for recent commits (agent may be working normally)
2. If commits are recent: extend timeout by 15 min
3. If no recent commits: commit whatever exists, spawn fresh agent
4. If repeated timeouts: task is too complex -- split it
