# OOB collector (APEX layer 2) — deploy-day server checklist  [PRIVATE]

Host: oob.vigilguard.net | Elastic IP: 54.73.236.42 | Instance i-06880b45050f969fd (vigil-prod, eu-west-1) | SG sg-05828a0ffec3603c3

## Cloudflare (owner, Option A — full delegation, done/in-progress 2026-11-24)
- A  ns1.oob  -> 54.73.236.42  (DNS only / grey)
- NS oob      -> ns1.oob.vigilguard.net
- Do NOT add *.oob or oob A records in Cloudflare (the collector's own DNS answers them).

## Done now (2026-11-24 setup; box stopped, nothing listens yet)
- [x] SG ingress 53/udp 0.0.0.0/0 + ::/0  (sgr-057b98fd9cf69ea70 / sgr-0bafeaa9030014e83)
- [x] SG ingress 53/tcp 0.0.0.0/0 + ::/0  (sgr-0e37dc5a4c0ed8ecf / +v6)
  (rules are greppable by description "VIGIL OOB collector"; remove with revoke-security-group-ingress if ever unwanted)

## OPEN DECISION before deploy — web-callback ports (80/443)
The app's 80/443 are deliberately locked to Cloudflare IP ranges only (origin cannot be reached except via the Cloudflare proxy/WAF).
oob.vigilguard.net is grey-cloud, so targets hit 54.73.236.42 DIRECTLY. DNS OOB already works. HTTP/HTTPS OOB does NOT yet, because
direct :80/:443 from a target is blocked by the Cloudflare-locked SG. Pick ONE (settle while building AP04):
  (A) Second Elastic IP / ENI for OOB only, SG open 53+80+443 to world; app keeps its Cloudflare-locked IP. Cleanest; default ports;
      then ns1.oob + *.oob point at the OOB IP, NOT 54.73.236.42 (would change the Cloudflare records).  [recommended if HTTP OOB matters]
  (B) Keep one IP (54.73.236.42); serve OOB HTTP on a dedicated high port opened to world (e.g. 8443/8080); callback URLs carry that
      port. Keeps the owner's current Cloudflare records; lower HTTP catch rate (default-port callbacks missed). DNS OOB unaffected.
  (C) Open 80/443 to the world AND move app-origin protection to the app layer (Cloudflare Authenticated Origin Pulls / origin secret),
      so direct non-Cloudflare hits are refused by the app. Preserves hardening + default ports, but is an app-origin change.
  (D) DNS-only OOB for launch (do nothing more); HTTP OOB deferred. DNS callback alone confirms a large share of blind classes.
DO NOT simply open 80/443 to 0.0.0.0/0 with no compensating control — it undoes the origin hardening (direct Host-spoof to the app).

## Deploy day (collector built = AP04; box started; owner approves)
1. Collector binds authoritative DNS on 0.0.0.0:53 (needs root or CAP_NET_BIND_SERVICE) answering *.oob.vigilguard.net -> 54.73.236.42
   and capturing every lookup as a signed OOB interaction.
2. TLS for *.oob.vigilguard.net: wildcard cert via Let's Encrypt DNS-01 answered by the collector's own authoritative DNS (not Cloudflare).
3. Env / LiveConfig: CRUCIBLE_OOB_RELAY_SECRET (owner-typed), oob_enable=1, oob_relay_host=oob.vigilguard.net,
   oob_advertise_base_url per the chosen scheme, + VIGIL_APEX_OOB=1 and the master VIGIL_APEX_ENABLED=1 (AP11).
4. Pin the collector public key into the owner-signed engagement authority via `vigil provision` (owner-signed).
5. Verify: dig NS oob.vigilguard.net -> ns1.oob.vigilguard.net ; dig x123.oob.vigilguard.net -> 54.73.236.42 (or OOB IP) ;
   mint a canary against the MERIDIAN range and confirm a signed OOB interaction is captured.
