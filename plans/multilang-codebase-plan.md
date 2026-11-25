# VIGIL multi-language codebase vulnerability analysis + proof-carrying fix — implementation plan

**Status:** planning only (no code). **Repo:** `/home/kali/vigil` (read from `origin/main`,
HEAD `2c93ed94`). All citations are `file:line` against `origin/main`.

**Goal:** make VIGIL's *consumer codebase code-review* work at FACT-grade on **any** programming
language, production-grade, landing as **one coherent PR** (plugin framework + a flagship set of
languages), with any further language an **additive plugin** rather than a core change.

---

## 0. Context and the live-vs-codebase boundary (state it up front)

VIGIL has **two** review surfaces, and only one of them is language-coupled:

* **Live-system scanning (black-box HTTP) — ALREADY language-agnostic. Out of scope here.**
  The live oracles adjudicate *bytes a running target produced over HTTP* — differential, boolean,
  timing, the request-side `*_BREAKOUT` parse-proofs, OOB callbacks, achieved-state, TLS/cert, etc.
  (`engine/crucible/framework/v2/verify/oracles.py`). None of them parses source, so a Django app,
  a Rails app and a Spring app are the same problem to them. **Nothing in this plan touches the live
  path.** It is named only to draw the boundary.

* **Codebase (source) scanning — language-coupled. THIS plan.**
  A codebase target has *no live HTTP oracle*; its findings are grounded in a **re-runnable
  deterministic rule over the operator's own source bytes**
  (`integration/vigil_integration/codescan.py:1-24`). Parsing source is where language matters, and
  that is the whole subject below.

**The two grades of code finding (this distinction drives the entire plan).** VIGIL already mints
code findings at two different grades, with *different* language-coupling:

1. **DAA-fact grade** — a DAA rule (regex / semgrep / external tool) matched real bytes; written to
   the signed spine as `status="fact"`, `source="daa:<rule_id>"`
   (`codescan.py:242-264`). Grounded in the **re-runnability of the rule**
   (`verify_finding_cleared`, `codescan.py:356-401`). It never claims runtime exploitability
   (`codescan.py:17-19`). **This grade is already multi-language** — any extension, any regex/semgrep
   rule — and its fix-verification (`build_code_fix_oracle`, `codescan.py:341-353`) is a
   language-agnostic rule re-run. Its fix status caps at the honest `verified-no-pr` (no signed
   certificate — `CodeFixVerdict`, `codescan.py:327-338`).

2. **Oracle-STATIC-FACT grade** — a VIGIL-owned deterministic oracle (`STATIC_RULE`) **re-parses the
   retained source region itself** and re-derives the code property, then mints a **signed,
   offline-re-verifiable certificate** through the single mint site. **This grade is Python-only
   today** (`verify/oracles.py:6212-6217` hard-refuses any non-Python language; `ast.parse` at
   `:6222`). This is the FACT-grade path and is where ~all the real per-language work lives.

"FACT-grade on any language" therefore means: **carry grade (2) — the sound, VIGIL-owned re-parse that
backs a signed cert — to every flagship language**, while preserving the honest LEAD boundary that
already keeps unsound classes (inter-procedural taint, provenance-free randomness) at LEAD even in
Python.

There is a **third FACT route** for code, already language-agnostic: the **Red-Pen reproduction
engine** (`integration/vigil_integration/redpen/engine.py`) — an LLM lens proposes a PoC, the bwrap
sandbox reproduces it twice, and an O-1 oracle confirms it (request-side `*_BREAKOUT` + sandbox-signal
oracles). Injection-class FACTs on any language come via this route (a PoC that fires in the sandbox),
*not* via static re-parse. The sandbox runs an arbitrary `/bin/sh -c` (`live/sandbox_exec.py:152-163`),
so it is already language-neutral; only toolchain availability inside the box is language-specific.

---

## 1. Audit — what exists today (language-specific vs language-agnostic)

### 1.1 Already language-AGNOSTIC (no change, or signature already parameterized)

| Component | Evidence (`file:line`) | Note |
|---|---|---|
| File-type recognition / tree walk | `analysis/models.py:18-21` (`DEFAULT_EXTENSIONS` = `.py .js .ts .tsx .jsx .go .rb .php .java .rs .yaml .yml .tf .sh .sql`), `:34-46` (`iter_files`) | Multi-language already. |
| Normalized finding / analyzer contract | `analysis/models.py:49-104` (`AnalysisFinding`, `Analyzer` Protocol) | Language-neutral shape. |
| `Symbol` / `SymbolIndex` shape | `analysis/index.py:25-57` | Docstring `:10-11`: "`Symbol` shape is language-agnostic so they can be added". |
| Pattern analyzer (DAA builtin) | `analysis/analyzers/builtin.py:56-93` | Regex rules **per-extension**; several already fire on `.js .ts .rb .php` (`DAA-EVAL :57-58`, `DAA-MD-INNERHTML :91-92`). Shallow multi-lang, LEAD-grade. |
| Secret scanners | `analyzers/external.py:336` (gitleaks), `:451` (trufflehog) | File/entropy based — language-agnostic. |
| Semgrep adapter (engine) | `analyzers/external.py:124-211` | Semgrep is multi-language; only the **shipped rulesets** are language-bound (see 1.2). |
| DAA-fact write path | `codescan.py:194-324` (`run_codescan`) | Writes *any* DAA match (any extension) as a re-runnable spine fact. |
| Rule-re-run fix oracle | `codescan.py:341-401` (`build_code_fix_oracle`, `verify_finding_cleared`) | Regex re-match; `moved`/`path_removed` evasion guards (`:377-395`) are extension-driven and conservative. Language-agnostic. |
| `FindingContext` carrier + `to_verifier_context` | `verify/adapter.py:236-357`, `:2603-2690` | `static_rule` ctx key `:357`; `from_code_region(... language=)` already takes a `language` param. |
| **Single mint site** | `oracle_adapter.py:113-164` (`certify_admitted`), `:178-421` (`confirm_and_certify`) | Fully language-agnostic: drives the oracle over `oracle_context`, enforces known-class (`:297`), provenance G4 (`:359`), m-of-n sign (`:406-414`). **No language coupling.** |
| Red-Pen engine | `redpen/engine.py:197-311` | map→attack→reproduce→certify; `poc_command` is a shell string; lenses generic. |
| Sandbox executor | `live/sandbox_exec.py:152-226` (`build_bwrap_argv`, `run_sandboxed`) | Arbitrary `/bin/sh -c`, zero-egress (`--unshare-all`), workspace-confined, `ro_binds` for a dep cache. Language-neutral. |
| Red-Pen O-1 certify gate | `redpen/certify.py:57-65` (`_FACT_ELIGIBLE_ORACLE_KINDS`) | Request-side `*_BREAKOUT` + sandbox-signal + `static_rule`. Keyed on the fired **oracle kind**, not language. |
| Remediation (negative) cert | `remediation/remediation_cert.py:1-22`, fix oracle `remediation/fix_oracle.py:65-152` | Re-fires the oracle over the patched `oracle_context`, certifies "oracle-silent". Language-agnostic at the cert layer (depends only on the oracle re-firing). |
| Fix driver (clone / apply / PR) | `live/codefix_runner.py:285-432` (`clone`,`build`,`open_pr`) | `git apply` (`:350-364`) is language-agnostic; the build/test leg delegates to buildsys. |

