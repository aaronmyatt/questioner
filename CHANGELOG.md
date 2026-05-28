# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-05-28

Initial public release.

### Added
- `questioner` bash script that bundles a repository via
  [repomix](https://github.com/yamadashy/repomix) and prompts an LLM CLI for a
  tiered set of exploration questions, then nominates a top-N.
- Three end-user invocation modes:
  - default (`invoke`) — bundle, prompt, pipe to `$QUESTIONER_AGENT` (default
    `claude -p`), print the response;
  - `--context-only` — print the bundled repo context to stdout for an in-IDE
    agent already in conversation;
  - `--prompt` — print the full prompt + context to stdout to route by hand.
- Tiered prompt that spans Tier A (purpose & surface), Tier B (architecture &
  seams), and Tier C (deep cuts), with a strict TOP-N spread rule so the
  headline output covers the project from the outside in rather than stacking
  deep-cut walkthroughs.
- Knobs: `--agent`, `--count`, `--candidates`, `--exclude`, `--version`.
- `SKILL.md` so any agent (Claude Code, Cursor, Zed, etc.) knows when and how to
  invoke the tool.
- Maintainer-only prompt-evaluation tooling (`--baseline`, `--baseline-list`,
  `--baseline-show`, `--baseline-rate`): pairwise better/worse/same rating of
  question-set runs against a per-repo champion, stored repo-locally under
  `<repo>/.questioner/`. Documented as a prompt-development tool, not part of
  normal end-user use (see DESIGN.md §4.4).

### Known limitations
- `--passes N` (multi-pass generation) and `--verify` (presupposition checking)
  are designed but **not implemented**; invoking either fails with exit code 64
  by design rather than silently ignoring the flag. See DESIGN.md §7.1.
- Bundling works while a repo fits in the agent's context window; the
  signal-augmented fallback for very large repos (DESIGN.md §7.2) is not built.
- All tuning to date has used a single model family; output character will shift
  with provider.

[Unreleased]: https://github.com/aaronmyatt/questioner/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/aaronmyatt/questioner/releases/tag/v0.1.0
