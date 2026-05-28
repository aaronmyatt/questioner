---
id: TASK-20
title: 'Author non-code prompt variants (notes, research, content)'
status: To Do
assignee: []
created_date: '2026-05-26 12:12'
updated_date: '2026-05-28 10:21'
labels:
  - dev
  - v0.1
  - agnostic
dependencies:
  - TASK-19
priority: low
ordinal: 20000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Once --kind exists, the prompt heredoc needs to branch. Keep this LIGHT - the user asked for 'simple guidelines to steer the model', not a template engine. Re-use the existing structure (constraints / negative examples / output contract) and only swap the kind-specific lines.

Three variants to author:

  notes/brain-dump corpus — emphasize:
    - cross-document threads picked up and dropped
    - contradictions or evolving positions across notes
    - recurring concerns that never got resolved
    - unanswered questions the author asked themselves
    - explicit non-goal: do NOT ask about file structure or tags
    Negative example to add: 'What is in note X?' (answered by reading it).

  research corpus — emphasize:
    - methodology choices and their implications
    - citation gaps or single-source claims
    - contested claims across sources
    - hypotheses stated but not tested
    - explicit non-goal: do NOT ask for summaries of individual papers
    Negative example: 'What does paper X argue?' (a summary, not exploration).

  website/marketing content — emphasize:
    - voice and positioning consistency across pages
    - implicit audience assumptions
    - structural/navigation gaps
    - claims that lack supporting evidence on-site
    - explicit non-goal: do NOT propose SEO keywords or copy edits
    Negative example: 'How should we rewrite the homepage?' (prescriptive, not exploratory).

Keep the output contract identical across kinds (numbered list, files cited, TOP N line) so consumers do not have to branch on kind. The 'files:' field still works for any corpus - it just points at .md notes or .html pages instead of .ts files.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Prompt branches on KIND with kind-specific constraints and negative examples
- [ ] #2 Output contract (numbered list + TOP N) is identical across all four kinds
- [ ] #3 Each non-code variant has at least one WRONG and one RIGHT negative-example pair
- [ ] #4 Manual test: run questioner --kind notes on an Obsidian-style folder; top 5 read as notes-shaped, not code-shaped
- [ ] #5 Manual test: run questioner --kind content on a Jekyll/Hugo content folder; top 5 read as content-shaped
- [ ] #6 Manual test: existing code-repo output is unchanged from v0 (regression check)
- [ ] #7 Each kind variant inherits the tier structure from TASK-22 (tier budget + cross-tier TOP-N selection rule); tiers may be re-labeled per kind but the spread requirement is preserved
<!-- AC:END -->