### 1.2 Language-SPECIFIC today (hard-coded Python, or ad-hoc per-language spread)

| Component | Evidence (`file:line`) | Current scope |
|---|---|---|
| **STATIC_RULE oracle (the FACT re-parser)** | `verify/oracles.py:6144-6241` | **Python-only.** Hard-refuses non-Python (`:6212-6217`); `ast.parse` (`:6222`); tier helpers `_broken_crypto_hit`/`_insecure_flag_hit` dispatched at `:6132-6141` are Python-AST walkers (`:5728-5825`). Only two FACT-capable tiers: `broken-crypto-invocation`, `insecure-flag-literal` (`_FACT_RULE_IDS`, `:5696-5714`). |
| AST symbol index | `analysis/index.py:2,10-11,60-116` | stdlib `ast`, `.py`-only (`:116`). Docstring: "Other languages would need their own parser." |
| DAA→STATIC-FACT promoter | `codescan.py:146-154` (`_static_rule_for`), `:140-143` (`_STATIC_RULE_FOR_DAA`) | Returns `""` for any non-`.py` file (`:151-153`); maps only `DAA-WEAK-HASH`, `DAA-TLS-VERIFY-OFF`. |
| Retained-region builder | `codescan.py:157-191` (`_retain_source_region`) | Python `ast` import-provenance + enclosing-stmt retention. Python-only. |
| `from_code_region` language default | `static_facts.py:85-97`, `verify/adapter.py:951-1015` | `language="python"` default; only `"python"/"py"` honored downstream (oracle `:6212`). |
| Bandit adapter | `analyzers/external.py:225-315` | Python-only (bandit is a Python SAST). |
| Joern adapter | `analyzers/joern.py:14-29` | "**one language per run, and that language is Python**"; source/sink model Python-web-shaped (`:84-146`). Docstring `:26-29` **names exactly what multi-lang needs**: a per-language `importCode` pass + a distinct source/sink model per frontend. |
| **Build/test detector** | `remediation/buildsys.py:229-275` (`_detect_build_plan_inner`), `:294-340` (`compose_offline_test_command`) | **Already multi-language** — Python (`:236-246`), JS/TS (`:248-263`), Go (`:185-198,265-268`), JVM Maven/Gradle (`:200-212,270-273`). **Missing: Ruby, PHP, Rust, C/C++.** Ad-hoc `_has()`/`_any_ext()` dispatch. |
| Dependency-cache fetch | `remediation/depfetch.py` (pip) vs `remediation/js_depfetch.py` (npm/yarn/pnpm) | Per-language **cousins**. Go/JVM/Ruby/PHP/Rust fetchers missing. Both gated host-side fetch + offline install under `VIGIL_EGRESS_GUARD` (`depfetch.py:1-10`). |
| Coder cross-file context | `remediation/graph_context.py` (framework Python AST index) vs `remediation/js_graph_context.py` (regex import-graph) | Per-language cousins. `js_graph_context.py:3` states plainly: "the framework AST index is Python-only." |

**Audit conclusion.** There is **no `LanguagePlugin` abstraction today** (confirmed: no
`LanguagePlugin`/`register_language`/`LANGUAGE_PLUGINS` symbol anywhere in `integration/**` or
`framework/v2/{analysis,verify}/**`). The per-language logic is **scattered across ~8 sites** and has
grown *ad-hoc Python+JS (+Go/JVM for build only) cousins* (`depfetch`/`js_depfetch`,
`graph_context`/`js_graph_context`, buildsys's four branches). The single mint site, the sandbox, the
Red-Pen engine, the negative cert, the DAA-fact path and the `FindingContext`/`Symbol` shapes are
**already language-agnostic**. So the core deliverable is: **consolidate the scattered per-language
decisions behind one `LanguagePlugin` interface, and extend the one genuinely Python-only FACT
chokepoint — the `STATIC_RULE` re-parse — to each flagship language, soundly.**

---

## 2. The `LanguagePlugin` architecture (the 6 components)

### 2.0 The unifying decision: tree-sitter as the offline re-parse substrate

The FACT path's hard requirement is a **pure, offline, deterministic, in-process re-parse** (no network,
no exec) that re-derives a code property from retained bytes — exactly what Python's stdlib `ast` gives
today (`oracle.py:6222`). To get the same for other languages **without a toolchain per language at
re-parse time**, adopt **tree-sitter** (the `tree-sitter` Python binding + per-language precompiled
grammar `.so`) as the plugin substrate:

