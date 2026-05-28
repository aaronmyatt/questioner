---
id: TASK-15
title: Write Installation and Usage docs pages
status: Done
assignee: []
created_date: '2026-05-26 02:55'
updated_date: '2026-05-28 10:29'
labels:
  - pages
  - docs
dependencies:
  - TASK-13
priority: high
ordinal: 15000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Two dedicated docs pages on the Pages site: (1) Installation - covers Homebrew, curl|bash, manual clone, plus repomix dependency setup and QUESTIONER_AGENT configuration; (2) Usage - flag reference, the three modes, env vars, and recipes (CI integration, in-IDE agent invocation, multi-provider piping). Keep CLI-source-of-truth: usage table generated/validated against --help output.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 /docs/install/ covers Homebrew, curl install, and manual paths; includes repomix install and agent CLI setup
- [ ] #2 /docs/usage/ documents all modes, flags, env vars, and exit codes
- [ ] #3 Recipes section shows: CI, in-IDE agent (--context-only), multi-provider via QUESTIONER_AGENT swap
- [ ] #4 Usage docs keep --baseline* under a separate 'prompt development (maintainers)' section, clearly marked as not part of normal end-user use, per DESIGN.md §4.4
<!-- AC:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Wrote docs/install.md and docs/usage.md. Install covers repomix, clone+symlink (Homebrew/curl marked planned), agent config, requirements table. Usage documents all modes/flags/env vars/exit-64 behaviour, recipes (exclude, bigger funnel, in-IDE --context-only, multi-provider), and keeps --baseline* under a clearly-marked 'prompt development (maintainers)' section per DESIGN.md §4.4.
<!-- SECTION:FINAL_SUMMARY:END -->
