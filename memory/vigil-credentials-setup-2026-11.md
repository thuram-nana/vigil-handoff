---
name: vigil-credentials-setup-2026-11
description: Owner's pre-deploy credential setup for the SaaS/customer app (started 2026-11-23) — what is created where, what is still owed, and where each value lands on the server
metadata:
  type: project
---

Owner 2026-11-23: after PR-H merges, PAUSE program work and set up credentials so the customer app can go live on deploy.
Infra already done by owner: Cloudflare, AWS SES + email DNS, Namecheap → Cloudflare DNS. NO EC2/VIGIL server yet
(deploy = docs/runbooks/AWS-DEPLOY.md: provision.sh → EC2 → bootstrap --production → KMS vault → units → nginx → DNS).

Done:
- 2026-11-23 Anthropic key stored in AWS SSM Parameter Store `/vigil/anthropic-api-key`, SecureString, data type text,
  KMS alias/aws/ssm. Region = eu-west-1 (Europe, Ireland) — owner confirmed 2026-11-23; provision.sh REGION, VIGIL_AWS_REGION and the EC2 host must all be eu-west-1.

- 2026-11-23 Google Cloud project VIGIL (vigil-515611, org thuram-org) created; Google Auth Platform: External, In production,
  scopes openid/email/profile (no verification needed for data access); Branding NOT yet verified (no logo — keep it off
  to avoid review); vigilguard.net ownership VERIFIED in Search Console (DNS via Cloudflare). After deploy: submit
  Branding → Verify branding (needs https://vigilguard.net + /privacy live). Public site has NO /terms page yet (gap).

- 2026-11-23 SES eu-west-1: domain vigilguard.net + noreply@vigilguard.net both VERIFIED → SIGIL_EMAIL_FROM=noreply@vigilguard.net (no hyphen), SIGIL_EMAIL_HOST=email-smtp.eu-west-1.amazonaws.com.
  SMTP login = existing IAM user vigil-ses-smtp (SMTP user name = its access key id); owner HAS the SMTP password saved. Do not create another.
  SES eu-west-1 is OUT of the sandbox (50k/day, 14/s): production access done. us-east-1 SES is sandboxed and unused, ignore it.
  Still to confirm: the SMTP password logs in against eu-west-1 (owner's local smtplib login test).
- 2026-11-23 Google OAuth web client "VIGIL customer app" CREATED (redirect https://app.vigilguard.net/api/customer/oidc/callback); id+secret in owner password manager.

- 2026-11-23 Stripe LIVE (owner chose live; account settles EUR, VIGIL catalog charges USD → conversion fee; EUR pricing = separate change if wanted):
  restricted key "VIGIL server" rk_live_…naEy created with exactly Charges&Refunds R, Disputes R, Invoices R, PaymentIntents W,
  Products W, Prices W (added: plan change creates prices), Subscriptions W, Checkout Sessions W. Deploy-day check = one real
  purchase → credits land → refund. API version = 2026-10-28.endive (event destination default) → VIGIL_STRIPE_API_VERSION=2026-10-28.endive; payload style SNAPSHOT (VIGIL parses v1 snapshot events, not thin).
  Event destination "VIGIL billing" (we_1Uey4f…) ACTIVE → https://console.vigilguard.net/webhooks/stripe, scope Your account,
  exactly the 5 events; owner to save whsec. Adaptive Pricing for VIGIL (Checkout) turned OFF 2026-11-23. Stripe pre-deploy setup COMPLETE.
  Account activation OK (owner); payouts: automatic weekly Monday, EUR → owner's Shine (FR) account. A USD balance also
  exists (VIGIL charges USD): confirm whether USD settles/converts to EUR or needs a USD payout account. Next: 1c public
  details (statement descriptor VIGILGUARD, support email/URL), 1d customer receipt emails.

- Company facts (from sigilsovereign.com legal notice, owner-delegated 2026-11-23): SIGIL SARL (Sovereign Integrity Governance
  Infrastructure Labs), Buea, Cameroon, RCCM RC.BUA.2026.B.0190, contact info@sigilsovereign.com; no street/NIU/capital published.
  Terms governing law chosen: Cameroon law incl. OHADA; courts of Buea after 30-day amicable step; mandatory local consumer/data
  rules preserved. /terms page drafted on claude/public-terms-page (+ Stripe webhook edge carve-out on claude/edge-stripe-webhook-route).
- Stripe receipt support email stays vigil@digittworld.com until after deploy (owner). Stripe receipts: Successful payments + Refunds ON.
- DNS 2026-11-23: _dmarc TIGHTENED by owner to "v=DMARC1; p=quarantine; adkim=r; aspf=r" (verified live 1.1.1.1+8.8.8.8);
  SES Easy-DKIM 3 CNAMEs live; custom MAIL FROM bounce.vigilguard.net (MX feedback-smtp.eu-west-1 + SPF amazonses) live → SPF+DKIM both align.
  Apex A 162.255.119.227 + www CNAME parkingpage.namecheap.com are Namecheap PARKING → replace with the EIP at deploy. Old note: SPF = Cloudflare email routing only
  (SES passes DMARC via aligned Easy-DKIM). Owner keeps: backup passphrase, Anthropic spend limit, vigil-engine key in password manager.

Owed (place after deploy): scan-engine Anthropic key (console API Keys screen).
Placement map: secrets → ~/.sigil/cockpit.env; non-secrets → customer.env; Stripe → billing-drain-stripe.env +
.vigil-live/service.env; only the gateway Anthropic key + GitHub App key are read natively from SSM.
Webhook edge: PR #1216 MERGED 2026-11-23 (ec53fb0c) — hooks.vigilguard.net vhost (nginx+Caddy, request_buffers 1MB), console private.
  Owner 2026-11-23: Rocket Loader OFF; Email Obfuscation + Web Analytics stay ON → privacy/cookies must say "we add an
  email-protection script and an analytics script" and what they do, WITHOUT naming Cloudflare in that wording (vendor stays in
  the provider list). Analytics runs on the public site only (pubui CSP allows the beacon origins; custui CSP stays strict).
  Follow-up agent for claude/legal-pages-edge-scripts was STOPPED BY THE OWNER 2026-11-23 — do not restart without asking;
  until resolved, the "no third-party scripts" wording stands and must be reconciled with the edge settings before go-live. PR #1215 terms+privacy+cookies
  MERGED 2026-11-23 (d805c779; round-3 review PASS); code of conduct → info@. Cloudflare speed settings chosen: Rocket Loader OFF,
  Early Hints ON, Cloudflare Fonts off (no Google Fonts used), Speed Brain OFF (beta; prefetch risk on app), Always Use HTTPS ON at deploy. Deploy-day Cloudflare: proxied `hooks` A record, Bypass cache rule, no challenges on hooks, Email Obfuscation/Rocket Loader/RUM OFF.
  GOTCHA: run gh with -R thuram-nana/vigil-sovereign (cwd PENTEST-main is another repo → 'Could not resolve PR').
OWNER DECISIONS 2026-11-23: (1) DEPLOY = OPTION B — Claude runs the non-secret deploy steps (AWS provisioning, EC2, bootstrap,
  units, nginx, checks) from this machine via the AWS CLI, owner approves each command and types every secret himself; this
  brings the VIGIL customer-app deployment into scope (supersedes the 2026-11-16 no-server rule for THIS deployment only).
  (2) Stripe webhook host = dedicated hooks.vigilguard.net (Cloudflare-proxied, only POST /webhooks/stripe|billing → uiproxy);
  console stays private (no public console record). Stripe destination URL CHANGED to https://hooks.vigilguard.net/webhooks/stripe (owner, verified on screen).
  AWS CLI v2.38.1 installed on this Kali box (apt awscli); next: owner creates vigil-admin access key + `aws configure` (eu-west-1),
  DONE: CLI = arn:aws:iam::521124646177:user/vigil-admin (AdministratorAccess), region eu-west-1.
  Inventory 2026-11-23: only the SSM SecureString exists; no KMS alias/bucket/role/SG/EIP/instance; default VPC vpc-054f943ab53a2c56e.
  provision.sh staged from main d08a161a at ~/.cache/vigil-deploy/provision.sh (sha256 0926…7f13); owner IP 102.244.45.32.
  PROVISIONED 2026-11-23 (owner yes): KMS CMK 2cd2456c-… alias/vigil-owner-kek; bucket vigil-backups-521124646177 (versioned,
  SSE-KMS, public blocked); role+profile vigil-host (inline vigil-host-policy + AmazonSSMManagedInstanceCore attached);
  SG sg-05828a0ffec3603c3 vigil-edge (443/80 from 22 Cloudflare CIDRs, 22 from 102.244.45.32/32); EIP 54.73.236.42
  (eipalloc-0d797f5cc1ed3bf63). www.cloudflare.com/ips-* is RESET from this network → local copy provision-cfapi.sh uses
  api.cloudflare.com/client/v4/ips (2-line diff). EC2 LAUNCHED 2026-11-23 (owner chose recommended): i-06880b45050f969fd t3.xlarge eu-west-1a, Ubuntu 24.04 (ami-0ae26e505e119d6d4,
  Canonical), profile vigil-host, SG vigil-edge, IMDSv2 hop 1, root 40GB gp3 enc, DATA vol /dev/sdf 50GB gp3 enc DeleteOnTermination=false
  (not yet formatted/mounted), EIP 54.73.236.42 associated, SSM Online (agent 3.3.4793.0). Break-glass key pair vigil-ec2 →
  ~/.ssh/vigil-ec2.pem (0600). Operate via `aws ssm send-command` / start-session. Instance STOPPED 2026-11-23 at owner request until deploy day
  (EIP stays associated). AWS Budget vigil-monthly $200/month → alerts to thuram@thuramnana.com at actual 80%/100% + forecast 100%.
  Deploy-day next (ONLY after owner approval, see [[vigil-deploy-gate-owner-approval]]): start instance, mount data vol at
  /mnt/vigil-data, user vigil, clone, bootstrap.sh --production.
  PRs: #1215 terms (fix round on review B1-B5/M1-M11), #1216 hooks edge (under review). Deactivate the key after deploy.
  2026-11-23: three agents (PR-H rvH5 fixer, terms/webhook builder mid-Caddy-templates, terms reviewer) STOPPED on the 5-hour usage
  limit (reset 12:50pm America/New_York) — resume each via SendMessage, do not respawn.
  Terms page PR #1215 (owner says counsel approved). Still open: server env files vs AWS-only secrets PR.
Never ask the owner to paste a secret into chat. Related: [[vigil-owner-flip-delegation]], [[vigil-payment-provider-stripe]].

- 2026-11-24 (owner, verified by metadata only): SSM eu-west-1 now holds /vigil/anthropic-api-key (gateway, 11-23),
  /vigil/anthropic-engine-api-key (SCAN-ENGINE key, SecureString aws/ssm, v1 21:46 — the 'owed' engine key, now stored),
  /vigil/gh-app-id (String 5804935), /vigil/gh-app-private-key (new 2026-11-24 key), /vigil/gh-webhook-secret.
  Server role vigil-host: inline policy scopes ssm:GetParameter to /vigil/anthropic-api-key ONLY, but the attached AWS-managed
  AmazonSSMManagedInstanceCore grants ssm:GetParameter(s) on * → the box CAN read all five (works), scoping is moot. Proposed to
  owner: add the 4 new ARNs to the inline policy (+ update apps/sigil/deploy/aws/provision.sh in code). DEPLOY DAY: load
  /vigil/anthropic-engine-api-key into the engine (ANTHROPIC_API_KEY via sigil.env / UI API Keys seal — engine does not read SSM),
  gh-app-id/private-key via VIGIL_GITHUB_APP_ID + VIGIL_GITHUB_APP_PRIVATE_KEY_SSM + VIGIL_AWS_REGION, webhook secret → VIGIL_GH_WEBHOOK_SECRET.
- 2026-11-24 22:2x (owner YES): role vigil-host inline policy vigil-host-policy — ssm:GetParameter Resource widened from the single
  gateway-key ARN to the 5 explicit /vigil/ ARNs (anthropic-api-key, anthropic-engine-api-key, gh-app-id, gh-app-private-key,
  gh-webhook-secret); before/after JSON in ~/.cache/vigil-tmp/iam/; simulate-principal-policy = allowed for all 5. TODO in code:
  apps/sigil/deploy/aws/provision.sh still grants only PROVIDER_KEY_PARAMETER — update it to the same 5 ARNs (fold into AP01 /
  the Red-Pen go-live slice) so a re-provision does not shrink the list back.
- 2026-11-24 ~22:50 owner DONE in Cloudflare (verified live): A hooks -> 54.73.236.42 PROXIED (resolves to CF 172.67.216.90/104.21.86.67 on
  authoritative + 1.1.1.1 + 8.8.8.8); cache rule "VIGIL hooks — bypass"; WAF custom Skip rule for POST /webhooks/{stripe,billing,github-app}
  on hooks; Bot Fight Mode OFF. POSTs to all 3 paths return 522 (origin off) with no challenge = correct pre-deploy. oob NS delegation intact.
  Still deploy day: SSL Full (Strict) + origin cert for hooks; app A record; Stripe/GitHub test deliveries (expect 202). GitHub path needs RP01.
