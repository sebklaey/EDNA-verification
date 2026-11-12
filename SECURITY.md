# EDNA — Security Notes (main live production, v1.4.1-main.1)

This document states what the public **production** endpoint does, what it deliberately
does **not** do, and the checklist an integrating provider must satisfy before an
**enterprise** deployment. EPCAL-1.0 applies; commercial hardening and SLAs are part of
license contracts — sebklay@me.com.

---

## 1. The public production endpoint — paid, key-authenticated

`https://fbxzacsslrrmivdzjctx.supabase.co/functions/v1/edna-mcp/mcp` is the live product endpoint (main live
production v1.4.1-main.1). By design:

| Property | Production status |
| --- | --- |
| Authentication | **Per-customer account key** (`edna2_…`) — `Authorization: Bearer edna2_…` or `X-EDNA-Key`; created once at https://fbxzacsslrrmivdzjctx.supabase.co/functions/v1/edna-mcp/, shown exactly once, stored only as SHA-384 |
| Free-Beta keys | **Revoked** — every `edna1_…` key is rejected with `beta_key_revoked` since the v1.4.1-main.1 Go-Live |
| TLS | Terminated by Supabase (HTTPS only) |
| Access control | Valid account key required for every `tools/call`; account-key scoping on `edna.submit_proof` |
| Metering | 1 credit per tool call (€0.01), metered per account; annual licenses include usage |
| Rate limiting | Signup rate limiting per client IP (account-farming protection); per-account metering bounds abuse |
| Session isolation | Per-session rows in `edna_mcp_sessions`, server-mediated by the `edna-mcp` Edge Function (isolated state per session id); the session id itself is the only "credential" and is shareable by design for the sensor-bridge pairing |
| Data at rest | Per-session snapshots in Supabase Postgres (quantized values + chain export state — no private key, no raw signals, no plaintext handle); no TTL — overwritten on next write, destroyed on `edna.id.revoke`; accounts, balances and ledgers in `edna_mcp_accounts` |
| Audit logging | Ledger-level: every credit purchase, grant, consumption and license purchase is appended to the account's CNSA-2.0 Credits Hash-Chain (SHA-384 linkage, AES-256-GCM payloads, ECDSA-P384 signatures) |
| Availability / SLA | Best effort on the shared endpoint; SLAs are part of license contracts |

**Consequences you must accept when using the production endpoint:**

- Anyone who learns a session id can read that session's stream (`edna.get_*`) and
  submit values into it (`edna.submit_proof`). Session ids are therefore **not secret
  in a security sense** — treat them like unlisted URLs.
- Submitted values are unauthenticated as to their origin. The AI client cannot know
  whether a stream genuinely comes from the EDNA engine on the user's device or was
  fabricated. Verify the optional attestation (`setAttestation`, WebAuthn) before
  acting on values; otherwise label them `unverified` in the model context.
- Perimeter security for the shared endpoint is best-effort. Enterprise integrations
  requiring per-tenant isolation, custom auth or SLAs license a hardened deployment
  (see checklist below) — sebklay@me.com.

## 2. What is already enforced server-side

- **Account authentication**: every `tools/call` requires a valid production account
  key; revoked Free-Beta keys (`edna1_…`) are rejected with `beta_key_revoked`.
- **Consent enforcement**: `edna.submit_proof` refuses submissions without
  `consent: { granted: true, scope: { mcp: true }, … }`; `edna.id.*` additionally
  requires `scope.id`. The consent record is stored and returned on every read.
- **Input shape discipline**: the server stores only the documented fields; there is
  no code path that accepts raw biometrics (content, coordinates, timing streams).
- **Session-keyed state**: one Durable Object per sanitized session id
  (`[a-zA-Z0-9_-]{1,64}`); cross-session reads return `no_data`.

## 3. Enterprise-hardening checklist (required before enterprise integration)

An AI provider integrating EDNA into a paid, customer-facing product at scale is expected
to operate (or license) a hardened deployment:

- [ ] **Authentication & authorization** — per-customer credentials (mTLS, OAuth, or
      signed requests); per-customer key scoping in `edna.submit_proof`.
- [ ] **TLS everywhere** — including between the user's device/browser and your
      ingestion endpoint; HSTS; no plaintext fallback.
- [ ] **Rate limiting & abuse controls** — per-key quotas, per-session submission
      ceilings, anomaly rejection on impossible cadences.
- [ ] **Session isolation guarantees** — session ids treated as bearer-capable
      references: server-side unguessable ids, or authenticated session binding, so
      one tenant cannot read another tenant's stream by guessing ids.
- [ ] **Data minimization & retention** — store only what the use case needs; define
      TTLs for per-session state; honor deletion end-to-end (the chain's
      `edna.id.revoke` destroys local state; your backend retention must match).
- [ ] **Supply-chain integrity** — pin the engine version you deploy
      (`npm install edna-engine@<version>` + integrity hash); verify the seal
      attribution per the license.
- [ ] **Unauthenticated-stream labeling** — if your deployment accepts
      engine-attested submissions, verify the attestation (`setAttestation`, WebAuthn)
      before acting on values; otherwise label them `unverified` in the model context.
- [ ] **Incident response** — documented path to revoke customer keys, rotate
      endpoints, and purge retained session state.
- [ ] **Independent verification** — run the open harness
      (`node verification/run-tests.mjs`) on the exact version you ship; log your run
      in `verification/ai-verification-log.md`.

## 4. Reporting

Security concerns about the public production endpoint: sebklay@me.com (PGP on request).
Please allow disclosure coordination before public write-ups.

---

*Engineering guidance, not legal advice. For GDPR obligations see
[PRIVACY.md](PRIVACY.md).*
