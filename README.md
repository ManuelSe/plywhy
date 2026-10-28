# PlyWhy

**Understand every move.** An Android chess-training app in which a strong local engine evaluates your candidate moves and an AI coach explains *why* a move is better, rather than merely revealing the answer.

> Status: **specification and Codex bootstrap only**. This repository starter does not yet contain a buildable Android app. Codex is expected to implement it.

## Quick start

1. Install Android Studio (Android SDK + emulator or connect a physical Android device), Git, a suitable JDK, and the latest Codex CLI.
2. Check `codex --version` (Goals require Codex CLI >= 0.128.0).
3. Open the project root in a terminal and run `codex`; authenticate normally.
4. Paste the complete text from [`prompts/START_CODEX.txt`](prompts/START_CODEX.txt) into Codex. Keep the session running while it works; Goals are persistent within a thread, **not an unattended background service**.
5. If Codex asks for ChatGPT sign-in or an Android device approval, perform that step. Follow progress in [`STATUS.md`](STATUS.md).

The launch prompt directs Codex to plan and implement milestones autonomously, build and run tests, fix failures, and stop only when success is verified or a real blocker requires action. Codex must not silently substitute paid API usage for ChatGPT plan usage.

## Product scope

- First 8 **full moves** by default (configurable 5–10): normally allow play, intervening only on meaningful mistakes or missed opportunities.
- Afterwards: accept any move sufficiently close to the best Stockfish move; reject materially inferior moves, explain why, and let the player retry indefinitely.
- Progressive, non-spoiling hints; explanatory feedback for **accepted** moves as well as mistakes; optional questions to the AI coach.
- Adjustable-strength **opponent** separate from high-strength evaluator.
- Local game persistence, responsive board, reliable legal moves, and an AI-free fallback when offline or not authenticated.
- **Not v1:** learning-progress tracking, spaced repetition, curated lesson courses, cloud accounts/sync, multiplayer, monetization.

## Specifications

- [PRODUCT.md](docs/PRODUCT.md): user journeys, scope, acceptance criteria.
- [COACHING.md](docs/COACHING.md): exact coaching state machine, comparison logic, hints, anti-spoiler rules.
- [ARCHITECTURE.md](docs/ARCHITECTURE.md): Android, engine, architecture, game-state handling.
- [AUTH_AND_AI.md](docs/AUTH_AND_AI.md): Sign in with ChatGPT, inference, security and fallbacks.
- [UX.md](docs/UX.md): screens, content, accessibility and visual guidance.
- [TESTING.md](docs/TESTING.md): validation strategy and chess test positions.
- [REFERENCES.md](docs/REFERENCES.md): documentation links and constraints to reverify.
- [PLANS.md](PLANS.md): autonomous implementation milestones.
- [STATUS.md](STATUS.md): live progress and evidence maintained by Codex.
- [AGENTS.md](AGENTS.md): concise repository-wide Codex instructions.

## License

PlyWhy starter materials are licensed under GPL-3.0-only (see `LICENSE`). Any shipped Stockfish binaries must comply with Stockfish GPLv3 license, source availability, notices, and attribution requirements. Codex should verify licensing for all other dependencies before including them.
