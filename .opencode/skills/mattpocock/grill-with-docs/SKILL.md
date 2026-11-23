---
name: grill-with-docs
description: Clarify a request against a repository, its glossary and ADRs; do not write documentation without explicit approval.
---

In OpenCode, the user can invoke this skill directly by mentioning `@grill-with-docs` in a request. Agents can also discover it when relevant.

Explicitly load `grilling` and `domain-modeling`. Use the progressive interview from `grilling`, grounded in the code, glossary, ADRs and repository instructions, to verify context and sharpen terminology.

This adaptation is intentionally non-destructive: use `domain-modeling` to identify and propose clarifications, but do not create or edit files without the user's explicit approval (relayed by the Coordinator when running as a subagent). A glossary is not a spec; an ADR records only a decision that meets all three `domain-modeling` criteria.

At the end, report resolved or ambiguous terms, conflicts with existing documents, confirmed product decisions, open questions and any ADR candidate to the user or calling agent. Do not publish to an issue tracker.
