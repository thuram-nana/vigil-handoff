# VIGIL vs ARTEX — End-to-End Gap Analysis

**PRIVATE working note. Do not commit. The repo is public; this file names unmerged branches and internal plans.**

Date: 2026-11-24. Scope: what VIGIL is missing or leaving out next to the public repo `BaaRai/ARTEX`. Built from the ARTEX inventory and the per-area comparisons (orchestration, recon/graph, evidence/traffic). The ARTEX zip was never opened; facts came from the public tree and the GitHub API only.

---

## 1. What ARTEX is (five lines)

- A self-hosted, model-agnostic autonomous penetration-testing platform. Tagline: "autonomous pentesting that keeps its receipts." AGPL-3.0, ~19k stars.
- Ships as one Go binary with an embedded Next.js UI; PostgreSQL is the only hard dependency.
- Runs a closed multi-agent loop: a planner splits a goal into intents, workers run real tools (Kali/Bash/HTTP) through a recording MITM proxy, and results are written back into two linked Postgres graphs — a shared asset graph (what exists) and a per-task exploration graph (what was tried).
- Findings are bound to the exact captured request/response that proves them, carry a proof-of-exploit level and verification status, and a retester agent re-runs them.
- Autonomy is bounded by an approval/interception gate, scope and target gates, token/turn budgets, a per-host noise governor, and a nonce-tagged untrusted-output boundary.

---

## 2. Where VIGIL is already stronger (and why)

VIGIL is not a thinner ARTEX. On the load-bearing axis — *is a claim true?* — VIGIL is more rigorous. ARTEX's own notes say "fact content itself is model-asserted." VIGIL never lets the model mint truth.

| Area | VIGIL advantage | Why it matters |
|---|---|---|
| Fact creation | A finding becomes a signed FACT only when a deterministic **oracle re-fires over retained target bytes** and the bug class is oracle-mapped; otherwise it is an honest LEAD. | ARTEX's `record_fact` / `prove_goal` are model-asserted. VIGIL structurally enforces "quantitative acceptance" instead of trusting a prompt. |
| Evidence | Bodies are content-hashed **and** the certificate is Ed25519-signed, hash-chained with anti-rollback, re-verified by re-running the oracle, and shippable to an **offline verifier**. | ARTEX binds bodies by hash under a DB lock. VIGIL gives an outside auditor a self-checking bundle. |
| Task structure | Goal tree + cursors are **checkpointed to disk** every 60s. | ARTEX's todolist is in-memory only; a crash loses the plan. |
| LLM failover | Failover never escapes the operator's **sovereignty tier**; redaction runs at the egress chokepoint before any provider round-trip. | ARTEX shares one provider with rate limiting but no tier binding. |
| Containment | Every tool call passes a conjunctive gate (authority → scope → destructive-confirm → budget → rate-limit → egress), fail-closed; the gateway drops everything by default at L3/L4. | ARTEX's Kali-Bash worker runs arbitrary shell. |
| Ownership | Authorization rests on **proven control** of the asset (DNS TXT / `.well-known` / meta tag), not a declared scope rule. | ARTEX attributes by operator-declared domain/IP/CIDR/keyword rules. |
| Imports | Lead-only, cover far more formats (nuclei, zap, burp, sqlmap, nikto, wapiti, ffuf, httpx, hydra, SARIF); a fact needs an oracle re-drive. | ARTEX trusts a scanner's own scope label. |
| Keyless mode | Ships an always-available DryRun backend and a keyless "attest-first, propose-nothing" engage path. | Tests stay hermetic; the honest no-key mode never fabricates activity. |

Net: VIGIL's gaps are almost all about **organisation, persistence, and operator surface** — not about truth-handling.

---

## 3. The gaps, ranked by customer and launch value

Ranking lens: what a paying tenant sees and what unblocks launch comes first; internal robustness and niche capture modes come last. Every recommendation extends an existing store, is default-OFF, keeps leads lead-only, and keeps the oracle as the sole authority.

