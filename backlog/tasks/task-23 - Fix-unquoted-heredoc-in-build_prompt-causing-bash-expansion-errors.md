---
id: TASK-23
title: Fix unquoted heredoc in build_prompt() causing bash expansion errors
status: Done
assignee: []
created_date: '2026-05-26 13:52'
updated_date: '2026-05-26 14:04'
labels:
  - dev
  - bug
dependencies: []
priority: high
ordinal: 23000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Running questioner against $HOME/Development/scratchpad produced three bash errors before the LLM output:

    ./questioner: line 140: /tmp/sp: Permission denied
    ./questioner: line 140: EventStore.events: command not found
    ./questioner: command substitution: line 141: syntax error: unexpected end of file

These come from build_prompt()'s 'cat <<PROMPT' (line 140). An unquoted heredoc delimiter lets bash perform variable expansion AND command substitution inside the body. If anything in the prompt text or in a modified version of the script contains $(...) or backticks - or if the body has been edited to inline some computed content - bash tries to execute it.

Two fixes, choose one:

  (a) Quote the delimiter: change 'cat <<PROMPT' to 'cat <<'\''PROMPT'\'''. This disables all expansion. ${CANDIDATES} and ${COUNT} then need to be substituted in another way: easiest is a small sed pass on the heredoc output, e.g. 'cat <<'\''PROMPT'\'' | sed -e "s/{{CANDIDATES}}/$CANDIDATES/g" -e "s/{{COUNT}}/$COUNT/g"' with the placeholders rewritten in the template.

  (b) Keep the unquoted heredoc but defensively escape every $ and backtick in the body that is not an intended variable expansion. Brittle - any future edit to the prompt body can re-introduce the bug.

Recommendation: (a). The prompt body is fixed prose with only two variables; the indirection cost is tiny and the safety gain is large.

Also worth doing while in here: confirm whether the script in the working tree differs from the original. The error text ("EventStore.events", "/tmp/sp") looks suspiciously like content that could only appear if Scratchpad source was inlined into the heredoc, which would mean an accidental edit. Diff against git HEAD before applying the fix to make sure no intentional work is undone.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 build_prompt() uses a quoted heredoc delimiter; running questioner against any directory produces zero bash errors on stderr
- [ ] #2 ${CANDIDATES} and ${COUNT} still substitute correctly in the rendered prompt
- [ ] #3 A bats test (or shell-script smoke test) asserts that 'questioner --prompt' stderr is empty on a clean run
- [ ] #4 If the working-tree script had drifted from git HEAD, that drift is either committed intentionally or reverted before this fix lands
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Quoted the build_prompt() heredoc delimiter (<<'PROMPT'). Switched ${CANDIDATES}/${COUNT} to {{CANDIDATES}}/{{COUNT}} placeholders post-substituted via sed in the same pipe — preserves the inline-template ergonomics without re-introducing expansion.

Confirmed working tree had drifted from the initial design: a Tier-B-style positive example was added inline with backticked symbols (`/tmp/sp`, `EventStore.events`, `WindowController.show()`). Those backticks were the source of the 'Permission denied', 'command not found', and 'unexpected end of file' errors. Drift folded into TASK-22's restructure (the same example now lives as the Tier B RIGHT example).

Smoke tested with --prompt mode on a 2-file test directory and with non-default --count 7 --candidates 40: zero stderr output, both placeholders substituted correctly, backticks in the body render as literals.
<!-- SECTION:NOTES:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Fixed unquoted heredoc bug. Quoted delimiter + sed-based placeholder substitution. Smoke tests pass clean.
<!-- SECTION:FINAL_SUMMARY:END -->
