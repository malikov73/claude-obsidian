---
name: obsidian-brainstorm
description: "Use when working on a business idea / strategy brainstorm where reasoning, decisions, and research must persist across sessions in Obsidian. Extends obsidian-conventions with idea-portfolio structure, MADR decision logs, and research-note patterns."
---

# Obsidian Brainstorming Specialization

Use this skill alongside `obsidian-conventions` (which carries the universal rules — frontmatter, naming, linking, atomicity, append-vs-rewrite). This skill adds patterns specific to brainstorming portfolios of business ideas — tracking 5+ ideas in parallel, persisting decisions and research across sessions.

## Why this skill exists

- **Context loss between sessions:** chat history is truncated; reasoning evaporates. A decision made in session N is gone in session N+1 unless it's in a note.
- **Decision drift:** without a log, decisions get re-litigated arbitrarily — the same trade-off considered over and over with no record of why we settled on X.
- **Repeated research:** web research is expensive in time and tokens. Persisting findings lets sessions build on prior work instead of re-googling.

## Idea portfolio shape

```
<vault root or subfolder>/
├── 00 Index.md                      # cross-idea index + status snapshot
├── NN <Idea Name>/
│   ├── 01 Идея.md                   # concept + value prop + ICP
│   ├── 02 Конкуренты.md             # landscape per market/segment
│   ├── 03 План реализации.md        # stack, build phases
│   ├── 04 Трафик и воронка.md       # acquisition, SEO
│   ├── 05 P&L и экономика.md        # unit economics, scenarios
│   ├── 06 <topic>.md, 07 <topic>.md, ...    # additional research / topics
│   └── NN Решения брейншторма.md    # decision log; NN = next free number
```

Section/file titles can be in any language — examples use Russian because that's a common convention in the asman vault. English equivalents work equally well: `01 Idea.md`, `02 Competitors.md`, `NN Decisions <project>.md`. The numeric prefix matters more than the language.

## 00 Index.md structure

```markdown
---
tags: [type/moc, brainstorm/index]
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: moc
status: active
---

# Brainstorm Idea Portfolio

> Cross-idea snapshot. Updated at end of each brainstorm session.

## Active ideas
| # | Idea | Status | Last update | Next milestone |
|---|------|--------|-------------|----------------|
| 07 | [[07 Wedding Speech Generator/01 Идея\|Wedding Speech]] | brainstorm | 2026-05-02 | domain + brand naming |

## Paused / on hold
- [[NN Idea/01 Идея|Idea]] — why paused, when to revisit

## Archived
- [[NN Idea/01 Идея|Idea]] — why killed

## Cross-cutting themes
- Multi-language SEO: [[07 Wedding Speech]], [[NN Other]]
- AI-quality moats: [[07 Wedding Speech]], [[NN Other]]
```

(Note: the `\|` in the table is a deliberate escaped pipe — wikilinks inside a table cell require escaping the pipe so it doesn't end the cell.)

## Decision log convention (MADR)

- Start a decision log only when **>3 key decisions** exist for an idea. Below that, inline rationale in `01 Идея.md` is sufficient.
- File: `<NN Idea>/NN Решения <topic>.md`. NN = next free number after research files.
- Each entry follows MADR (template at `templates/adr-madr.md`). Required fields: id, status, context, drivers, options, decision, consequences, revisit-when, related.
- **Append-only.** Old decisions never deleted; mark `superseded-by: "[[ADR-NNNN]]"`.
- Cross-reference from other files via wikilink + heading: `[[NN Решения брейншторма#ADR-0012]]`.
- ID scheme: `ADR-NNNN` (4-digit, sequential, never reused).

Example entry:

```markdown
## ADR-0017: Use OpenRouter as LLM gateway

- **Status:** accepted
- **Date:** 2026-05-01
- **Deciders:** self

### Context
Need access to Claude Opus 4.7 + fallback options without managing direct Anthropic billing during MVP.

### Considered options
- Direct Anthropic SDK
- OpenRouter
- Multi-provider abstraction (LangChain)

### Decision
**OpenRouter**, because: 5.5% credit-fee is noise at MVP scale, gives free fallback to Sonnet/GPT-4 on failures, consolidated billing.

### Consequences
- Positive: instant multi-model A/B testing; simple API.
- Negative: extra 5.5% becomes meaningful at $5k+ MRR.

### Revisit when
- MRR > $5k → switch to direct Anthropic.
- OpenRouter SLA degrades.

### Related
- [[03 План реализации#LLM tier]]
- [[05 P&L и экономика#COGS]]
```

## Research note convention

- **Inline UPDATE** (preferred for small findings): single finding <150 words → append `## UPDATE YYYY-MM-DD: <finding>` block to the relevant existing file.
- **Separate file** (when finding doesn't fit): topic >150 words OR multi-source synthesis → create `<NN Idea>/NN <Topic> Research.md` using `templates/research-note.md`.
- Sources captured in frontmatter `sources: [url1, url2]` for inline notes; via Source-folder note (one per source) for permanent records.
- Always link from the parent idea note: e.g., `01 Идея.md` references `06 Multi-language Research.md` via a "Related" or "Research" section.

## Cross-references between ideas

- In `00 Index.md`: every idea is a wikilink with alias (`[[NN Idea/01 Идея|Idea Name]]`).
- For shared themes (tech stack, market, target segment): tag both ideas with `cross-cutting/<theme>` and add a "Related ideas" section to each.
- When pausing/killing an idea: **never delete the folder**. Update `01 Идея.md` frontmatter `status:` to `paused` or `archived`. Add a section `## Why paused: YYYY-MM-DD` with reasoning. Update `00 Index.md` accordingly.

## Workflow heuristics

1. **Session start:** read `00 Index.md` first. Then read the relevant idea's files (typically `01 Идея.md` + recently-updated files via `obsidian_get_recent_changes` filtered to that folder).
2. **During research:** save findings *immediately* to notes — not at session end. Even partial findings are valuable; lost context is the bigger risk.
3. **After each key decision:** append to the decision log with full MADR fields. Don't postpone — context is freshest now.
4. **Session end:** update `00 Index.md` if scope/priority changed. Update `updated:` frontmatter on touched files.

## Anti-patterns (do NOT save)

- ❌ Intermediate brainstorm iterations → only the final decision/finding belongs in notes.
- ❌ Information already in code or git history.
- ❌ Ephemeral session details ("just chatted about X").
- ❌ Long quotes from research without your own synthesis — synthesize then cite.
- ❌ Creating new idea folders for variations of an existing idea — append to existing or use a sub-section.

## Extends obsidian-conventions

> All frontmatter, naming, linking, atomicity, and append-vs-rewrite rules from the `obsidian-conventions` skill apply unchanged. This skill only adds the brainstorm-specific layer on top. If a question isn't answered here, defer to `obsidian-conventions`.