| # | Gap (short) | ARTEX has | VIGIL today (incl. built-not-merged) | Recommendation (the VIGIL way) | Program | Effort |
|---|---|---|---|---|---|---|
| 1 | **Merge the evidence-verified graph** | Findings bound to a re-verified proof cert; goal closed only on matching bound traffic | S1 evidence resolver + S2 verified-only join **built, not merged**; on `main` an `evidence_ref` is just a non-empty string | Merge S1 then S2. Add a demote-only check that the cert's oracle-context host equals the finding's asset host and is in charter scope. No new store. | Investigation graph | S |
| 2 | **Persistent searchable traffic index** | Recording MITM proxy = one searchable index of every exchange ("keeps its receipts") | Executor writes `request.http`/`response.http` per action dir; Strix traffic lives only in in-sandbox Caido and is discarded except cited ids; `TrafficCorpus` has no production data source | Build a default-OFF per-engagement exchange index inside the existing evidence tree (host/method/status/body-sha/action_id). Export Caido history at teardown, tagged `sandbox-capture`, LEAD-only. Point `TrafficCorpus` at it. Keep the trusted gateway MITM-free. | CRUCIBLE engine; invgraph for links | M |
| 3 | **Org-wide asset inventory** | Company-scoped asset graph persisting across tasks, program-deduplicated | World-model is a JSON file per run; intel store keyed per slug; **no `tenant_id`**; world-model and the F4 bridge share no asset key | Build an org-scoped read-model over the existing intel store + signed spine. Key every asset by canonical `EntityRef` id; use the same key in F4. Stamp `tenant_id`. Derive `seen_in` per asset. Add a structural `SUBDOMAIN_OF` edge from the pinned PSL. Default-OFF. | Investigation graph + consumer app | M |
| 4 | **TESTED anchor + coverage view** | Per-task coverage graph marks in-scope assets tested/untested; findings-by-asset tree | Attack Graph / Coverage tabs are **per run**; a clean test leaves no anchor, so per-asset coverage can't be computed; S5 coverage view has **no branch yet** | Build S5 with a `ChainStep -[TESTED]-> asset` edge written from the executor's **pinned** target (what the gate authorized), keyed by `EntityRef` id. Keep coverage states separate from verdict. `tested_no_finding` requires a retained benign response. | Investigation graph | M |
| 5 | **Attribution of discovered assets** | Deterministic engine attributes discovered assets to a company; recompute under lock | Ownership proven per exact host, but nothing attributes discovered subdomains/IPs to an org or recomputes on revoke/expiry; legal/gov S5 `current_holders` (sole-proof-holder primitive) is **unmerged** | Add an attribution projection as a pure fold over the proof log, so revoke/expiry drops attribution automatically. Attribute only to the sole current proof-holder, or under a verified apex with explicit opt-in. RDAP/ASN stay displayed LEADs. | Consumer app + CRUCIBLE engine | M |
| 6 | **Charter scope hardening** | Scope rules with fixed precedence; CIDR kind | `parse_scope`/`host_matches_scope` accept `*.com` / `*.co.uk` and have **no CIDR kind** | At authority mint, refuse public-suffix and bare-TLD wildcards using the pinned PSL, fail-closed. If CIDR is needed, add it only as an owner-signed netblock with a minimum-prefix bound and the categorical `.gov/.mil/.edu/.int` floor. | CRUCIBLE engine | S |
| 7 | **Operator-facing replay pane** | Operator-driven request replay | A fully gated Repeater exists (entitlement + scope at two layers + destructive-confirm binding) but **nothing exposes it** — no console route, no UI pane, and no capture index to seed it | Add a default-OFF console route + UI pane that calls only the existing Repeater facade (no second send path). Seed from an evidence-dir exchange or a capture-index row. Add request-hygiene refusals with tests. Show results as provenance-labelled observations with a "hand to oracle" action. | Consumer app + CRUCIBLE engine | M |
| 8 | **By-asset findings tree + query filter** | Asset query DSL; company→root→subdomain→service→endpoint with subtree counts | Every view is per run; the customer app lists only proven hosts, not the surface under them | Once #3 and #4 land, add a read-only, tenant-scoped by-asset tree that counts CONFIRMED and LEAD separately. Add a small fixed-grammar filter (allowlisted fields → predicate, no SQL/eval). Expose in console and customer app. | Consumer app + investigation graph | M |
| 9 | **Edge-triggered, multi-cycle loop** | Event-driven planner: wakes on graph change with 800ms debounce + heartbeat patrol; always-on | Coordinator is **tick/round-robin**; the real closed OODA loop (`engage_autonomous.py`) is opt-in and bounded to **one cycle**; the default engage is a fixed pipeline | Wake the Coordinator on blackboard append (it is already the append-only source of truth), coalesce with a debounce, keep a heartbeat patrol as deadlock floor. Extend `engage_autonomous` to a budget-/watchdog-bounded multi-cycle loop. Keep **serial-for-PROOF**. Default-OFF. | CRUCIBLE engine | L |
| 10 | **LLM goal decomposition into objectives** | One LLM step splits the operator goal into independently-verifiable objective nodes, rule-based fallback | Decomposition is archetype-seed + ≥5 LLM hypotheses; durable Objective nodes exist only in **unmerged** invgraph S3 (`PURSUES`/`REACHES`) | Add a kernel goal-decomposition binding (sibling to hypothesize/threat_model): split the charter objective into falsifiable objectives, each with a `refute_on` and an explicit oracle-class mapping, feeding `planner/seed.py`. Fall back to the existing archetype seed. Persist via S3 and merge it (inert). | CRUCIBLE engine + invgraph | M |
| 11 | **Per-leaf claim lease (SEARCH only)** | DB-backed claim lease + heartbeat + fencing token + claim-sweep; crash resets running claims | Leases stop at the **engagement** boundary (`slug_lease.py`); claiming a goal-tree leaf is an in-process flag; `resume.py` checkpoint has no production caller; nothing re-opens leaves stuck "claimed" after a crash | For the SEARCH phase only (concurrency already doctrine-permitted), extend the `slug_lease` primitive down to leaf granularity over the blackboard: per-leaf claim nonce, heartbeat, sweep that returns a crashed leaf to the frontier, explicit terminal states. Keep PROOF serial. Default-OFF. | Investigation graph / CRUCIBLE engine | M/L |
| 12 | **Per-leaf control (pause/cancel/steer)** | Per-intent pause/cancel into a recoverable state; steer drained before next tool call, no progress lost | Control is **engagement-granular** (kill-switch, budget pause, watchdog halt); steering folds into the **next** think, not before a specific worker's next tool call | Add a per-leaf steer queue drained by the executor **before** its next gated tool call (reuse the WARDEN pre-exec hook shape); add a per-leaf cancel that marks the leaf recoverable instead of tripping the global kill-switch; tag every abort with a typed cause. Keep steering advisory + gated. Surface in the consumer app. | CRUCIBLE engine + consumer app | M |
| 13 | **Narrow cross-session inheritance** | Direct related tasks inherit terminal intents, tagged read-only, non-transitive | Connected sessions pass only finding/lead summaries; asset nodes, coverage, terminal outcomes and **failure lessons** never cross; `query_prior_chains` (carries lessons) is **unwired** | Extend `retrieve_priors` to also union consented partitions: world-model asset nodes as GROUNDING_INTEL (filtered through the current charter first) and terminal outcomes / failure lessons as dead-end priors. Wire `query_prior_chains` into the think step. Keep read-only, non-transitive, consent-gated. | Investigation graph | M |
| 14 | **Background enrichment queue** | Non-AI worker pool: dnsx resolve + gated HTTP probe, cooldown, writes status/title/length back | Auto-probe happens only inside the opt-in discover path or explicit tool runs; **no per-asset cooldown**; no first-party probe writes HTTP metadata onto the SERVICE/ENDPOINT node | Add a deterministic enrichment step to the opt-in discover path, serial or small bounded pool, with a per-`(kind,id)` cooldown keyed on monotonic seq. Send every probe through the gated `HttpExecutor`, retain the exchange, record status/title/length as INTEL observations. Never mint liveness as FACT. Default-OFF. | CRUCIBLE engine | M |
| 15 | **Graph compaction (hot/cold digest)** | Cold-digest compaction folds settled nodes into digest nodes with `covers` edges, members retained | The `chainast` compactor works on the **message chain, not the graph**, and is unwired; the goal tree and F4 graph have no folding | Add a read-only digest view over the F4 `GraphView`, default-OFF. Hot set computed deterministically; cold nodes grouped into digests citing member ids + a Merkle root over member spine hashes (reuse `chainast.merkle_root`). Tag SUMMARY, never FACT; never delete members. Wire `chainast` into the think loop in the same slice. | Investigation graph | M |
| 16 | **bbot import parser** | bbot JSONL ingest (typed fields only, free text as one inert quoted string) | No bbot parser (`imports/parsers.py` covers nuclei/zap/burp/sqlmap/nikto/wapiti/ffuf/httpx/hydra/SARIF but not bbot) | Add a `bbot` format: read only typed fields, map `DNS_NAME→DOMAIN` / `IP_ADDRESS→HOST` / `OPEN_TCP_PORT→SERVICE` / `URL→ENDPOINT` as LEAD observations; keep `scope_distance` as an attribute only (charter gate decides); keep `data`/`data_json` inert. | CRUCIBLE engine | S |
| 17 | **nmap -oX file import** | Deterministic scanner-file ingestion incl. operator-supplied nmap XML | nmap XML is parsed only inside the sensor's **own** run; an operator-supplied `-oX` file can't be imported | Register an `nmap` import format that reuses the sensor's XML normaliser and the shared `service_observations` minter, lead-only, with the item cap + truncation reporting. | CRUCIBLE engine | S |
| 18 | **HAR import/export** | HAR 1.2 export + import | No HAR reader or writer on `main` | Import: a passive Tier-1 sensor shaped like `sensors/tshark.py`, bounded, drops out-of-scope hosts, maps entries to world-model observations as GROUNDING_INTEL (never FACT). Export: project evidence-dir exchanges as a non-authoritative view, redacted by default, each entry carrying its `action_id`/cert digest. | CRUCIBLE engine + consumer app | S |
| 19 | **Request rewrite rules (request-phase only)** | Rewrite/tamper-hook engine + sandboxed hook VM | Only pre-send pure payload rules (`intruder/processing.py`) and `mutate()`; no in-flight rewrite | Build it as **declarative rules only** (no hook VM — don't take on a hand-written interpreter as a trust surface), applied inside VIGIL's own gated send path, WARDEN-gated, default-OFF. Record each applied rule's id+digest as evidence. **Request-phase only** — a response-rewritten exchange must be marked ineligible for minting, because oracles may judge only target-produced bytes. | CRUCIBLE engine | M |
| 20 | **WebSocket / SSE frame recording** | Stream recording of WS/SSE traffic | `scanner/websocket.py` keeps only the handshake transcript + verdicts; frames aren't recorded as ordered artifacts | When the capture index (#2) lands, write WS/SSE frames from VIGIL's own client as one ordered, bounded `frames` artifact per stream in the action evidence dir, so the manifest hashes them. No separate store. | CRUCIBLE engine | S |
| 21 | **SOCKS5 / transparent capture modes** | SOCKS5 (+UDP) and transparent inbound capture | Gateway accepts only HTTP CONNECT + absolute-form HTTP on allowlisted ports; no SOCKS5, no `SO_ORIGINAL_DST` | If needed, add a default-OFF SOCKS5 CONNECT front end that calls the **same** `authorize()` (one gate, no parallel policy). Keep UDP ASSOCIATE **refused** (UDP egress widens containment). Metadata-only until the capture index exists. | CRUCIBLE engine | M |
| 22 | **Idle-turn nudge** | `steerHooks.Stop` detects a thinking-only turn and injects a capped one-shot rerun | A no-action/no-text turn falls to fail-closed `ASK_USER`, not a nudge | In the think loop, detect a no-action-no-text turn and re-prompt once (hard cap ~2), suppress on provider refusal, then fall back to ASK_USER. Bound under existing budget + watchdog. | CRUCIBLE engine | S |
| 23 | **LLM health / circuit-breaker memory** | Shared provider pool with health/failover | Failover + per-call availability probe exist, but each failover re-probes from scratch — no persistent cross-call health memory | Optionally add an in-process cooldown per backend that reorders `permitted_preference()`, still re-asserting sovereignty on construction and never leaving the tier. Advisory to selection only. | CRUCIBLE engine | S |

Built-not-merged that directly unblocks the top of this list: invgraph **S1** (evidence resolver), **S2** (verified-only join), **S3** (Objective nodes), **S4** (Remediation/Retest lifecycle); legal/gov **S5** `current_holders` (reused for gap 5). **S5 coverage view has no worktree yet** and must be built for gaps 4 and 8.

---

## 4. Things ARTEX does that VIGIL should deliberately NOT do

These are outside VIGIL's authorized, prove-don't-guess, defensive-proof doctrine. Keep them out.

| ARTEX trait | Why VIGIL must not adopt it |
|---|---|
| **Ungoverned Kali-Bash worker** (arbitrary shell) | VIGIL gates every tool call WARDEN-fail-closed and keeps the local terminal read-only-allowlist. An open shell is an ungoverned offense portal. If more tool breadth is wanted, add tools **behind** the conjunctive gate + retained-bytes capture, never an open shell. |
| **LLM mandatory, no simulation mode** | VIGIL inverts this on purpose: the DryRun backend is always available and the keyless engage attests first and proposes nothing. ARTEX's mandate would break hermetic tests and remove the honest keyless mode. |
| **TLS MITM on the containment plane** | TLS keys and plaintext do not belong on the gateway that enforces containment. Capture belongs in the evidence tree and (at teardown) the sandbox proxy, not on the trusted gate. |
| **Response rewriting that can reach an oracle** | An oracle may judge only target-produced bytes. A rewritten response would let VIGIL mint a FACT over bytes it produced itself. Allow request-phase rewrite only; mark any response-rewritten exchange ineligible for minting. |
| **UDP egress (SOCKS5 UDP ASSOCIATE)** | Widens containment. Keep it refused. |
| **Authorization from declared scope rules** (domain/IP/CIDR/keyword) | VIGIL authorizes on **proven control** of the asset's own infrastructure. A declared rule is not proof. Keep RDAP/ASN/registrant inference as displayed LEADs that never authorize. |
| **ICP-license scope rules** | China-specific registry, outside VIGIL's markets. Not applicable. |
| **Keyword scope rules** | No equivalent and none needed — ARTEX itself never uses keywords to attribute. |

---

## 5. ARTEX's strongest area vs the investigation-graph program (S1–S5)

ARTEX's strongest area is the **dual knowledge graph with evidence-bound findings** — the "keeps its receipts" core: a persistent asset graph plus a per-task exploration graph, anchored together, with every finding tied to the captured exchange that proves it and re-run by a retester.

**What the investigation-graph program already covers:**

| ARTEX strength | Covered by |
|---|---|
| Finding bound to a verified proof | **S1** (evidence resolver by `cert_digest`) + **S2** (CONFIRMED demoted unless the `evidence_ref` resolves to a same-engagement, same-finding cert that re-verifies under the pinned root) |
| Goal/objective nodes | **S3** (inert `Objective` nodes; `PURSUES`/`REACHES`; a goal with no actions shows uncovered, never CLEAN) |
| Retester + verification lifecycle | **S4** (Remediation/Retest nodes, verified/unverified lifecycle) |
| Tested/untested coverage | **S5** (coverage view) — **planned, no worktree yet** |

**Where it still misses ARTEX, and exactly what to add:**

1. **No org-wide persistent asset graph.** Invgraph is per-engagement projection. Add an org-scoped asset inventory read-model (gap 3) over the existing intel store + spine, keyed by canonical `EntityRef` id, with `tenant_id`. No new database.
2. **No shared asset key.** The world-model and the F4 bridge (Host/Port/Technology/Endpoint) have no common key, so anchors can't join cleanly. Unify on the `EntityRef` id across both, inside the invgraph asset-identity slice.
3. **No structural hierarchy edge.** Only an `apex` attribute exists. Add a `SUBDOMAIN_OF` edge derived at canonicalisation time from the vendored, sha256-pinned PSL, INTEL grounding only.
4. **No TESTED anchor.** A clean test leaves no trace. Build S5 with a `ChainStep -[TESTED]-> asset` edge from the executor's **pinned** target (gap 4), so per-asset coverage is computable; keep coverage states separate from the verdict.
5. **No cold-digest compaction.** The goal tree and F4 graph don't fold. Add a read-only digest view (gap 15): hot set computed deterministically, cold nodes grouped into SUMMARY digests citing member ids + a Merkle root; members never deleted.
6. **Narrow inheritance.** Extend the consented-session union to carry asset nodes and failure lessons, not just finding/lead summaries, and wire `query_prior_chains` (gap 13).

Invariant for all of it: the graph stays **inert** — a one-way projection that authorizes nothing. CONFIRMED comes only from a re-verified evidence cert; the gate never reads the graph.

---

## 6. Proposed order of work

**Wave A — Launch credibility (merge what's built).**
1. Merge **S1** then **S2** (gap 1): CONFIRMED depends on a re-verified evidence cert. Add the host-in-scope demote check.
2. Merge **S3** (gap 10 persistence) and **S4** so objectives and retest lifecycle are first-class.

Rationale: these are already built, raise the credibility bar a tenant and an auditor see, and unblock the asset/coverage work.

**Wave B — Customer-visible asset & coverage story.**
3. Org-wide asset inventory read-model with `tenant_id` + `SUBDOMAIN_OF` + shared `EntityRef` key (gap 3).
4. Build **S5** coverage view with the **TESTED anchor** (gap 4).
5. Attribution fold over the proof log + charter wildcard/PSL + CIDR guard (gaps 5, 6).
6. By-asset findings tree + fixed-grammar filter in console and customer app (gap 8).

Rationale: this is the "what did you cover, on which of my assets" product that a buyer evaluates, and it reuses Wave A.

**Wave C — Receipts & operator surface.**
7. Persistent per-engagement traffic **capture index** in the evidence tree (gap 2).
8. Operator-facing **replay pane** over the existing Repeater facade, with hygiene refusals (gap 7).
9. HAR import/export and the bbot + nmap import formats (gaps 16, 17, 18); WS/SSE frame recording once the index lands (gap 20).

Rationale: delivers ARTEX's "keeps its receipts" feel on VIGIL's stricter base, plus interoperability. Depends on Wave B's evidence plumbing.

**Wave D — Loop depth & robustness (engine-internal, default-OFF).**
10. Edge-triggered wake + multi-cycle autonomy (gap 9), keeping serial-for-PROOF.
11. Per-leaf claim lease for the SEARCH phase (gap 11) and per-leaf control (gap 12).
12. Background enrichment queue (gap 14); graph compaction (gap 15); cross-session inheritance of assets + lessons (gap 13).

**Wave E — Polish / niche (only if pulled by a need).**
13. Idle-turn nudge (gap 22); LLM health memory (gap 23); declarative request-rewrite rules (gap 19); SOCKS5 CONNECT front end (gap 21, UDP stays refused).

Sequencing note: Wave A gates Waves B and C. Within each wave the items are listed in dependency order. Everything ships default-OFF, lead-only, oracle-authoritative, with no parallel store — each gap extends the intel store, the signed spine, the F4 graph, or the ownership proof log.
