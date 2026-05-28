---
id: TASK-17
title: Build Examples / Case Studies page
status: To Do
assignee: []
created_date: '2026-05-26 02:55'
updated_date: '2026-05-26 12:13'
labels:
  - pages
  - marketing
dependencies:
  - TASK-13
priority: medium
ordinal: 17000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Marketing leverage: show what the output actually looks like on real repos. Start with jira-sprint-metrics-app (Pass 1 vs Pass 2 from DESIGN.md §2), then add 2-3 popular OSS repos run through questioner with the top 5 captured verbatim. Each case study notes: repo size, agent used, repomix exclusions, surprising/unsurprising results. Acts as both demo and honesty signal - users can see what they're going to get.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 At least 3 case studies published, each with repo URL, command used, and full top-5 output
- [ ] #2 Pass-1 vs Pass-2 ablation example from DESIGN.md §2 reproduced as a case study
- [ ] #3 Each case study includes a 'what this shows / what this misses' annotation
- [ ] #4 At least one case study features a non-code corpus (notes / research / website content) run with --kind, demonstrating kind-shaped output
<!-- AC:END -->
