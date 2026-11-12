# EDNA-2 Specification — The Chladni Chain & the Three Memory Layers

**Status:** Draft v0.2 · 2026-11-09 · Target: EDNA ID (product layer on top of EDNA-1)
**Positioning:**
> **EDNA ID — A persistent, privacy-first human continuity layer for the age of AI agents.**
> Not just another bot detector — a possible new trust layer between human, device, and AI.

EDNA-1 = the Sensoric Engine (published, v1.3.1). **EDNA ID = the Identity &
Continuity layer built on it (this spec — new architecture, not yet published functionality).**
**License:** EPCAL-1.0 — free for evaluation and testing; commercial use requires a license
agreement (sebklay@me.com).

---

## 0. Provability — what is proven and what is not

This section is the boundary between architecture and scientific proof. It is stated here
exactly as it will be stated in all public claims:

| Claim | Status |
|-------|--------|
| The chain is append-only, tamper-evident, and locally verifiable | **Provable** (cryptographic construction; v2 vectors 3–4) |
| Pause/resume works via reference comparison + new behavior data | **Provable** (v2 vector 2) |
| Replay of recorded segments is rejected | **Provable** (v2 vector 3) |
| 99 % re-recognition | **Not yet proven.** Requires long-term studies with real participants, across sessions and devices, under realistic attack scenarios. Until then: an honest, measured confidence curve (§5) |
| Bot prevention | **Not claimed.** A hash chain does not stop bots that convincingly simulate human patterns. Behavioral analysis + challenge-response do the detection work; the chain makes attacks *continuously expensive* |
| Tamper protection | Holds **as long as the attacker cannot replace the entire local history and its trust anchors** (device key, genesis record). State exactly this way |
| Privacy | Long-term behavioral profiles are sensitive. Mitigations: device-bound keys, local-only storage, transparent consent, real deletion (revocation), exportability |

`compareTo` in EDNA-1 is an explicit stub; Identity Continuity is listed as v2 development in
the public manifest. The Chladni Chain and long-term memory are therefore a **new architecture
under evaluation** — not already-published functionality.

---

## 1. The Three Pillars

| # | Pillar | What it means | How the architecture solves it |
|---|--------|---------------|--------------------------|
| 1 | **Human Sensoric for AI** | The AI perceives the user *over time* — not "how is the human now" but "what is the human going through". | The three memory layers (§2.4) give the AI live dynamics, session continuity and long-term evolution — deltas against the user's **own** history, not a universal ideal of human behavior. |
| 2 | **Continuous Bot Detection** | Humanness is verified *throughout the session*, not once at the door like CAPTCHA. | Verification **expires**: every segment must grow organically from live input. Replay is rejected; uniformity across segments is a chain-level signature. Challenges (v1.3) are woven in. |
| 3 | **EDNA ID — Same-Human Recognition** | A stable, **device-bound** identity that gets more certain the longer the chain grows; bound to a user handle + device key. | Genesis block at enrollment; every resume re-confirms live behavior against the local reference **and** requires fresh behavior data; confidence grows with verified history. New device = new EDNA ID by design. |

**The core principle:** EDNA compares the human primarily with *their own past*, not with a
universal template.

---

## 2. The Three Memory Layers

```
┌─────────────────────────────────────────────────────────────┐
│ LAYER 3 — LONG-TERM MEMORY            weeks · months        │
│ Evolution of the individual Chladni pattern against the     │
│ user's own historical reference profile; personal signature;│
│ stability/change over long horizons. Local, exportable.     │
├─────────────────────────────────────────────────────────────┤
│ LAYER 2 — SESSION MEMORY              hours · days          │
│ Continuity across sessions: re-recognition, behavioral      │
│ change between sessions, session comparison. Anchored by    │
│ the chain; survives pauses of any length.                   │
├─────────────────────────────────────────────────────────────┤
│ LAYER 1 — LIVE MEMORY                 seconds               │
│ The emergence of human recognition: signal changes,         │
│ interaction dynamics, confidence path in real time.         │
└─────────────────────────────────────────────────────────────┘
        immutable append-only chain below everything
```

