---
name: domain-modeling
description: Clarify domain terms and invariants, respect the glossary, and identify the few decisions that warrant an ADR.
metadata:
  "opencode/autoinvoke": false
---

# Domain modeling

Use this discipline when a discussion touches domain language, invariants, boundaries between concepts, the glossary or a durable architectural decision. It complements a specification: a glossary is neither a spec nor an implementation scratchpad.

## Understand before proposing

- Find `GLOSSARY-MAP.md`, relevant `GLOSSARY.md` files, ADRs, repository instructions and code. Follow the map when the repository has multiple contexts.
- If a term conflicts with the glossary, cite its definition and ask the user to clarify (or ask the Coordinator to relay the question when running as a subagent).
- Identify vague or overloaded words and suggest a canonical term. Test domain relationships with concrete scenarios and edge cases.
- Compare statements about behavior with repository evidence. Report contradictions as verified facts, assumptions or questions.

## Glossary

- Propose a term only if it is specific to the project's domain. General programming concepts do not belong.
- Keep definitions short (one or two sentences), describe what the concept is, and list terms to avoid where useful.
- Preserve existing structure. For a single-context repository without a glossary, propose a root `GLOSSARY.md`; for multiple contexts, follow `GLOSSARY-MAP.md`.
- Propose entries to the user or calling agent first. Do not create or edit a glossary without explicit user approval.
- Use [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md) as a guide, unless the repository already has a convention.

## ADRs

An ADR is justified only when all three conditions are true:

1. The decision is costly to reverse.
2. It would be surprising to future contributors without context.
3. It results from a genuine trade-off between plausible options.

Easy-to-reverse, obvious or uncontentious decisions do not need an ADR. For a candidate, present context, options, recommendation and consequences with status “proposed.” Do not write it or present it as approved until the user explicitly authorizes its creation.

Follow the repository's ADR format and numbering. If none exists, propose `docs/adr/NNNN-slug.md` and consult [ADR-FORMAT.md](./ADR-FORMAT.md). Create the directory only when an approved ADR is actually needed.
