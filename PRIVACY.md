# EDNA — Privacy Notes for Deployers (GDPR-oriented)

EDNA's architecture minimizes data by design: raw behavioral biometrics never leave the
user's device; only consented, quantized values may be released (see **Data flow** in
[README.md](README.md)). This document lists what a deploying organization must still
clarify for a lawful production deployment. **Engineering guidance, not legal advice —
run this past your DPO/legal counsel for your concrete use case.**

---

## 1. Classification: quantized behavioral patterns are (likely) personal data

- Pseudonymization ≠ anonymization. Salted hash patterns and quantized signal vectors
  relate to an identifiable natural person when they can be linked to a session,
  account, device, or EDNA ID — Recital 26 GDPR.
- Behavioral biometric *templates* may fall under **Art. 9 GDPR** (biometric data for
  uniquely identifying a person) **depending on the deployment's purpose**: uniquely
  identifying users (EDNA ID continuity) points toward Art. 9; mere interaction
  adaptation without identification points toward regular personal data. **Get a legal
  classification for your use case before launch.**
- Even "only statistics" become personal data when combined with account metadata.

## 2. Checklist for a lawful deployment

- [ ] **Lawful basis (Art. 6 / 9)** — explicit consent is the natural basis;
      enterprise deployments may alternatively rely on legitimate interest or
      contract, documented per purpose. Art. 9 processing (if applicable) requires an
      Art. 9(2) condition — typically **explicit consent**.
- [ ] **Consent quality (Art. 4(11), 7)** — the on-device gate (`setConsent`) is the
      technical floor, not the legal ceiling: consent must be informed (what is
      released, to whom, for what), specific, freely given, and revocable without
      detriment. Bundle your privacy notice with the consent UI.
- [ ] **Purpose limitation (Art. 5(1)(b))** — release contexts must be bound to the
      stated purpose (e.g. "adapt response style"). Do not repurpose streams for
      profiling, worker monitoring, or ad targeting without new consent.
- [ ] **Data minimization (Art. 5(1)(c))** — prefer `edna.get_writing_process` /
      `getHumanContext` levels over full signal vectors when the use case allows;
      disable fields you don't consume server-side.
- [ ] **Storage limitation (Art. 5(1)(e))** — define TTLs for per-session state on
      your endpoint (the shared production endpoint has **no TTL** — see
      [SECURITY.md](SECURITY.md)).
      Honor `edna.id.revoke` end-to-end: local deletion must propagate to your
      backend copies.
- [ ] **Transparency (Art. 12–14)** — users must know that interaction dynamics are
      measured, what categories leave the device, and their rights. The on-page
      honest-limits blocks in the demos are a starting point, not a privacy notice.
- [ ] **Data subject rights (Art. 15–21)** — access/erasure workflows: EDNA supports
      local export (`exportState`, report downloads) and real deletion (`revoke`);
      your backend must implement the same for anything it stores.
- [ ] **Processor arrangements (Art. 28)** — if a provider (e.g. an AI platform)
      processes released EDNA data on your behalf, a DPA is required. If you deploy
      the MCP endpoint for your own customers, *you* are typically the processor or
      controller depending on the setup — clarify roles.
- [ ] **DPIA/DSFA (Art. 35)** — large-scale behavioral processing likely requires a
      Data Protection Impact Assessment. The public repo (engine, spec, these notes)
      can serve as annex material describing the system.
- [ ] **Cross-border transfers (Ch. V)** — the shared production endpoint is operated on
      Supabase infrastructure; transfers outside the EEA need an appropriate
      safeguard (SCCs etc.) or an EU-hosted enterprise deployment.
- [ ] **Security (Art. 32)** — see [SECURITY.md](SECURITY.md) §3.

## 3. What EDNA deliberately does NOT do

- No raw biometric transmission (no keystroke content, pointer paths, motion samples).
- No cross-device linkage by design: export salts are fresh per export; an EDNA ID is
  device-bound and starts a new chain on a new device.
- No emotional, health, age, or intent inference — the output contract states this in
  every context object. Deployers must not re-enable such inference downstream.

## 4. Production endpoint notes

- The shared production endpoint requires a per-customer account key (`edna2_…`);
  Free-Beta keys (`edna1_…`) were revoked at the v1.4.1-main.1 Go-Live.
- It stores per-session quantized state in Durable Object
  storage without TTL until overwritten or revoked.
- Everything you need to self-host the endpoint is in `deploy/` and `mcp/` —
  self-hosting is the recommended path for any enterprise pilot.

---

Questions and licensing: sebklay@me.com — SEBKLAEY© Agency Sebastian Kläy.
