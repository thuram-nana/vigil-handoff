---
name: vigil-strix-shell-hosting-decision
description: "Approved hosting architecture for the Strix-style (WE-host, WE-hold-keys) VIGIL product — apply at the S5+ isolation/scale infra wave, not before."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-27T19:14:15.951Z
---

APPROVED 2026-10-27 (user: "proceed with this approach when the time comes"). This is for the **Strix-style managed version** of the "Strix shell on VIGIL's spine" program — WE host it and WE hold the keys (per sigilsovereign.com positioning) — which is DISTINCT from the already-existing **customer-sovereign VIGIL** (the customer self-hosts and holds their own keys; that one deploys into the customer's cloud/on-prem as Terraform/Helm). Do NOT conflate the two: the sovereignty/customer-key-custody concern applies to the OTHER product; here the offensive egress + key custody are OUR responsibility.

**How Strix does theirs (grounding, from usestrix/strix OSS + docs; app.strix.ai internals are inference):** execution = a **sandbox-per-scan** — an isolated Docker sandbox per engagement with a tool server (terminal/browser/file HTTP API), a **Caido proxy logging all HTTP/HTTPS**, a shared `/workspace`, and **an isolated network with controlled internet access**; multiple agents collaborate inside one sandbox. The OSS core runs in your own Docker; `app.strix.ai` is a **control-plane SaaS** orchestrating those sandboxes + continuous scan + repo/domain connect + integrations. This maps 1:1 to VIGIL: Strix sandbox-per-scan = VIGIL **S5+ ephemeral per-job worker**; Strix controlled-egress+proxy = VIGIL **egress gating + evidence capture**; Strix OSS-self-host vs app.strix.ai = VIGIL **sovereign version vs this we-host version**. So the hosting topology IS the FATAL-2 two-plane split.

**The approved architecture (apply at S5+):**
- **Control plane** (UI, queue, governance, signed spine, results): **managed Kubernetes** (the ephemeral-worker + autoscaling orchestrator).
- **Execution plane**: **ephemeral, strongly-isolated microVM/container sandboxes — one per engagement, torn down after** (the direct analog of Strix's sandbox-per-scan + VIGIL's S5+ worker).
- **Egress**: **dedicated, PUBLISHED, pinned IPs** (NAT/BYOIP) so customers allowlist + traffic is attributable (matches VIGIL OPSEC "stable correlatable source IP"); **settle the provider security-testing AUP EARLY** — scanning authorized third-party targets from a hyperscaler is a go-to-market gate, not a footnote (vendors expose this as "private locations").
- **Keys (WE hold them)**: cloud **KMS/HSM** (per-tenant DEKs) + **confidential compute** for the vault — the TPM-sealed vault maps onto KMS + a confidential VM/enclave. "We hold the keys" is OUR assurance story, so confidential compute is what makes it credible.

**Provider pick — DEFAULT = AWS** (best fit for THIS product because of **Firecracker**, built for ephemeral-isolated-sandbox-per-request = Lambda/Fargate): EKS (control plane) + Fargate/Firecracker (per-engagement sandboxes) + **Nitro Enclaves** (vault; host operator can't read keys) + KMS + BYOIP egress. **GCP** (GKE Autopilot + gVisor/GKE Sandbox) = strong close alternative. **Azure** (AKS + Confidential Containers) = pick if a GOVERNMENT buyer shows up. NOT a single VPS (no isolation/autoscale/HA/confidential-compute); the current Contabo VPS was fine only for the single-tenant stage.

**NEXT when the time comes:** at the S5+ wave, ship this as a Terraform/Helm reference-arch skeleton in the repo (EKS + a Firecracker/Fargate worker pool wired to the S5+ SecurityJob queue + NAT egress w/ static EIP + KMS/Enclave vault) so S5+ is deployable infra, not a design note. Related: [[vigil-strix-shell-mcp-wave]] · [[vigil-sovereign-fabric-vision]].
