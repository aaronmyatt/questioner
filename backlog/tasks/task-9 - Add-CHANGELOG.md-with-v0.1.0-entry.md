---
id: TASK-9
title: Add CHANGELOG.md with v0.1.0 entry
status: Done
assignee: []
created_date: '2026-05-26 02:54'
updated_date: '2026-05-28 10:29'
labels:
  - release
dependencies: []
priority: medium
ordinal: 9000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Adopt Keep a Changelog format (https://keepachangelog.com/) and SemVer (https://semver.org/).

v0.1.0 entry should capture:
  - initial public release
  - repomix-based bundling
  - three invocation modes (default / --context-only / --prompt)
  - SKILL.md for agent integration
  - explicitly note --passes >1 and --verify as 'designed but not implemented; invoking them fails with exit code 64 by design' per DESIGN.md section 7.1
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 CHANGELOG.md follows Keep a Changelog format
- [ ] #2 v0.1.0 entry lists features shipped and explicit known-not-implemented items
- [ ] #3 Date stamped to release day, not commit day
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Compare/release links assume the repo at github.com/aaronmyatt/questioner — adjust if the owner/repo differs.
<!-- SECTION:NOTES:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Added CHANGELOG.md (Keep a Changelog + SemVer). v0.1.0 entry lists shipped features and explicitly notes --passes/--verify as designed-not-implemented (exit 64 by design).
<!-- SECTION:FINAL_SUMMARY:END -->
