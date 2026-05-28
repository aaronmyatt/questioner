---
id: TASK-3
title: Add --baseline better/worse/same workflow for prompt iteration
status: Done
assignee: []
created_date: '2026-05-26 02:53'
updated_date: '2026-05-28 10:12'
labels:
  - dev
  - v0.1
dependencies: []
priority: medium
ordinal: 3000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Replace 'blessed question sets' (TASK-3 original) and the abandoned 0-10 absolute scoring with a pairwise champion/challenger workflow. Each new run is rated against the current champion for the same repo: 'better' promotes the new run to champion; 'worse' and 'same' record the verdict but leave the champion unchanged. Inspired by Elo / ranked-pairs / RLHF reward models, where humans rate by comparison rather than absolute scale because comparative judgment is more stable over time.

Why pairwise instead of 0-10:
  • Absolute scores drift. '7/10' six months apart doesn't measure the same thing.
  • Pairwise verdicts are stable claims because the comparison target is held fixed.
  • Chain of 'better' verdicts forms a monotone improvement record - each champion provably beat its predecessor.
  • 'Worse' and 'same' verdicts are equally useful eval signal - they diagnose changes that didn't help.
  • Trade-off: loses magnitude (you know A>B but not by how much). User judged this an acceptable price.

Design choices settled:
  • Verdict values: better / worse / same / skip. Single keypress prompt (b/w/s/skip).
  • Optional one-line notes per verdict.
  • Storage: JSON, one file per repo at <baselines-dir>/<repo-slug>.json. Sibling runs/ subdir holds full question text per run as .txt files.
  • Prompt versioning: SHA256 of the heredoc body in build_prompt(), truncated to 12 chars. Extracted between heredoc delimiters so unrelated script edits don't bump the version.
  • Location: default $XDG_DATA_HOME/questioner/baselines (fallback ~/.local/share/questioner/baselines). Configurable via --baselines-dir flag or QUESTIONER_BASELINES_DIR env var.
  • Per-tier ratings are a designed-not-shipped extension - same data model, one verdict per tier. Defer until single overall verdict has been used for real iterations.

User experience for a single rating:

  $ questioner --baseline ~/Development/scratchpad
  [normal questioner output: 25 candidates + TOP 5]
  ...
  ─────────────────────────────────────────────────
  Current champion for 'scratchpad':
    prompt sha:    9876543210ab
    captured:      2026-05-25T14:00:00Z
    output:        ~/.local/share/questioner/baselines/runs/scratchpad-9876-...txt

  Open the champion output and compare to the run above, then enter a verdict:
    b = better  (this run beats the champion)
    w = worse   (champion still wins)
    s = same    (no meaningful difference)
    skip        (rate later)

  Verdict [b/w/s/skip]: b
  Notes (optional, single line): broader landscape, Tier A questions now present
  ─────────────────────────────────────────────────
  Recorded.
    -> NEW CHAMPION for scratchpad
       was: 9876543210ab (2026-05-25)
       now: a1b2c3d4e5f6 (2026-05-26)

First run for a repo: no comparison; tool announces 'no prior champion, this run becomes the inaugural champion' and stores it.

JSON schema:

  {
    "repo": "scratchpad",
    "repo_path": "/Users/aaronmyatt/Development/scratchpad",
    "champion": {
      "run_id": "scratchpad-a1b2c3d4-20260526T140000Z",
      "promoted_at": "2026-05-26T14:00:00Z"
    },
    "runs": [
      {
        "id": "scratchpad-a1b2c3d4-20260526T140000Z",
        "timestamp": "2026-05-26T14:00:00Z",
        "prompt_sha": "a1b2c3d4e5f6",
        "agent": "claude -p",
        "candidates": 25,
        "count": 5,
        "verdict": "better",
        "compared_against": "scratchpad-9876-20260525T140000Z",
        "notes": "broader landscape, Tier A questions now present",
        "output_file": "runs/scratchpad-a1b2c3d4-20260526T140000Z.txt"
      }
    ]
  }

  Notes:
    • verdict is 'inaugural' for the first run, 'better'/'worse'/'same'/'skipped' thereafter.
    • compared_against is null for inaugural runs and skipped runs.
    • champion pointer is updated only on 'better' verdicts.

Supporting commands (small to ship alongside; share the same JSON layer):

  --baseline-list                 List all repos with current champion (prompt sha + promotion date + total runs).
  --baseline-show <repo>          Print the current champion's full question text + verdict history (chronological with prompt SHAs and verdicts).
  --baseline-rate <repo> <verdict> Rate a previously skipped run without re-running the LLM (looks up the latest skipped run for that repo).

