# Critique Module: Wave Completion Stage

## Pre-Mortem Prompt

"This wave's output was merged into the main branch and broke the entire build
pipeline. Tests that passed in isolation failed in integration. A contract violation
went undetected and corrupted downstream data. What did we miss?"

## Dimension Checklist

### 1. Plan Drift (Actual vs. Expected)
- For each task in the wave: compare expected output vs. actual output, expected
  files modified vs. actual, expected duration vs. actual
- Check scope drift: did any task produce outputs not in the plan? Skip specified outputs?
- Calculate drift metric: (planned items not completed + unplanned items completed) /
  total planned items. Drift >20% warrants a replan. Drift >30% is S1.
- Check architectural drift: do actual module boundaries and dependency directions
  match the architecture plan?

### 2. Test Passage Verification
- Re-run all tests in a clean environment (not the agent's environment) if possible
- Check test coverage delta: did coverage increase proportional to new code?
- Audit test quality: look for tests that always pass, tests that test mocks instead
  of real behavior, tests with no assertions
- Verify ALL pre-existing tests still pass -- a wave that passes its own tests but
  breaks existing ones is a net negative (S0 if regressions found)
- Count new tests vs. new code -- zero new tests for substantial new code is S1

### 3. Contract Compliance
- Verify implemented function signatures exactly match specified contracts
- Check behavioral contracts: do implementations satisfy preconditions, postconditions, invariants?
- Run cross-module integration check: do modules produced by different agents work together?
- Check API backward compatibility if wave modified existing APIs

### 4. Technical Debt Assessment
- Scan for new TODO/HACK/FIXME markers added during the wave
- Compare cyclomatic complexity before and after -- significant increases are debt
- Check dependency debt: were new dependencies added? Are they maintained and licensed?
- Count disabled or skipped tests -- each is tracked debt
- Calculate debt trend: are new TODOs outnumbering resolved ones?

### 5. Budget Burn Rate
- Calculate CPI (Cost Performance Index) = Earned Value / Actual Cost. CPI < 1.0 = over-budget
- Calculate SPI (Schedule Performance Index) = Earned Value / Planned Value. SPI < 1.0 = behind schedule
- Calculate EAC (Estimate at Completion) = Budget at Completion / CPI
- Track per-wave trend: is burn rate accelerating or decelerating?
- Account for retry costs: retry-heavy waves indicate quality problems AND budget problems

### 6. Go/No-Go Assessment

**Hard blockers (any one = NO-GO for next wave):**
- Regression tests failing
- Contract violations between modules
- Security vulnerabilities introduced
- Drift >30% from plan
- Budget CPI < 0.7 (>40% over budget)

**Soft signals (accumulate toward NO-GO):**
- Test coverage decreased
- Technical debt ratio increased >5%
- Multiple tasks required >2 retries
- Agent self-reported results differ from independent verification
- New TODOs outnumber resolved ones

**GO conditions (all must be true):**
- All planned tasks completed or explicitly deferred with rationale
- All tests pass in clean environment
- All contracts verified
- Budget within 20% of projection (CPI > 0.83)
- No unresolved SPOF introduced
- Next wave's dependencies all satisfied

## Forward Impact Assessment

After evaluating the wave, assess the remaining execution plan:

- Based on actual performance, are remaining wave budget projections still realistic?
- Did this wave reveal assumptions that invalidate remaining wave plans?
- Should remaining task assignments be adjusted based on agent performance?
- Are there dependency changes that require re-sequencing remaining waves?
- Does the adaptive re-planning strategy need to be triggered?

Present forward impact findings separately from wave critique findings, labeled
as "Forward Impact" category.

## Deterministic Pre-Checks

- Verify the wave's task list matches what was specified in the execution plan
- Check that all merge operations followed the specified merge order
- Verify gate scripts were run and results recorded
- Check git log: do actual commits match expected task completion pattern?
- Count files modified vs. file manifest: flag unexpected file modifications