### 2.1 Layer 1 — Live Memory (seconds)

The emergence of recognition: each micro-event contributes to the confidence path toward full
verification. Recorded as **emergence data** inside segments (first crossing, ramp, stability
runs) — not as per-event persistent blocks.

### 2.2 Layer 2 — Session Memory (hours · days)

Segments (batched, chained) + the anchored rolling window. Detects: re-recognition after
pauses, within-session drift, tempo arcs, session-vs-session comparison.

### 2.3 Layer 3 — Long-Term Memory (weeks · months)

The append-only chain log (the **historical truth**: what EDNA recognized and cryptographically
chained, when) **plus** the evolving reference profile (the **memory model**: what EDNA has
learned about the individual pattern). Two strictly separated artifacts:

1. **Historical truth** — immutable, append-only, verifiable via checkpoints. Updates to the
   profile never touch it.
2. **Memory model** — derived from the chain, continuously refined, replaceable without
   invalidating history.

### 2.4 Adaptive bundling & cryptographic checkpoints

High-frequency sensor streams do **not** produce one persistent block per event. Instead:

- **Micro-events** accumulate in memory (Layer 1) with full emergence data.
- **Segments** batch micro-events; a segment closes on verification loss, pause, challenge
  weave, or cap. Segments are chained: `segHash = SHA-256(prev ‖ pattern ‖ meta ‖ counter)`.
- **Checkpoints** every N segments (default 16) commit the interval:
  `checkpoint_n = SHA-256(checkpoint_{n-1} ‖ SHA-256(seg_i ‖ … ‖ seg_j))`.
  Checkpoints make the temporal statement **cheaper to verify** — a sampled run covers
  O(N/16) segments plus the checkpoint chain instead of the full log. This is a constant-factor
  reduction (still asymptotically linear), not a logarithmic membership proof. A Merkle-tree
  accumulation over checkpoints (O(log N) inclusion proofs for any segment) is explicitly
  **future work**, not part of this draft.

The same temporal evidence, a fraction of the verification cost.

---

## 3. Block / Segment Format (v1)

```
Segment {
  counter:        uint64            // strict monotonic — replay rejection
  prevHash:       bytes32           // previous segment hash (genesis: 0x00…00)
  pattern:        quantized vector  // per-channel means, 2 decimals, fixed order
  proofStatus:    enum              // verified | challenge_passed | resumed
  emergence:      compact object    // { events, firstCross, ramp, stableRun }
  challengeRef:   optional bytes16  // woven v1.3 challenge id
  deviceKeyId:    bytes16           // fingerprint of the device PUBLIC key (§4)
  timestamp:      relative ms       // segment start offset in session — wall
                                    // clock is deliberately NOT chained
  sig:          bytes64             // ECDSA-P256 over segHash (device key, §4)
}
segHash = SHA-256( "edna2|segment|" ‖ counter ‖ prevHash ‖ pattern ‖ proofStatus ‖
                   emergence ‖ challengeRef ‖ deviceKeyId ‖ timestamp )
```

**Deliberately excluded:** wall-clock time, raw signals, keystroke content, pointer paths, PII,
private key material. The chain carries quantized pattern commitments and signatures only.

**Checkpoint format:**
```
Checkpoint { index, prevCheckpoint, intervalHash: SHA-256(concat(segHashes since last)),
             segmentRange: [from, to], sig }
```

---

## 4. Genesis, Device Key & EDNA ID

1. **Enrollment (explicit user action):** user chooses a **handle**.
2. **Device keypair:** an ECDSA P-256 keypair is generated on-device at genesis
   (`crypto.subtle.generateKey`, private key non-extractable, platform-keystore wrapped where
   available). The chain stores only a **public-key fingerprint** — never private material.
3. **Handle commitment:** `handleCommit = ECDSA-Sign(devicePrivateKey,
   SHA-256("edna2|handle|" ‖ normalize(handle) ‖ "|" ‖ deviceKeyId))` —
   an **ECDSA signature**, not an HMAC: the device key is an asymmetric ECDSA-P256 keypair
   (and the private key is non-extractable, so a raw-key HMAC would not even be possible).
   The commitment is publicly **verifiable against `devicePublicJwk`** — anyone holding the
   exported public key can confirm that this device key signed this handle binding, without
   any shared secret. No plaintext handle anywhere in the chain.
