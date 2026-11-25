---
name: vigil-data-retention-decisions-2026-11-24
description: Owner-approved data retention schedule for hosted VIGIL (2026-11-24, decision 3) and what enforces each item
metadata:
  type: project
---

Approved by the owner 2026-11-24:
- Raw scan evidence (captured requests/responses from customer targets): **90 days after the assessment finishes**, then
  deleted automatically; findings + reports kept. NEEDS BUILDING (evidence clean-up sweep) — check proof-bundle/PCF
  dependence on evidence before deleting.
- Findings/reports: while account open; deleted on request after closure (manual first).
- Customer-supplied credentials (target logins, repo/cloud connections): until disconnected / assessment ends; removed at closure.
- Account details: while open; revoked immediately at closure (decommission, built).
- Billing records/invoices: **10 years** (OHADA accounting).  Security/audit logs: **3 years**.
- Web server logs: **30 days** (set on deploy day).  Server backups: 30 days, newest 8 kept (built).
- Off-site S3 backups: **90 days** (S3 lifecycle rule on vigil-backups-521124646177, applied by Claude).
- Deletion requests: answered within **30 days** via info@sigilsovereign.com.
Privacy page (/privacy retention section) must be updated to state exactly this once the evidence sweep exists.
Related: [[vigil-launch-pricing-decisions-2026-11-24]], [[vigil-credentials-setup-2026-11]].

**Decision 4 (support, owner-approved 2026-11-24):** support = info@sigilsovereign.com, "we aim to reply within 2 business
days" (a target, no SLA). /support rewritten for hosted → PR #1219 (claude/support-page-hosted; merge AFTER #1218).
S3 lifecycle APPLIED 2026-11-24: rules vigil-offsite-backups-90-days (prefix vigil/, expire 90d, noncurrent 1d, abort MPU 7d)
+ vigil-clean-expired-delete-markers. Evidence sweep design: delete evidence NOT backing a finding 90 days after the
assessment; keep finding evidence (proof bundles/PCF re-verify from evidence_root/action_id) until deletion request/closure.
Build with agents after the Nov 29 reset together with the /privacy retention text.
