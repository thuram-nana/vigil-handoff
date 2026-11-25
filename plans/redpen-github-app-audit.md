
## Added by the completeness check (2026-11-24, read-only)

Refs: `origin/main` = dceea03, `claude/consumer-prE-ciretest` (E) = 27a793d. NOTE: the base audit body was NOT present at
this path when the check ran; this section was appended to an absent file (so it created it). Scratch + probes:
`/home/kali/.cache/vigil-tmp/ghapp-audit/completeness/`.

**Exact path (OBSERVED, both refs; uiproxy.py is identical on E):** `/webhooks/github-app` — uiproxy.py:222, exact
`split.path` match, POST only, uiproxy.py:1657-1659. Mounted only when `VIGIL_GH_WEBHOOK_SPOOL` is set (read ONCE at
`vigil up` start, uiproxy.py:4316 → restart vigil-command after setting) AND `VIGIL_REDPEN_ENABLED` is truthy (read per
request, uiproxy.py:1435-1440, re-checked :2211). The owner's URL path matches. No trailing slash, no query string.

1. **Edge hop blocks it (OBSERVED, identical on both refs).** The hooks vhost forwards ONLY `/webhooks/stripe` and
   `/webhooks/billing`: vigil-hooks.nginx.conf:98-122 (`location /` → 404), vigil-hooks.Caddyfile:29-33,49-52,
   infra/edge/Caddyfile.example:42-50. So `https://hooks.vigilguard.net/webhooks/github-app` is a 404 at nginx/Caddy and
   never reaches the proxy. apps/sigil/tests/test_hooks_webhook_edge.py:36,77,105 pins exactly those two paths. Adding
   the route means changing the vhost (same `$request_uri` exact-match pattern) AND that test.
2. **DNS (OBSERVED from this host, 2026-11-24 ~22:10 EST):** `hooks.vigilguard.net` and `app.vigilguard.net` do not
   resolve (getent rc=2). The apex does. Every delivery, including the ping GitHub sends when the URL is saved, fails today.
3. **Secret hop (OBSERVED):** nothing reads `/vigil/gh-webhook-secret` from SSM. The proxy reads env
   `VIGIL_GH_WEBHOOK_SECRET` only (uiproxy.py:2181; an empty value gives a 503). The value must be copied in plain text into
   `~/vigil/.vigil-live/service.env` (vigil-command.service:67-68). `sigil doctor` checks the SOVEREIGN process env
   (apps/sigil/sigil/cli.py:3533 → github_app.py:129-132), not the proxy's, so it can say "webhook secret set" while the
   proxy answers 503. The same secret is shared with `/webhooks/github` (repo-connect).
4. **IAM (OBSERVED, same on E):** provision.sh:78-82 grants `ssm:GetParameter` only on the provider-key parameter.
   `VIGIL_GITHUB_APP_PRIVATE_KEY_SSM=/vigil/gh-app-private-key` (github_app.py:42,98-105 → `aws ssm get-parameter
   --with-decryption`, aws_ssm.py:57-59) would get AccessDenied. INFERRED: if the parameter uses a customer CMK rather than
   `aws/ssm`, the role also needs `kms:Decrypt` on that key.
5. **Env names (OBSERVED):** on main, no env example or runbook mentions `VIGIL_GITHUB_APP_ID`, `VIGIL_GITHUB_APP_SLUG`,
   `VIGIL_GITHUB_APP_PRIVATE_KEY_SSM`, `VIGIL_GH_WEBHOOK_SECRET` or `VIGIL_GH_WEBHOOK_SPOOL`. E's
   infra/systemd/vigil-redpen-checks.env.example:23-24 uses the placeholder `/vigil/github-app/private-key`, which is NOT the
   owner's `/vigil/gh-app-private-key`. Also required: `VIGIL_GITHUB_APP_ID=5804935`, `VIGIL_AWS_REGION=eu-west-1`
   (aws_ssm.py:46-50), and `VIGIL_GITHUB_APP_SLUG=vigil-red-pen`; without the slug, Connect GitHub refuses
   (config.py:481-485, provision.py:3135-3137).
6. **Event actions (OBSERVED: main's `_redpen_event_coords` was run in scratch, probe_coords.py):** every `check_run`
   (rerequested, requested_action) and `check_suite` (requested, rerequested) payload maps to None, so the proxy acks 202
   and spools nothing. Two of the three subscribed events do nothing, and "Re-run" on the check is a no-op.
   `pull_request` ready_for_review/edited are spooled, but the drain enqueues only opened/synchronize/reopened
   (redpen_webhook_drain.py:53,194). `installation`/`installation_repositories` events (INFERRED: GitHub sends them to
   every App without a subscription) are spooled and then only acknowledged. No disconnect is reconciled.
7. **Setup URL / OAuth (OBSERVED):** connecting a repo needs GitHub to redirect to `/api/customer/code/connected`
   (custui.py:732,1415-1432) with installation_id + setup_action + state + code. With the Setup URL blank there is no
   callback, so no connected-repo record is ever made. Even with it, `installation_verifier` defaults to None
   (provision.py:544,607-611), so the result is always `verification_required`. That verifier needs the OAuth `code`, which
   requires "Request user authorization during installation" plus the client secret (both off/unwired). Expected value
   (host INFERRED from vigil-customer.nginx.conf:66): `https://app.vigilguard.net/api/customer/code/connected`.
8. **Installability (OBSERVED):** an unauthenticated `GET api.github.com/apps/vigil-red-pen` returns 200. INFERRED from
   GitHub's rule that private Apps 404 without auth: the App is PUBLIC, so any account can install it.
9. **Review → check chain (OBSERVED; E code executed in scratch, probe_orgid.py):** `vigil redpen-review` authenticates
   with `GITHUB_TOKEN`, not the App (integration cli.py:4127), and its reserve fails closed (cli.py:4142-4151), so every PR
   is refused. The refused outcome has `org_id ""` (redpen_review.py:338 main / :353 E; the drain sets `tenant_id ""` at
   redpen_webhook_drain.py:207), and the check writer refuses that (redpen_check.py:148 main / :184 E). Result:
   RedpenCheckError, the outcome is quarantined, and NO check (not even neutral) is posted on a webhook PR. This contradicts
   E's LIMIT-redpen-review-execution-live-flip ("neutral for a review that did not run"). Main has no check-writer verb at
   all (E adds `sigil redpen-checks`, cli.py:3819).
10. **Runners (OBSERVED):** neither ref has a systemd unit for `vigil redpen-webhook-drain` or `vigil redpen-review`, so the
    spool would only fill.
11. **Test gaps (OBSERVED):** test_uiproxy_redpen_webhook.py, test_redpen_webhook_drain.py and
    test_hooks_vhost_uiproxy_e2e.py have no check_run/check_suite case. No test sends `/webhooks/github-app` through the
    hooks vhost, and none feeds a webhook-shaped refused job to the check writer.
12. **Minor (OBSERVED):** installation repo lists are cut at 200 per key (uiproxy.py:1470). The edge caps bodies at 1 MB,
    the proxy at 5 MB (uiproxy.py:210). `mint_installation_token` posts `{}` (github_app.py:207), so the token covers every
    repo in the installation, not one repo as its docstring says.
