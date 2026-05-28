---
id: TASK-5
title: 'Run §9 validation experiments (zero-docs, large repo, multi-provider)'
status: To Do
assignee: []
created_date: '2026-05-26 02:54'
updated_date: '2026-05-28 10:21'
labels:
  - dev
  - research
dependencies: []
priority: low
ordinal: 5000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
DESIGN.md §9 names three experiments that gate v0.1 prioritization decisions: (1) repo with zero curated docs - does output thin out or hold up? (2) ~500k LOC repo - where does repomix break? (3) same repo across Claude/GPT/Gemini - do the top 5 overlap meaningfully? Outcomes determine whether signal-augmented fallback (§7.2) or multi-agent jury (§7.2) become urgent.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Results from all three experiments documented in DESIGN.md §9 (or a new EXPERIMENTS.md)
- [ ] #2 Decisions explicitly recorded: does signal-augmented fallback move to v0.1 scope?
- [ ] #3 Decisions explicitly recorded: does multi-agent jury move to v0.1 scope?
<!-- AC:END -->
