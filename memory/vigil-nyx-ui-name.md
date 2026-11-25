---
name: vigil-nyx-ui-name
description: "Owner 2026-11-16 — the Strix agent is shown as \"NYX\" in the UI ONLY; nothing changes in the backend (modules, STRIX_* env, API values, signed \"strix.*\" ids, CI check names, vendor/strix)"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-16T15:25:11.701Z
---

Owner decision (2026-11-16): "nyx will be only in ui, nothing changes in backend."

**How to apply:** change only what a person SEES in a UI (labels, headings, buttons, tooltips, help text, rendered
agent names) from Strix to NYX. Never change identifiers the UI sends to or reads from the backend (API values like an
`agent: "strix"` field, route paths, CSS classes / element ids / data attributes the code keys on, persisted
`strix.*` finding kinds). Where the UI renders a backend-provided name, map it to "NYX" at display time in the UI layer.
The vendored upstream `vendor/strix` keeps its name and its Apache-2.0 NOTICE credit ("Strix — usestrix/strix").
Backend-only footprint measured 2026-11-16: 661 files / ~8,800 mentions, 59 STRIX_* env vars, the required check
`strix Claude-runtime (P8)`, ~20 signed `strix.*` ids — all deliberately untouched. Related: [[vigil-strix-shell-mcp-wave]].
