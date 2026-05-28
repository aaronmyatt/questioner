---
id: TASK-16
title: Port DESIGN.md and SKILL.md to docs site
status: To Do
assignee: []
created_date: '2026-05-26 02:55'
updated_date: '2026-05-26 02:55'
labels:
  - pages
  - docs
dependencies:
  - TASK-13
priority: medium
ordinal: 16000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
DESIGN.md is the load-bearing rationale doc; SKILL.md is the agent integration spec. Port both to the Pages site as /docs/design/ and /docs/skill/. Keep the source markdown files in the repo as the canonical version; the site pages should be thin wrappers that include them, so the two don't drift. Jekyll can include them with a Liquid {% include_relative %} tag.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 /docs/design/ page rendered from DESIGN.md without manual duplication
- [ ] #2 /docs/skill/ page rendered from SKILL.md without manual duplication
- [ ] #3 Internal links between landing, usage, design, and skill pages all work
<!-- AC:END -->
