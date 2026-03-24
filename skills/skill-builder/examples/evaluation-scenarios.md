# Skill Builder Evaluation Scenarios

Three test cases for validating the Skill Builder skill itself.

---

## 1. Happy Path: Bootstrap a Simple Analysis Skill

**Input**: "Build me a skill that audits my project's dependencies for outdated packages"

**Expected behavior**:
- Detects Bootstrap mode
- Asks intake questions (some may be pre-answered from the prompt)
- Classifies as Medium freedom / Analysis pattern
- Generates 2-3 candidate descriptions in third person with trigger keywords
- Produces SKILL.md under 500 lines with Constraints section
- Reads principles.md and runs quality checklist before presenting
- Generates 3 evaluation scenarios for the new skill
- Supporting files organized in correct directories

**Verify**: Generated skill passes the quality checklist with 18+ Pass items.

---

## 2. Edge Case: Partial Intake in Initial Request

**Input**: "Create a skill called deploy-preview that deploys a preview environment. It should use Bash and Read tools, be user-invocable only, and produce a URL."

**Expected behavior**:
- Detects Bootstrap mode
- Recognizes that Purpose, Tools, Invocation, and Outputs are already answered
- Skips those intake questions — only asks about Trigger and Complexity
- Does NOT re-ask questions the user already answered
- Proceeds through remaining phases normally

**Verify**: No redundant questions asked. All user-provided values are preserved in the generated SKILL.md.

---

## 3. Failure Case: Optimize a Badly Structured Skill

**Input**: Provide a SKILL.md with these problems:
- No frontmatter at all
- Description embedded in the body as prose
- Over 800 lines of instructions
- No Constraints section
- Nested reference files (ref → ref → ref)
- Mixed terminology ("endpoint" / "URL" / "route" used interchangeably)

**Expected behavior**:
- Detects Optimize mode
- Reads the skill and all supporting files
- Runs quality checklist — should produce multiple Fail items
- Structural analysis flags: missing frontmatter, over line limit, buried instructions, nested refs
- Presents audit report with clear Pass/Warn/Fail scoring
- Recommendations ordered by impact (frontmatter and description first, then structure, then content)
- Asks before applying fixes
- After fixes, re-runs audit to verify improvement

**Verify**: Audit report accurately identifies all planted problems. Fixed skill scores significantly higher on re-audit.
