---
name: grilling
description: Structure a clarification interview into dependency-aware rounds, with recommendations and a clear stopping point.
metadata:
  "opencode/autoinvoke": false
---

# Progressive interview

Clarify a plan, decision or idea by mapping decisions and their dependencies. Find verifiable facts in the context, files and available tools; the user should decide only the decisions that belong to them.

## Process

1. Identify unresolved decisions and their dependencies. The frontier is the set of questions whose prerequisites are already settled.
2. Ask the frontier questions in one round, excluding questions that depend on another unresolved answer.
3. Give each question a clear title, concise wording, useful options if needed, and a recommendation. Phrase it so “yes” accepts the recommendation.
4. Wait for answers before generating the next round. Incorporate corrections and explicit “I don't know” answers.
5. Stop when the brief is clear enough for the requested deliverable. List remaining uncertainties instead of extending the interview without benefit.

## Round size and format

- Ask at most 3–5 related questions per round in the usual case. Use fewer for difficult or independent decisions.
- Do not ask a long sequence of one-at-a-time questions when they can be grouped without inventing dependencies.
- Use this format for questions:

```text
❓ **Q1 — <title>**: <question and useful context>

➡️ Recommendation: <option and concise reason>
```

- In a primary session, address the user directly and wait for their answers. As a subagent, address the calling agent to relay the questions; do not assume the user can see your child session, and wait for answers to be passed back.

## Boundaries

- Do not ask the user for facts available from the repository or other accessible sources. Report inaccessible research or contradictions.
- If a question requires a mockup, prototype or observed behavior, mark the evidence needed instead of rephrasing the question indefinitely.
- Do not present a recommendation as a decision. Do not begin implementation, edit files or take external action before explicit approval from the user and calling agent.
