---
id: TASK-4
title: Add shellcheck linting and bats smoke tests
status: To Do
assignee: []
created_date: '2026-05-26 02:53'
updated_date: '2026-05-26 12:19'
labels:
  - dev
  - quality
dependencies: []
priority: medium
ordinal: 4000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Bash script currently has no automated quality checks. Add:

  - shellcheck (https://www.shellcheck.net) to catch quoting/portability bugs
  - bats-core (https://github.com/bats-core/bats-core) smoke tests covering:
      * questioner --help exits 0
      * questioner --context-only emits XML on stdout
      * questioner --prompt includes both the prompt header and bundled context
      * questioner --passes 2 fails with exit code 64 and prints the expected 'not implemented' error to stderr (this is the intentional guard until TASK-1 ships - the test verifies the guard, then will need updating once --passes is implemented)
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 shellcheck passes with no warnings on the questioner script
- [ ] #2 bats test suite covers all three modes and known error paths
- [ ] #3 CI workflow (.github/workflows/test.yml) runs shellcheck and bats on push and PR
<!-- AC:END -->