Out of scope for this task (designed-not-shipped):
  • Per-tier verdicts. Same data model with a tier dimension; defer until single overall verdict is in use.
  • Magnitude / confidence fields. Adds friction without proportional signal; revisit if pairwise alone proves insufficient.
  • Cross-repo aggregate views. Trade-offs across repos are best read individually, not averaged.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 questioner --calibrate runs against a built-in set of reference repos+blessed questions
- [ ] #2 Output shows overlap score and divergence per repo
- [ ] #3 Calibration corpus stored in a calibration/ folder with README explaining how to add new repos
- [ ] #4 Calibration corpus includes at least one non-code example (notes or research folder) with blessed questions tuned to --kind notes/research
- [ ] #5 questioner --baseline <path> wraps a normal run, prompts for a 0-10 score on /dev/tty after output, and records the result;Prompt SHA is computed from the heredoc body only (not the whole script), so unrelated edits do not bump the version;JSON file per repo records timestamp, prompt SHA, agent, candidates/count, score, notes, and a path to the run's question text;Run output is preserved verbatim in a sibling runs/ subdir so past baselines can be re-read without re-running the LLM;After scoring, the tool prints prev-best (for this repo) and tags the run as NEW BASELINE or 'did not beat';--baselines-dir flag and QUESTIONER_BASELINES_DIR env var override the default location;--baseline-list and --baseline-show <repo> commands operate on the JSON and let the user inspect the leaderboard;--baseline --score N --notes ... supports non-interactive scoring for scripted use;DESIGN.md gains a short section describing the baseline workflow as the empirical successor to the blessed-question-sets idea originally in §7.1
- [ ] #6 questioner --baseline <path> wraps a normal run, prints the current champion (if any) with a path to its output, prompts for a b/w/s/skip verdict on /dev/tty;Prompt SHA is computed from the heredoc body only (not the whole script) so unrelated edits do not bump the version;First run for a repo is stored as 'inaugural' champion with no comparison prompt;'better' verdict promotes the new run to champion; 'worse' and 'same' record the verdict but leave the champion unchanged;Every run is preserved verbatim in a sibling runs/ subdir so historical champions stay re-readable without re-running the LLM;--baselines-dir flag and QUESTIONER_BASELINES_DIR env var override the default location ($XDG_DATA_HOME/questioner/baselines, fallback ~/.local/share/questioner/baselines);--baseline-list and --baseline-show <repo> commands operate on the JSON and surface the champion + verdict history;--baseline-rate <repo> <verdict> retroactively rates the latest skipped run for that repo without re-running;--baseline --verdict b|w|s --notes ... supports non-interactive verdict entry for scripted use;DESIGN.md gains a short section describing the pairwise baseline workflow as the empirical successor to blessed question sets, naming pairwise rating as the deliberate choice over absolute scoring;Manual smoke test: capture inaugural baseline for $HOME/Development/scratchpad, then re-run with the same prompt and rate 'same' - champion unchanged, run recorded
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
FOLLOW-UP (2026-05-28): Storage relocated from $XDG_DATA_HOME/questioner/baselines/<slug>.json to repo-local <repo>/.questioner/baseline.json, in line with repositioning baseline as a maintainer/dev tool whose corpus versions alongside the codebase. Consequences: (1) one store per repo, so the per-slug JSON keying is gone — single baseline.json + runs/ per repo; (2) run ids and run filenames dropped the slug prefix (now <prompt-sha>-<timestamp>); (3) --baseline-show / --baseline-rate no longer take a slug arg — every baseline mode keys off the [path] positional (default '.'); (4) --baseline-list now summarizes the current repo's runs (champion marked) instead of enumerating repos across a shared dir; (5) --baselines-dir flag and QUESTIONER_BASELINES_DIR env remain as override escape hatches. DESIGN.md §4.4 and --help updated. Re-smoke-tested all flows + shellcheck clean.
<!-- SECTION:NOTES:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Pairwise baseline workflow shipped: --baseline (inaugural+better/worse/same/skip), --baseline-list, --baseline-show, --baseline-rate. JSON storage at $XDG_DATA_HOME/questioner/baselines/<slug>.json with runs/<id>.txt for full output preservation. Prompt SHA derived from heredoc body only. Full smoke tests pass.
<!-- SECTION:FINAL_SUMMARY:END -->
