# Phase Structure & Dependency Mapping

## The Phased Approach

AI coding agents perform better with dependency-ordered, testable phases than with
monolithic specifications. Each phase must leave the codebase in a runnable state.

### Why Phases Matter

**Context window limits**: Frontier LLMs follow ~150-200 instructions reliably
before performance degrades (HumanLayer, 2025). A single phase with 30-50
requirements stays well within this budget, even accounting for the ~50 instructions
consumed by the agent's own system prompt.

**Verification gates**: The METR study found developers were 19% slower with AI tools,
partly because of time spent reviewing and correcting AI output. Phases create natural
review points where the user verifies before the agent continues.

**Error isolation**: CodeRabbit found AI-generated code has 1.7x more issues than
human code. Smaller phases mean fewer variables when something goes wrong, making
debugging tractable.

**No dead ends**: Half-implemented features, commented-out code, and placeholder
functions create ambiguity. The agent cannot distinguish work-in-progress from
intentional scaffolding. Each phase must be complete.

## Phase Sizing Heuristic

Each phase should:
- Represent **5-15 minutes** of agent work (at current frontier model capability)
- Contain **30-50 requirements** maximum
- End with **functionality the user can manually verify**
- Leave the codebase in a **runnable state**

If a phase would take more than 15 minutes or has more than 50 requirements, split it.
If a phase would take less than 5 minutes, consider merging it with an adjacent phase
to reduce handoff overhead.

## Dependency Mapping

### Common Dependency Patterns

Foundations must precede features. Map dependencies before writing phases:

```
Database Schema
  └── Data Access Layer (ORM models, queries)
       ├── API Endpoints (CRUD operations)
       │    └── UI Components (forms, lists, detail views)
       └── Background Jobs (data processing, notifications)

Authentication
  └── Authorization (role-based access)
       └── Protected Routes & Features

Shared Utilities
  └── Feature Components (using those utilities)
       └── Integration Points (composing features together)
```

### Standard Phase Ordering

For a typical web application, phases follow this order:

1. **Foundation**: Project setup, database schema, core configuration
2. **Data Layer**: Models, migrations, seed data, data access patterns
3. **API / Backend Logic**: Endpoints, services, business rules, auth
4. **Core UI**: Layout, navigation, routing, shared components
5. **Feature Implementation**: Individual features (one phase per major feature if complex)
6. **Integration**: Connecting features, cross-cutting concerns
7. **Polish & Hardening**: Error handling, loading states, edge cases, accessibility

### Parallel-Safe vs Sequential Phases

Some phases can be built in parallel if they share no dependencies:

```
Phase 1: Database Schema (sequential -- everything depends on this)
  ├── Phase 2a: User Management API (parallel-safe)
  ├── Phase 2b: Content Management API (parallel-safe)
  └── Phase 2c: Notification System (parallel-safe)
Phase 3: UI Shell & Navigation (depends on Phase 1)
Phase 4: Feature UIs (depends on Phase 2a/2b/2c AND Phase 3)
Phase 5: Polish (depends on all prior phases)
```

Mark parallel-safe phases in the PRD so agents or teams with parallel execution
capabilities (Cursor with 8 agents, Google Antigravity) can exploit this.

## Phase Transition Checklist

Before moving from Phase N to Phase N+1:

- [ ] All Phase N acceptance criteria pass
- [ ] All Phase N verification commands succeed
- [ ] All prior phase functionality still works (regression check)
- [ ] No commented-out code or TODO placeholders remain
- [ ] No lint warnings or type errors introduced
- [ ] The codebase runs without errors

## Handling Phase Failures

If a phase fails:
1. **Do NOT proceed to the next phase** -- fix the current phase first
2. **Roll back to the last known-good state** if fixes are not converging
3. **Re-examine the phase requirements** -- the spec may need adjustment
4. **Split the failed phase** into smaller sub-phases if it was too large
5. **Update the PRD** to reflect any discoveries or changes

## Example: Phased Decomposition

Traditional monolithic requirement:
> "The system must allow users to upload, organize, and play audio files with
> real-time frequency visualization and playlist management."

Phased decomposition:

| Phase | Deliverable | Depends On | Verification |
|-------|------------|------------|--------------|
| 1 | Database schema (users, audio_files, playlists tables) | None | Migration runs, tables exist |
| 2 | Audio upload API + file storage | Phase 1 | Upload endpoint returns 200, file persisted |
| 3 | Audio library UI (list, search, metadata display) | Phase 2 | Files appear in UI after upload |
| 4 | Playback engine (play, pause, seek, volume) | Phase 2 | Audio plays through browser |
| 5 | Frequency visualization (FFT, canvas rendering) | Phase 4 | Visualizer responds to audio |
| 6 | Playlist management (create, reorder, delete) | Phase 1, 3 | Playlists persist and display |
| 7 | Polish (loading states, error handling, responsive) | All prior | No console errors, all states handled |
