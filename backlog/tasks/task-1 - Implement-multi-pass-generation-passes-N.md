---
id: TASK-1
title: Implement multi-pass generation (--passes N)
status: To Do
assignee: []
created_date: '2026-05-26 02:53'
updated_date: '2026-05-28 10:21'
labels:
  - dev
  - v0.1
dependencies: []
priority: low
ordinal: 1000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
The --passes flag is not implemented yet. Running 'questioner --passes 3' today fails with exit code 64 (intentional - the script refuses unsupported flags rather than silently ignoring them; see questioner lines 208-214).

Implement N runs of generation, optionally with deliberately varied emphases per pass (git churn, static structure, prose docs, test names), then surface only questions that appear across passes. Cheapest form of noise reduction available. See DESIGN.md section 7.1.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Running questioner --passes 3 completes without exit 64
- [ ] #2 Questions appearing in <2 passes are filtered out by default
- [ ] #3 Optional --bias <angle> flag biases a single pass; --passes cycles through angles internally
- [ ] #4 Behaviour documented in --help and DESIGN.md updated to mark feature shipped
<!-- AC:END -->
