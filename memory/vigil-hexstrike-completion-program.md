---
name: vigil-hexstrike-completion-program
description: "VIGIL: pull-forward program to wire the WHOLE HexStrike catalogue + make every soundly-FACT-capable tool mint a signed FACT to enterprise/national standard; grounded 8-wave plan"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-09-25T20:43:45.219Z
---

Operator (2026-09-25) chose to PULL FORWARD the HexStrike catalogue completion (was FACT-coverage
Wave 8): "I want ALL of hexstrike, all fully wired and FACT-capable end-to-end, enterprise and
national tool standard." Grounded by a 13-agent read-only workflow over `/home/kali/vigil-dev` @
HEAD `ecce8a78`. See sibling [[vigil-strix-fact-breadth-program]] (the same for Strix) and
[[vigil-strix-hexstrike-wiring-program]] (wiring already complete; all 12 invariants MET).

**HONEST CEILING (the deliverable definition — do NOT overclaim past it).** "All 56 fact-capable"
is NOT sound: a FACT needs a deterministic VIGIL-OWNED oracle re-firing over a gated live capture +
offline re-verify. Sound completion = **18 tools mint genuine signed offline-re-verifiable FACTs**
(15 achieved-state reachability/liveness + 3 TLS posture-fact) — up from **2 truly end-to-end today**
(nmap, sslscan; masscan/rustscan/naabu are FACT-wired but UNAVAILABLE at the executor = the JOIN
DEFECT). The rest: ~21 LEAD-only (sound; feed the agreement ladder / a VIGIL re-drive, tool bytes
never mint), 17 BLOCKED (excluded offense/credential/binary). Making all 56 "fact_capable" would
require fabricating oracles VIGIL doesn't own (L2-ARP liveness, UDP-137, TLS suite-enumeration
completeness, per-CVE reachability, param-inference certainty) — the one thing we never do.

**CENSUS (verified, pinned by test_capability_matrix_sync.py):** 56 catalogued (the "79" is prose
only in hexstrike.json:3), 36 brain-proposable (_TOOL_DANGER), 17 excluded (1 also proposable =
sqlmap → BLOCKED), 5 fact_capable (nmap/masscan/rustscan/naabu/sslscan), 9 typed argv builders
(_BUILDERS; 2 = wapiti/zaproxy are VIGIL-native, not hexstrike). Catalogue = a JOIN across 3
registries sharing no key: capability matrix `docs/capability-matrix/hexstrike.json` · brain
`_TOOL_DANGER` (brains/hexstrike_brain.py:74) · executor `_BUILDERS` (live/executor.py:1740, mirror
tools/profile.py:46 `_TYPED_BUILDER_TOOLS`); joined by capability_join.py:79/163. VIGIL-DIRECT fact
families (no tool is authority): VERSION_RANGE via live.sbom, ACHIEVED_STATE via live.web_redrive.