* offline and in-process (grammars are vendored/pinned `.so` blobs — no network at parse time);
* deterministic (same bytes → same CST), so the signed cert re-verifies offline exactly like the `ast`
  path;
* available for **every** flagship language (js, ts, go, java, ruby, php, rust, c, cpp);
* keeps Python on stdlib `ast` (already sound, zero-dep) — the plugin interface abstracts over *which*
  parser, so "add a language" = "vendor a grammar + write the tier predicates," never "bring a new
  parser toolchain into the FACT path."

tree-sitter is a **new offline dependency**: pin exact grammar versions, vendor the `.so` artifacts,
and **lazy-import it only inside the offense-plane re-parse, behind the default-OFF flag** (FATAL-2 +
byte-identical-off, see §5). It is never imported on the sovereign/console leg.

> Soundness caveat tree-sitter does **not** remove: a *concrete-syntax* match is not automatically a
> sound property. The two FACT-capable tiers are **call-shape + import-provenance** properties (see
> §3), which a CST exposes faithfully; anything needing dataflow/inter-procedural reasoning stays
> LEAD-only in **every** language (the existing Python boundary, `oracle.py:5702-5714`), and the plan
> **must not** regress that.

### 2.1 The interface (a bounded, additive unit per language)

A `LanguagePlugin` is the single object a language supplies. Proposed location:
`integration/vigil_integration/langplugins/` (offense-plane, import-clean, lazy) with a registry
`langplugins/registry.py` keyed by canonical language id. Each plugin provides the **six** components:

```
LanguagePlugin
├─ id: "python" | "javascript" | "typescript" | "go" | "java" | "ruby" | "php" | "rust" | "c" | "cpp"
├─ (1) detection:        extensions: frozenset[str]; manifests: frozenset[str];
│                        detect(path) -> bool
├─ (2) parse:            parse_symbols(source, path) -> list[Symbol]          # → the agnostic Symbol shape
│                        retain_region(source, line) -> str                   # import-provenance + enclosing decl
├─ (3) rules:            taint_ruleset_dir() -> Path                          # semgrep YAML(s), like the 2 today
│                        pattern_rules() -> tuple[PatternRule, ...]           # DAA regex, per-ext
├─ (4) sandbox recipe:   build_plan(repo) -> BuildPlan                        # compile floor + real test cmd
│                        offline_install(plan, cache_in_box) -> str           # the /bin/sh -c composer
│                        depfetch(repo, cache_dir) -> FetchResult             # GATED host-side fetch
├─ (5) oracle mapping:   static_tier_predicate(rule_id, cst) -> (bool, str)   # the FACT re-derivation
│                        fact_capable_rule_ids: frozenset[str]                # subset that can mint a FACT
└─ (6) PCR fix:          graph_context(repo, file) -> list[str]              # cross-file neighborhood
                         (fix-apply + negative-cert reuse the agnostic layer; see §2.7)
```

The registry is the **only** new dispatch point. Every existing scattered site becomes a thin call
into `registry.for_path(path)` / `registry.for_language(id)`:

* `codescan._static_rule_for` (`:146-154`) → `plugin.fact_capable_rule_ids` + `plugin.detect`;
* `codescan._retain_source_region` (`:157-191`) → `plugin.retain_region`;
* `analysis/index.build_symbol_index` (`:109-116`) → `plugin.parse_symbols`;
* `buildsys._detect_build_plan_inner` (`:229-275`) → `plugin.build_plan`;
* `remediation.graph_context` / `js_graph_context` → `plugin.graph_context`;
* `remediation.depfetch` / `js_depfetch` → `plugin.depfetch`;
* `verify/oracles.static_rule_oracle` (`:6144`) → dispatch `_static_tier_hit` through
  `registry.for_language(ctx["language"]).static_tier_predicate` (§2.6).

### 2.2 Component (1) — file / ecosystem detection

**Today:** scattered — `DEFAULT_EXTENSIONS` (`models.py:18-21`), per-rule `extensions` in the pattern
analyzer (`builtin.py:132-133`), the `.py` test in `_static_rule_for` (`codescan.py:151`), and
`_has()`/`_any_ext()` manifest probes in buildsys (`buildsys.py:53-74,237,249`).

