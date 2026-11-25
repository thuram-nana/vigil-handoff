---
name: clean-verdict-soundness
description: "What it takes for a CLEAN (absence-of-vulnerability) to be sound in VIGIL — and the two ways it has failed: a positive-only detector's silence, and a single-peer assumption across two connections"
metadata:
  node_type: memory
  type: project
---

CLEAN is the highest-stakes verdict the product emits: it tells an operator a vulnerability is ABSENT.
Get it wrong and they ship a hole believing it was checked. Two independent failures found in 2026-09.
See [[runtime-redrive-live-false-fact]], [[claim-discipline-meet-up]], [[vigil-fact-coverage-program]].

## FAILURE 1 — silence is not a negative (the standing rule)
A positive-only detector that does not fire has observed NOTHING. Rounding that to CLEAN is the oldest
trap here. `probe_verdict` must return `clean` only from a signal built with `conclusive=True`; several
branches declare `clean_capable:true` while no code path can produce a conclusive negative for them
(`service_reachability.tcp_handshake`, `tls_weakness.tls_handshake` — shipped as named `blocking_work`).

## FAILURE 2 — a channel-confirmed negative on ONE CONNECTION says nothing about the PEER
**I got this one wrong out loud, so it is recorded as a correction.** Early in the HexStrike W3 review I
said deprecated-TLS-protocol REFUSAL was "one of the very few genuinely sound CLEANs — a channel-confirmed
negative, unlike a detector's silence." The negative IS channel-confirmed: the peer really sends
`alert(70) protocol_version`. That was not the flaw.

**The flaw: the witness handshake and the refusal happen on TWO DIFFERENT TCP CONNECTIONS, and nothing in
the evidence binds them to the same peer.** An ordinary L4 pool breaks it. Reproduced with one listening
socket and two real OpenSSL backends round-robining per connection (A `min=TLSv1`, B `min=TLSv1_2`),
through the real runner and gate:
```
VIGIL verdict for offer_tls1_0: ['clean']
GROUND TRUTH, 6 TLS1.0-only handshakes to the SAME host:port:
  ['TLSv1','REFUSED','TLSv1','REFUSED','TLSv1','REFUSED']
offline re-derivation of the CLEAN: True
```
Every guard held individually; the offline verifier faithfully reproduced the false conclusion. Realistic
shapes: a k8s Service with >1 pod, an haproxy/nginx TCP pool, an L4 VIP, a DNS A-record pool mid-rollout —
precisely when one straggler still speaks the old protocol. It is also undetectable as a
self-contradiction, because `min_version=TLSv1.1` is a legal config, so "CLEAN at 1.0 + FACT at 1.1" is
internally consistent.
**Why it cannot be patched:** `spki_sha256` exists on the COMPLETED leg, but a REFUSED handshake yields no
certificate, so there is no material to bind the two legs to one peer. Repeating the probe k times only
makes the claim probabilistic, which the oracle doctrine forbids. RESOLUTION: `clean_capable:false`,
protocol refusal becomes INCONCLUSIVE, the CLEAN-capable pin reverts 7→6, and the witness machinery is
KEPT (it did close the bare-alert-70 hole and every sub-condition is mutation-defended) plus the VIP
two-backend fixture is kept as a permanent negative control.

## THE RULE THAT GENERALISES
**A CLEAN must be derivable from a SINGLE observation of a SINGLE identified peer.** Any absence-claim
assembled from two observations needs a cryptographic or structural binding that BOTH observations can
supply. Ask of every CLEAN: *what exactly did I observe, and could a different responder have produced
each half?*

## AND: A CLEAN CARRIES NO CRYPTOGRAPHY
`certify_admitted` mints a signed certificate only for a FACT. A CLEAN comes back as an unsigned
`AdapterResult(status="lead", outcome="clean")` — no signature, no digest, no trust root. Its only
integrity is provenance-by-custody: it came out of this runner. Anyone re-deriving a CLEAN from a
third-party-supplied context is trusting the supplier. True of EVERY clean in the product, and it should
be stated wherever a CLEAN is presented.