**THE 8-WAVE PLAN (cadence: parallel worktree builds → red-pen per slice → sequential additive merge;
every fact_capable flip gates on conformance.REQUIRED_PROPERTIES [all 7] through the REAL
run_external_tool + the corpus zero-FP gate; NO new OracleKind in the frozen _ALL_ORACLES fallback —
imds_verify/secret_verify present-only pattern; keyless offense/FATAL-2 function-local imports;
never-liftable egress floor; LEAD-until-oracle; only VIGIL's gated re-drive crossing verdict.admit →
certify_admitted('live_redrive') mints):**
**STATUS: Wave 0 MERGED (#846 @db3bccee, 2026-09-25) — red-pen PASS.** Took the UNIFY path (not mirror): framework-free SSOT `SPEC_BUILDER_TOOLS` in oracle_families.py; capability_join.resolve() unions `_BUILDERS | SPEC_BUILDER_TOOLS` (function-local stdlib import → resolve stays framework-free); masscan/rustscan/naabu now end-to-end. UDP soundness: capture_handshake refuses non-tcp BEFORE any socket + parsers drop proto!=tcp (closed at 3 layers). Red-pen O1 (operator-visible honesty divergence: profile.py build_profiles said REFUSED while capability_join said EXECUTABLE) CLOSED via a REAL meet-up — profile.py now recognizes the R4 spec-builder tools as `cli` control surface (clean: no roster change, counts relational, gate unmoved, drift-guard test pins the set); O2 test-fidelity fixed. Gate byte-identical 11/0/0, FATAL-2 intact. NOTE the stale plan anchors: `_SPEC_BUILDER_TOOLS` is in brains/hexstrike_body.py:76 (NOT executor.py); error_signature_oracle is oracles.py:2792 (not :1875).
- **Wave 0 (foundation, lands first):** 0.1 THE JOIN DEFECT — add masscan/rustscan/naabu to executor
  `_BUILDERS` + profile `_TYPED_BUILDER_TOOLS` (preferred: unify capability_join.resolve to read the
  ToolSpec-builder set so registries can't drift) → makes the 3 already-FACT-wired port scanners
  end-to-end. 0.2 UDP SOUNDNESS — gate the reachability re-drive to protocol=='tcp' (external_tool.py:1060,
  reachability.py:112) so a udp-open row never mints a TCP-handshake FACT.
**STATUS: Wave 1 MERGED (#847 @a258bd58, 2026-09-25) — red-pen PASS.** zmap+unicornscan → fact_capable (pure wiring; builders pre-existed; fact_capable 5→7 {nmap,masscan,rustscan,naabu,zmap,unicornscan,sslscan}). Mints only via real capture_handshake; default fact_capable=False spec stays LEAD-only; two surfaces agree; CAP_NET_RAW honesty (UNAVAILABLE-in-sandbox, never false EXECUTABLE); gate byte-identical 11/0/0. DEFERRED to W7 sweep: evidence-branches.json:270 blocking_work under-claims (see W7 note).
- **Wave 1 (port breadth, reuses service_reachability.tcp_handshake):** zmap + unicornscan → fact-capable
  (builders exist; flip matrix + _SPEC_BUILDER_TOOLS + _BUILDERS). Red-pen: raw-socket-needs-root →
  report fact-capable-but-proposer-UNAVAILABLE, not EXECUTABLE; scope-pin host not printed saddr.
- **Wave 2 (web-discovery achieved-reachability — LARGEST; builds ONE new endpoint_liveness branch+oracle,
  L7 analogue of tcp_handshake, reuses web_redrive `_gated_web_send` NO canary):** 2.0 liveness scaffold;
  2.1 httpx+ffuf (builders exist); 2.2 gobuster+feroxbuster+dirsearch; 2.3 katana+hakrawler; 2.4
  dirb+wfuzz(content-mode); 2.5 passive-archive (gau/waybackurls/paramspider) URLs → liveness re-drive
  (mints CURRENT liveness, never the archive's historical claim). Red-pen: soft-404, crawl/enumeration
  COMPLETENESS never a FACT or CLEAN; CLEAN bounded to the EXACT probed URL only.
- **Wave 3 (TLS breadth, reuses _TLS_REDRIVES/TLS_WEAKNESS):** testssl+sslyze → posture-fact (leaf/protocol
  only). Red-pen: per-cipher ENUMERATION rows + vuln probes (Heartbleed/ROBOT/…) have NO oracle → LEAD.
- **Wave 4 (scanner per-finding re-drive; LEAD-feeds-VIGIL-oracle, tool never mints):** nuclei/nikto/jaeles/
  wpscan finding whose class ∈ WEB_FACT_CLASSES → web_redrive → ACHIEVED_STATE on the EXISTING branch;
  else LEAD. Red-pen: info/misconfig template auto-mint; CVSS treated as evidence; tool label upgrading class.
- **Wave 5 (cloud/SBOM posture):** trivy/grype → VERSION_RANGE posture-fact where a parsed pinned version in
  a vuln range is re-derivable from the retained SBOM; prowler/scout-suite/checkov/terrascan/kube-bench/
  kube-hunter → posture LEAD unless a VIGIL artifact-scoped re-drive exists; read-only, egress floor holds.
- **Wave 6 (injection LEAD→VIGIL oracle):** re-home x8 web_discovery→injection; wire arjun/x8/dalfox(+wfuzz
  param) discovered params into VIGIL's differential injection re-drive (mints only after VIGIL's own
  control-vs-payload differential fires); sqlmap stays EXCLUDED/BLOCKED (never run).
- **Wave 7 (honesty gate, MUST be final merge):** machine-checked boundary — and refresh the DEFERRED doc nit: evidence-branches.json ~:270 `hexstrike.service_reachability` blocking_work text now UNDER-claims for zmap/unicornscan (says "pending promotion/default emits LEAD" while _spec_for_kind builds them fact_capable=True by default). Conservative, passes validation, non-blocking — fold into the coverage-truth sweep. — extend validate_family_routing
  into a REQUIRED CI test so no matrix row can claim a FACT its family can't mint, and arp-scan/nbtscan/
  every excluded tool can never silently drift to fact_capable.

**DECISION 2026-09-26 — endpoint_liveness / `sibling_response_differential` DOWNGRADED TO A PERMANENT LEAD after FIVE red-pen rounds.** Every round found a NEW false-FACT class minting signed, offline-re-verifying FACTs for URLs where nothing exists: (1) fixed non-same-shape control + status-only predicate; (2) hex/UUID route-class miss + bounded-body control collision (~30%); (3) checksum/validation routes (Luhn ~32%, base58 ~7%) — char-class subset does NOT imply route-VALID; (4) unknown in-branch validator; (5) per-position FORMAT routes (`^report\d$`, `^user\d{3}$`, `^dashboard\d$`) at 11-55% direct and 29-40/80 on BOTH runner legs, because the twin cohort draws from a per-SEGMENT UNION alphabet so a Hamming-1 probe at a digit slot becomes a letter ~74% of the time and the route rejects it (no twin found => mint); plus (6) a SKEWED per-PATH bounded body space (13/100 at P=1/20).
**THE CLINCHER was not the rate but the INFORMATION CONTENT:** on a route where /o/dashboard9 EXISTS and /o/dashboard1 does NOT, it minted 40/40 for the existing id and 21/40 for the phantom — it barely discriminates. And the honest narrowing forced in round 4 (disclaiming not-found-ness/phantom-ness/existence/liveness) removed the very thing that made it worth minting: what shipped was a raw measurement of response variation across a lexical neighbourhood, at confidence 0.9 with a token consumers read as "VIGIL confirmed something". LESSON: when honest narrowing leaves a proposition too weak to matter, that is itself the signal to ship a LEAD — soundness and MEANINGFULNESS are separate bars, and a FACT must clear both. The whole evidence pipeline is RETAINED as a lead-enricher (gated GETs + same-branch baseline + multi-sampling + validator-aware controls + NO-TWIN cohort) — genuinely good LEAD evidence, it just never mints. This is the THIRD honest boundary alongside clickjacking-achieved and GraphQL-cost-absent.
**HONEST COVERAGE REVISION (supersedes the 18-tool ceiling above):** the original grounded ceiling of 18 FACT-minting tools was 15 achieved-state (6 port scanners + 9 web-discovery liveness) + 3 TLS posture. With liveness a LEAD, the 9 web-discovery tools (httpx/ffuf/gobuster/feroxbuster/dirsearch/katana/hakrawler/dirb/wfuzz) stay LEAD-only, so the realistic FACT-minting ceiling from W0-W3 is **9 tools** (6 port: nmap/masscan/rustscan/naabu/zmap/unicornscan + 3 TLS: sslscan/testssl/sslyze). 7 are merged today (6 port + sslscan). Later waves may add FACTs via OTHER oracles (nuclei/nikto findings re-driven into existing web oracles W4; trivy/grype VERSION_RANGE posture W5) — but do NOT quote 18.
**CONSTITUTION VIOLATION found in the same review (fix regardless of the downgrade):** the new httpx ToolSpec omitted the User-Agent pin and httpx's `-random-agent` DEFAULTS TRUE => every probe went out under a RANDOM BROWSER IDENTITY (identity rotation, forbidden by §VI.4 "you are not evading them"). The repo had ALREADY found+fixed+documented this in the governed executor (executor.py:924-936 explains it, :570 pins `-H User-Agent: OBSIDIAN/1.0 (authorized owner-test)`) — the new leg reintroduced it. ALWAYS check a new tool spec against the governed executor's existing hardening. Also OPSEC: the re-drive was UNCAPPED (63 gated GETs/URL, serial, no delay, every proposed URL re-driven) => an ffuf run proposing 200 URLs = ~12,600 GETs at a production target. Needs a hard cap + documented budget (§VI).

**TWO TRANSFERABLE SOUNDNESS LESSONS (from 4 red-pen rounds on endpoint_liveness, 2026-09-26) — apply to EVERY differential FACT:**
1. **A same-shape RANDOM control is not enough; you need a VALIDITY-NEIGHBOURHOOD control.** Drawing controls to maximize INDEPENDENCE from the target is exactly wrong when the app has an unknown semantic validator (checksum / prefix / suffix / positional rule): every random control lands in the REJECT set, so the target looks "distinguishable" when it is just the only validator-VALID probe. The fix is a MINIMAL-EDIT-DISTANCE (Hamming-1) sibling cohort + a **NO-TWIN rule** (if ANY other probe returns the target's exact (status, body-hash) -> LEAD), added as extra `not eq(probe_k_sha, target_0_sha)` AST clauses and NEVER counted toward the control floor (twin-search only, so it can only turn FACT->LEAD). Measured: phantom 39/40 -> 0/40 FACTs with ALL true positives preserved 40/40, for ~20 extra gated GETs. This FALSIFIED a self-certified "irreducible" claim — never accept an irreducibility argument without an independent attempt to find a discriminator. What remains genuinely irreducible is only a validator constraining the id JOINTLY ACROSS POSITIONS (whole-string checksum) with an in-branch reject.
2. **A claim that does not reach the WIRE is not a disclosure.** The narrowed claim lived in a `WebLivenessResult.note` that `AdapterResult.detail`/`RunnerResult` silently dropped, so the signed certificate shipped only the token `endpoint_liveness @0.9` — precisely the assertion the module said it must never make. ALWAYS put the narrowed claim INTO the digest-bound certificate and rename any bug_class token that overclaims. Corollary: a stale description in the engine's own registry (verifier.py) can document the INVERSE of the shipped predicate — grep the registry + test comments whenever a predicate changes.
Also: with a tiny valid-fraction p and an IN-BRANCH reject, raising the control count does NOT mitigate (40/40 at N=10, 40/40 at N=160, ~1874 controls for 50%) — do not describe that as "bounded, trades traffic for it".
SAFETY GOTCHA found in the same review: a brain-supplied `scheme` interpolated into a tool argv escaped the ScopeGate (packets to an out-of-scope host before `_pin_url_host` blocked the FACT). Validate every brain-supplied argv value (scheme against {http,https}, wordlist against an allow-list) BEFORE the send, not just at mint.

**KEY SOURCE ANCHORS for build agents:** live/oracle_families.py (registry+routing, fuse returns literal
"LEAD") · live/external_tool.py:588,614,650,657,677,1060 (spec builders + re-drives) · live/web_redrive.py:
27,98 (WEB_FACT_CLASSES + gated send) · live/capability_join.py:79,163 · live/conformance.py:30
(REQUIRED_PROPERTIES) · brains/hexstrike_body.py:76,86 (_SPEC_BUILDER_TOOLS + _spec_for_kind) · live/
executor.py:1740 (_BUILDERS) · tools/profile.py:46 (_TYPED_BUILDER_TOOLS) · verify/reachability.py:112 +
verify/tls.py:71 (the two gated captures) · docs/capability-matrix/{hexstrike.json,evidence-branches.json}.

**BOUNDARY LEDGER (permanent LEAD — VIGIL owns no sound re-drive):** arp-scan (L2 ARP ≠ TCP port),
nbtscan (UDP-137 non-response≠absence), gau/waybackurls/paramspider (archive say-so; emitted URLs CAN feed
the liveness FACT = current liveness only), arjun/x8/wfuzz-param (inferred param = heuristic), subfinder/
amass (subdomain existence; resolved host:port is a DIFFERENT predicate), enum4linux-ng/smbmap (no SMB
oracle), autorecon (orchestrator), nuclei/nikto/jaeles/wpscan (match = LEAD; VIGIL re-drive mints),
prowler/scout-suite/checkov/terrascan/kube-*(posture unless artifact re-drive), trufflehog/gitleaks (secret
candidate; FACT path = credential-validity check not in catalogue), chromium/playwright (DOM = LEAD; DOM_EXECUTION
re-drive mints), source SAST (LEAD here). Enumeration/COMPLETENESS + fingerprint/tech/banner never mint & never
a universal-negative CLEAN. BLOCKED = the 17 excluded offense/cred/binary (sqlmap/hydra/john/metasploit/pwntools/
angr/…). [[vigil-fact-coverage-program]] [[vigil-wave-cadence]] [[claim-discipline-meet-up]]
