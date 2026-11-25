---
name: vigil-payment-provider-stripe
description: "Owner 2026-11-16 — VIGIL uses STRIPE for payments; Flutterwave development is PAUSED (keep its code, build nothing new on it); W5 / A1 / Gate 5 target Stripe"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-16T15:01:42.926Z
---

Owner decision (2026-11-16, verbatim intent): "we won't be using Flutterwave so leave its development paused, focus on
Stripe."

**What exists on main (verified 2026-11-16):** `governor/billing.py::StripeBillingProvider` (acts on
`checkout.session.completed`, `checkout.session.async_payment_succeeded`, `invoice.paid`, `invoice.payment_failed`;
re-fetches the object from Stripe, never trusts the body; refund / reconcile NOT implemented), `governor/stripe_egress.py`,
Stripe checkout via `Provisioner._stripe_checkout_provider` when `VIGIL_BILLING_PROVIDER=stripe`. The code's default when
`VIGIL_BILLING_PROVIDER` is unset is still Flutterwave — fails closed without FLW secrets; switching the default to Stripe
is a small product-default change to make in its own slice (or with W5).

**How to apply:**
- W5 (refunds / chargebacks): Stripe events, verified from docs.stripe.com/api/events/types —
  disputes `charge.dispute.created | .funds_withdrawn | .updated | .closed (status won | lost | warning_closed) |
  .funds_reinstated` (data.object = Dispute); refunds `charge.refunded` (Charge, incl. partial) and
  `refund.created | .updated | .failed` (Refund). **DECIDED by the owner 2026-11-16 (pick-one): remove the disputed
  credits when the dispute OPENS / funds are withdrawn; give them back automatically when the dispute is won
  (`charge.dispute.funds_reinstated` / closed `won`).**
- A1 auto top-up: Stripe saved payment method + off-session PaymentIntent (verify the API at A1).
- Gate 5: Stripe TEST mode, never a live key or a real card.
- Never commit a Stripe secret (`sk_live_…`, webhook signing secret); tests use dummies.
Related: [[vigil-owner-flip-delegation]], [[vigil-strix-shell-hosting-decision]].
