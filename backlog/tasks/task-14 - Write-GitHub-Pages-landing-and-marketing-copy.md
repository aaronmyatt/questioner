---
id: TASK-14
title: Write GitHub Pages landing and marketing copy
status: In Progress
assignee: []
created_date: '2026-05-26 02:54'
updated_date: '2026-05-28 10:29'
labels:
  - pages
  - marketing
dependencies:
  - TASK-13
priority: high
ordinal: 14000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Landing page should answer in <10 seconds: what does this do, who is it for, how do I run it. Lead with a hero block stating the hypothesis from DESIGN.md §1, show a 'before/after' example (raw repo vs top 5 questions), and a 30-second quickstart. Avoid feature-listing; lead with the cold-start problem it solves.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Hero section with one-sentence value prop, hypothesis statement, and primary CTA (install)
- [ ] #2 Before/after example showing the top-5 output from a real repo (e.g. jira-sprint-metrics-app)
- [ ] #3 30-second quickstart with copy-paste install + first run
- [ ] #4 Footer with links to GitHub repo, DESIGN.md, SKILL.md, and license
- [ ] #5 Landing copy frames the tool around 'any organised corpus' (not just repos); at least one non-code example referenced
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Landing page docs/index.md written: hero with one-line value prop + CTA, the hypothesis statement, a 'what the output looks like' tier explainer, a 30-second quickstart, and a footer links section. TWO gaps: (1) AC#2 wants a REAL before/after top-5 from an actual repo — I cannot generate that without an LLM run (your agent/quota), so the example block is clearly LABELLED 'Illustrative' with a TODO to swap in a real capture (ties to TASK-17). (2) AC#5 ('frame around any organised corpus + non-code example') is SUPERSEDED by the prompt-freeze decision: the --kind/agnostic feature (TASK-19/20) is deprioritised and won't ship in v0.1, so the landing is framed around code repos (what actually ships). Revisit AC#5 if/when the agnostic feature ships.
<!-- SECTION:NOTES:END -->
