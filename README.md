# questioner

**Surface a handful of high-signal exploration questions about any repository — grounded in its actual code, not generic priors.**

`questioner` bundles a repo into a single blob (via [repomix](https://github.com/yamadashy/repomix)), asks an LLM for ~25 candidate questions that span the project from purpose to internals, and nominates the strongest few. It's a fast way to orient in an unfamiliar codebase — or to hand a newcomer better starting points than a directory listing.

```bash
# Bundle the current repo, ask your agent, print the top 5.
questioner

# Point it at another repo.
questioner ~/code/some-project

# No LLM CLI handy? Just emit the full prompt and route it yourself.
questioner --prompt | pbcopy
```

## How it works

The prompt asks for questions across three tiers, and forces the headline picks to span them:

- **Tier A — Purpose & surface:** what the project does, its entry point, the smallest end-to-end example.
- **Tier B — Architecture & seams:** the major modules, how they communicate, where the boundaries are.
- **Tier C — Deep cuts:** cross-cutting concerns, non-obvious decisions, the things worth a guided walkthrough.

This keeps the output a *map* of the project rather than five deep-dive walkthroughs. See [DESIGN.md](DESIGN.md) for the full rationale.

## Install

`questioner` is a single bash script. It needs [repomix](https://github.com/yamadashy/repomix) on your `PATH` and an LLM CLI that reads a prompt on stdin.

**Clone + symlink (current method):**

```bash
git clone https://github.com/aaronmyatt/questioner.git ~/Development/questioner
ln -s ~/Development/questioner/questioner /usr/local/bin/questioner   # or anywhere on $PATH

# Dependency: repomix (any one of these)
npm install -g repomix      # global install
brew install repomix        # if available on your platform
# or use it on demand: bunx repomix / npx repomix
```

A Homebrew tap and a `curl | bash` installer are planned but not yet shipped.

**Configure your agent** (any CLI that reads a prompt on stdin):

```bash
export QUESTIONER_AGENT="claude -p"        # default
# export QUESTIONER_AGENT="llm -m gpt-4o"  # or any other
```

## Usage

| Mode (mutually exclusive) | Behaviour |
|---|---|
| *(default)* | bundle → pipe prompt to the agent → print the response |
| `--context-only` | print the bundled repo context to stdout (for an in-IDE agent already in conversation) |
| `--prompt` | print the full prompt + context to stdout (route it yourself) |

| Option | Default | Purpose |
|---|---|---|
| `--agent CMD` | `$QUESTIONER_AGENT` or `claude -p` | agent CLI invocation (must read prompt on stdin) |
| `--count N` | 5 | final questions to present |
| `--candidates N` | 25 | raw candidates to generate |
| `--exclude PATTERN` | — | passed to `repomix --ignore` (repeatable) |
| `--version` | — | print version and exit |

`--passes N` and `--verify` are designed but not yet implemented; invoking them fails with exit code 64 by design (see [DESIGN.md §7.1](DESIGN.md)).

> **For maintainers:** `questioner` also ships a prompt-evaluation workflow (`--baseline*`) for iterating on the prompt itself. It's not part of normal use — see [DESIGN.md §4.4](DESIGN.md).

## Using it from an agent

[SKILL.md](SKILL.md) tells an agent (Claude Code, Cursor, Zed, any other) when to invoke `questioner`, which mode to use, and how to present the output.

## Documentation

Full docs, examples, and rationale: **https://aaronmyatt.github.io/questioner/**

- [DESIGN.md](DESIGN.md) — design rationale, tier structure, planned enhancements
- [SKILL.md](SKILL.md) — agent-facing instructions
- [CHANGELOG.md](CHANGELOG.md) — release history

## License

[MIT](LICENSE) © Aaron Myatt
