---
name: vigil-customer-kit-mirror-guard
description: Any edit to packages/vigil-ui tokens.css/components.css/ui.js must be mirrored into the customer app copy — test_customer_kit_copies.py fails SIGIL governor-gates + fold-cache CI
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T16:15:53.140Z
---

`apps/sigil/tests/test_customer_kit_copies.py` enforces that `apps/sigil/sigil/ui/customer_app/` vendors the console kit `packages/vigil-ui/`:
- `tokens.css` and `components.css` must be **byte-identical** copies ("copy the kit file over").
- `ui.js` is a **fork**: it may differ from the kit ONLY inside its two customer regions (request helpers); any line added outside them (e.g. new icons in the `P` icon map) must be added to the customer fork too, and `_exports` must match.
- `app.js` is NOT mirrored (the customer app is its own).

**Why it bites:** it runs in the SIGIL suite, so it reds BOTH "SIGIL governor gates (P7)" and "SIGIL suite with the fold cache ON" at once — which LOOKS like the known governor timing flake. #1195 (session folders, 2026-11-19): I added folder CSS + 4 icons to the kit and assumed "flake" from circumstantial evidence (main green, two correlated checks); the actual job log showed this guard. Lesson: never call a red check a flake until the job log names the failing test.

**How to apply:** after ANY kit edit, `cp packages/vigil-ui/{tokens,components}.css apps/sigil/sigil/ui/customer_app/` and transplant out-of-region ui.js lines (extract the block from the kit programmatically, don't retype), then run `apps/sigil/tests/test_customer_kit_copies.py`. Related: [[vigil-ui-wiring-matrix-guard]].
