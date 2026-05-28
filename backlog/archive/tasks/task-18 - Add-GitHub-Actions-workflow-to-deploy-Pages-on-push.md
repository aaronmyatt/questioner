---
id: TASK-18
title: Add GitHub Actions workflow to deploy Pages on push
status: To Do
assignee: []
created_date: '2026-05-26 02:55'
updated_date: '2026-05-26 02:55'
labels:
  - pages
  - ci
dependencies:
  - TASK-13
priority: medium
ordinal: 18000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Automate Pages build/deploy on push to main. Use the official Jekyll deploy action (https://github.com/actions/jekyll-build-pages + https://github.com/actions/deploy-pages) configured to build the /docs folder. Triggers only on changes under docs/** to avoid wasted runs. If MkDocs/Docusaurus is chosen instead, swap to their corresponding actions.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 .github/workflows/pages.yml builds and deploys the site on push to main
- [ ] #2 Workflow uses path filters so non-docs commits don't trigger a deploy
- [ ] #3 First green run visible in Actions tab; site updates within 2 minutes of merge
<!-- AC:END -->
