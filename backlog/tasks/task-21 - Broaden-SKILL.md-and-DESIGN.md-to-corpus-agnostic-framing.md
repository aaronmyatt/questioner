---
id: TASK-21
title: Broaden SKILL.md and DESIGN.md to corpus-agnostic framing
status: To Do
assignee: []
created_date: '2026-05-26 12:12'
updated_date: '2026-05-28 10:21'
labels:
  - docs
  - agnostic
dependencies:
  - TASK-19
  - TASK-20
priority: low
ordinal: 21000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
The two doc files currently lock the project to repos:
  - SKILL.md 'When to invoke' lists only repo-shaped triggers
  - DESIGN.md §1 hypothesis says 'a repository bundled into a single context blob'
  - Both treat 'developer + codebase' as the only user/object pair

Broaden the framing without rewriting from scratch.

SKILL.md changes:
  - Add notes/research/content directory triggers to 'When to invoke' (e.g. 'help me get oriented in this research folder', 'what questions does this set of notes raise?')
  - Document the --kind flag and when an agent should set it explicitly vs trust auto-detection
  - Use neutral 'corpus' or 'directory' where 'repository' is not specifically meant; keep 'repository' in the code examples
  - Add a 'failure mode' note: questioner is for *exploration*, not summarization - if a user wants a TL;DR of a notes folder, that is a different tool

DESIGN.md changes:
  - Insert a new short section (§3.5 'Scope: any organised corpus') explaining: bundling has always been agnostic; only the v0 prompt was code-specialized; --kind generalizes the prompt
  - Articulate WHY corpus-kind awareness is in scope while user-intent tags (onboarding/debugging/etc) are not: kinds describe the *material*, tags described the *user*. Pass-2 ablation showed material distinctions matter; tags did not.
  - Update §7.4 (non-goals) to clarify that corpus-kind awareness is NOT a non-goal (distinct from per-user intent tagging which remains a non-goal)

Keep both docs lean - this is positioning + a flag, not a rewrite.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 SKILL.md 'When to invoke' lists at least three non-code use cases
- [ ] #2 SKILL.md documents --kind and the auto-detection behaviour
- [ ] #3 DESIGN.md has a new short section articulating the corpus-agnostic stance
- [ ] #4 DESIGN.md §7.4 clarifies kinds-are-in-scope vs user-tags-out-of-scope distinction
- [ ] #5 No existing repo-focused examples removed - additions are additive
<!-- AC:END -->
