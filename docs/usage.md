---
title: Usage
layout: default
nav_order: 3
---

# Usage

```
questioner [options] [path]
```

`path` defaults to the current directory.

---

## Modes

The three modes are mutually exclusive. Pick based on whether you have an LLM CLI handy and whether an agent is already in conversation with you.

| Mode | Behaviour | When to use |
|---|---|---|
| *(default)* | bundle → pipe prompt to `$QUESTIONER_AGENT` → print the response | shell / CI workflows with an LLM CLI available |
| `--context-only` | print the bundled repo context to stdout — no prompt, no agent call | an agent already in conversation that just wants fresh context to reason over |
| `--prompt` | print the full prompt + context to stdout — no agent call | route into any LLM CLI by hand, or paste into a web UI |

```bash
questioner                              # default: top 5 for the current repo
questioner ~/code/some-project          # another repo
questioner --context-only > /tmp/repo.xml
questioner --prompt | pbcopy
```

## Options

| Option | Default | Purpose |
|---|---|---|
| `--agent CMD` | `$QUESTIONER_AGENT` or `claude -p` | agent CLI invocation (must read prompt on stdin) |
| `--count N` | 5 | final questions to present |
| `--candidates N` | 25 | raw candidates to generate (the funnel; more room to think) |
| `--exclude PATTERN` | — | passed to `repomix --ignore` (repeatable) |
| `--version` | — | print version and exit |
| `-h`, `--help` | — | show help |

### Not yet implemented

`--passes N` (multi-pass generation) and `--verify` (presupposition checking) are designed but not implemented. Invoking either fails with exit code 64 by design — the script refuses unsupported flags rather than silently ignoring them. See the design rationale, §7.1.

## Environment variables

| Variable | Purpose |
|---|---|
| `QUESTIONER_AGENT` | default agent CLI invocation |
| `QUESTIONER_BASELINES_DIR` | override the baselines storage location (maintainer tools) |

## Recipes

**Trim bulky or generated paths** so they don't eat the context window:

```bash
questioner --exclude 'vendor/**' --exclude 'dist/**' --exclude '**/*.lock'
```

**A bigger funnel** for a large repo with rich surface area:

```bash
questioner --candidates 40 --count 7
```

**In-IDE agent** (Claude Code, Cursor, Zed): emit just the context and let the agent reason in your existing conversation:

```bash
questioner --context-only > /tmp/repo.xml
```

**Different providers:** swap the agent per run to compare readings:

```bash
questioner --agent "claude -p"      ~/code/proj
questioner --agent "llm -m gpt-4o"  ~/code/proj
```

---

## Prompt development (maintainers)
{: .text-grey-dk-000 }

These commands are **not part of normal end-user use**. They exist to evaluate and iterate on `questioner`'s own prompt by rating whole question-sets across prompt versions. Since end users don't edit the prompt, there's little here for them. Full rationale: design doc §4.4.

Storage is repo-local: each evaluated repo gets a `<repo>/.questioner/` directory (a `baseline.json` plus a `runs/` archive), meant to be committed alongside the code.

| Command | Behaviour |
|---|---|
| `--baseline [path]` | invoke, then rate the run **better / worse / same** against this repo's current champion |
| `--baseline-list [path]` | list this repo's recorded runs (champion marked) |
| `--baseline-show [path]` | print this repo's champion output + verdict history |
| `--baseline-rate [path]` | retroactively rate this repo's latest skipped run (use with `--verdict`) |
| `--baselines-dir DIR` | override storage location (default `<repo>/.questioner`) |
| `--verdict V` | non-interactive verdict (`b`/`w`/`s`/`skip` or full words) |
| `--notes "TEXT"` | one-line note attached to the rating |

Only a `better` verdict (or the inaugural run) promotes a run to champion. Ratings are pairwise on purpose — a stable "is this better than the last best?" judgement, rather than an absolute score that drifts over time.
