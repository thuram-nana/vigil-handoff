---
name: vigil-launch-pricing-decisions-2026-11-24
description: Owner launch-pricing decisions (2026-11-24) — decided credit packs, plans retired at launch, 450 trial credits, NO purchase limits/custom amount at launch
metadata:
  type: project
---

Owner answers 2026-11-24 (reconciling the code with the 11-11/11-15/11-16 decisions in docs/decisions/PRICING-RED-PEN-DESIGN.md):
- **Credit packs = the decided ones:** $5/$10/$16/$25/$50/$100 → 500/1,000/1,600/2,500/5,000/10,000 credits, $1 = 100 credits,
  NO volume bonus. (Code had only $10→1,000 and a $50→6,000 bonus placeholder.)
- ~~plans stop selling~~ SUPERSEDED same day: owner wants plans AT LAUNCH: Starter $29/3,200 (price_id vigil_starter_monthly,
  plan_id starter; was the never-sold 'research' placeholder), Pro $99/12,000, Team $299/39,000 credits/month (+10/+21/+30% vs
  packs). Plan credits first, expire at period end; then purchased credits; cancel = end of paid month, no refund.
  Flags on in deploy examples: VIGIL_CUSTOMER_PLAN_CHECKOUT_ENABLED, VIGIL_CUSTOMER_PLANS, VIGIL_CUSTOMER_PLAN_CHANGE (edge reads
  PLANS/PLAN_CHANGE/CHECKOUT too); go-live VIGIL_CUSTOMER_CHECKOUT_ENABLED stays off until the drain is proven. The cockpit ALSO
  needs the Stripe lane (VIGIL_BILLING_ENABLE_STRIPE, SECRET_KEY, WEBHOOK_SECRET, API_VERSION) since it creates checkouts.
  PR #1217 (claude/launch-pricing-catalog, wt vigil-wt-launch-pricing) — awaits independent review after the Nov 29 reset.
- **Trial = 450 credits** (code default was 100, deploy example 300).
- **Launch WITHOUT the §4.6 purchase limits and WITHOUT the custom-amount option** — owner accepts the stolen-card risk for now
  (packs only). Do not build them unless the owner asks.
- Unchanged: published rate card (§0.2), margin set privately at deploy via VIGIL_BILLING_TARGET_MARGIN (never committed),
  AEGIS per-host prices stay placeholders (AEGIS not in the launch), seats not sold.
Catalog rule: never edit a sold entry in place — retire it and add a new price_id (GO-LIVE §8). Related:
[[vigil-payment-provider-stripe]], [[vigil-credentials-setup-2026-11]], [[vigil-deploy-gate-owner-approval]].

**Tax decisions (owner, 2026-11-24, decision 2):** SIGIL SARL IS VAT-registered in Cameroon (charges TVA 19.25% to
Cameroonian customers); customers are BUSINESSES **AND INDIVIDUALS** (owner correction 2026-11-24, superseding "businesses only"):
collect billing country always + VAT/tax id optionally (B2B reverse charge); consumer sales trigger destination VAT (EU OSS /
UK VAT from the first sale for a non-EU seller) → accountant must confirm registrations; prices are EXCLUSIVE of tax (tax added at checkout); tax calculation must be LIVE AT
LAUNCH → finish the legal-gov S6 Stripe Tax slice (wt vigil-wt-legalgov-s6 @c7f1196e WIP, VIGIL_BILLING_TAX, owner input
catalog.tax_settings = behaviour 'exclusive' + registrations) and review it after the Nov 29 reset.
OPEN RISK to raise with the owner: Stripe does not onboard Cameroon-based merchants; this Stripe account settles EUR to a
French bank (Shine) with Paris time zone, so it may be registered as a FRENCH business — the seller of record / VAT regime
(French TVA 20%?) must be confirmed by the owner + accountant before Stripe Tax registrations are set.

Consequences of B2C (owner told 2026-11-24): terms page says "for business and professional use" (line ~50) and "internal
business purposes" (~287) → must be updated for individuals + EU consumer right of withdrawal (express consent to immediate
performance; Stripe Checkout consent_collection terms_of_service) — legal text change, owner/counsel OK needed.
PR #1218 (claude/terms-individuals, wt vigil-wt-terms-individuals): terms opened to individuals + EU consumer 14-day
withdrawal (unused credits refunded pro rata; right ends once a purchase's credits are used) + VIGIL_STRIPE_TERMS_CONSENT
(Stripe consent_collection.terms_of_service=required + immediate-supply text; fail-closed echo check; on in cockpit example;
needs Stripe Public details Terms URL). NEEDS owner's counsel OK on the new wording + independent review before merge.
NOTE for S6 tax: Stripe checkout _checkout_link checks amount_total == catalog amount for payment mode — with exclusive tax
amount_total includes tax, so S6 must compare against the pre-tax subtotal.
**SUPERSEDED (owner, 2026-11-24 later): KEEP TAX OUT ENTIRELY FOR NOW.** No Stripe Tax, no tax IDs, no TVA collected at
launch; legal-gov S6 (VIGIL_BILLING_TAX) stays unmerged/off — do NOT finish it for launch unless the owner asks. VIGIL
charges no tax by default, so no code change. Terms §6 wording ("taxes added where the law requires") left as is. Owner
told the risk once: if SIGIL SARL is TVA-registered, TVA on Cameroonian sales may be owed out of its own revenue.
- Owner 2026-11-24 (explicit): NO TVA even on Cameroonian customers. Do not re-raise unless asked.
- 2026-11-24: owner says counsel APPROVES the #1218 terms wording (individuals + consumer withdrawal + checkout consent). Still needs the independent review after Nov 29.

**Decision 5 (owner, 2026-11-24):** NO promotions at launch (no promo codes/bonus credits; never create Stripe coupons —
VIGIL's underpay check would quarantine discounted payments). **SEATS ARE SOLD AT LAUNCH** (reverses HANDOFF "do not build
the seat SKU yet"): plans include seats — pay-as-you-go 1 (founder), Starter 1, Pro 3, Team 10 — each extra member
$25/month (access only, no credits); adding a seat mid-month charges the prorated rest of the month NOW (then $25 from the
next renewal); removing a member = access lost immediately, seat billing stops at the next renewal, no refund; founder +
org_admin can buy/remove seats. Needs: seat billing on Stripe (quantity, proration_behavior always_invoice), member-limit
enforcement on invites, Team-screen UI, a seat-only subscription for pay-as-you-go orgs, terms §6 seat wording (counsel).
- 2026-11-24 verification (wf_dd931100-ec0, PROVEN on main dceea03a): login rate-limited; pack/plan double-click safe; one payment credited
  once; auto top-up one charge. Gaps → PL01 fix slice wf_82f1e175-00f (branch claude/pay-login-hardening): HIGH second plan sellable after
  period_end while Stripe still renews the old sub; HIGH Caddy example/Tunnel lose client IP; MEDIUM login counter reset on success / IPv6 /
  full store; LOWs. Report ~/.claude/plans/login-and-double-pay-verification.md. Deploy uses the nginx edge (works); never ship the Caddy
  example until PL01 merges.
