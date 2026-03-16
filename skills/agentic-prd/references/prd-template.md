# Agentic PRD Template

Use this template to generate PRDs optimized for AI coding agent implementation.
Adapt section depth based on project complexity -- small features need fewer sections,
large projects need all of them.

---

```markdown
# PRD: [Product/Feature Name]

## Overview

[2-3 sentences: What is being built, why it matters, and who it serves.
This section is for human alignment -- it provides strategic context that
AI agents do not need but stakeholders do.]

## Research Summary

### Verified Platform Capabilities
- [Capability]: [Evidence/source, any constraints discovered]
- [Capability]: [Evidence/source, any constraints discovered]

### Existing Codebase Patterns
- Framework: [name] v[version]
- Database: [type] via [ORM/driver]
- Testing: [runner] with [file naming pattern]
- Style: [key conventions -- naming, formatting, import order]
- Auth: [mechanism -- JWT, sessions, OAuth, etc.]

### Constraints Discovered
- [Constraint]: [Source/evidence]

### Risks & Unknowns
- [Risk]: [Mitigation]

## User Stories

[Keep these outcome-focused. They anchor the spec in user value.]

- As a [role], I want to [action] so that [outcome].
- As a [role], I want to [action] so that [outcome].

## Success Metrics

[Quantifiable outcomes. Include AI-specific metrics if the feature uses AI.]

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| [Metric name] | [Threshold] | [How to measure] |
| [Metric name] | [Threshold] | [How to measure] |

## Non-Goals (DO NOT Implement)

[AI cannot infer scope from omission. State every boundary explicitly.]

- DO NOT implement [feature/capability] in this project
- DO NOT modify [existing system/component]
- DO NOT add [technology/dependency] unless explicitly specified in a phase
- DO NOT optimize for [concern] at this stage

## Tech Stack & Conventions

### Stack
- Language: [language] v[version]
- Framework: [framework] v[version]
- Database: [database] via [ORM/driver]
- Testing: [test runner]
- Package manager: [npm/yarn/pnpm/bun]

### Conventions
- Naming: [camelCase/snake_case/kebab-case for files, variables, components]
- Components: [pattern -- functional components, hooks, etc.]
- Imports: [ordering convention]
- Error handling: [pattern -- try/catch, Result types, error boundaries]
- State management: [approach]

### Commands
- Install: `[command]`
- Dev server: `[command]`
- Build: `[command]`
- Test: `[command]`
- Lint: `[command]`
- Type check: `[command]`

## Project Structure

[Show the directory layout. Use full paths. Mark new directories/files.]

```
project-root/
├── src/
│   ├── components/       # React components
│   ├── hooks/            # Custom hooks
│   ├── lib/              # Utility functions
│   ├── routes/           # Route handlers / pages
│   ├── services/         # Business logic
│   ├── types/            # TypeScript type definitions
│   └── db/
│       ├── schema.ts     # Database schema
│       └── migrations/   # Migration files
├── tests/
│   ├── unit/
│   └── integration/
├── public/               # Static assets
└── package.json
```

## Boundaries

### Always (every phase, every commit)
- Run the full test suite before considering a phase complete
- Follow the naming conventions specified above
- Use existing patterns found in the codebase
- Handle errors explicitly -- never swallow exceptions silently
- Validate all external inputs (user input, API responses)

### Ask First (requires user approval before proceeding)
- Database schema changes beyond what the current phase specifies
- Adding new dependencies not listed in the phase requirements
- Changing public API signatures
- Modifying authentication or authorization logic
- Any changes to CI/CD configuration

### Never (categorically prohibited)
- Commit secrets, API keys, or credentials to files
- Modify files in node_modules/, vendor/, or generated directories
- Remove or skip failing tests without explicit approval
- Use `any` type in TypeScript (use `unknown` and narrow instead)
- Add TODO/FIXME comments as substitutes for implementation

---

## Phase 1: [Foundation Name]

### Objective
[One sentence: what this phase delivers as a working, verifiable unit.]

### Prerequisites
[What must exist before this phase starts. For Phase 1, this is typically
"clean project initialized" or "existing codebase at commit [hash]".]

### Requirements

1. [Requirement with specific, verifiable outcome]
   - File: `/src/path/to/file.ts`
   - Details: [Implementation specifics if non-obvious]

2. [Requirement with specific, verifiable outcome]
   - File: `/src/path/to/file.ts`
   - Details: [Implementation specifics if non-obvious]

[Continue numbering. Stay under 50 requirements per phase.]

### Acceptance Criteria

```gherkin
Scenario: [Descriptive name]
  Given [precondition with specific values]
  When [action with specific input]
  Then [verifiable outcome with specific expected values]

Scenario: [Descriptive name]
  Given [precondition]
  When [action]
  Then [outcome]
```

### Verification

Run after completing this phase to confirm success:

```bash
[test command targeting this phase's functionality]
```

Manual verification:
- [ ] [Observable behavior the user can check]
- [ ] [Observable behavior the user can check]
- [ ] All previous functionality still works (if not Phase 1)

---

## Phase 2: [Feature Layer Name]

### Objective
[One sentence.]

### Depends On
Phase 1 completion (specifically: [what from Phase 1 this phase uses]).

### Requirements

1. [Requirement]
   - File: `/src/path/to/file.ts`

[Continue...]

### DO NOT CHANGE
[List every file/system that must remain stable during this phase.]

- `/src/db/schema.ts` -- database schema is frozen after Phase 1
- `/src/lib/auth.ts` -- authentication logic must not be modified
- [Any other protected files or systems]

### Acceptance Criteria

```gherkin
Scenario: [Name]
  Given [precondition]
  When [action]
  Then [outcome]
```

### Verification

```bash
[test command]
```

Manual verification:
- [ ] [Check]
- [ ] [Check]
- [ ] All Phase 1 functionality still works

---

## Phase 3-N: [Subsequent Phases]

[Same structure as Phase 2. Each phase adds:
- Depends On section referencing prior phases
- DO NOT CHANGE section growing with each phase
- Verification confirming all prior phases still work]

---

## Final Phase: Polish & Hardening

### Objective
Error handling, edge cases, loading states, and UX polish. No new features.

### DO NOT CHANGE
- All database schemas
- All API endpoint signatures
- All business logic from prior phases
- All authentication/authorization flows

### Requirements
1. Add loading states for all async operations
2. Add error boundaries / error handling for all user-facing flows
3. Add input validation for all forms
4. Verify responsive behavior at mobile/tablet/desktop breakpoints
5. Run full test suite and fix any failures
6. Run linter and fix all warnings

### Verification
```bash
[full test suite command]
[lint command]
[type check command]
```

Manual verification:
- [ ] All features from all phases work end-to-end
- [ ] Error states display helpful messages
- [ ] Loading states appear during async operations
- [ ] No console errors or warnings in browser
```

---

## Template Sizing Guide

| Project Size | Phases | Requirements/Phase | Total PRD Length |
|-------------|--------|-------------------|-----------------|
| Small (single feature) | 2-3 | 10-20 | 1-2 pages |
| Medium (multi-feature) | 4-6 | 20-40 | 3-5 pages |
| Large (full application) | 6-10 | 30-50 | 5-10 pages |

Never exceed 50 requirements per phase. If a phase has more than 50, split it.
Never create a PRD with more than ~200 total requirements across all phases --
break the project into multiple PRDs instead.
