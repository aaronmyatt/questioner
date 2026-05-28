---
title: Installation
layout: default
nav_order: 2
---

# Installation

`questioner` is a single bash script. It has two runtime dependencies:

1. **[repomix](https://github.com/yamadashy/repomix)** — bundles a repo into one LLM-friendly blob.
2. **An LLM CLI** that reads a prompt on stdin (e.g. `claude -p`, `llm`, `pi`).

---

## 1. Install repomix

Any one of these works:

```bash
npm install -g repomix      # global install
brew install repomix        # if available on your platform
bunx repomix --version      # no install — bun runs it on demand
npx repomix --version       # no install — npm runs it on demand
```

`questioner` does not auto-install repomix; it fails fast with an install hint if it's missing.

## 2. Install questioner

### Clone + symlink (current method)

```bash
git clone https://github.com/aaronmyatt/questioner.git ~/Development/questioner
ln -s ~/Development/questioner/questioner /usr/local/bin/questioner
# (any directory on your $PATH works)
```

Verify:

```bash
questioner --version
```

### Homebrew tap *(planned)*

A `brew install aaronmyatt/tap/questioner` path is on the roadmap but not yet shipped.

### `curl | bash` installer *(planned)*

A one-line install script is on the roadmap but not yet shipped. Until then, use the clone + symlink method above.

## 3. Configure your agent

`questioner` pipes its prompt to whatever `$QUESTIONER_AGENT` names (default `claude -p`). Any CLI that reads a prompt on stdin works:

```bash
export QUESTIONER_AGENT="claude -p"        # default
# export QUESTIONER_AGENT="llm -m gpt-4o"  # OpenAI via simonw/llm
# export QUESTIONER_AGENT="pi -p"          # or any other
```

Add the `export` to your shell profile (`~/.zshrc`, `~/.bashrc`) to make it stick. You can also override per-run with `--agent`:

```bash
questioner --agent "llm -m gpt-4o" ~/code/some-project
```

## Requirements summary

| Dependency | Why | Required for |
|---|---|---|
| `repomix` | bundles the repo | all modes that read a repo |
| an LLM CLI on stdin | answers the prompt | default mode only (not `--prompt` / `--context-only`) |
| `jq` | reads/writes the baseline store | maintainer `--baseline*` commands only |

Next: [Usage](usage).
