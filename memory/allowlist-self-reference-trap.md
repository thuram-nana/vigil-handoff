---
name: allowlist-self-reference-trap
description: "An allowlist computed from the SAME untrusted lookup as the target authorises itself — VIGIL's private-tier egress lift was vacuous for every exact charter entry"
metadata:
  node_type: memory
  type: project
---

Found 2026-09-27 red-penning VIGIL's `live/egress_floor.py` (branch strix/s2-transport-intake). Generalises
to any allow/deny decision that compares a resolved value against a set derived from the same resolution.
See [[runtime-redrive-live-false-fact]], [[vigil-fusion-redamon-pentagi]] (the ONE-shared-helper lesson).

## THE TRAP
`pin_egress` resolves the target, then lifts the RFC1918/loopback tier if the resolved IP appears in
`allowed_ips`. `allowed_ips` comes from `charter_allowed_ips` → `CharterScopeSource.resolved_allowed_ips`
(`gateway/vigil_gateway/scope_source.py:136-158`), which **resolves every non-wildcard charter scope entry
AT REQUEST TIME**. The signed charter carries a NAME; the IP comes from a DNS answer taken milliseconds
before the target's own. **When the target IS the scope entry — the overwhelmingly common case — both
sides of the comparison are the same DNS query, so the lift ALWAYS fires.** The docstring's claim of an
"exactly-charter-authorized IP" is really an *exactly-DNS-asserted* IP.

Measured: charter lists `staging.example.com` exactly, resolver returns `10.0.0.5` →
`DIALLED=[('10.0.0.5',445)]` and a **signed `service_reachable` FACT about an internal host**. And with NO
resolver compromise at all: an exact-scoped dual-homed / round-robin name resolving to
`[10.0.0.5, 93.184.216.34]` puts BOTH in the lift set and the connect deterministically takes
`addresses[0]` — the internal one.

**Rule: an allowlist must be pinned in the SIGNED artifact (an IP literal), never re-derived at decision
time from the same untrusted source as the thing being checked.** A name in a signed charter authorises
the NAME; it cannot authorise whatever address a resolver returns for it today.

## WHAT HELD, AND WHY THE DISTINCTION MATTERS
The **Tier-1 floor (metadata/link-local) is real** and survived ~25 spellings WITH the charter explicitly
listing the address: `169.254.169.254` bare/trailing-dot/whitespace/bracketed/`%zone`, decimal
`2852039166`, octal `0251.0376.0251.0376`, hex `0xa9fea9fe`, `::ffff:169.254.169.254`, `::ffff:a9fe:a9fe`,
6to4 `2002:a9fe:a9fe::`, NAT64 `64:ff9b::…`, RFC-8215 `64:ff9b:1::…`, `fd00:ec2::254`, `fe80::1%eth0`,
`169.254.170.2`, `0.0.0.0`, `224.0.0.1`, `192.0.0.8`, `198.18.0.1`. **Zero leaks.** It works because it
classifies the RESOLVED value against a closed structural rule and nothing can lift it.
**The difference: Tier-1 is a property OF the address; the private-tier lift is a comparison against a set.
A structural predicate cannot be self-referenced; a derived set can.**

## COMPANION FINDINGS FROM THE SAME REVIEW
- **The class-bug reached SEVEN sites, not four.** After a shared helper guarded four, a wider grep found
  two integration sites with NO floor at all (`remediation/live_adapter.py:221`,
  `remediation/differential_adapter.py:292` — the remediation-VALIDATION adapters, which run on
  operator-triggered re-tests), plus two that run the floor but pass the **NAME** to the capture so the OS
  re-resolves at connect (`external_tool.py:631,816`) — reopening the exact rebinding window the pin
  exists to close. **After writing a shared helper, grep for the PATTERN (resolve-then-connect), not for
  the helper's name.**
- **A dead footgun must be made to raise, not just left uncalled.** `live/dns_pin.py:resolve_and_validate`
  is charter-relative, now called by nothing, and named like the safe thing — one import away from the
  next author. That is precisely how this bug reached seven sites.
- **An untested contract step is not a contract.** Mutation `if False and address_authorized is not None:`
  killed **zero** tests across five suites, and two of the four guarded sites never pass the predicate at
  all — so "every resolved address must be in scope" was absent there while the shared docstring presented
  it as the contract.
- **A derived-from-source test pin can silently cover less than a hand-written one.** A pin that builds
  `alias_keys` via `getattr(mod, f"_{ARM}_ALIASES", None)` silently DROPS an arm whose map is named
  differently — a third arm carrying a shadowing alias passed the disjointness test. Derivation must
  assert that every arm contributed.
- **Per-request re-derivation is an OPSEC signature, not just latency.** Re-resolving the whole charter
  scope per request measured ~15,000 DNS queries per scan (+15 s at 1 ms/query, +749 s at 50 ms) against
  the operator's resolver and the in-scope domains' authoritative servers. `make gate` is loopback with a
  1-entry charter and hides it entirely. Cache per (slug, charter mtime).

**Recurred 2026-11-19 (investigation-graph S1, evidence resolver):** `resolve_and_verify` pinned authenticity to the bundle's OWN shipped `trust-root.json` + `TRUST-ROOT-FINGERPRINT.txt` — both attacker-writable — instead of the configured governance root (`entitlement.store.load_trust_root()`). An attacker-signed bundle with a matching sidecar read back `ok=True authentic=True`. The build agent had even noted the in-bundle default as an "honest posture note" and shipped it anyway; only the adversarial red-pen BLOCKED it. Rule: an authenticity pin is NEVER sourced from the artefact being authenticated; with no configured/out-of-band pin, fail closed.
