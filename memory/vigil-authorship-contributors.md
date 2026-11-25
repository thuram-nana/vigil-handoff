---
name: vigil-authorship-contributors
description: VIGIL repo attribution — since 2026-11-16 follow HANDOFF-SESSION.md §0.8 (session Co-Authored-By trailer on commits, "Generated with Claude Code" on PRs); old 2026-07-29 no-trailer rule superseded; history rewrite still pending
metadata:
  node_type: memory
  type: feedback
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-16T13:04:23.413Z
---

For the **vigil-sovereign** repo (`/home/kali/vigil`, GitHub thuram-nana/vigil-sovereign).

**Current rule (2026-11-16):** the user made `HANDOFF-SESSION.md` the single source of truth for the takeover,
and its §0.8 "Attribution (binding)" says: end every commit and PR body with the attribution lines the session's
own system reminder specifies (commit `Co-Authored-By:` trailer; PR "Generated with Claude Code" line), and put
**no model identifier anywhere else** in a pushed artifact (code, comments, PR titles, docs). Merged PRs
#1129–#1133 carry the trailer, so this is the live practice.

**Superseded:** on 2026-07-29 the user asked for NO `Co-Authored-By: Claude` trailer (sole-contributor wish);
PRs #173 onward omitted it until the 2026-11 cloud sessions. If the user restates the old preference, it wins
again — ask rather than guess when the two conflict.

**Why:** explicit user direction both times; the newer one (handoff as source of truth) governs now.

**How to apply:** take the exact trailer text from the current session's attribution reminder; add nothing it
leaves out (e.g. no `Claude-Session:` line unless the reminder gives one). Keep git authorship as the user's
canonical `Junior Thuram Nana` identity.

**Still PENDING (needs the user's explicit go-ahead — destructive force-push):** a history rewrite to strip old
trailers and remap the user's split identities (`Water Hacker`/`satoshinakamotobull@gmail.com`) → @thuram-nana.
Concern not resolved: the repo subtree-imported Strix (Apache/NOTICE) with full history (~440+ commits by real
third-party authors: `0xallam`, `bearsyankees`, `octovimmer`, `0xhis`) — never collapse them. Related:
[[vigil-ui-terminal-dossier-program]], [[vigil-wave-cadence]].