**Plugin:** `extensions`, `manifests` (e.g. `go.mod`, `package.json`, `Gemfile`, `composer.json`,
`pom.xml`, `Cargo.toml`), and `detect(path)`. `DEFAULT_EXTENSIONS` becomes the **union** of all
registered plugins' extensions (so the tree walk is unchanged in behavior, derived not hard-coded).
Pure + total (mirrors buildsys's hostile-repo totality, `:216-226`).

### 2.3 Component (2) — parser/AST → the agnostic `Symbol` / retained region

**Today:** `analysis/index.py` (Python `ast` → `Symbol`, `:60-116`) and
`codescan._retain_source_region` (Python `ast` → retained import+enclosing region, `:157-191`).

**Plugin:** `parse_symbols(source, path) -> list[Symbol]` emitting the **existing** `Symbol` shape
(`index.py:25-32`: kind ∈ {function,class,import,call}, name, path, line, detail) — Python via `ast`,
others via tree-sitter queries. `retain_region(source, line) -> str` reproduces the Python retention
contract **per language**: the module/file's **import statements** (the provenance the oracle resolves)
+ the **top-level/enclosing declaration** containing `line`, concatenated so it re-parses standalone
(`codescan.py:179-191`). This is the bytes that travel verbatim into the cert so the FACT re-verifies
offline and a tamper that removes the property no longer re-fires (`codescan.py:162-164`). Fail-closed:
an unparseable region falls back to whole-file then one-line, and the oracle then REFUSES → LEAD (never
a crash), exactly as today (`codescan.py:170-174`).

### 2.4 Component (3) — taint + rule set

**Today:** two shipped semgrep rulesets `analysis/rules/taint-python.yaml`,
`taint-javascript.yaml` (`analyzers/external.py:118-121,136` point semgrep at `rules/`), plus the
per-extension regex pack in `builtin.py:56-93`.

**Plugin:** `taint_ruleset_dir()` returns the plugin's semgrep YAML(s); the Semgrep adapter is handed
the **union** of registered rulesets (semgrep is already multi-language — only the *shipped rules* were
the limit). `pattern_rules()` returns the plugin's DAA regex rules (migrating `builtin._RULES`'
per-ext entries into their owning plugins; the merged set is byte-identical to today's `_RULES`).
These feed grade-(1) **DAA-facts** and grade-(3) **semgrep LEADs** — already multi-language; this
component is mostly *consolidation + rule authoring*, the lowest-risk per-language work.

### 2.5 Component (4) — sandbox-run recipe (toolchain image + build/run a PoC)

**Today:** `remediation/buildsys.py` is already a 4-language dispatcher (Python/JS/Go/JVM,
`:229-275` + composers `:294-340`); `depfetch.py`/`js_depfetch.py` are the gated host-side cache
fillers; `codefix_runner` runs the composed command in `run_sandboxed` (`codefix_runner.py:378-431`)
and **degrades to apply-check-only** when no build system is detected (`:391`).

**Plugin:** `build_plan(repo) -> BuildPlan` (the dependency-free **compile/syntax floor** + the **real
test command**), `offline_install(plan, cache_in_box) -> str` (the `/bin/sh -c` that installs OFFLINE
from the mounted read-only cache then runs the suite — mirrors `compose_offline_test_command`'s
per-language branches, `buildsys.py:304-340`), and `depfetch(repo, cache) -> FetchResult` (the
**gated** host-side fetch, `VIGIL_EGRESS_GUARD`, mirroring `depfetch`/`js_depfetch`). The sandbox
itself is unchanged (`run_sandboxed`, zero-egress, `ro_binds` for the cache).

**Key honesty that keeps the PR tractable:** the build/test gate (Axis A) is **optional** — the FACT
and its fix-verification do **not** depend on it. A static finding's fix is verified by the **re-parse
re-fire** (`STATIC_RULE` goes silent) or the **DAA re-run** (`verify_finding_cleared`), *not* by the
test suite (`codescan.py:341-401`; `codefix_runner.py:391` ships "apply-check only" as a first-class
outcome). So a language can ship **FACT-grade findings + proof-carrying fix** with only the
light/offline re-parse; the heavy toolchain image buys only the *additive* "passes the real suite"
confidence tier. This is why 7 languages fit in one PR: the FACT path needs tree-sitter, not a JDK.

### 2.6 Component (5) — the oracle mapping that mints a FACT

**This is the one genuinely Python-only chokepoint and the heart of the work.**

**Today:** `static_rule_oracle` (`verify/oracles.py:6144-6241`) refuses non-Python (`:6212-6217`),
parses with `ast` (`:6222`), and dispatches two FACT-capable tiers (`_static_tier_hit`, `:6132-6141`)
to Python-AST predicates. A FACT is minted **only** through the single site
(`oracle_adapter.certify_admitted` → `confirm_and_certify` → `build_certificate`/`sign_certificate`,
`:113,406,414`), which is already language-agnostic.

**Plugin + oracle change (minimal, additive, same `OracleKind`):**

1. The `static_rule` ctx already carries `language` (`adapter.py:1011`; oracle reads it at `:6176`).
   Replace the hard `language not in ("python","py")` refuse (`:6212-6217`) with a **registry
   dispatch**: `plugin = registry.for_language(language)`; if none, REFUSE → LEAD (identical
   fail-closed shape — an unregistered language is exactly today's non-Python refuse). If present,
   `plugin.parse(source)` (tree-sitter / `ast`) then `plugin.static_tier_predicate(rule_id, cst)`.
2. **No new `OracleKind`.** Multi-language routes through the **same** `STATIC_RULE` kind via the
   `language` field — so `_ALL_ORACLES` stays frozen at 15 and `make gate` is byte-identical
   (mirrors `test_gate_byte_identical.py:151-158`, which notes `STATIC_RULE` "is keyed on a
   `static_rule` ctx field no benchmark/scan finding carries").
3. **The FACT-capable vocabulary is unchanged and per-language-sound:** only `broken-crypto-invocation`
   and `insecure-flag-literal` (`_FACT_RULE_IDS`, `oracle.py:5696-5714`). Each plugin re-derives these
   **as a single-region call-shape + import-provenance property** in its own CST (concrete per-language
   forms in §3). `insecure-randomness-sink` and `direct-taint` stay **hard LEAD-only in every language**
   (`_LEAD_ONLY_RULE_IDS`, `:5711`) — the plan must preserve, not widen, this boundary.
4. **The mint remains the single site.** Nothing about `certify_admitted` changes. The M3 anti-relabel
   rule-id→class derivation (`adapter.py:977-1001`; mint-choke re-assert `oracle_adapter.py:315-330`)
   is language-independent (it binds on `rule_id`, not language) and keeps holding.

**Request-side + sandbox-signal FACTs (injection, etc.) are already multi-language** — they are minted
by the Red-Pen reproduction route (`redpen/certify.py:57-65`) over PoC bytes the sandbox produced, not
by static re-parse. A plugin contributes to that route only by (a) its lenses proposing a
language-appropriate `poc_command` and (b) its toolchain being present in the sandbox — both already
supported by the agnostic engine.

### 2.7 Component (6) — PCR fix support

**Today:** `codefix_runner` clones, `git apply`s the approved diff (`:350-364`), runs the buildsys gate
(`:378-431`), and opens a PR under m-of-n (`:433+`, `build_destruction_quorum :567`). `graph_context`
(Python AST) / `js_graph_context` (regex) feed the coder. The **negative cert** (`remediation_cert.py`,
`vigil-remediation-cert-v2`) re-fires the oracle over the patched `oracle_context` and certifies
"oracle-silent" (`:81-88`), signed Ed25519, offline-re-verifiable (3 checks, `:9-14`).

**Plugin:** supplies only `graph_context(repo, file)` (cross-file neighborhood — Python uses the
`Symbol` index, others a tree-sitter import/caller graph; consolidates `graph_context`/`js_graph_context`).
**Everything else is reused agnostically:** `git apply` is language-neutral; the build/test gate is
component (4); the negative cert re-fires the **same** `STATIC_RULE` re-parse (now language-aware via
§2.6) or the **language-agnostic** DAA re-run (`verify_finding_cleared`). So "the signed negative cert
re-fires in that language" is satisfied by the component-(5) dispatch — **no new cert machinery per
language.**

### 2.8 Why "adding a language" is a bounded additive unit

A new/niche language is: vendor one pinned tree-sitter grammar; write (2) `parse_symbols`/`retain_region`
queries; (3) a semgrep YAML + a few regex rules; (4) a `BuildPlan` branch + offline composer + a gated
fetch (optional — apply-check works without it); (5) the two tier predicates (broken-crypto,
insecure-flag) over the CST + fixtures; (6) a tree-sitter import graph. **No core edit** — it registers.
That is the definition of "any language is true": the registry dispatch and the single mint site never
change again.

---

## 3. Per-flagship-language sections (what each needs, real cost, offline parser/toolchain)

For each language the **FACT-path cost** is dominated by: **(P)** an offline in-process parser for the
re-parse, and **(T)** toolchain weight for the *optional* build/test gate. The two FACT-capable tiers
have a concrete per-language form; the request-side/injection FACTs ride the (already-agnostic) Red-Pen
sandbox route.

### 3.1 Python — the reference (already shipped)
* (P) stdlib `ast` — zero-dep, sound today. (T) venv + pip wheelhouse — light, **done** (`buildsys.py:236-246`, `depfetch.py`).
* Tiers: `broken-crypto-invocation` (`hashlib.md5`/`Crypto.Cipher.DES`, provenance via import, `oracle.py:5716-5825`), `insecure-flag-literal` (`verify=False`). **Cost: 0 (reference).**

### 3.2 JavaScript / TypeScript
* (P) tree-sitter-javascript / -typescript (offline, pinned). (T) node + npm/yarn/pnpm offline cache — **done** (`buildsys.py:248-263`, `js_depfetch.py`, `js_graph_context.py`).
* Tiers: broken-crypto `crypto.createHash('md5'|'sha1')`, `createCipheriv('des'|'rc4'|...)`; insecure-flag `rejectUnauthorized:false`, `NODE_TLS_REJECT_UNAUTHORIZED=0`. Call-shape + `require`/`import` provenance — sound single-region.
* **Cost: low–moderate** (grammar + tier predicates + TS generics; build/graph already exist). TS = JS grammar variant, not a separate ecosystem.

### 3.3 Go
* (P) tree-sitter-go (offline, in-process — avoids needing the Go toolchain just to re-parse). (T) `go test ./...`, vendored=offline / GOMODCACHE — build **done** (`buildsys.py:185-198,265-268`); **depfetch for Go is missing** (new, small).
* Tiers: broken-crypto `crypto/md5`,`crypto/des`,`crypto/rc4` (import-path provenance is **cleaner** than Python — the import path *is* the crypto namespace); insecure-flag `tls.Config{InsecureSkipVerify: true}`. Sound single-region.
* **Cost: moderate** (grammar + predicates + a Go depfetch; build composer exists).

### 3.4 Java
* (P) tree-sitter-java (offline), or pure-Python `javalang` (pip, offline) — **two** offline options, no JDK needed to re-parse. (T) Maven `mvn -o test` / Gradle `--offline` — build **done** (`buildsys.py:200-212,270-273`); depfetch (`~/.m2` / gradle cache) **missing** (new, heavier).
* Tiers: broken-crypto `MessageDigest.getInstance("MD5"|"SHA1")`, `Cipher.getInstance("DES"|"RC4"|"AES/ECB/...")` — the broken primitive is a **string-literal argument**, so the predicate reads the literal (per-language nuance, still sound single-region); insecure-flag an all-trusting `TrustManager`/`HostnameVerifier` or `setHostnameVerifier(ALLOW_ALL)`.
* **Cost: moderate** (grammar/javalang + literal-arg predicates). JDK toolchain image is **heavy** → the real-suite tier is owner-gated (§6); FACT + apply-check ship.

### 3.5 Ruby
* (P) tree-sitter-ruby (offline). (T) `bundle exec rspec`/`rake test`, `vendor/bundle` offline — **buildsys branch missing** (new); gated `bundle` fetch missing.
* Tiers: broken-crypto `Digest::MD5`/`Digest::SHA1`, `OpenSSL::Digest.new('MD5')`, `OpenSSL::Cipher.new('DES'|'RC4')`; insecure-flag `OpenSSL::SSL::VERIFY_NONE`, `verify_mode = VERIFY_NONE`. Constant/call provenance via `require`. Sound single-region.
* **Cost: moderate** (grammar + predicates + a Ruby BuildPlan branch + gated fetch).

### 3.6 PHP
* (P) tree-sitter-php (offline). (T) `composer install` + `vendor/`, phpunit — **buildsys branch missing** (new); gated composer fetch missing.
* Tiers: broken-crypto `md5()`/`sha1()` **builtins** (globally resolvable — **no import**, so the provenance floor is "known builtin", a per-language rule), `hash('md5'|'sha1', …)`, `openssl_encrypt(..., 'des-...'|'rc4')`; insecure-flag `CURLOPT_SSL_VERIFYPEER => false`, `"verify" => false` (Guzzle). Sound single-region, with the builtin-provenance nuance documented.
* **Cost: moderate** (grammar + predicates incl. the no-import builtin case + BuildPlan branch).

### 3.7 Rust
* (P) tree-sitter-rust (offline). (T) `cargo test`, vendored registry (`cargo vendor`) offline — **buildsys branch missing**; cargo compiles are **heavy**.
* Tiers: broken-crypto `md5::`/`sha1::`/`md-5` crate calls, `Des`/`Rc4` from `openssl`/`RustCrypto` — provenance via `use` paths; insecure-flag `danger_accept_invalid_certs(true)` (reqwest), `SslVerifyMode::NONE`. Call-shape sound, **but** idiomatic Rust crypto spreads across crates/trait methods → higher false-negative risk for a single-region predicate.
* **Cost: moderate (parser+rules) → high (sound FACT tier).** **Recommendation:** ship Rust in the PR at **DAA-fact + fix-apply + DAA-re-run** grade; the oracle-STATIC-FACT tier is the **first additive follow-on** (needs the crate-provenance predicate red-penned). Honest, not deferred-forever.

### 3.8 C / C++
* (P) tree-sitter-c / -cpp (offline). (T) `make`/`cmake` + a compiler — build is **project-specific**, hard to generalize; no standard offline dep cache.
* Tiers: broken-crypto `MD5()`/`SHA1()`/`EVP_md5()`/`DES_*`/`RC4` (OpenSSL) via `#include` provenance; insecure-flag `SSL_CTX_set_verify(ctx, SSL_VERIFY_NONE, …)`, `curl_easy_setopt(..., CURLOPT_SSL_VERIFYPEER, 0)`. Call-shape is visible, **but the C preprocessor** (macros, conditional `#include`) makes *sound* provenance genuinely hard for a pre-preprocessing CST.
* **Cost: high.** **Recommendation:** ship C/C++ in the PR at **DAA-fact + fix-apply** grade (detection + regex/semgrep rules + apply-check fix + DAA re-run). The oracle-STATIC-FACT re-parse tier **and** any build/test image are **owner-gated/deferred** (§6) — I will not claim a sound macro-expanded C crypto detector in one red-pennable diff.

---

## 4. The exact 1-PR scope (explicit + honest)

### 4.1 In the PR

**A. The plugin framework (core):**
1. `langplugins/` package + `registry.py` (dispatch by path/language), import-clean, lazy, offense-plane
   only.
2. `LanguagePlugin` interface (the 6 components, §2.1) + the `PatternRule`/`BuildPlan`/`FetchResult`
   typed carriers (reuse `BuildPlan` from `buildsys.py:40-50`).
3. Refactor the scattered sites (§2.1 list) to call the registry — **behavior-preserving for Python**
   (byte-identical when the flag is off; see §5).
4. The `static_rule_oracle` registry dispatch (§2.6) replacing the hard Python refuse
   (`oracle.py:6212-6217`) — **same `OracleKind`, same `_FACT_RULE_IDS`, same single mint site.**
5. `DEFAULT_EXTENSIONS` derived from the registry (union), replacing the hard-coded tuple
   (`models.py:18-21`) — same effective set.
6. tree-sitter vendored + pinned (grammars as build artifacts), lazy-imported behind the flag.

**B. Flagship languages shipped at FULL FACT grade:** **Python** (reference, already there),
**JavaScript/TypeScript**, **Go**, **Java**, **Ruby**, **PHP** — each with all 6 components: detection,
tree-sitter (or `ast`) parse+retain, taint+pattern rules, build/test recipe (+ gated depfetch; JVM
real-suite image owner-gated), the two FACT-capable tier predicates, graph-context, and PCR
negative-cert reuse. Per-language fixtures with a minted FACT + a same-shape benign control (§7).

**C. Flagship languages shipped at DAA-fact + proof-carrying-fix-apply grade (FACT re-parse deferred):**
**Rust**, **C/C++** — detection, pattern/semgrep rules (already multi-lang-capable), apply-check +
DAA-re-run fix-verification, buildsys branch where feasible. Their oracle-STATIC-FACT tier is the named
first follow-on (§4.2). **Stated honestly in the PR description**, not silently.

Justification for this cut: the FACT tier needs only the **offline re-parse** (tree-sitter) + **two
sound single-region predicates**; Python/JS/Go/Java/Ruby/PHP all satisfy that in one diff and are
red-pennable with fixtures. Rust and C/C++ have a *genuinely harder* sound predicate (crate-spread
crypto / preprocessor), so claiming their FACT tier in the same diff would be un-red-pennable — exactly
the overclaim the constitution forbids.

### 4.2 Additive follow-on (NOT in the PR — each a bounded plugin/slice)

* Rust oracle-STATIC-FACT tier (crate-provenance crypto predicate + fixtures).
* C/C++ oracle-STATIC-FACT tier (preprocessor-aware provenance) + a C/C++ build/test image.
* JVM / Ruby / PHP / Go **real-suite** heavy toolchain sandbox images + their gated depfetchers (the
  FACT + apply-check ship without them).
* Any niche language (Kotlin, Swift, Scala, C#, Elixir, …) — a new plugin, `register()`, done. No core
  change. **This is the architectural proof that "any language" is true.**
* Joern multi-frontend wiring (`analyzers/joern.py:26-29`) for deep inter-procedural LEADs per language
  — orthogonal LEAD-breadth work, not on the FACT path.

### 4.3 Why not "every language fully in one diff"
Un-red-pennable by construction: a sound FACT predicate needs a passing **same-shape benign control**
per language (§7), and a heavy toolchain image per language inflates the diff past review. The
architecture makes the *claim* true (a new language is a small additive plugin), which is the honest
form of "any language."

---

## 5. Invariants per component (the 5 non-negotiables)

**(I1) A FACT only via the oracle (single mint site).**
Unchanged. Every grade-(2) FACT still flows `static_rule_oracle` (now language-dispatched) →
`certify_admitted`/`confirm_and_certify` → `build_certificate`/`sign_certificate`
(`oracle_adapter.py:113,406,414`). Plugins supply *predicates and bytes*, never a verdict or a
signature. The M3 rule-id→class binding (`adapter.py:977-1001`, mint-choke `oracle_adapter.py:315-330`)
is language-independent and preserved. DAA-facts remain the honest `verified-no-pr` grade with no signed
cert (`codescan.py:327-338`).

**(I2) Target-touch via the gate.**
The codebase path is **local** (no target touch): `run_codescan`, the re-parse, `git apply`, and the
bwrap build/test gate touch only local files. Whole-tree deep analysis stays gated on
`Capability.DEEP_STATIC_ANALYSIS` (OFFENSIVE tier, `orchestrator.py:9,82-83`) — unchanged, applies to
every language. The **only** egress is the per-language **gated depfetch** (host-side, operator-approved,
`VIGIL_EGRESS_GUARD`), mirroring `depfetch.py:1-10` / `js_depfetch.py`; the sandbox install stays
**offline** from the mounted read-only cache (never a network-capable box, `sandbox_exec.py:1-33`).

**(I3) FATAL-2 lazy offense imports.**
`langplugins`, tree-sitter, and every grammar are imported **function-local**, offense-plane only —
mirroring `oracle_adapter.py:139,219-224`, `codescan.py:209-215`, `redpen/engine.py:31-37`,
`remediation_cert.py:20,84`. The sovereign/console leg never imports a plugin or tree-sitter. The
`static_rule_oracle` dispatch lazy-loads the plugin **inside** the oracle call (the oracle module
itself already lives behind the verify layer the sovereign leg does not drive for mint).

**(I4) Default-OFF `VIGIL_*` flag + byte-identical when off.**
New flag `VIGIL_MULTILANG_CODESCAN` (keyless env read, offense-plane, mirroring `VIGIL_REDPEN_ENABLED`
`redpen/engine.py:41-49`). **OFF (default):** `static_rule_oracle` takes the **existing** non-Python
refuse path for every non-Python language (identical bytes/verdict to `oracle.py:6212-6217`);
`_static_rule_for` returns `""` for non-`.py` (today's behavior, `codescan.py:151-153`); no tree-sitter
import occurs. Because multi-lang uses the **same** `OracleKind.STATIC_RULE` keyed on a ctx field no
benchmark/scan finding carries, `_ALL_ORACLES` stays frozen at 15 and `make gate` is **byte-identical**
(assert via the existing `test_gate_byte_identical.py:151-158,200-221` pattern). The DAA-fact path (grade 1)
is already multi-language and is **not** gated by this flag — it changes no verdict; only the new
**oracle-FACT** re-parse for non-Python is flag-gated.

**(I5) Clearances = signed, offline-re-verifiable negative certs.**
Unchanged and reused. A per-language clearance is the **same** `vigil-remediation-cert-v2`
(`remediation_cert.py:32`, Ed25519, 3-check offline re-verify `:9-14`), minted only when the oracle
**re-fires and goes silent** over the patched retained region (`:81-88`) — the language-aware
`STATIC_RULE` re-parse (§2.6) or the language-agnostic DAA re-run (`verify_finding_cleared`). No new
cert schema, no new signer, per language.

**Offense-plane / sandbox touches to flag for review:**
* The `static_rule_oracle` edit is on the **offense** verify plane (it already is) — but the re-parse
  stays a **pure, offline, in-process, non-exec** operation (tree-sitter `.parse(bytes)`), the same risk
  class as today's `ast.parse` (`oracle.py:6222`). No egress, no subprocess.
* The build/test gate runs in the **existing** bwrap sandbox (`sandbox_exec.py`), unchanged floors.
* Each new gated **depfetch** is a host-side egress step → routes through `VIGIL_EGRESS_GUARD` exactly
  like `depfetch`/`js_depfetch`; gate-marshal/red-pen review per language.

---

## 6. Honest owner-gated / deferred

* **Heavy toolchain sandbox images** (JDK/Maven+Gradle, Rust/cargo, C/C++ compilers, and their offline
  dep caches): the **real-suite** tier only. FACT + apply-check ship without them. Owner-gated because
  each is a large image + a gated fetcher that needs its own red-pen/gate-marshal pass.
* **Rust oracle-STATIC-FACT tier:** deferred to the first follow-on — the sound crate-provenance crypto
  predicate needs its own same-shape control corpus before it can mint.
* **C/C++ oracle-STATIC-FACT tier:** deferred — the preprocessor defeats a sound single-region
  provenance re-parse on pre-expanded source; needs a design (compile-commands-driven expansion) I will
  not hand-wave into one PR.
* **Any language whose offline in-process parser isn't feasible** (e.g. one with no maintained
  tree-sitter grammar): ships at **DAA-fact grade only** (regex/semgrep LEADs + DAA re-run fix), with the
  FACT tier explicitly marked unavailable — never a silent false FACT. (This is the same honest downgrade
  the Python path already applies to LEAD-only classes, `oracle.py:5702-5714`.)
* **Inter-procedural / provenance-resolved taint and insecure-randomness FACTs:** remain LEAD-only in
  **every** language (including Python today) — explicitly **not** promoted by this PR
  (`_LEAD_ONLY_RULE_IDS`, `oracle.py:5711`). Injection FACTs come via the Red-Pen sandbox PoC route, not
  static re-parse.

---

## 7. Test / verification strategy

**Per-language fixture pairs (the prover/red-pen corpus).** Mirror
`analysis/benchmark/python/{vulnerable,sanitized}.py` + `ground-truth.json`. For each FACT-capable
language and each FACT tier, ship:
* a **vulnerable** fixture that mints a signed FACT end-to-end
  (`run_codescan` → `STATIC_RULE` re-parse fires → `certify_admitted` signs → `framework.v2 verify`
  re-fires offline), and
* a **same-shape benign control** that must **NOT** fire — the critical near-miss per tier:
  broken-crypto → the *strong* primitive in the same call shape (sha256 / AES-GCM), or the explicit
  opt-out (Python's `usedforsecurity=False`, `oracle.py:5805-5824`; the per-language analogue);
  insecure-flag → verification *enabled* in the same assignment shape. A sound detector **refuses** the
  control (this is the fake-fixture discipline — the control must exercise the adversarial shape, not a
  cosmetically different one).

**Byte-identical gate when OFF.** Extend `test_gate_byte_identical.py` to assert: with
`VIGIL_MULTILANG_CODESCAN` unset, (a) `_ALL_ORACLES == 15`, (b) a non-Python `static_rule` ctx takes the
identical refuse path, (c) `_static_rule_for` returns `""` for non-`.py`, (d) no tree-sitter module is
imported (assert via `sys.modules`). `make gate` digest unchanged.

**FATAL-2 guards.** A test asserting the sovereign/console import graph pulls **no** `langplugins`,
`tree_sitter`, or grammar module (extend the existing two-env import-cleanliness tests). Every `framework`
/ plugin import in the new code is function-local (lint + test).

**Fix-verification + negative-cert per language.** For each FACT-capable language: apply a correct fix,
assert `verify_finding_cleared`/`STATIC_RULE` goes silent, mint `vigil-remediation-cert-v2`, and re-verify
it offline (3 checks). Plus the **evasion** controls already in Python (`codescan.py:377-395`): a moved
file (`moved`) and a rename-to-unscanned-extension (`path_removed`) must **not** read as cleared.

**Oracle soundness (re-execution, not string trust).** The re-parse must re-derive the property from the
retained bytes; a test that **mutates the retained region to remove the property** must flip the verdict
to silent (tamper-no-refire, `codescan.py:162-164`). A DAA rule_id/CWE is never trusted — only `path`/`line`
point at where to retain (`codescan.py:132-139`).

**Red-pen / adversarial-sweep per FACT-capable language** before merge: construct a false positive for each
new tier predicate; confirm the same-shape control holds; confirm no offense drift and the gate is
byte-identical off.

---

## 8. Acceptance checklist — what "100% (any language)" looks like

1. **One registry, one mint site.** All per-language dispatch flows through `langplugins/registry`; the
   single FACT mint site (`certify_admitted`) and the single `OracleKind.STATIC_RULE` are unchanged. ✅ = no
   second mint path, no new OracleKind.
2. **Flagship FACT set live:** Python, JS/TS, Go, Java, Ruby, PHP each mint a signed, offline-re-verifiable
   `broken-crypto-invocation` **and** `insecure-flag-literal` STATIC FACT from a fixture, with a passing
   same-shape benign control. ✅
3. **Flagship fix set live:** each of the above produces a proof-carrying fix (apply-check minimum;
   negative `vigil-remediation-cert-v2` re-verifies offline). ✅
4. **DAA-fact + fix grade for Rust & C/C++** (detection, rules, apply-check, DAA re-run), with the FACT
   tier **honestly marked** as follow-on. ✅
5. **Byte-identical off:** `VIGIL_MULTILANG_CODESCAN` unset ⇒ `make gate` digest unchanged, no tree-sitter
   import, Python behavior identical. ✅
6. **FATAL-2 clean:** sovereign/console imports no plugin/tree-sitter; all offense imports function-local. ✅
7. **Target-touch gated:** only the per-language gated depfetch egresses (operator-approved,
   `VIGIL_EGRESS_GUARD`); the sandbox installs offline; `DEEP_STATIC_ANALYSIS` still gates whole-tree. ✅
8. **LEAD boundary preserved:** `insecure-randomness-sink` and `direct-taint` remain LEAD-only in every
   language; no new class mints a FACT the single-region re-parse cannot soundly prove. ✅
9. **"Any language" demonstrated:** a *new* language (e.g. Kotlin) can be added as a plugin that
   registers, ships a grammar + 6 components + fixtures, and mints a FACT — **with zero edits to the
   registry, the oracle, the mint site, or the cert**. ✅ = the architecture, not a special case.
10. **Honesty:** every claim in the PR description maps to a passing fixture or is named as owner-gated/
    follow-on; no overclaim (constitution XI.10, claim-discipline). ✅

---

## 9. Build order (within the one PR)

1. Interface + registry + typed carriers; wire `DEFAULT_EXTENSIONS` from the registry (no behavior change).
2. Extract Python into the first plugin (prove byte-identical; this is the refactor safety net).
3. `static_rule_oracle` registry dispatch behind `VIGIL_MULTILANG_CODESCAN` (Python path unchanged off/on).
4. JS/TS plugin (reuses existing buildsys/js_depfetch/js_graph_context — proves consolidation).
5. Go, Java, Ruby, PHP plugins (parse+retain+tiers+rules+buildsys branch+fixtures each).
6. Rust, C/C++ at DAA-fact + fix grade (rules + apply-check + DAA re-run).
7. Byte-identical/FATAL-2/gate tests; per-language red-pen + adversarial-sweep.
8. Docs: capability-matrix rows per language (FACT-capable vs LEAD-only, real-suite owner-gated), PR
   description stating the exact in-PR vs follow-on cut.
