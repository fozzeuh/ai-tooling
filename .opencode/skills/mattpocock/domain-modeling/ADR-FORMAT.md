# ADR format

Follow the repository's existing format first. Otherwise, ADRs live in `docs/adr/` and use sequential numbering (`0001-slug.md`, `0002-slug.md`). Create the directory only when an approved ADR is needed.

## Minimal template

```md
# {Short decision title}

{Context, decision and reason in one to three sentences.}
```

Add only when useful: status (`proposed`, `accepted`, `deprecated`, `superseded`), considered options, or consequences.

Before writing, confirm that the user explicitly authorized this ADR. Propose one only if the decision is hard to reverse, surprising without context and the result of a genuine trade-off.
