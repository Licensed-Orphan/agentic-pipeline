# Orchestration Runtime Selection & Configuration

How to choose and configure the right runtime for dispatching your agent swarm.
The runtime determines how agents are spawned, how they communicate, and how
their work is collected and merged.

## The Three Runtimes

### 1. Subagents with Worktree Isolation

**Architecture:**
```
Main Claude Session (Lead/Orchestrator)
  ├── Subagent A (worktree: feature/auth)      → reports result back
  ├── Subagent B (worktree: feature/products)   → reports result back
  ├── Subagent C (worktree: feature/orders)     → reports result back
  └── Lead merges and integrates
```

**How it works:**
- Main session spawns subagents via the `Agent` tool with `isolation: worktree`
- Each subagent gets an isolated copy of the repository (git worktree)
- Subagents work independently and report results back to the main session
- Main session reviews results and manages the merge sequence
- No inter-agent communication during execution

**Configuration:**
- No special settings needed -- works out of the box
- Custom agent definitions in `.claude/agents/` are optional but recommended
- Subagents inherit CLAUDE.md and MCP server configuration

**When to use:**
- Tasks are independent and don't need inter-agent communication
- Clear file ownership boundaries (no overlapping edits)
- Budget-conscious execution (lowest overhead)
- Simple projects with 2-4 parallel agents
- Operator wants the simplest possible setup

**When NOT to use:**
- Agents need to communicate findings mid-execution
- Complex dependency chains requiring live coordination
- >5 parallel agents (management becomes complex)

**Cost profile:** Lowest. Each subagent is a focused session; the main session
coordinates without duplicating work.

---

### 2. Agent Teams (Experimental)

**Architecture:**
```
Team Lead (Main Claude Session)
  ├── Teammate: Foundation Builder     ← direct messaging →
  ├── Teammate: Auth Developer         ← direct messaging →
  ├── Teammate: Products Developer     ← direct messaging →
  └── Shared Task List with dependencies
```

**How it works:**
- Lead creates a team and spawns teammates with specific roles
- Teammates share a task list with dependency tracking and auto-unblocking
- Teammates can message each other directly (mesh communication)
- Lead coordinates work and synthesizes results
- Tasks auto-unblock when dependencies complete

**Configuration:**
```json
// In .claude/settings.json or project settings
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**When to use:**
- Complex projects requiring inter-agent coordination
- Tasks have runtime dependencies that may shift
- Need shared task tracking with auto-unblocking
- Agents might discover issues that affect other agents' work
- Large projects (>15 tasks) where lead coordination adds value

**When NOT to use:**
- Simple parallel work with no cross-talk needed
- Budget-sensitive execution (highest token cost)
- Sequential tasks that don't benefit from parallelism
- Need session resumption (Agent Teams don't support resume)

**Cost profile:** Highest. Each teammate is a full Claude instance with its own
context window. Lead overhead for coordination adds tokens. But mesh communication
can save time by avoiding sequential dependency resolution.

**Quality enforcement hooks:**
Agent Teams support two hooks that map directly to verification gates:
- **TeammateIdle**: Runs when a teammate is about to go idle. Exit with code 2
  to send feedback and keep the teammate working. Use this to enforce output
  quality before a teammate stops (e.g., "tests must pass before idling").
- **TaskCompleted**: Runs when a task is being marked complete. Exit with code 2
  to prevent completion and send feedback. Use this to enforce task-level gates
  (e.g., "lint must be clean before marking done").

Configure hooks in `.claude/settings.json` under `"hooks"`. These replace
manual gate checks with automated enforcement.

**Plan approval workflow:**
For high-risk modules, require teammates to plan before implementing:
- Teammate works in read-only plan mode until the lead approves
- Lead reviews and approves or rejects with feedback
- Rejected teammates revise and resubmit while staying in plan mode
- Influence the lead's judgment with criteria: "only approve plans that include
  test coverage" or "reject plans that modify the database schema"

**Display modes:**
- **In-process** (default): All teammates run in the main terminal. Use
  Shift+Down to cycle through teammates. Works in any terminal.
- **Split panes**: Each teammate gets its own pane (requires tmux or iTerm2).
  Set `"teammateMode": "tmux"` in settings.json, or pass `--teammate-mode tmux`.
  Note: split panes are NOT supported in VS Code terminal, Windows Terminal,
  or Ghostty.

**Known limitations (as of 2026):**
- Teammates sometimes fail to mark tasks as completed
- `/resume` doesn't restore in-process teammates
- Lead sometimes implements instead of delegating (tell it to wait for teammates)
- Teammates can't spawn their own sub-teams (no nested teams)
- No session persistence across restarts
- One team per session -- clean up before starting a new one
- Lead is fixed for the team's lifetime (can't promote a teammate)

---

### 3. Manual Worktree Sessions

**Architecture:**
```
Human Operator
  ├── Terminal 1: claude (worktree: feature/auth)
  ├── Terminal 2: claude (worktree: feature/products)
  ├── Terminal 3: claude (worktree: feature/orders)
  └── Terminal 4: claude (main branch, merge conductor)
