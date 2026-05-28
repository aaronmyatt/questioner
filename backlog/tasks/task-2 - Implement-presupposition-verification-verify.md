---
id: TASK-2
title: Implement presupposition verification (--verify)
status: To Do
assignee: []
created_date: '2026-05-26 02:53'
updated_date: '2026-05-28 10:21'
labels:
  - dev
  - v0.1
dependencies: []
priority: low
ordinal: 2000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
The --verify flag is not implemented yet. Running 'questioner --verify' today fails with exit code 64 (intentional - the script refuses unsupported flags rather than silently ignoring them; see questioner lines 208-214).

Implement a second LLM pass that, for each candidate question, asks the model to locate the file/lines supporting the question's factual presuppositions. Questions whose presuppositions cannot be substantiated are flagged speculative. Likely the single biggest slop reducer per DESIGN.md section 7.1.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 questioner --verify completes without exit 64
- [ ] #2 Output marks questions as 'speculative' when presuppositions cannot be substantiated
- [ ] #3 Verification pass reuses the same agent CLI; no second --agent flag required for v0.1
- [ ] #4 Token/cost envelope of the second pass documented in --help and README
<!-- AC:END -->
