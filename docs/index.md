---
title: Home
layout: default
nav_order: 1
description: "Surface high-signal exploration questions about any repository, grounded in its actual code."
permalink: /
---

# questioner
{: .fs-9 }

Surface a handful of high-signal exploration questions about any repository — grounded in its actual code, not generic priors.
{: .fs-6 .fw-300 }

[Get started](#30-second-quickstart){: .btn .btn-primary .mr-2 }
[View on GitHub](https://github.com/aaronmyatt/questioner){: .btn }

---

## The idea

> Given a repository bundled into a single context blob and a well-shaped prompt, a modern LLM can surface ~25 specific, repo-grounded exploration questions — and the strongest few are genuinely useful to new and existing developers.

Dropping into an unfamiliar codebase, the hardest part isn't reading any single file — it's knowing *which* questions are worth asking. `questioner` does one thing: it reads the whole repo and hands you those questions. It does not write code, author tours, or invent answers. It points; you explore.

## What the output looks like

`questioner` asks for questions across three tiers and forces the headline picks to span them, so you get a *map* of the project rather than five deep-dive walkthroughs:

- **Tier A — Purpose & surface:** what the project does, its entry point, the smallest end-to-end example.
- **Tier B — Architecture & seams:** the major modules, how they communicate, where the boundaries are.
- **Tier C — Deep cuts:** cross-cutting concerns and non-obvious decisions worth a guided walkthrough.

<!-- TODO(TASK-17): replace this illustrative block with a real captured top-5
     from a public repo (and link the full case study). Until a real run is
     captured, this is labelled illustrative so nothing here is mistaken for
     verified output. -->
> **Illustrative** (a real captured example is coming — see [examples](#)):
>
> ```
> 1. [A] What does running `questioner` with no arguments do, and which
>        block in the script prints the usage text?
> 4. [B] How does a bundled repo flow from repomix through build_prompt()
>        to the agent CLI, and where is the prompt assembled vs. piped?
> 7. [B] Where are the tool's trust boundaries — what does it shell out to,
>        and how is $QUESTIONER_AGENT split and executed safely?
> 12. [C] Why is the prompt heredoc quoted (`<<'PROMPT'`) and the variables
>        sed-substituted afterwards instead of expanded inline?
> 18. [C] How does the prompt-body SHA stay stable across unrelated script
>        edits, and why does that matter for the baseline workflow?
>
> TOP 5: 1, 4, 7, 12, 18
> ```

## 30-second quickstart

```bash
# 1. Install repomix (the bundler) and questioner.
npm install -g repomix
git clone https://github.com/aaronmyatt/questioner.git ~/Development/questioner
ln -s ~/Development/questioner/questioner /usr/local/bin/questioner

# 2. Tell it which LLM CLI to use (any that reads a prompt on stdin).
export QUESTIONER_AGENT="claude -p"

# 3. Run it on any repo.
cd ~/code/some-project
questioner
```

No LLM CLI handy? `questioner --prompt | pbcopy` gives you the full prompt to paste anywhere, and `questioner --context-only` emits just the bundled repo for an agent you're already chatting with.

See [Installation](install.html) for all the options and [Usage](usage.html) for the full flag reference.

---

## More

- [Installation](install.html) — dependencies, install paths, agent configuration
- [Usage](usage.html) — modes, flags, and recipes
- [Design rationale](https://github.com/aaronmyatt/questioner/blob/main/DESIGN.md) — why it works the way it does
- [Agent integration](https://github.com/aaronmyatt/questioner/blob/main/SKILL.md) — using `questioner` from Claude Code, Cursor, Zed, etc.
- [License](https://github.com/aaronmyatt/questioner/blob/main/LICENSE) — MIT © Aaron Myatt
