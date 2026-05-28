---
id: TASK-12
title: Ship Homebrew tap and curl install script
status: To Do
assignee: []
created_date: '2026-05-26 02:54'
updated_date: '2026-05-26 02:55'
labels:
  - release
  - distribution
dependencies:
  - TASK-11
priority: medium
ordinal: 12000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Two install paths to keep barrier-to-entry low: (1) Homebrew tap (e.g. aaronmyatt/tap/questioner) with a formula that depends on node (for repomix) and installs the script to bin/; (2) curl|bash install script hosted on GitHub Pages that drops the script into ~/.local/bin and prompts to install repomix. Both should be documented in README install section. Brew docs: https://docs.brew.sh/Taps. Install-script best practices: https://github.com/koalaman/shellcheck/wiki and similar reputable curl-pipe-bash patterns.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Homebrew tap repo created with a working formula; 'brew install <tap>/questioner' succeeds
- [ ] #2 install.sh published; 'curl -fsSL .../install.sh | bash' works on macOS+Linux
- [ ] #3 Both paths verified on a fresh machine (or container) before release
<!-- AC:END -->
