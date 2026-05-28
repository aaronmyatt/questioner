---
id: TASK-19
title: Add corpus-kind detection and --kind flag
status: To Do
assignee: []
created_date: '2026-05-26 12:12'
updated_date: '2026-05-28 10:21'
labels:
  - dev
  - v0.1
  - agnostic
dependencies: []
priority: low
ordinal: 19000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Today the prompt assumes 'developer reading a repository' - it asks for files/symbols, lifecycle/TTL/idempotency cross-cutting concerns, build-time vs run-time tradeoffs. Run it on a notes folder, research corpus, or website content directory and the model still produces code-shaped questions. Fix the steering, not the bundler: repomix already handles any directory.

Add a --kind flag with values: auto (default), code, notes, research, content. The 'auto' value uses lightweight extension-based heuristics to pick a kind; an explicit --kind always wins. Keep heuristics conservative - when uncertain, fall back to 'code' so existing behaviour does not regress.

Rough heuristic sketch (refine during implementation):
  - >70% files are .md/.txt and no .ts/.tsx/.js/.py/.go/.rs/.java/.rb/.php present → notes
  - presence of .bib, .tex, citations.json, or a references/ folder → research
  - presence of content/ or _posts/ with .md frontmatter and no package.json/src/ → content
  - otherwise → code

The detection result is exported as a shell variable (e.g. KIND) so the prompt-building step (see follow-up task) can branch on it.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 questioner --kind notes (etc.) accepted; --kind auto is the default
- [ ] #2 Heuristic detection runs only when --kind=auto and prints the detected kind to stderr so users can sanity-check
- [ ] #3 Unknown --kind values fail fast with a clear error listing valid kinds
- [ ] #4 Detection heuristics documented in script comments with rationale, and in DESIGN.md
<!-- AC:END -->
