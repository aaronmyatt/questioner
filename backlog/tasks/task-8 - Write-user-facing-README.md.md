---
id: TASK-8
title: Write user-facing README.md
status: Done
assignee: []
created_date: '2026-05-26 02:54'
updated_date: '2026-05-28 10:29'
labels:
  - release
  - docs
dependencies: []
priority: high
ordinal: 8000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
No README present. Needs to lead with the one-line value proposition (top-5 high-signal exploration questions for any repo), then install, quickstart, mode reference, and link to DESIGN.md for rationale. Keep marketing prose minimal in README - that lives on GitHub Pages per goal 3.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 README opens with a single-sentence value prop and a usage example block
- [ ] #2 Install section covers: clone+symlink, Homebrew (once tap exists), and curl-pipe-bash (if shipped)
- [ ] #3 Mode/flag reference table is consistent with the script --help output
- [ ] #4 Links to DESIGN.md, SKILL.md, and the GitHub Pages site
- [ ] #5 README must NOT feature the --baseline* commands in the main usage flow; mention them only under a brief 'Developing the prompt' / contributor note pointing to DESIGN.md §4.4
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
GitHub + Pages URLs assume aaronmyatt/questioner and aaronmyatt.github.io/questioner — adjust if owner/repo differs.
<!-- SECTION:NOTES:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Wrote README.md: one-line value prop + usage block, how-it-works tiers, install (clone+symlink working; Homebrew/curl noted as planned), agent config, mode/flag tables consistent with --help, baseline noted as maintainer-only, links to DESIGN/SKILL/CHANGELOG/Pages.
<!-- SECTION:FINAL_SUMMARY:END -->
