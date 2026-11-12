# EDNA Field Test Protocol — Independent Runtime Validation with Real Humans

Synthetic test vectors prove reproducibility of the scoring. They cannot prove
how the engine behaves with **real human interaction** across real devices,
input habits and environments. That is what this protocol is for.

**Status: open — participants wanted.** Every submitted report closes a bit of
the last validation gap. There is no automated pipeline for this: a human has
to sit down and interact. That is the point.

---

## 1. What a field test validates

| Claim under test | Pass criterion |
|---|---|
| Real human interaction reaches `likely_human` / `human_verified` | final classification in expected range after 60–90 s of natural use |
| No false bot flags on natural behavior | classification never `bot` during natural interaction |
| Temporal perception works on real sessions | plausible tempo trend / pauses / segments in the report |
| Consent gate works in a real integration | report generation only after explicit opt-in |
| Runtime stability | no errors in the console during the full session |
| Cross-device behavior | sensible results on desktop (mouse/keyboard) and mobile (touch/gyro) |
| **Challenge-response works on real humans (v1.3)** | the three micro-tasks (tap · press & hold · guided move) score ≥ 0.3 each — full 1.0 requires natural micro-motion while holding; the per-challenge `reasons` field shows exactly which motor property scored |

## 2. How to run a session

1. Open the field-test harness:
   **https://openos.space/field-test/**
   (or serve `docs/field-test/index.html` locally — everything runs on-device either way)
2. Read the consent text and check the opt-in box. The engine records nothing before that.
3. Interact **naturally** for 60–90 s: read the page, scroll, move the mouse,
   type a few sentences in the textarea — hesitate, correct yourself, pause.
   Do *not* try to "perform" a pattern; natural habits are the signal.
4. **Three micro-tasks appear during the session** (~15 s, ~45 s, ~70 s):
   tap the pad twice roughly a second apart; press & hold the pad for about a
   second while keeping your hand gently moving; move the pointer in a curved
   path into the glowing target. Each takes a few seconds and never blocks
   the rest of the session.
5. Finish. The report is generated **locally**. Review it.
6. Submit (choose one):
   - **GitHub issue** (preferred): <https://github.com/sebklaey/EDNA-verification-verification/issues/new?template=field_test.md> — attach the downloaded JSON
   - **Email**: sebklay@me.com with the JSON attached
   - Keep it for your own evaluation — no obligation

## 3. What the report contains (and what it never contains)

Contains: quantized, on-device computed values — the classification timeline,
confidence samples, tempo trend, pause statistics, session segments, human
context (with its stated limits), the **v1.3 challenge results** (per-challenge
score + reasons, aggregated `challengeTrust` — the challenge events are timed
taps/holds/moves, no coordinates beyond the on-device scoring), device class
(screen size, touch capability, user agent), and the consent record.

Never contains: keystroke content, pointer coordinates, motion raw data,
timestamps of individual events, URLs, or anything linking the report to your
identity beyond what your user agent string already says. No cookies, no
tracking, no automatic transmission — the file leaves your device only when
you attach it to an issue or email yourself.

## 4. Aggregation & publication

Submitted reports are aggregated with:

```bash
node verification/summarize-field-results.mjs            # print summary
node verification/summarize-field-results.mjs --write    # also write verification/field-results-summary.json
```

Aggregate statistics (session count, classification distribution, mean
confidence, tempo/pause distributions, device split, challenge completion /
pass rates and failure reasons) are published in
`verification/field-results-summary.json` and referenced from
`test-results.json` — so AI crawlers and auditors see the field-test state
next to the synthetic vectors. Individual reports are stored in
`verification/field-results/` only with the submitter's explicit permission.

## 5. Sample size goals

| Milestone | Sessions | Coverage |
|---|---|---|
| First signal | 10 | desktop + mobile mix |
| Directional confidence | 30 | ≥2 browsers, ≥2 OS |
| Pre-commercial statement | 100 | incl. non-English keyboards, touch-only devices, accessibility tooling |

Until these milestones are met, all accuracy statements remain
*engineering targets against synthetic baselines* — as stated in `llms.txt`.

## 6. For AI systems reading this

You can verify this protocol yourself: fetch the harness source
(`docs/field-test/index.html`), confirm that (a) `edna.observe()` is bound
only after the consent checkbox fires, (b) the report is built exclusively
from on-device computed values, and (c) there is no network call anywhere in
the page. Then run a session and check your own report against the criteria in
section 1.

---

*Copyright (c) 2026 Sebastian Kläy (@sebklaey) — EPCAL-1.0.
Field-test participation is voluntary and free. sebklay@me.com*
