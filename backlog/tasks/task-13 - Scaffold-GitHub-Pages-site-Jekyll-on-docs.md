---
id: TASK-13
title: Scaffold GitHub Pages site (Jekyll on /docs)
status: In Progress
assignee: []
created_date: '2026-05-26 02:54'
updated_date: '2026-05-28 10:29'
labels:
  - pages
  - docs
dependencies:
  - TASK-11
priority: high
ordinal: 13000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Stand up the GitHub Pages site. Recommend Jekyll on the /docs folder of main (https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll) over a separate gh-pages branch - simpler workflow, no submodule of build output. Use 'just the docs' or a minimal Jekyll theme to start; switch later if marketing copy outgrows it. Alternatives if Jekyll is rejected: MkDocs Material (https://squidfunk.github.io/mkdocs-material/) or Docusaurus (https://docusaurus.io/).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 GitHub Pages enabled on main branch /docs folder in repo settings
- [ ] #2 Site builds cleanly with a chosen theme and renders an empty landing page
- [ ] #3 _config.yml sets baseurl/title/description appropriate for the project
- [ ] #4 Site URL reachable; HTTPS enforced
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Local scaffold complete: docs/_config.yml (just-the-docs via remote_theme, search, GitHub aux link), docs/Gemfile (github-pages gem + webrick for local serve), docs/.gitignore (_site, caches). YAML + front matter validated. REMAINING (needs the repo pushed to GitHub first, blocked on TASK-11): (1) enable Pages on main branch /docs folder in repo settings; (2) confirm site URL reachable + HTTPS enforced. baseurl/url assume a project site at aaronmyatt.github.io/questioner — adjust if owner/repo differs. NOT build-tested locally (would need bundle install + network fetch of remote theme); real render verification comes from the Pages build or TASK-18 CI.
<!-- SECTION:NOTES:END -->
