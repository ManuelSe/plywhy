# PlyWhy — coding agent instructions

Goal: implement a secure, tested native Android chess-learning app described in `docs/PRODUCT.md`, `docs/COACHING.md` and `PLANS.md`.

## Before coding
- Read `README.md`, `docs/PRODUCT.md`, `docs/COACHING.md`, `docs/AUTH_AND_AI.md`, `docs/ARCHITECTURE.md`, `docs/TESTING.md`, and `PLANS.md`. Use `docs/REFERENCES.md` to verify external integration details; official up-to-date documentation supersedes stale assumptions.
- Maintain `PLANS.md` (work plan and decisions) and `STATUS.md` (completed work, evidence, blockers, next steps). Do not label an untested function as complete.
- Prefer simple maintainable Kotlin + Jetpack Compose, unidirectional state flow, testable domain logic, and small focused modules. Reuse vetted chess libraries where appropriate instead of implementing chess rules from scratch.

## Autonomous execution
- Work through the milestones in `PLANS.md` without requesting approval for normal coding choices, local builds, tests, refactors, or non-destructive local commits.
- Run appropriate unit tests and Android build/lint after each milestone; repair regressions before advancing. Add an automated regression test when fixing a bug.
- If an emulator/connected device is available, run on it and validate major interactions. Never claim device verification unless executed.
- If blocked by account sign-in, missing hardware, access limitations, unsupported OAuth, or external services, collect evidence, document the exact blocker, and continue with independent tasks where safe. Do not fake login/inference verification or replace ChatGPT auth with an insecure workaround.
- Never push, publish, deploy, use a paid API, request purchases, perform destructive Git operations, or change license policy without user approval. Do not embed secrets, tokens, keys or machine-specific paths in tracked files.
- Do not use `--dangerously-bypass-approvals-and-sandbox` or equivalent unrestricted execution flags by default.

## Non-negotiable behavior
- Stockfish is the authority on move strength, not the LLM. No LLM text may change game legality or move acceptance decisions.
- Keep the high-strength **evaluator** separate from the deliberately weaker **opponent**.
- Rejecting a candidate move must not alter the committed board, game history, castling/en-passant rights, clocks, or repetition state.
- Accept near-equivalent moves; handle mate and stalemate correctly; avoid overinterpreting small score differences.
- Before solution reveal, hints and explanations must not accidentally disclose the best move, UCI/SAN notation or destination squares.
- AI is a progressive enhancement. Chess play and local evaluation remain usable without AI access.

## Definition of done
- Android debug APK builds successfully; deterministic unit tests pass; instrumented UI tests and device checks pass when an Android device is available; `STATUS.md` clearly distinguishes verified, unverified and blocked checks.
- Required coaching flows work with real engine evaluations, including no-spoiler retries and an accepted-move explanation.
- All third-party licenses and shipped Stockfish source/build provenance are documented; no credentials are committed.
- Provide reproducible build/run instructions and the output APK path. Do not claim completeness on mock-only proof.
