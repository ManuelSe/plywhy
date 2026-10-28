# PlyWhy — product specification (MVP)

## Vision

PlyWhy teaches **how to reason about chess moves** through active play, retrying poor decisions, receiving graded hints, and understanding the difference between merely legal/okay and genuinely strong moves. It targets a new player (approximately 600–800 Elo initially) but should remain useful as their playing ability develops. Avoid rote opening memorization as the primary experience.

## Product principles

1. **Attempt before answer.** The learner chooses a legal candidate before seeing the engine's solution.
2. **Intervene sparingly in openings.** Natural opening practice should not become a best-move guessing game.
3. **Accept alternatives.** Equivalent or nearly equivalent moves count as successes; no arbitrary single-move puzzles unless objectively forced.
4. **Explain the why.** Every accepted move should have a short evidence-grounded explanation, with details available on request.
5. **The user controls retries and hints.** No retry limits or timer pressure.
6. **Reliable chess first.** The board is a genuine game; castling, en passant, promotion, checks, draws and history always work.
7. **Local by default.** Stockfish analysis runs on the device; no chess server needed. Optional authenticated AI enhances explanations.

## Primary flow

1. Open app → resume the last game or start a new game.
2. Choose color (White by default), coaching preset, opponent difficulty, opening-coaching length (default **8 full moves**, adjustable 5–10).
3. Board shows the current position, whose turn, last move, and controls. Tap/drag a piece to submit a legal **candidate**. Illegal moves are never accepted.
4. **Opening stage (first N full moves by the learner's game, counting from move 1):** normally commit the candidate move immediately. If evaluation finds a consequential blunder/mistake or clear missed tactical opportunity, suspend and open the coaching overlay rather than committing it. Opening deviations that are sound are welcome.
5. **Best-move stage:** evaluate each candidate against Stockfish's strongest viable alternatives. Accept moves effectively as strong, otherwise reject provisionally and allow another attempt. No forced revealing.
6. **On rejection:** show a short explanation of what goes wrong without spoilers; offer Retry, Hint, Ask coach, Reveal solution. The board remains at the **pre-attempt position**. The attempted move may be ghosted/previewed, not committed.
7. **On acceptance:** commit the candidate once; show a concise explanation of its merit. If it is nearly equivalent rather than the principal variation, say so; provide optional “Compare alternatives” and “Ask why.” Then the opponent plays a legal move, and the flow repeats.
8. When either side is checkmated, stalemated or drawn (including rule-dependent claimable draws), show outcome and allow review/new game.

## In-scope capabilities

- Native Android app (phone portrait first), smooth accessible chessboard, legal move highlights, tap-to-move (drag optional), board flip, captured pieces optional.
- Play as White or Black vs configurable-strength engine; analysis always uses an unrestricted or strong separate evaluator; do not equate advertised opponent level to exact Elo without validation.
- Candidate preview, strong/mistake assessment, opening leniency, strict best-move stage, multiple acceptable moves, unlimited retries.
- Explanation after every accepted learner move, and after rejected moves. Explanation must distinguish: forcing tactic, positional/strategic preference, prophylaxis, or an unclear/engine-only preference. Never invent causes.
- Progressive spoiler-safe hints, explicit reveal, follow-up coach chat grounded in position and verified engine continuation.
- Local Save/Resume, New Game, Undo (see below), Settings, Offline/AI status, clear error messages.
- Gracefully handle poor device performance, missing engine, AI sign-out, token expiry, unavailable model, network failure, quota exhaustion, and cancellation.

## User-facing settings

- Learner side: White / Black; default White.
- Opening duration: 5 to 10 **full move numbers**, default 8. Counting is based on the game move number, not number of coaching requests; for Black, opening ends after Black's 8th move.
- Opponent strength: e.g. Beginner / Developing / Club / Strong (labels, not guaranteed Elo). Default Beginner, with readable description.
- Coach strictness: Relaxed / Balanced / Strict; default Balanced. Clear that strictness may vary by position; show the reason for a decision.
- AI provider: ChatGPT plan (preferred when eligible); Offline engine-only. A future optional separately billed API mode requires explicit opt-in and secure architecture.
- Model: prefer `gpt-6.1-sol`, reasoning `low` where available to the authenticated account. Show what is *actually* available.
- Board orientation/color and sound/haptics options; no background analytics requirement.

## Undo and retry semantics

- **Rejected candidate:** not added to game history. Retry resets preview only, never the actual game.
- **Undo:** clear distinct action to take back the last completed learner+opponent move pair where possible (or last ply if game not paired); restore complete reversible game state including FEN rights and repetition data. Do not allow Undo to bypass the retry mechanism unintentionally during an unresolved candidate.
- **Resumption:** persist committed move list, starting FEN, game options, current state, in-progress coaching attempt if appropriate, and versioned save schema. Never store OAuth access tokens inside game saves.

## Out of scope for v1

- Learning/weakness tracking, personalized lesson generation, spaced repetition, achievements, cloud sync, multiplayer, opening database explorer, chess accounts, camera board recognition, Play Store publication, advertising or monetization.
- Building a new chess engine or a comprehensive training content library.

## MVP acceptance criteria

- A new user can complete a full legal game against a weaker opponent with the strong evaluator coaching each learner move as specified.
- A sound, non-top-ranked opening move is not routinely interrupted; a clear blunder triggers intervention.
- Later, a near-equal second-best move is accepted, while a substantially inferior move yields a spoiler-safe explanation and unlimited retries.
- After revealing a solution, the user explicitly chooses a legal move to proceed (or selects a clear “Play revealed move” action); no invisible move changes.
- Every accepted learner move receives a short explanation, including explicit uncertainty when engine evidence does not support a confident human-readable reason.
- Offline and auth-unavailable scenarios still support play and Stockfish analysis, with an honest “AI explanation unavailable” state, not fabricated LLM feedback.
- FEN legality, game history, repetitions and draw states are correct; the automated tests in `TESTING.md` pass.
- Signed-in ChatGPT model availability is **verified on device** before claiming that GPT-6.1 or plan usage works there. Successful builds alone do not establish this.
