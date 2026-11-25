---
name: vigil-onboarding-guide
description: "Owner 2026-11-16 — IN-APP onboarding guide (skippable first-login tour whose done/skipped answer is kept SERVER-SIDE per account, a Guide screen covering the whole app, VIGIL-wordmark loader); PR #1141 behind VIGIL_CUSTOMER_ONBOARDING (default OFF) until the screens it describes exist"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-16T22:48:17.015Z
---

Owner intent (2026-11-16, several messages): not a stand-alone page — a fully interactive guide INSIDE the customer
app, in the app's own theme, clicked through on the FIRST sign-in, with Skip; detailed and step by step, end to end
(incl. AEGIS and "how a GitHub account is connected / what Red-Pen does"); the loader is the VIGIL logo brightening
and dimming; then "server-side tracking for first login" and "the guide covers the app end to end, nothing left out".

Built: PR #1141 (`claude/customer-onboarding-guide`). `customer_app/guide.js` (8 chapters + Questions tab, 19-step
tour), wired in app.js; server side: edge `GET/POST api/customer/onboarding` → provision `onboarding_get/_set` →
`Provisioner.onboarding_state/_record` → `ui/onboarding_store.py` (`SIGIL_HOME/customer/onboarding/`, outside
backups). Re-posting is bounded (unchanged = no write; only the first close is audited). A screen census test fails
when app.js gains a route the guide does not cover; the Team step is pinned to `_CUSTOMER_GRANTABLE_ROLES`.

**Why default OFF:** the guide describes screens not built yet (launch/pause/retest UI, Team, Account security,
Monitoring, W6 billing views, Code/Red-Pen) — listed in HANDOFF §15.11 C. **How to apply:** keep the copy in step
as those land; add Max to the launch step only when L1 ships; copy rules are test-enforced (no model/engine names,
no prices or credit amounts, estimates only negated). Related: [[vigil-nyx-ui-name]], [[vigil-payment-provider-stripe]].
