# Technical architecture

## Technology

- Kotlin, Jetpack Compose, AndroidX architecture components, Gradle, Kotlin coroutines/Flow; select current stable compatible releases at implementation time. Use the Android SDK available locally, and document `minSdk`/`targetSdk` choices. Suggested initial `minSdk=26`, subject to engine compatibility.
- Prefer ViewModel + immutable UI state + repository interfaces, with a domain layer independent of Android frameworks, especially for move evaluation and the coaching state machine.
- Use a well-maintained chess rules / PGN / FEN library with suitable licensing if available. Validate move legality fully; don't build naive piece movement in UI code.
- Run a *real Stockfish engine on-device*: a separate high-strength evaluator plus adjustable-strength opponent. Choose an Android-compatible approach after verifying current Android execution restrictions and native packaging; do not assume a desktop UCI binary can just be copied to app-private storage and executed. Any JNI/NDK or native executable technique needs instrumented validation on a real Android runtime.
- Store games/settings locally with Room or DataStore (choose appropriately); secure tokens separately with Android Keystore-backed storage; don't put credentials in logs, preferences exports, saved games or backups.

## Modules / interfaces (suggested, not mandatory)

- `chess-core`: `GameState`, legal moves, FEN/PGN, SAN/UCI conversion, history, clocks/counters, draw claims.
- `engine`: `EngineSession` (UCI/process/NDK bridge), `PositionAnalysis` (score, mate, normalized perspective, PV), `OpponentEngine` (limited-strength separate instance/configuration).
- `coaching`: pure `MoveAssessmentPolicy`, `CoachSession` state machine, `HintStage`, typed evidence and analysis snapshots. Contains **no API/Android UI code**.
- `ai`: `CoachProvider` interface, `ChatGptOAuthProvider`, potential optional `ApiProvider`, prompt/evidence formatter, streaming, follow-up question handler, error handling.
- `data`: local game repository, settings repository, migrations.
- `ui`: Compose screens and accessibility, reactive flows.

Interfaces should be mockable in tests. Avoid making inference requests from composables.

## Turn pipeline

```
User candidate (legal UCI)
  -> store as provisional attempt, board remains committed
  -> high-strength evaluator computes MultiPV and candidate-specific score
  -> move-policy determines ACCEPT / INTERVENE / UNDECIDED
  -> if ACCEPT: commit once -> request bounded AI explanation -> opponent picks legal move
  -> if INTERVENE: evidence-grounded feedback -> unlimited retries/hints/reveal
  -> if UNDECIDED: increase search effort within time budget or conservatively accept
```

An engine-only analysis screen should remain usable when AI fails. Do not let an API response change whether a candidate is legal or accepted.

## Engine analysis details

- Maintain one coherent search configuration per comparison; evaluate from the same parent position and depth/nodes budget, normalize scores to the learner's perspective. Engine search can be non-deterministic with time limits; prefer fixed nodes for reproducible automated tests.
- Request MultiPV (e.g. top 3–5 moves) and also evaluate a candidate move that may not be in MultiPV (restricted root search / searchmoves if supported). Ensure comparisons represent the *same* position and side-to-move; do not subtract incompatible scores.
- Treat mate scores separately from centipawns, including sign and distance. Stable legal forced wins/losses take precedence over small centipawn tolerances.
- Normalize/check principal variations against legal moves from parent; reject corrupt/stale engine responses.
- Use a bounded analysis budget and allow user cancellation; never block main UI. Cache keyed by exact position and analysis parameters, not just board placement (castling, en-passant, move counters matter for rules).
- Opponent strength settings must not weaken evaluator strength. Distinguish engine quality from opponent strength in configuration.

## State consistency

- The authority for board state is a committed immutable sequence of legal moves plus initial FEN.
- Candidate moves, ghost previews, hint stage, streaming explanation, and chat transcript are **session UI state**, not board moves.
- Every committed move updates castling rights, en passant square, halfmove/fullmove counters, repetition tracking and turn exactly once.
- Use SAN for user-facing move notation and UCI internally; avoid mixing SAN and UCI in API payloads without labels.
- Promotion must offer queen/rook/bishop/knight; handle checkmate, stalemate, insufficient material, 50/75-move and 3/5-fold repetition correctly, including the distinction between automatic and claimable draws. Test both colors.

## Offline operation

- Local play, basic engine assessment and candidate retries do not require an account or network.
- If AI is offline, show engine-backed concise evidence (e.g. evaluation difference and a legal variation) and a clear AI-unavailable label. Do not generate fake strategic explanations.
- AI requests are triggered by acceptance, intervention, hint or explicit follow-up, not on every board render or repeated recomposition.

## Performance, UX and logging

- Debounce simultaneous taps; cancel stale analysis when game state changes; ignore out-of-order AI responses by request ID/position hash.
- Show analysis spinner and fallbacks; use short-lived per-position context rather than dumping entire game every request.
- Keep development logs useful but avoid tokens, emails, conversation histories, private account identifiers and raw authorization headers. Provide optional opt-in diagnostic export with sanitized data only.
- APK should build with documented Gradle wrapper and Android Studio; Codex must choose compatible versions and validate them.
