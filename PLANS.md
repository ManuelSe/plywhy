# PlyWhy autonomous implementation plan

This is a **living plan**. Codex owns routine execution and updates. Verify each milestone before moving on; if an external dependency blocks a milestone, document evidence and proceed with independent implementation using a clearly marked test fake/offline mode. Keep milestones focused and independently shippable.

## M0 — Feasibility and repository foundation

- [ ] Create complete Gradle/Kotlin/Compose Android project, version catalogs/Gradle wrapper, basic UI and unit test harness, reproducible build steps.
- [ ] Investigate official SIWC Android loopback auth constraints; implement safe OAuth proof of concept if compatible. Verify on emulator/device when possible. Handle signed-in model catalog and streaming response completion. Verify `gpt-6.1-sol` low reasoning **only if account offers it**.
- [ ] Establish `CoachProvider`/auth abstraction and fully functional offline/fake provider to unblock the rest without paid services.
- [ ] Verify chosen approach for local Stockfish on Android (NDK/process/JNI etc.), document exact plan and license/source provenance. Include smoke-test harness.
- [ ] Run initial compile/test/real-device checks as available; record failures and blocked external verification.

## M1 — Complete playable local chess

- [ ] All chess rules via tested library, legal tap moves, board rotation, promotion, terminal outcomes, safe undo, local save/resume.
- [ ] Start actual on-device Stockfish, build high-strength evaluator and independent weaker opponent controls, enforce no-UI-blocking analysis.
- [ ] Engine moves legally, ratings appropriately described; integration tests and APK build pass.

## M2 — Deterministic coaching controller

- [ ] Pure coaching state machine and candidate-versus-best policy with opening leniency and strict post-opening mode, configurable presets.
- [ ] Reject/preview/retry flows; near-equal alternatives accepted; mate-aware comparisons; no loss of game-state rights.
- [ ] Create 24+ verified golden positions and automated tests per `docs/TESTING.md`.
- [ ] Engine-only explanations/variations and fallback work when AI offline.

## M3 — AI coaching with guarded explanations

- [ ] Feed typed validated Stockfish evidence to the AI provider, including stage-specific spoiler context. Concise feedback for accepted/rejected moves.
- [ ] Progressive hints, explicit reveal, follow-up question UI; verify any hypothetical line using bounded engine analysis.
- [ ] Validate output and refuse to display obvious leaks/hallucinated variations; fall back gracefully on errors.
- [ ] Real SIWC inference verification if feasible; account/consent and quota failure UI. Do not use paid API unless expressly authorized.
- [ ] Review >=20 representative position explanations with real responses **if auth available**, otherwise explicitly mark this validation blocked.

## M4 — Harden and finish the MVP

- [ ] Polished Material 3 UI, TalkBack labels, loading/error states, settings, navigation and local persistence migrations.
- [ ] Regression/instrumented tests, unit tests, lint, emulator/device smoke tests, graceful sign-out/network failure.
- [ ] License audit (GPLv3 source/build provenance for Stockfish, notices), no-secrets audit, update README.
- [ ] Produce/installable debug APK, SHA-256, exact build commands and an evidence report in `STATUS.md`.

## Decision log

- 2026-10-28: Native Kotlin/Compose, offline engine analysis, AI as enhancement, GPLv3 project starter.
- 2026-10-28: Opening defaults to 8 **full moves**, not 8 plies; adjustable 5–10. Strong evaluator separate from weak opponent.
- 2026-10-28: Prefer ChatGPT plan OAuth with GPT-6.1 Sol `low` if actually supported by the user's account; no implicit paid API fallback.

## Working rules

- At each milestone: implement → test → fix → document → commit locally → continue.
- Respect `AGENTS.md` boundaries; no remote publish/push or paid API without user consent.
- Blockers are exact and evidence-backed. Do not mark OAuth or device tests as passed based only on a unit mock or desktop proof.
- Do not endlessly chase non-essential polish. Prioritize functional coaching and a verifiable working app.
