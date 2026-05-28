# Questioner

**Status:** v0 design + initial implementation
**Date:** 2026-05-26
**Origin:** Spun out of a Waystation cold-start discussion. The goal here is deliberately narrower than Waystation: surface ~5 high-signal exploration questions about any repository, and stop there. Anything downstream (file:line tours, commentary, flow authoring) belongs to whoever consumes the questions, not to this tool.

---

## 1. Hypothesis

> Given a repository bundled into a single context blob and a well-shaped prompt with a justification clause, a modern LLM can surface ~25 specific, repo-grounded exploration questions — and the top 5 of those are often genuinely useful to new and existing developers.

The hypothesis is testable by running it against repos where we already know what's interesting and seeing if the output overlaps with that.

---

## 2. The minimal test we ran

1. Picked `jira-sprint-metrics-app` (PocketBase + Mithril, small-medium, rich in gotchas).
2. Bundled it with **repomix** (https://github.com/yamadashy/repomix) into a single XML blob.
3. Prompted the agent to surface the top 25 exploration questions, focusing on complex logic, unusual patterns, and high-churn files.

Two passes:

| Pass | Context | Outcome |
|---|---|---|
| 1 | Full repo (including `CLAUDE.md`, `backlog/`, types file) | Strong output. Some questions clearly re-surfaced curated prose. |
| 2 | Excluding `CLAUDE.md`, `backlog/`, big types file | Still strong. Leaned harder on code patterns — Goja reflect bridge, view introspector quirks, `DELETE`+bulk insert vs upsert, `dueDateTolerance` clamping, the `=` vs `?=` PB rule operator bug. |

**The ablation result was the important one.** Pass 2 demonstrated the system is not purely reciting prose; meaningful questions surface from code and migration history alone. The cold-start story holds up at small-repo scale. The remaining unknown is how it scales (in repo size and in repos with zero curated docs); that is the next experiment.

---

## 3. What changed from the original Waystation design

The discussion narrowed an originally elaborate question-surfacing design (`§4–§7` of the Waystation doc) into this small tool. Specifically:

| Decision | Status |
|---|---|
| Elaborate Phase 1 signal-gathering (TODO greps, hot-file scoring, coupling-hub detection, comment density spikes…) | **Dropped.** Repomix already bundles the repo. The signal pipeline only earns its keep when repos exceed the context window. Held in reserve as a fallback (see §7). |
| Justification clause in the prompt | **Kept.** Highest-leverage piece of the original design. |
| Over-generate then rank | **Kept.** Generate 25, present 5. |
| Intent-tag bucketing (onboarding/debugging/extending/reviewing) | **Dropped.** Pass-2 output didn't suffer from clustering. Tags add UX complexity for unclear gain. |
| "None of these" free-text regen loop | **Dropped.** User can rerun the tool freely. |
| Multi-pass triangulation | **Designed, not yet implemented (§7).** Cheap noise-reducer. |
| Presupposition verification | **Designed, not yet implemented (§7).** Cheap slop-reducer. |
| Line numbers / file:line citations / "first flow step" output | **Explicit non-goal.** Belongs to consumers of the question stream, not to this tool. |
| Caching, scheduling, weekly regen | **Out of scope.** Reruns are user-initiated. |

The operational stance is: simple, on-demand, agnostic of consumer, agnostic of LLM provider.

---

## 4. Tool shape

Two artifacts:

1. **`questioner`** — bash script. Bundles a repo via repomix, builds the prompt, either invokes a user-chosen agent CLI or emits the prompt+context for the user to pipe manually.
2. **`SKILL.md`** — agent-facing instructions, agnostic of provider. Tells an agent (Claude Code, pi, llm, any other) when to invoke the script, which mode to use, and how to present the output.

### 4.1 Script modes

| Mode | Behaviour | When to use |
|---|---|---|
| **default** | Bundle → invoke `$QUESTIONER_AGENT` via stdin → print agent's output | Shell or CI workflows with an LLM CLI available |
| `--context-only` | Print bundled repo context to stdout. No prompt, no agent call. | An agent already in conversation with the user, that just wants fresh context to reason over |
| `--prompt` | Print full prompt + context to stdout. No agent call. | Any other tool — pipe into anything by hand |

The two non-default modes keep the script useful when no LLM CLI is installed, in restricted environments, or when an existing in-IDE agent (Claude Code, Cursor, Zed Assistant) wants to do its own reasoning rather than spawn another CLI.

### 4.2 Knobs

| Flag | Default | Purpose |
|---|---|---|
| `--agent CMD` | `$QUESTIONER_AGENT` or `claude -p` | Which agent CLI to pipe to. Must accept the prompt on stdin. |
| `--passes N` | 1 | Run question generation N times. With N>1, only questions appearing across runs survive *(planned, v0 single-pass only)*. |
| `--verify` | off | Second pass that checks each candidate's presuppositions against the code; flag unverifiable questions speculative *(planned, v0 off)*. |
| `--count N` | 5 | Final questions presented to the user. |
| `--candidates N` | 25 | Raw candidates the LLM is asked to produce per pass. |
| `--exclude PATTERN` | none | Passed through to repomix `--ignore`. Repeatable. |

#### Why these knobs

- **`--candidates 25, --count 5`** keeps the LLM in a regime where it has room to think. Asking for exactly 5 in one shot produces flat, over-pruned output; asking for 25 and post-filtering gives consistently better top picks. The funnel happens inside a single LLM call to avoid a second round-trip.
- **`--passes`** is the cheap form of noise reduction. A question only one run surfaces is plausibly noise; questions appearing across runs are robust signal.
- **`--verify`** addresses the specific risk that generated questions encode claims about the code ("X always happens before Y") that the LLM cannot actually substantiate. A second pass asks the model to locate the supporting line(s); if it can't, the question is flagged speculative.

### 4.3 The prompt

The prompt is a quoted heredoc inside the bash script (delimiter `<<'PROMPT'`; placeholders `{{CANDIDATES}}` / `{{COUNT}}` are post-substituted via `sed`, since a quoted heredoc disables `${var}` expansion). Quoting is mandatory: the prompt body uses backticked code references like `` `EventStore.events` `` that an unquoted heredoc would execute as commands.

Four load-bearing parts, in order:

1. **Tier definitions.** The candidate set must span three tiers, each with a target proportion of the total. The tiers are mutually constraining — each tier defines what shape its questions must take *and* what they must avoid:
   - **Tier A — Purpose & surface** (≈1/5). Names a specific surface-level file/command/symbol; answerable in 2–3 minutes by reading ONE file; does NOT require multi-file synthesis. Opens the project to a complete newcomer.
   - **Tier B — Architecture & seams** (≈2/5). Spans 2–4 files; reveals a structural decision visible directly in the code; a reader can chew on the answer in 5–10 minutes. Hands the reader a usable mental model.
   - **Tier C — Deep cuts & non-obvious decisions** (≈2/5). Synthesizes across 3+ files OR encodes a non-obvious cross-cutting concern (lifecycle, ordering, TTL, idempotency, runtime quirks); worth a guided walkthrough rather than just reading the file — generic questions cannot pass this bar; a reader needs 15–30 minutes to really sit with.
2. **Negative + positive examples per tier.** Each tier carries one WRONG and one RIGHT concrete example with reasons. Belt-and-braces over the tier definitions — negative examples define the floor, positive examples define the ceiling. LLMs internalise rules from examples more reliably than from prose constraints.
3. **Output contract.** Numbered list with tier label first (`N. [A] question text`), followed by a `TOP <N>:` line. Tier labels are mandatory; if a question fits no tier cleanly, label it `[C]`. Strict format makes downstream consumption — and the spread rule below — auditable.
4. **TOP-N spread rule (strict).** The headline picks MUST include at least one Tier A, at least one Tier B, and at most two Tier C. Pick for breadth of the mental map first, depth of insight second. This rule is what prevents the headline output from drifting into five deep-cut walkthroughs even when individual Tier C questions score highest — see the calibration history with `~/Development/scratchpad` for the symptom this defends against.

The prompt does not ask for line numbers, flow steps, intent tags, or commentary. Those are not this tool's job.

### 4.4 The baseline workflow (a maintainer tool, not an end-user feature)

**Scope.** `--baseline` and its sibling subcommands (`--baseline-list`, `--baseline-show`, `--baseline-rate`) exist to evaluate and iterate on questioner's *own prompt*. They are maintainer tooling, not part of the end-user surface. The reason is structural: the comparison axis is the prompt version (a SHA of the heredoc body), and end users never edit the prompt — it's baked into the script. For an end user, every run carries the same SHA, so a better/worse/same verdict would only be measuring the LLM's run-to-run nondeterminism on an unchanged prompt, which is close to noise. The value accrues entirely to whoever is tuning the prompt.

Keeping the workflow maintainer-scoped also preserves the **stateless operational stance** the tool commits to elsewhere (see §7.3): a default end-user run writes no persistent state and reads no preferences. State is created only when a maintainer explicitly invokes a `--baseline*` command, and it lives *inside the repo being evaluated* (see Storage below), so it versions with that codebase rather than accumulating in a hidden global directory. The flags stay in the one script (already built and tested) but are documented under a "prompt development" heading in `--help` and kept out of the headline usage docs.

`--baseline` wraps a normal `--invoke` run with a *pairwise* rating step against the current champion for the same repo. The model deliberately rejects absolute scoring (0–10, A–F): scores drift over time and "7/10 six months ago" doesn't measure the same thing as "7/10 today." Pairwise verdicts hold the comparison target fixed, so each verdict is a stable directional claim. This is the same reason Elo, ranked-pairs, and RLHF reward models all use pairwise human judgement instead of absolute scales.

**Verdict vocabulary.** Four values: `better`, `worse`, `same`, `skip`. Only `better` (or the inaugural run) promotes the new run to champion; `worse` and `same` record the verdict but leave the champion in place. `skip` records the run without rating it, so a busy user can capture now and rate later (`--baseline-rate --verdict ...` from within the repo).

**Prompt versioning.** Each run is tagged with the first 12 hex chars of `sha256(heredoc-body)`. The body is extracted with `awk` between the `<<'PROMPT'` opener and the `^PROMPT$` closer, so unrelated edits to the script (argparse tweaks, comment fixes, new flags) do *not* bump the version. The version moves iff the prompt itself moves. Implementation: see `prompt_body_sha()` in the script.

**Storage.** Repo-local. Each evaluated repo gets a `<repo>/.questioner/` directory containing a single `baseline.json` plus a `runs/` subdir holding the full agent output for every recorded run as `<prompt-sha>-<timestamp>.txt`. Because the store lives inside the repo, it is meant to be committed alongside the code — the baseline corpus travels with the codebase it describes. Default location is `<repo>/.questioner`; override with `--baselines-dir DIR` or `$QUESTIONER_BASELINES_DIR` (e.g. to point at a scratch location, or to deliberately recreate a single shared store). Since each store is single-repo, run files and run ids no longer carry a slug prefix — the containing directory identifies the repo. The JSON schema (sketch):

```json
{
  "repo": "scratchpad",
  "repo_path": "/Users/aaronmyatt/Development/scratchpad",
  "champion": { "run_id": "<prompt-sha>-<timestamp>", "promoted_at": "..." },
  "runs": [
    { "id": "<prompt-sha>-<timestamp>", "timestamp": "...", "prompt_sha": "...",
      "agent": "...", "candidates": 25, "count": 5,
      "verdict": "better|worse|same|skipped|inaugural",
      "compared_against": "previous-run-id-or-null",
      "notes": "...", "output_file": "runs/<prompt-sha>-<timestamp>.txt" }
  ]
}
```

All baseline subcommands (`--baseline`, `--baseline-list`, `--baseline-show`, `--baseline-rate`) key off the repo `[path]` argument (default `.`); none take a slug, since there is one store per repo. Writes are atomic: every update goes to a same-dir tempfile and is moved into place, so a Ctrl-C mid-write can't leave the champion pointing at a not-yet-recorded run.

**Why this works as both eval and historical record.** The chain of `better` verdicts forms a monotone improvement record — each champion provably beat its predecessor in pairwise judgement. `worse` and `same` verdicts are equally useful eval signal: they diagnose changes that didn't help. Trade-offs across repos surface naturally: a prompt edit that scores `better` on one repo and `worse` on another tells you exactly that, instead of being averaged away. The trade-off is that you lose magnitude — you know a change moved the needle, but not by how much. That price was deemed acceptable in exchange for noise-resistance over the long iteration timeline this workflow is meant to support.

**Per-tier verdicts are a designed-not-shipped extension.** Same data model with a tier dimension (one verdict per tier). Deferred until single-overall verdict has been used for real iterations and a need for per-tier resolution actually surfaces.

#### Why tiers rather than a flat constraint set

The pre-tier prompt had a single hard constraint (synthesis across ≥3 files or a non-obvious cross-cutting concern) and a "what an experienced developer would point a newcomer at" framing. Both biased structurally toward Tier C: the synthesis constraint *disqualifies* every Tier A question by construction; the framing selects for what is surprising to someone who already knows the project. The output was high-quality on its own terms but stacked at one depth, which made it useless for the actual cold-start use case the tool is named for. Tiers fix the bias by making different shapes legitimate first-class outputs rather than allowing one shape to monopolize the set.

---

## 5. Usage

```bash
# Default — bundle current dir, ask claude -p for top 25, print top 5.
questioner

# Different agent (anything that reads prompt on stdin).
QUESTIONER_AGENT="llm -m gpt-4o" questioner ~/code/my-repo

# Just bundle the repo, give me the context (for an in-IDE agent).
questioner --context-only > /tmp/repo.xml

# Just give me the full prompt; I'll route it myself.
questioner --prompt | pbcopy

# Exclude vendored or generated stuff that bloats context.
questioner --exclude 'vendor/**' --exclude 'dist/**' --exclude '**/*.lock'

# Ask for a bigger funnel (e.g., a large repo with rich surface area).
questioner --candidates 40 --count 7
```

---

## 6. Files

- `DESIGN.md` — this document
- `questioner` — the bash script (executable)
- `SKILL.md` — agent-facing instructions

---

## 7. Enhancements worth envisioning

Ranked roughly by leverage relative to implementation cost. The first three sit in v0's near-future scope; the rest are speculative.

### 7.1 High-leverage, low-cost (likely v0.1)

#### Multi-pass with deliberately varied emphases
Today `--passes N` is planned as N runs of the same prompt. A small extension: each pass emphasises a different angle — git churn, static structure, prose docs, test names — so the union of passes covers the repo from different vectors. Questions appearing across angles are *extra*-robust signal. Implementation: add `--bias <angle>` to single-pass mode, and have multi-pass mode cycle through angles internally.

#### Presupposition verification (`--verify`)
Many generated questions encode factual claims about the code ("Phase 1 always bumps `lastSyncedAt`"). If the model is wrong, the question misleads. A second LLM pass asks: *"For each presupposition in this question, locate the line(s) supporting it. If you cannot, mark the question speculative."* Cheap single-call addition. Probably the single biggest slop reducer available.

#### Calibration suite
Bundle 2–3 known repos with "blessed" question sets (questions a thoughtful human authored after deep exploration). Running `questioner --calibrate` produces fresh questions on the bundled repo and shows overlap with the blessed set. Lets users gut-check whether the tool is working in *their* environment before trusting it on novel repos. Doubles as a regression test when iterating on the prompt.

### 7.2 Medium-leverage, medium-cost

#### Signal-augmented context for big repos
When repomix output exceeds the agent's context window, fall back to a curated bundle: top-N hot files (from `git log --since=90.days`), entry points (per-language heuristics or `aider`-style repo map), confusion markers (`rg TODO|FIXME`), import-graph hubs (`madge` for JS/TS, `pydeps` for Python). This is the original Waystation Phase-1 design, but invoked only as a fallback — never the primary path on small repos. The interface is a `signals/` directory of scripts, one per signal, each emitting JSON.

#### Per-language entry-point detectors
Pluggable interface in `signals/`: JS/TS first (route definitions, exported APIs, `package.json` bin entries), Python next (`__main__`, Flask/FastAPI routes, click entry points), Go (`main.main`, HTTP handler registrations), Rust (`fn main`, `pub fn` in `lib.rs`). One file per language; adding a language is a contained change.

#### Multi-agent jury
Generate questions with one agent, rank with another. Reduces single-model bias and lets you exploit cost/quality tradeoffs (cheap model generates 50 candidates, expensive model picks 5). Implementation: `--rank-agent CMD` distinct from `--agent CMD`. Output of the generator is fed to the ranker with an explicit "pick the 5 strongest, justify rejections" prompt.

#### History-aware mode
A `--since=14d` flag that biases generation toward files changed in the recent window. Same machinery, different framing: *"questions worth asking about what just shipped."* Useful for reviewers, tech leads, and post-merge retrospectives.

### 7.3 Lower-leverage, speculative

#### CI integration
Same engine, different prompt: *"questions a reviewer should consider on this PR diff."* Run as a comment on every PR. Tested-on-master before considering. Different product, same machinery.

#### Domain corpus mode
Run across N repos in a shared domain (a company's services, a polyglot monorepo) and look for cross-repo theme overlap. Useful for tech leads thinking about consistency or for spotting where one service deviates from a pattern the others share.

#### Diff mode
User reruns periodically; tool diffs against the prior question set and surfaces "new since last run." Becomes a change-radar. Skipped from v0 per user-led-rerun design; revisit if users start asking "what's different."

#### Negative-output mining via feedback
Capture thumbs-up/down on which of the 5 was useful; over time learn what to suppress. Requires storing feedback state. Skipped in v0 because the operational stance is stateless.

#### User exemplar steering (few-shot from vetted question sets)
Let an end user save question sets they rate highly and have questioner inject them as few-shot exemplars on future runs — *"produce sets like these."* Distinct from the maintainer baseline workflow (§4.4): baseline only *records and compares*, never feeds anything back into generation; steering would feed user-curated exemplars *into* the prompt to bias output toward a personal taste. The mechanisms are unrelated even though both involve storing question sets.

Deliberately deferred, not just unscheduled. It would make end-user questioner **stateful and preference-aware**, a direct fork from the current stateless/agnostic stance (the same stance that defers negative-output mining above). It also raises real questions worth designing through before any code: where exemplars live and how they're scoped (per-repo? global? per language?), how exemplar injection interacts with the tier budget (§4.3) without the exemplars simply overriding the tier balance, and whether a user's high-rated set from one repo should influence questions on an unrelated repo at all. Revisit only if users actually ask to steer output toward a saved taste — and treat it as a conscious change to the product's philosophy, not an incremental flag.

### 7.4 Explicit non-goals (will not be added)

- Line numbers, file:line citations, "first flow step" output. These belong to downstream consumers (Waystation; whatever else). Adding them to questioner would creep into specific product territory and break agnosticism.
- Persistent caching of question sets. Reruns are cheap and user-initiated.
- Intent tags (onboarding / debugging / extending / reviewing). Pass-2 output didn't cluster; tagging adds complexity for unclear gain.
- A built-in answerer. The tool produces questions, not answers. Answering is exploration; exploration belongs to the user or the user's agent.

---

## 8. Known limitations

- **Scale.** Repomix bundling works while the repo fits in the agent's context window. ~50k LOC comfortable; ~500k LOC needs the signal-augmented fallback (§7.2). The fallback is not yet built.
- **Curated-prose dependency.** Even after the pass-2 ablation, some questions clearly benefit from inline migration comments, descriptive function names, and small in-tree README files. Truly undocumented repos may produce thinner output. How much thinner is the next experiment.
- **Single-model bias.** All testing so far has been with one model family. Output character will shift with provider. The multi-agent jury enhancement (§7.2) is the planned mitigation; for now, treat output as one model's reading of the repo and adjust expectations accordingly.
- **v0 is single-pass, no verify.** The two highest-leverage enhancements (§7.1) are designed but not implemented. Single-pass output is already useful on the pilot repo, so v0 is honest about what it does today.

---

## 9. The next experiment

Before iterating further, run questioner on:

1. A repo with **zero curated docs** (no CLAUDE.md, no design notes, no architecture markdown). Measure whether output thins out or holds up. If it holds up, the borrowed-knowledge concern is dead. If it thins, the signal-augmented fallback (§7.2) becomes urgent rather than speculative.
2. A repo at the **500k LOC** scale. Measure where repomix breaks and what the fallback should produce.
3. The same repo at **3+ providers** (Claude, GPT, Gemini at minimum) with the same prompt. Measure whether the top 5 overlap meaningfully. If not, multi-agent jury (§7.2) moves up the priority list.

The output of those three experiments determines what v0.1 should be.
