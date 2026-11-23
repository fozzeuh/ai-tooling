---
name: grill-me
description: Clarify an idea through progressive questions without editing files; use for an early-stage or ambiguous request.
---

In OpenCode, the user can invoke this skill directly by mentioning `@grill-me` in a request. Agents can also discover it when relevant.

Load the `grilling` skill and progressively interview the user about an idea or decision. This mode does not require a repository or domain documentation. For a request grounded in a codebase and its vocabulary, prefer `grill-with-docs`.

Clarify the problem, actors, outcome, scope, scenarios and acceptance criteria. In a primary session, ask the user directly and wait for their answers; as a subagent, send questions to the calling agent to relay. Do not edit files or create tickets.
