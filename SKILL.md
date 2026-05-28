---
name: questioner
description: Surface ~5 high-signal exploration questions for an unfamiliar repository, grounded in actual code rather than generic priors. Use when a user wants to orient in a new codebase or get starting points for exploration.
---

# Skill: questioner

Use this skill when a user wants to explore an unfamiliar repository, when they ask for a "tour" or "starting points", or when **you yourself** need orientation in a repo you haven't seen before and want better entry points than a directory listing.

## When to invoke

Invoke when:
- "What's interesting in this repo?"
- "Help me get oriented in `<repo>`."
- "What should a new developer ask about here?"
- "Give me a tour" / "show me the unusual stuff"
- You are about to do non-trivial work in a repo with no prior context, and want a focused starting point.

Do **not** invoke when:
- The user has a specific bug, feature, or task. This skill is for orientation, not problem-solving.
- The repo already has authored documentation/tours covering the user's question — read those first.
- The user is several turns deep in a focused exploration. Switching to question-surfacing mid-stream is a derail.

## Backing tool

A bash script: `questioner`. Typically at `~/Development/questioner/questioner`; may be on `$PATH`. Try `command -v questioner` to check.

The script bundles the repo via `repomix` and runs an LLM CLI against it. Three modes — pick based on whether you have an LLM CLI handy and whether you're already in conversation with the user.

### Mode A — you are already in a conversation with the user (preferred for in-IDE agents)

```
questioner --context-only [path]
```

Emits the entire bundled repo (XML, repomix format) to stdout. **Read it yourself**, then produce 25 candidates + top 5 inside this conversation using the prompt template shown below. This is the preferred mode for in-IDE agents (Claude Code, Cursor, Zed) because it keeps everything in the user's existing context — no extra CLI invocation, no extra spend.

The prompt template you should follow when given a bundled repo:

> You are surfacing the top **25** exploration questions a developer would want to ask about the repository bundled below.
>
> Every question MUST:
> - Name specific files/symbols from the bundle (no generic phrasing).
> - Require synthesis across ≥3 files OR a non-obvious cross-cutting concern (lifecycle, TTL, idempotency, runtime quirks, ordering invariants).
> - Carry a one-line justification answering: *"why is this question worth a walkthrough rather than just reading the file?"*
>
> Surface complex logic, unusual patterns, high-churn areas, decisions that look arbitrary until explained.
>
> After 25 numbered candidates (question / why / files), produce a `TOP 5:` line nominating the strongest five.

### Mode B — you have an LLM CLI available

```
questioner [path]
```

Default behaviour: bundle, prompt, pipe to `$QUESTIONER_AGENT` (default `claude -p`), print the response. Useful for shell-based or CI workflows.

### Mode C — you want the raw prompt for some other tool

```
questioner --prompt [path]
```

Emits the full prompt + bundled context to stdout. Pipe into any LLM CLI, paste into a web UI, etc.

## Useful flags

| Flag | Purpose |
|---|---|
| `--exclude PATTERN` | Repomix `--ignore` passthrough. Use for `vendor/**`, `node_modules/**`, big generated files, large lockfiles, etc. |
| `--count N` | Number of final questions (default 5). Bump to 7–10 for larger repos. |
| `--candidates N` | Raw candidate pool (default 25). Bump for bigger surface area. |

## What to do with the output

The output is two things:
1. **~25 numbered candidate questions** with one-line justifications and referenced files.
2. **A `TOP 5:` line** calling out the strongest five.

Present **only the top 5 to the user**, verbatim, with their justifications and file references. Do not paraphrase. Do not invent answers — these are starting points for *exploration*, not problems you've already solved.

If the user picks a question, switch into ordinary exploration mode: read the referenced files, follow the thread, answer based on what you actually observe. If you can't substantiate something, say so. The skill's value is destroyed by confident bluffing on the back end.

If the user says "none of these are what I want," ask what they actually want to know and then either rerun with that as guidance, or skip the skill and go straight to exploration.

## What this skill is NOT

- **Not a flow author.** It surfaces questions, not file:line bookmarks, intent tags, or commentary tracks. Downstream tools (Waystation and the like) own that.
- **Not a documentation generator.** It will not invent rationales for code it hasn't deeply read.
- **Not provider-specific.** Any LLM CLI that reads a prompt on stdin works.

## Failure modes to watch for

- **Borrowed knowledge.** If the repo contains rich prose docs (`CLAUDE.md`, `ARCHITECTURE.md`, design folders, `docs/`), generated questions may largely re-surface that prose rather than reason about code. The output is still useful — but credit the docs, not the model, and don't claim insight beyond what's grounded.
- **Small repo, thin output.** Tiny repos may not have 25 distinct questions; expect duplicates or weak candidates. Lower `--candidates`.
- **Big repo, context overflow.** If `questioner` exits with a context-too-large error, exclude bulky folders (`--exclude 'vendor/**' --exclude 'node_modules/**' --exclude 'dist/**'`). If still too large, the repo exceeds what repomix-bundling can do; fall back to manual exploration or a signal-driven approach (see `DESIGN.md §7.2`).
- **Presupposition risk.** Some generated questions encode factual claims about the code (e.g. "X always happens before Y"). Treat such claims as hypotheses to verify, not as facts. The planned `--verify` flag (DESIGN.md §7.1) will eventually do this automatically.

## Configuration

| Env var | Purpose |
|---|---|
| `QUESTIONER_AGENT` | Default LLM CLI invocation, e.g. `claude -p`, `pi -p`, `llm -m gpt-4o`. Must accept the prompt on stdin. |

## See also

- `DESIGN.md` — full design rationale, ablation findings, planned enhancements.
- repomix — https://github.com/yamadashy/repomix
