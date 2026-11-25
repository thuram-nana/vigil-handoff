---
name: vigil-engine-egress-floor-residual
description: "CLOSED — WS1.1 #874 floored capture paths; WS1.1b #879 the 3 raw-socket sites; G2 #919 the LAST site (http_executor httpx via SovereignHttpxTransport/_FloorPinnedBackend) WITH a gateway-less stdlib backstop that also fixed the #881 'floor too broad breaks loopback tests' park. Bakes the gateway-less-degrade pattern + 2 test-rigor traps."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-07T22:33:47.308Z
---

VIGIL's never-liftable engine egress floor (block metadata 169.254.169.254 / link-local / RFC1918 / CGNAT /
multicast / reserved — directly or via a DNS-rebinding answer) lives in two caller-invoked helpers (NOT a
global socket/getaddrinfo chokepoint): `integration/.../live/egress_floor.py::pin_egress` (integration
re-drive, the real-external-egress plane — ALWAYS has vigil_gateway, stays strictly fail-closed) and
`engine/crucible/framework/v2/common/egress_pin.py` (`resolve_and_pin` / `resolve_and_validate_all` — the
engine side; resolves ONCE, validates EVERY resolved IP via `vigil_gateway.denylist.is_egress_denied`, dials
the pinned IP, never re-resolves).

**ALL engine capture/egress sites now floored:** #874 (reachability/tls/reachability_cloud-check) · #879 (the
3 raw-socket sites: websocket CSWSH / race raw_race / smuggling raw_send+_desync_leg) · **G2 #919 the LAST one
— `agents/http_executor.py` httpx** via `SovereignHttpxTransport` + `_FloorPinnedBackend` (installed only on
the REAL egress path = when the caller injects no inner transport; resolve-once→validate-every-IP→dial-pinned,
Host/SNI preserved so a vhost still routes+validates its cert). G2 MERGED #919 → main 08f1b3f6 (2026-10-08); mergeStateStatus CLEAN.

**G2 also resolved the #881 park ("http_executor floor TOO BROAD → 32 clean-CI failures / 16-22 engine
tests fail-closed").** Root cause: flooring the central httpx path made it HARD-REQUIRE vigil_gateway, so on
the gateway-LESS CRUCIBLE-core engine plane every LOOPBACK scan fail-closed (`EgressFloorRefused`,
"no destination can be authorized"). The fix is the **gateway-less-degrade pattern** below.

### Load-bearing lesson: the gateway-less-degrade of a shared security control
A floor wired onto a path that runs in BOTH planes — gateway-present (offense/real-egress) AND gateway-less
(CRUCIBLE-core engine isolation, loopback-only by design) — must NOT hard-require the gateway (fail-closes
legit loopback) and must NOT fail-OPEN. The sound third option (`egress_pin.py::_stdlib_floor_denied`, G2):
when `vigil_gateway` is unimportable, DEGRADE to a pure-stdlib `ipaddress` **never-liftable backstop** that is
a strict structural subset of the authority — refuses metadata (incl. IMDSv6 `fd00:ec2::254` + embedded IPv4
`::ffff:169.254.169.254`/`::169.254.169.254`/`64:ff9b::a9fe:a9fe`), link-local, multicast, reserved/unspec,
RFC1918/CGNAT/ULA via `is_private/is_link_local/is_multicast/is_reserved/is_unspecified` + exact-CIDR mirror —
applying the SAME charter-literal/loopback lift, so loopback+public scans work while the critical tier stays
refused (GUARDED, never unguarded). It is NEVER consulted when the gateway IS present (full denylist = sole
authority). Loopback is EXCLUDED from Tier-1a (stdlib `::1.is_reserved` is True — would wrongly hard-deny a
scoped loopback target; the authority splits loopback out identically). The integration `pin_egress` is the
real-external-egress plane (always has gateway) → it correctly STAYS fail-closed; do NOT add the backstop
there. This is the [[vigil-crucible-core-gateway-importorskip]] DISARM principle made precise: degrade to the
SAFE structural floor, not fail-open, not fail-closed-break.

### Two test-rigor traps the G2 red-pen caught (both now closed in #919)
1. **The importorskip coverage trap.** A test guarded by `pytest.importorskip("vigil_gateway")` SKIPS in the
   exact gateway-LESS config a backstop exists to serve → the 165-line backstop had ZERO CI coverage (its
   correctness rested only on the red-pen's probes). FIX (`common/tests/test_egress_backstop.py`): the test
   FORCES the absent condition in-process (meta_path finder raising ModuleNotFoundError for `vigil_gateway*` +
   `sys.modules` eviction, restored in a fixture) so it RUNS in any leg (incl. CRUCIBLE-core, where it is most
   needed, NO importorskip) and asserts a distinctive `[stdlib backstop]` refusal marker that only the degrade
   branch emits — so it can never accidentally pass against the real authority. (`.venv-offense` has
   vigil_gateway editable-installed, so a PLAIN run masks the degrade branch — only the meta_path sim or CI is
   faithful; the recurring [[vigil-crucible-core-gateway-importorskip]] masking trap.)
2. **The hand-copied-mirror drift hazard.** A backstop MUST copy the authority's deny-set (the authority is
   unimportable in that plane → cannot reuse the one shared helper there). Without a committed guard, a future
   edit to `denylist._ALWAYS_DENY_*` silently makes the backstop LESS strict = a latent fail-open. FIX
   (`integration/tests/test_egress_backstop_mirror.py`, importorskip gateway → runs in the P5/gateway-present
   leg): pin `backstop-CIDRs == authority _ALWAYS_DENY minus {127.0.0.0/8, ::1/128}` AND behaviourally refuse a
   representative of EVERY authority always-deny net (so a future globally-routable always-deny entry not
   covered by the predicate fallbacks also trips it). Mutation-proven RED on both a dropped CIDR and an
   unmirrored authority entry.

### A third G2 fix — a CI guard's substring false-match
The P5 regression guard `test_the_charter_relative_dns_pin_is_removed...` grep'd the SUBSTRING
`resolve_and_validate` to prove the removed footgun `dns_pin.resolve_and_validate` has no callers — and
false-matched G2's new SOUND `resolve_and_validate_all` (+ its docstrings). Fix = tighten to word-boundary
`resolve_and_validate\b` (`_` is a word char, so `_all` no longer matches; a real `dns_pin.resolve_and_validate(`
revival still matches). General lesson: a "removed-footgun" guard must match the footgun PRECISELY, or a sound
sibling API trips it.

**Eval-lab belt regardless of the app floor:** enforce egress at the VM/cloud FIREWALL + disable IMDS /
require IMDSv2 hop-limit-1 per VM. The `.gov/.mil/.edu/.int` block (`vigil_core/hard_guardrail.py`, default
ON) is a HOSTNAME string match and by design does NOT catch IP literals — it never substitutes for the IP
floor. Do NOT move the denylist into vigil_core (breaks the stdlib-only gateway Docker image; tried+reverted).
Relates to [[allowlist-self-reference-trap]] [[vigil-gold-standard-program]] and the 17-org eval lab
([[vigil-live-external-program]]).
