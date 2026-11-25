# Login and double-payment check (main dceea03a, 2026-11-24)

PROVEN = test or probe run. READ = code only. Every gap re-checked in the code.

## 1. Is login rate-limited?
Yes (PROVEN). 5 wrong passwords for one account in 15 minutes lock it for 1 minute. Each new lock doubles, up to 1 hour. During a lock even the right password is refused. Other customers can still log in. Unknown emails get identical answers, hiding who has an account.

Behind Cloudflare, the shipped nginx setup works (PROVEN). The Caddy example in AWS-EC2-DEPLOY and the Cloudflare Tunnel option do not. There every visitor looks like 127.0.0.1: anyone's 5 wrong passwords lock all customers out (PROVEN).

## 2. Double click or double credit?
- **Packs:** a double click opens one Stripe page. One payment is credited once, even with repeated or simultaneous Stripe messages. Paying twice needs two finished Stripe pages, each credited once (all PROVEN).
- **Plans:** 2 to 8 clicks or tabs make one subscription. A repeated "invoice paid" adds credits once. A double "change plan" sends one change (all PROVEN).
- **Auto top-up** (off by default): two simultaneous triggers make one charge (PROVEN, fake Stripe).

## 3. Real gaps
1. **HIGH, plan billed twice:** once the paid month ends, the "already has a plan" check turns off (provision.py:2122), but Stripe still renews or retries the old subscription. A second plan can be sold: double payment monthly, one allotment (PROVEN). Fix: also refuse while past_due and for ~72 hours after the month ends.
2. **HIGH, proxy:** add `header_up X-Real-IP {remote_host}` to infra/edge/Caddyfile.example, and `set_real_ip_from 127.0.0.1;` to the tunnel step.
3. **MEDIUM, login:** a successful login clears the address counter (provision.py:1120), so an attacker with a free account escapes it. IPv6 counts per address. A full store stops counting (PROVEN). Fix: keep the counter on success, use ip_rate_key, refuse when full.
4. **LOW:** a known email can be kept locked, even after a reset (PROVEN). Auto top-up can show "failed" after charging the card (Stripe 409, PROVEN). The authenticator-code limit is off by default (READ).

Not checked here: the Anthropic outage.
