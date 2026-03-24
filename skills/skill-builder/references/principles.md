# Skill Builder Principles Reference

These are the research-backed principles that the Skill Builder enforces. Each principle includes its confidence level, source domain, and application guidance. Sourced from 80+ academic papers, Anthropic guidance, frontier model provider documentation, and community practitioner knowledge.

---

## A. Architecture Principles

### A1. Progressive Disclosure is Non-Negotiable
**Confidence: Very High** | Sources: Claude Code, OpenAI Codex, LangChain, Cursor

Load skill metadata upfront; load full instructions only when relevant.

- **Level 1 — Metadata** (~100 tokens): `name` and `description` always in context
- **Level 2 — SKILL.md body** (<500 lines): Loaded when skill triggers
- **Level 3 — Bundled resources** (unlimited): Loaded only when Claude needs them

**Application**: Every skill MUST use this three-level structure. SKILL.md is the focused entrypoint. Detailed content goes in `references/`, `examples/`, `scripts/`.

### A2. Generate-Critique-Refine is the Core Loop
**Confidence: Very High** | Sources: Self-Refine (NeurIPS 2023), PromptWizard (ACL 2025), OPRO (ICLR 2024)

Single-pass generation is suboptimal. The three-step loop yields ~20% improvement:
1. Generate a draft
2. Critique against known principles
3. Refine based on critique

**Application**: Never present a skill to the user without first running a self-critique pass against the quality checklist.

### A3. Simplicity-First Escalation
**Confidence: High** | Sources: Anthropic (building-effective-agents), cross-provider consensus

Start with the simplest skill structure. Only add complexity when demonstrably needed.

1. Simple instructions (heuristic guidance)
2. Multi-step workflow (numbered steps with decision points)
3. Tool orchestration (multiple tool backends, fallback chains)
4. Subagent delegation (fork context, spawn specialists)

**Application**: Default to the simplest pattern that handles the use case. If the user's skill can be a list of guidelines, don't scaffold it as a multi-phase workflow.

### A4. Single Responsibility Per Skill
**Confidence: High** | Sources: LangChain, CrewAI, AutoGen, OpenAI Agents SDK, Cursor

Each skill should have one focused responsibility.

**Application**: If a skill does two distinct things, recommend splitting into two skills. "deploy" and "test" not "deploy-and-test."

---

## B. Content Principles

### B1. Conciseness is Empirically Critical
**Confidence: Very High** | Sources: Context Rot (Chroma 2025), Prompt Length (arXiv 2025)

- Target under 2,000 tokens / 500 lines for SKILL.md body
- After ~2,000 tokens, LLM performance degrades measurably
- Redundant information actively harms performance
- Every model degrades at every input length increment

**Application**: Challenge every line — "Would removing this cause Claude to make mistakes?" If the answer is no, remove it.

### B2. Critical Instructions at Start and End
**Confidence: Very High** | Sources: Lost-in-the-middle effect, OpenAI GPT-4.1 guide

Models attend best to the beginning and end of context. The middle gets the least attention. 30%+ accuracy drops observed.

**Application**: Place the most important instructions in the first and last sections of the SKILL.md body. Never bury critical constraints in the middle.

### B3. Structured Formatting Provides Signal
**Confidence: High** | Sources: Structured prompts research (2025), cross-provider consensus

Structured formats reduce entropy and narrow the model's output distribution.

**Application**: Use YAML frontmatter + Markdown headings + bullet points. Keep hierarchy shallow (max 2 heading levels in body). Pick one format and use it consistently — don't mix XML and Markdown.

### B4. Imperative Directives Over Prose
**Confidence: High** | Sources: OpenAI GPT-4.1, Gemini 3, Cursor, GitHub Copilot

Modern models follow instructions literally. Precision > eloquence.

**Application**: Write "Parse the config file and extract the API key" not "You should consider parsing the configuration file to look for the API key." One action per bullet. Imperative verb at the start.

### B5. Include Explicit Constraints (What NOT to Do)
**Confidence: High** | Sources: SWE-bench research, Anthropic agent design

Negative instructions prevent entire classes of failure that positive instructions alone miss. Scaffold design (including constraints) accounts for 10x performance variation.

**Application**: Every skill should have a Constraints section with explicit "Do NOT" statements. Example: "Do NOT modify test files" or "Do NOT commit without user confirmation."

### B6. Few-Shot Examples Are Essential for Complex Tasks
**Confidence: High** | Sources: Google PE whitepaper, OpenAI, MIPRO, cross-provider

Examples guide structure, logic, and tone. 1-5 diverse, realistic examples is the sweet spot.

**Application**: If the skill produces formatted output, generates code, or uses tools in non-obvious ways, include examples. Wrap in `<example>` tags. Include `<thinking>` tags to show reasoning. Ground in actual use cases, not hypotheticals.

