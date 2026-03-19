# Scoping Question Bank

Use these questions to scope research before diving in. Select questions
based on the detected research type. Never ask more than 4 questions total --
pick the highest-signal questions for the specific topic.

## Universal Questions (Any Research Type)

1. **Intent**: What decision will this research inform? Are you evaluating
   options, understanding a space, or preparing to build something specific?
2. **Known context**: What do you already know or assume about this topic?
   Any prior work or existing research I should build on?
3. **Constraints**: Are there hard constraints I should know about? (timeline,
   mandated technology, budget, compliance, team skills)
4. **Success criteria**: What would make this research "done" for you? What
   would a great output look like?

## Technical Research

5. **Evaluation criteria**: What matters most -- performance, developer
   experience, ecosystem maturity, cost, or something else?
6. **Scale requirements**: What scale does this need to operate at? (users,
   data volume, throughput, latency)
7. **Integration context**: What existing systems must this integrate with?
   What's the current tech stack?
8. **Build vs buy**: Are you open to using third-party services, or does this
   need to be built in-house?
9. **Maturity tolerance**: Are you comfortable with cutting-edge/experimental
   technology, or do you need battle-tested solutions?

## Product/Market Research

10. **Target user**: Who specifically is the target user? What's their role,
    context, and level of technical sophistication?
11. **Competitive awareness**: Are there specific competitors or products you
    already know about that I should analyze?
12. **Differentiation hypothesis**: Do you have an initial theory about what
    would make your approach different or better?
13. **Market signals**: Have customers or prospects asked for this? Are there
    Gong calls, support tickets, or Slack threads I should mine?
14. **Business model**: How would this be monetized or justified? Is there a
    revenue target or cost savings goal?

## Domain Research

15. **Depth vs breadth**: Should I survey the full domain broadly, or go
    deep on specific aspects you've already identified?
16. **Expertise level**: How familiar is your team with this domain? Should
    the research include foundational concepts or focus on advanced topics?
17. **Academic vs practical**: Should I prioritize academic rigor (papers,
    formal methods) or practical implementation experience (case studies,
    production systems)?
18. **Time horizon**: Are you interested in the current state of the art, or
    also emerging/future directions?

## Internal Context Questions

19. **Internal sources**: Should I search Slack, Confluence, Jira, Gong, or
    Snowflake for relevant internal context? Any specific channels, spaces,
    or projects to focus on?
20. **Stakeholders**: Are there specific teams or people whose perspective
    matters for this research?
21. **Prior decisions**: Are there past architecture decisions or technical
    choices that constrain what's possible here?

## When to Skip Scoping

Skip scoping entirely when ALL of the following are true:
- The topic is specific and well-defined (not "research AI" but "evaluate
  CRDT libraries for collaborative editing in TypeScript")
- The user has provided clear context about why they need the research
- The research type is obvious from the topic
- There are no ambiguous constraints

When skipping, briefly state your assumptions at the start of the research
plan so the user can correct course early if needed.