4. **Genesis segment:** first verified segment, `prevHash = 0x00…00`, signed with the device key.
5. **EDNA ID (display form):**
   `ednaId = "edna1:" + base32hex(SHA-256(genesisHash ‖ handleCommit))[0:26]`
   — no personal data, not reversible to the handle.
6. **Device binding:** the identity **is** the key. A new or different device generates a new
   keypair and therefore a **new EDNA ID** — no cross-device identity in v1.
7. **Revocation / deletion (scoped to local storage):** private key destroyed → no future
   segment can be signed → chain permanently unextendable; vault and memory model erased.
   Deletion here means **complete deletion of everything EDNA ID holds on this device**. It
   does **not** reach copies that already left the device: segments or exports transmitted or
   backed up earlier cannot be un-transmitted or retracted by the engine (same semantics as
   `profile-hash` in EDNA-1). "Complete deletion" is therefore always stated as *local*.

---

## 5. Identity Confidence — an honest, measured curve

The "99 %" is a **research target, not a shipping promise**:

```
confidence(identity) = f(verifiedSegments, distinctSessions, challengePasses,
                         overlap(liveAggregatedWindow, referenceVault))
```

- First minutes: ~0.85–0.90 — "probable same human". **This range is a preliminary design
  hypothesis, not a measured value** — until the long-term studies land, every published
  number on the early curve is labeled provisional. The implementation's starting constants
  (0.85 base + segment/session/challenge growth, capped at 0.999) are placeholders for that
  curve, chosen to be monotonic and conservative, and are subject to recalibration.
- Multi-session growth with challenges: converging toward >0.99 **as measured by long-term
  studies** (§0)
- The measured curve is published with the v2 study results — never as a blanket claim

**Resume after pause (spec-critical):** resuming is **not** decided by the last hash. It is
decided by comparing **stored reference + freshly aggregated behavior data** (Layer 2 overlap)
against a threshold; below threshold a challenge is required. The hash chain proves
continuity *after* the match; the match itself is behavioral.

---

## 6. What the chain adds to bot detection (and what it doesn't)

| Attack | Defense |
|--------|---------|
| One-time pass, then bot | Verification expires; chain stalls; challenge fires |
| Replay of recorded segments | Counter + prevHash binding; device-key signature |
| Cross-session replay | Resume gate = reference + fresh behavior, not stored hash |
| Uniform synthetic input | Chain-level entropy signature across segments |
| Session/ID theft | EDNA ID is behavioral + key-bound, not a bearer token |

**Does not:** stop a bot that sustains convincing mimicry (§0). The honest pitch: *continuous
cost*, not *impossibility*.

---

## 7. API Surface (EDNA ID layer)

```
ednaId.chain.status()         → { segments, checkpoints, counter, head, integrity,
                                  lastVerification, pausedFor }
ednaId.chain.genesis(handle)  → { ednaId, deviceKeyId }      // user-initiated
ednaId.identity.confidence()  → { confidence, basis }        // measured curve
ednaId.identity.verifyResume(liveAggregatedWindow)
                              → { match, threshold, resume: ok|challenge_required }
ednaId.memory.live()          → Layer 1: emergence, confidence path
ednaId.memory.session()       → Layer 2: timeline deltas, session comparison
ednaId.memory.longterm()      → Layer 3: evolution vs own historical reference
ednaId.export()               → hashes/checkpoints only, consent-gated
ednaId.revoke()               → destroy key + vault + model (complete LOCAL deletion —
                                already-exported copies cannot be retracted, §4.7)
```

### 7.1 EDNA ID Account identity tools (v1.4.2-main.1)

Device-bound identity enrollment and login on top of the chain (MCP:
`edna_identity_create` / `edna_identity_verify`):

