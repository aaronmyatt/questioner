---
id: TASK-6
title: Pin repomix version compatibility
status: To Do
assignee: []
created_date: '2026-05-26 02:54'
labels:
  - dev
  - quality
dependencies: []
priority: low
ordinal: 6000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Script uses repomix with --style xml --output --ignore flags. These have varied across repomix versions (per script comment line 117). Pin a minimum supported version, surface a clearer error when an older repomix is detected, and document compatibility in README. Ref: https://github.com/yamadashy/repomix/releases.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Script checks 'repomix --version' against a documented minimum and exits with a helpful message if too old
- [ ] #2 README states the minimum required repomix version and links to install paths
<!-- AC:END -->