```

**How it works:**
- Operator creates git worktrees manually
- Operator launches separate Claude sessions in each worktree
- Each session receives its spawn prompt as the first message
- Sessions work independently (no communication)
- Operator manages merge sequence manually

**Configuration:**
```bash
# Create worktrees
git worktree add ../project-auth -b feature/auth
git worktree add ../project-products -b feature/products

# Launch Claude with specific agent
cd ../project-auth && claude --agent feature-implementer
cd ../project-products && claude --agent feature-implementer
```

**When to use:**
- Maximum control over each agent session
- Want to use different models for different agents
- Need to monitor and interact with individual agents
- Large projects where you want to inspect each agent's work in real-time
- Experienced operator who is comfortable managing worktrees

**When NOT to use:**
- Large number of agents (>5 becomes unwieldy)
- Automated execution without human supervision
- Need inter-agent communication

**Cost profile:** Medium. Comparable to subagents, but human management time
is a factor. No coordination overhead tokens.

## Decision Matrix

| Factor | Subagents | Agent Teams | Manual Worktrees |
|--------|-----------|-------------|-----------------|
| Setup complexity | Low | Medium | High |
| Inter-agent communication | No | Yes (mesh) | No |
| Task dependency tracking | Manual | Automatic | Manual |
| Token cost | Low | High | Medium |
| Human oversight needed | Medium | Low | High |
| Session resumability | Yes | No | Yes |
| Max recommended agents | 4-5 | 4-6 | 3-5 |
| Model mixing | No (same model) | No (same model) | Yes |
| File isolation | Automatic | Manual discipline | Automatic |
| Best for | Simple parallel work | Complex coordination | Maximum control |

## Runtime-Specific Execution Patterns

### Subagent Pattern: Wave Dispatch

```
Wave 1: Main session dispatches S0 (Foundation Builder)
  └── S0 completes, result returns to main session
  └── Main session runs Wave 1 gate

Wave 2: Main session dispatches S1, S2, S3 in parallel
  └── Each agent completes, results return
  └── Main session runs Wave 2 gate

Wave 2→3: Main session (or Merge Conductor subagent) runs merge
  └── Sequential merge with test-after-each

Wave 3: Main session dispatches S4 (Integration Wirer)
  └── S4 completes, result returns
  └── Main session runs Final gate
```

### Agent Teams Pattern: Task Flow

```
Lead creates team with Foundation Builder, 3 Feature Developers

Lead creates task list:
  T0.1: Foundation types (assigned: Foundation Builder)
  T0.2: Foundation schema (depends: T0.1, assigned: Foundation Builder)
  T1.1: Auth module (depends: T0.R1, assigned: Auth Dev)
  T2.1: Products module (depends: T0.R1, assigned: Products Dev)
  T3.1: Orders module (depends: T0.R1, assigned: Orders Dev)

Foundation Builder completes T0.R1 → Auto-unblocks T1.1, T2.1, T3.1
Feature developers claim and start their tasks
Lead monitors, resolves conflicts, coordinates merge
```

### Manual Worktree Pattern: Terminal Orchestration

```
Terminal 0 (main): Operator manages execution
Terminal 1: claude --agent foundation-builder (tier-0/foundation worktree)
  └── Operator monitors, provides the spawn prompt
  └── Agent completes, operator verifies
Terminal 0: Operator merges foundation, runs gate

Terminal 2: claude --agent feature-implementer (tier-1/auth worktree)
Terminal 3: claude --agent feature-implementer (tier-1/products worktree)
Terminal 4: claude --agent feature-implementer (tier-1/orders worktree)
  └── Agents work in parallel
  └── Operator monitors all terminals
Terminal 0: Operator runs merge sequence after all complete
```

## Hybrid Patterns

For complex projects, combine runtimes:

**Pattern: Manual Lead + Subagent Workers**
- Human runs the lead session manually on main
- Lead dispatches feature agents as subagents with worktrees
- Human monitors and manages gate verification
- Best of both: automation of parallel work + human oversight

**Pattern: Agent Teams for Features + Subagent for Integration**
- Use Agent Teams for the complex feature development phase
- Spawn a subagent for the integration phase (fresh context)
- Avoids context pollution from the long feature development conversation

**Pattern: Subagent Research + Manual Implementation**
- Use Haiku subagents for exploration and research phases
- Use manual Opus/Sonnet sessions for implementation
- Maximizes cost efficiency during exploration