### B7. Description Quality is the Highest-Leverage Variable
**Confidence: Very High** | Sources: PE2 (ACL 2024), Claude Code architecture, cross-provider progressive disclosure

The description determines whether the skill gets loaded at all. It's the gatekeeper.

**Rules:**
- Third person ("Processes X when Y")
- Include specific trigger keywords in quotes
- Specify WHAT the skill does AND WHEN to use it
- Under 1024 characters
- Include negative triggers if needed ("Not for X")

**Application**: Generate the description BEFORE the body. Write 2-3 candidates. Optimize for trigger accuracy.

---

## C. Optimization Principles

### C1. Evaluation-Driven Development
**Confidence: Very High** | Sources: Mollick (Wharton 2025), Anthropic skill authoring, OpenAI, OPRO

No universal prompt pattern works. Testing each question 100 times reveals substantial variability.

**Application**: Every skill should ship with 3 evaluation scenarios (happy path, edge case, failure). Use the two-Claude pattern: Claude A designs, Claude B tests.

### C2. The Meta-Prompt is the Key Variable
**Confidence: High** | Sources: PE2 (ACL 2024)

Three critical components for effective meta-prompts:
1. Detailed description of what good looks like
2. Rich context about capabilities and constraints
3. Step-by-step template for construction

**Application**: The Skill Builder itself is a meta-prompt. Its effectiveness depends on how well it describes good skills, provides context about Claude Code, and templates the construction process.

### C3. Domain-Grounded Examples Beat Abstract Instructions
**Confidence: High** | Sources: MIPRO, DSPy, APE

Instructions grounded in actual task inputs/outputs outperform abstract descriptions.

**Application**: When generating examples for skills, use realistic data from the user's actual domain, not generic placeholders.

### C4. Role Prompting is Conditional
**Confidence: Medium-High** | Sources: Role prompting research (2024)

- Effective for open-ended/creative tasks
- Minimal impact on accuracy-based tasks
- Domain alignment matters when used

**Application**: Only include role/persona framing when the skill involves creative, open-ended work. For procedural/accuracy tasks, skip the role and focus on clear instructions.

### C5. CoT Scaffolding is Task-Dependent
**Confidence: Medium-High** | Sources: Mollick (2025), CoT research

Value of explicit chain-of-thought is decreasing as models improve.

**Application**: Include reasoning scaffolds for complex multi-step skills. Omit for simple tasks. Don't add "think step by step" reflexively — only when visible reasoning actually helps.

---

## D. Robustness Principles

### D1. No Magic Pattern — Test Across Runs
**Confidence: Very High** | Sources: Mollick "Complicated and Contingent" (2025)

What helps in one context may hurt in another. Substantial variability under identical conditions.

**Application**: Generated skills should be tested in real scenarios, not assumed correct from structure alone.

### D2. Tool Descriptions Deserve Same Care as Tool Code
**Confidence: High** | Sources: Anthropic tool design, OpenAI function calling

The "intern test": Can someone unfamiliar correctly use the tool given nothing but the description?

**Application**: When skills reference tools, include clear descriptions with: what the tool does, expected inputs, example calls, edge cases, and what errors look like.

### D3. Hoard What Works
**Confidence: High** | Sources: Simon Willison, community consensus

Successful patterns should be documented and reused as templates.

**Application**: When a skill works well, study why and extract reusable patterns for future skills.

### D4. Prompts Are Infrastructure
**Confidence: High** | Sources: Augment Code, community consensus

Skills deserve the same rigor as code: version control, testing, consistent terminology, documentation.

**Application**: Treat skill development as engineering, not copywriting. Version, test, iterate.

---

## E. The Three Agentic Reminders

From OpenAI's GPT-4.1 research, these three behavioral anchors improved SWE-bench scores by ~20% when included in agent prompts:

1. **Persistence**: Keep going until the task is completely resolved
2. **Tool use**: Use tools to gather information; do NOT guess or make up answers
3. **Planning**: Plan extensively before each action, reflect on outcomes after

**Application**: Every workflow/task skill should encode these three behaviors, adapted to the skill's domain.

---

## F. Degrees of Freedom Framework

From Anthropic's skill authoring best practices:

| Freedom Level | When to Use | Instruction Style |
|---------------|-------------|-------------------|
| **High** | Multiple valid approaches, creative tasks | Goals + constraints, heuristic guidance |
| **Medium** | Preferred pattern exists, some variation OK | Pseudocode, parameterized steps with decision points |
| **Low** | Fragile operations, consistency critical | Exact commands, validation at every step |

**Application**: Assess degree of freedom before writing instructions. Match instruction specificity to task fragility.
