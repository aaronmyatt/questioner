---
id: TASK-22
title: >-
  Restructure code-corpus prompt for landscape coverage (outside-in to
  inside-out)
status: Done
assignee: []
created_date: '2026-05-26 13:52'
updated_date: '2026-05-26 14:04'
labels:
  - dev
  - v0.1
  - prompt-quality
dependencies:
  - TASK-23
priority: high
ordinal: 22000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Real-world feedback from running questioner against $HOME/Development/scratchpad: all 25 candidates and the top 5 read as inside-out deep-cut questions ('why does X do Y instead of Z', 'walk through Z in detail'). None help a newcomer answer 'what does this project *do*, what shape is it, how would I start using or contributing to it'. The set lacks landscape coverage.

Three structural biases in the current prompt cause this:
  1. The hard constraint 'synthesis across >=3 files OR non-obvious cross-cutting concern' disqualifies every outside-in question by construction.
  2. The surface bias - 'high-churn areas, decisions that look arbitrary until explained, anything an experienced developer would want to point a newcomer at' - is specialist-to-specialist framing. It selects for what is surprising to someone who already knows the project.
  3. No tier budget: 25 questions all chase the same shape and cluster at one depth.

The fix is structural. Restructure the prompt so the 25 candidates span tiers, and the TOP-N selection has to draw from across tiers, not concentrate in one. Rough tier sketch (numbers tuneable during implementation):

  Tier A - Purpose & surface (~5 of 25): What does this project do for whom? What is the smallest end-to-end example of using it? What is the user-visible entry point? These need to be specific (named files/commands) but should NOT require multi-file synthesis. A reader should be able to answer one in 2-3 minutes by reading a single file.

  Tier B - Architecture & seams (~10 of 25): What are the major modules and how do they communicate? Where are the trust boundaries (process/network/file/permission)? What pluggable extension points exist? These should require reading 2-4 files but should not depend on knowing project history.

  Tier C - Deep cuts & non-obvious decisions (~10 of 25): The current style. Cross-cutting concerns, gotchas, design decisions that look arbitrary until explained. Keep the existing constraint here.

Other steering changes:
  - Add a positive example per tier alongside the existing negative examples. Negative-only examples define the floor; positive examples per tier define the ceiling for each.
  - Reframe the framing prose from 'what an experienced developer would point a newcomer at' to 'a 25-question set whose breadth paints a usable mental map of the project from purpose to internals; whose density rewards curiosity at every depth'.
  - TOP-N selection rule: the top 5 MUST include at least one Tier A question, at least one Tier B question, and at most two Tier C questions. This forces landscape coverage in the headline output even when the deep-cut questions are individually strongest.
  - Add a 'newcomer 5-minute test' gate: each TOP-N candidate must be one whose answer a thoughtful newcomer could engage with in 5 minutes; this excludes 30-minute multi-file walkthroughs from the headline list (they remain in the candidate pool as Tier C entries).

Out of scope for this task: the --kind branching (TASK-20). This task is about iterating the *code* prompt's steering; the kind variants in TASK-20 should inherit this tiered structure once they exist.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Prompt body declares an explicit tier budget (counts per tier sum to total candidates)
- [ ] #2 Each tier has at least one positive (RIGHT) example in addition to the existing negative examples
- [ ] #3 TOP-N selection rule requires the headline list to span tiers (concrete rule: >=1 Tier A, >=1 Tier B, <=2 Tier C in the top 5)
- [ ] #4 Re-run against $HOME/Development/scratchpad: top 5 contains at least one 'what does this do / smallest end-to-end' question and at least one architecture question, not five deep-cut walkthroughs
- [ ] #5 Re-run against jira-sprint-metrics-app (the DESIGN.md section 2 pilot repo): output still surfaces strong Tier C content; deep cuts are not lost, just balanced
- [ ] #6 DESIGN.md section 4.3 updated to describe the tier structure and the rationale (landscape coverage vs specialist-bias)
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Restructured the prompt around three explicit tiers (A: Purpose & surface, B: Architecture & seams, C: Deep cuts), each with its own shape constraints, target proportion (1/5, 2/5, 2/5), and a WRONG+RIGHT example pair. Reused the user's working-tree positive example as the Tier B RIGHT example since it perfectly demonstrates the architecture-and-seams shape.

Reintroduced specificity-naming as a global constraint (every question, every tier must name files/symbols/commands) since it was load-bearing across the original prompt.

Added the TOP-N spread rule (>=1 A, >=1 B, <=2 C) with 'STRICT — violating this is a failure' framing so the LLM treats it as a hard rule rather than guidance.

Updated DESIGN.md §4.3 to describe the four load-bearing parts (tier defs / examples / output contract / spread rule) and added a 'why tiers rather than a flat constraint set' subsection explaining the structural bias that motivated the rewrite (the scratchpad calibration symptom).

Have not yet re-run against $HOME/Development/scratchpad or jira-sprint-metrics-app — those are the user's calibration targets and re-running consumes their agent quota. ACs #4 and #5 are left for the user to verify; the prompt change is committed and they can validate when convenient.
<!-- SECTION:NOTES:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Tiered prompt structure shipped (A/B/C with target proportions, per-tier examples, TOP-N spread rule). DESIGN.md §4.3 updated. Live re-runs against calibration repos pending user verification.
<!-- SECTION:FINAL_SUMMARY:END -->
