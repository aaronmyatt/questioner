---
id: TASK-10
title: Add VERSION constant and --version flag
status: Done
assignee: []
created_date: '2026-05-26 02:54'
updated_date: '2026-05-28 10:29'
labels:
  - release
dependencies: []
priority: medium
ordinal: 10000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Script has no version output today. Add a VERSION constant near the top of the script, surface it via --version (printing 'questioner X.Y.Z'), and reference it in error/help output. Anchors release tagging and lets users self-report versions in bug reports.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 questioner --version prints semver string and exits 0
- [ ] #2 VERSION is defined once near the top of the script and reused in help text
<!-- AC:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Added VERSION=0.1.0 constant near the top of the script, a --version flag (prints 'questioner 0.1.0', exit 0), and a version header line in --help that reuses the constant.
<!-- SECTION:FINAL_SUMMARY:END -->
