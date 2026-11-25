---
name: shell-commit-message-backticks
description: "NEVER put backticks in a double-quoted bash string — git commit -m \"...`tool`...\" executes the tool. Use -F file."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-08-14T13:50:00.337Z
---

Backticks inside a **double-quoted** bash string are command substitution. Writing
`git commit -m "... nuclei's \`-disable-update-check\` ..."` does not quote the tool name — it
**runs the tool**.

This happened for real on 2026-08-14 while committing the egress-guard slice: the message contained
`` `nuclei` `` and `` `require` ``. Bash executed both. `nuclei` ran with no target, contacted
ProjectDiscovery, and **downloaded 84MB / 13,841 templates** to `~/.local/nuclei-templates` — an
outbound egress event on a host whose charter forbids external egress, in the very commit that added
the no-egress guard. It also silently **ate both words from the commit message**.

**Why:** the words most likely to be backticked in a commit message about tooling are *tool names* —
so this failure mode preferentially executes exactly the programs that scan, fetch, and egress.

**How to apply:** write every non-trivial commit message to a file with the Write tool and use
`git commit -F <file>` (or `--amend -F`). Never `-m` with a message containing backticks. The same
hazard applies to `gh pr create --body`, `gh issue`, and any `-m/--body` flag. If a message must be
inline, use single quotes — but prefer `-F`, which has no quoting hazard at all.

Related: [[vigil-authorship-contributors]] (this repo takes no `Co-Authored-By: Claude` trailer),
[[claim-discipline-meet-up]].
