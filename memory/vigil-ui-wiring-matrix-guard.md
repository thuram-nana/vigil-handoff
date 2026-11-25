---
name: vigil-ui-wiring-matrix-guard
description: The no-dead-buttons/no-orphan-routes UI-wiring matrix guard — how new console routes must be registered + its golden regen
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T15:05:29.211Z
---

`integration/tests/test_ui_wiring_matrix.py` (runs in the **two-env boundary P5** CI job, sovereign venv) is a load-bearing guard every route-adding PR trips. It builds a model of UI calls vs registered backend routes and fails on a **dead button** (a `getJSON/postJSON(SOV|OFF("…"))` UI call with no matching registered route) or an **orphan route** (a registered route with no UI caller and not allowlisted).

**How routes are extracted (TEXTUAL, line-scoped — the trap):**
- `_PATH_EQ = path\s*==\s*"([^"]+)"` and `_PATH_IN = path\s+in\s+\(([^)]*)\)` + `_DICT_KEY`/`_PATH_SW`/`_FAPI`, each matched **per single line**.
- `_PATH_IN` needs the **closing `)` on the same line**. A `path in (` tuple split across lines registers only the literals on the trigger line; continuation-line literals are DROPPED.
- A route handled by a bare `else:` (no `path == "…"` of its own) is **never registered** → its UI call is a dead button.

**Rule for build agents / new routes:** register every route with an explicit `path == "…"` line (or a single-line `path in (…)` tuple with the closing paren). Never rely on `else:` or a multi-line tuple continuation to "cover" a path.

**Golden matrix:** `docs/capability-matrix/ui-wiring-matrix.json` is committed. Any button/route change drifts it; regenerate with `VIGIL_UIWIRING_REGEN=1 <sovereign-python> -m pytest integration/tests/test_ui_wiring_matrix.py`. Allowlists for genuinely headless routes: `INTENTIONALLY_HEADLESS` / `PENDING_UI_WIRING` (+ `_VERBS`), one line of reason each.

**First hit (#1195 session folders, 2026-11-19):** `/api/session/folder` was handled by `else:` inside a 2-line `path in (...)` tuple → dead button; the other 3 folder POSTs registered via their inner `path ==` elif lines. Fix = `else:` → `elif path == "/api/session/folder":` (behaviour-identical, `path` already ∈ the 4-tuple) + regen the golden. Local pre-merge runs (console/rbac/docs UI suites) did NOT cover this — always run `integration/tests/test_ui_wiring_matrix.py` + `test_two_env_boundary.py` before pushing a route-adding PR. Related: [[vigil-consumer-cascade-lessons]], [[vigil-crucible-core-gateway-importorskip]].