```
chain.enrollStatus()          → { state: none|building|sealed, verifiedMs,
                                  remainingMs, requiredMs: 180000 }
chain.enrollIdentity(handle)  → building… then seals a device-signed identity anchor
                                block; requires full human verification time — pauses
                                farm nothing (clock ticks only in feed(), max 2 s/call)
chain.verifyIdentity(handle, liveWindow)
                              → { handleMatch, match, channels, streak,
                                  authenticated }   // 3 consecutive 100 % matches
                                                  // → authenticated (EDNA ID login)
```

Login growth: every successful verify at streak ≥ 3 closes an `identity_login` growth
segment (rate-limited to one per 10 s), so the chain keeps strengthening while the user
is logged in. Genesis still requires `human_verified` at confidence ≥ 0.999.

Everything returns hashes, quantized patterns, signatures, derived scores. **Raw behavior
never leaves the device** — invariant since v1.

---

## 8. Verification & Test Vectors (v2 suite)

1. `human_organic_10min` — organic session: chain + checkpoints grow; confidence curve recorded
2. `pause_resume_2h` — 2 h interruption: resume decided by reference + fresh data; seamless continuation
3. `replay_attack` — recorded segments replayed: rejected (counter/prevHash/signature)
4. `tamper_past` — modified history: checkpoint verification fails; integrity = broken
5. `mimicry_bot_30min` — sustained mimicry: behavioral scorers + challenges flag (chain alone: documented non-claim)
6. `challenge_woven` — v1.3 challenge inside chain: missing/false link breaks verification
7. `device_change` — same handle, new keypair: NEW EDNA ID; old chain unextendable
8. `revocation` — key destroyed: permanent; nothing re-derivable
9. `checkpoint_efficiency` — sampled verification covers O(N/16) segments + checkpoint chain
   (constant-factor reduction, still linear; Merkle accumulation = future work, §2.4)

---

## 9. Decisions Log (locked)

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-11-09 | Event-driven growth with adaptive bundling + checkpoints every 16 segments | Temporal evidence at a fraction of verification cost; no per-event persistence |
| 2026-11-09 | Three memory layers (live / session / long-term); historical truth append-only, memory model replaceable | Profile updates never rewrite history |
| 2026-11-09 | Device binding via ECDSA-P256 keypair (non-extractable private key); chain stores public-key fingerprint only | Identity **is** the key; new device = new EDNA ID |
| 2026-11-09 | Resume = stored reference + fresh aggregated behavior vs threshold — never last hash alone | Hash proves continuity, behavior proves identity |
| 2026-11-09 | Wall-clock time never enters the chain (relative offsets only) | No timing side-channels, no cross-session linkage |
| 2026-11-09 | Provability chapter is part of the spec | Boundary between architecture and scientific proof is explicit |
| 2026-11-09 | Product split: EDNA-1 (Sensoric Engine) / EDNA ID (Identity & Continuity layer) | Independent evolution; ID layer is new, not retroactive |
| 2026-11-09 | Handle commitment = ECDSA signature (not HMAC) over the normalized handle; verifiable against the exported public key | Device key is an asymmetric, non-extractable ECDSA key — a raw-key HMAC would be impossible and less verifiable |
| 2026-11-09 | Checkpoint verification stated as constant-factor (O(N/16), still linear); Merkle accumulation over checkpoints = future work | Honest complexity claim; logarithmic membership proofs are a v2+ option, not implied |
| 2026-11-09 | Deletion claims scoped to local storage; already-exported copies explicitly out of reach | Revocation destroys everything the engine holds, but cannot un-transmit history (same semantics as EDNA-1 profile-hash) |
| 2026-11-09 | Early confidence range (0.85–0.90) labeled provisional design hypothesis | Prevents a placeholder from being read as a measured result until the long-term studies land |
| 2026-11-11 | Identity enrollment = ≥ 3 min organic human-verified interaction (ENROLL_MIN_MS 180000), device-signed anchor; 3× 100 % verify-to-login; login growth segments | An EDNA ID Account must be earned by live behavior, not minted; the chain strengthens while logged in |

*SEBKLAEY© Agency — Sebastian Kläy · sebklay@me.com*
