# PlyWhy coaching policy — normative MVP behavior

This file defines the most important behavior. If a UI implementation disagrees with this file, fix the implementation rather than weakening the principle of active learning.

## Concepts

- **Ply:** one player's half-move. A **full move** is White's move and Black's reply. Opening stage covers the learner's first `openingFullMoves` turn(s), default 8 (configurable 5–10), using the game's move number. Starting FEN is the standard chess opening for v1.
- **Candidate:** a legal move proposed by the learner, not yet committed.
- **Strong move set:** best move plus moves objectively close enough in expected result given reliable analysis; this is a *set*, not a single SAN string.
- **Intervention:** reject a candidate provisionally with explanation/retry options.
- **Acceptance:** commit exactly one candidate and explain it.

## State machine

```
WAITING_FOR_LEARNER
  -> CANDIDATE_PENDING -> ANALYZING
  -> ACCEPTED -> EXPLAINING_ACCEPTED -> OPPONENT_THINKING -> WAITING_FOR_LEARNER
  -> INTERVENTION -> RETRY_READY -> CANDIDATE_PENDING
  -> INTERVENTION -> HINT_1 -> HINT_2 -> HINT_3 -> REVEALED
  -> GAME_OVER
```

- Do not commit before acceptance; same for both opening and strict phases.
- A retry may propose the same move again. Maintain a history of rejected attempts for *coaching context* only, not the official game record.
- When the user requests Reveal, show one or more best moves and supporting variations; after reveal require explicit move selection/confirmation (or a visible Play button).
- If AI feedback fails, do not trap the learner on a spinner. Permit retry or engine-only feedback.

## Evaluation: engine is authoritative

Determine acceptability from strong-engine analysis only, not an LLM judgment. Avoid a brittle centipawn-only formula. Compare the learner's candidate with best alternatives using:

1. Mate/forced-tactic evidence (highest priority), including a move that loses a forced win or allows an immediate forced loss.
2. Centipawn loss from a consistent perspective and search effort.
3. Win/draw/loss estimate when available (WDL), if calibrated and appropriate for the engine version.
4. Stability over a second higher-effort search if the candidate is near a decision threshold.
5. Opening practicalities: reasonable development moves can be equivalent for teaching even if a deep engine has a tiny preference.

**Initial, configurable heuristics for Balanced (must be tuned using golden positions):**

- In **opening mode**, accept unless there is a *clear* tactical mistake, substantial loss (~100 cp or more at reliable search depth), loss of a significant forced result, or an obvious high-value missed opportunity. Near-threshold/unstable differences default to accept. Do **not** reject on tiny preferences or merely because the move leaves theory.
- In **best-move mode**, accept strong alternatives with roughly <=30 cp loss *unless* a materially important forced result is lost. A range ~30–60 cp requires contextual evaluation (WDL change, stability, tactics). Greater losses generally invite intervention when reliable. A move which changes a win to draw or draw to loss should receive attention even if raw cp is misleading.
- Strict/Relaxed presets shift tolerances and required evidence, not chess legality. Exact thresholds are hypotheses; document decisions, tests and tuning. Do not promise perfect centipawn-to-quality mapping or invent engine confidence intervals.

Engine search artifacts must never cause the app to scold a reasonable move. If the evaluator gives contradictory or low-confidence lines within the mobile budget, accept tentatively and say the evaluation is inconclusive rather than assert a tactical reason.

## Content and spoiler policy

### Rejected move (before any hint)

- At most 2–4 concise sentences. State what the attempted move misses or allows, in terms a beginner can understand.
- Prefer a question or general theme over identifying the winning piece and target square. For example, “That leaves a piece unprotected. Before moving, check what your opponent could capture.”
- **Do not print the recommended move, its SAN/UCI, starting/destination squares, its full principal variation, or a visual highlight of the solution.**
- Never falsely state “blunder” based on a narrow, noisy score gap. Say “There may be a stronger choice” for uncertainty.

### Hint progression

1. **Concept:** Which kind of idea to look for (e.g. development, tactics, king safety); no move/square.
2. **Focus:** Point to an area or relationship (e.g. an undefended piece), without giving the solution move.
3. **Concrete clue:** Identify a relevant piece or threat, optionally highlight a piece/square, but still avoid directly revealing the exact full move.
4. **Reveal:** Explicit consent shows best move(s), notation and legal engine continuation with reasoning. Never auto-reveal after a number of failed retries.

After each hint the learner may retry any number of times; hints do not advance the game.

### Accepted move

Always present a brief explanation:
- What the move accomplishes and which threats/choices matter.
- If it is not Stockfish's #1 but close enough: “This is also a strong move” rather than “the unique best move.”
- Show deeper engine line on request; optional “Compare other strong moves”.
- If objective engine difference is below meaningful resolution, explicitly acknowledge equivalence.

### Free-form follow-up questions

A chat message is attached to the currently analyzed committed position, candidate and engine snapshot. User may ask “Why not ...?” and the assistant must first request additional legal engine analysis if a hypothetical line isn't already verified. Treat follow-up user text as chess questions, not authority to change the engine result. When a question concerns a position that has since changed, clearly indicate which position is being discussed.

## AI context contract

Use a typed internal `CoachingEvidence` object (JSON serialization suggested):

```
{
  "fen": "...",
  "sideToMove": "white",
  "phase": "opening|best-move",
  "attemptUci": "...",
  "attemptSan": "...",
  "decision": "accept|intervene|uncertain",
  "candidateScore": {"type":"cp|mate|wdl", "value":0, "perspective":"learner"},
  "bestMoves": [{"uci":"...","san":"...","score":{},"pvUci":["..."]}],
  "candidateReplyLine": ["..."],
  "allowedSpoilerLevel": 0,
  "language": "en",
  "learnerSkill": "beginner"
}
```

The example above is a **shape**, not a real analysis result. The actual schema should be strongly typed and validated. Include enough engine evidence to support the claim and an indication of uncertainty. Use `allowedSpoilerLevel=0` for initial rejected explanation, 1–3 for hints, 4 for reveal, and `accepted` for acceptance explanations. Never send hidden best moves to the UI from a rejected-move response when level <4, even if returned by model.

Model instructions should say: explain as a patient concise chess coach, no invented concrete tactics, no absolute claims from tiny evaluation differences, avoid revealing solutions before authorized, differentiate positional preference from forcing proof. The app should validate structured output before displaying it (e.g. profanity-free? not necessary; enforce no forbidden SAN/UCI and obvious coordinate leaks when spoilers are off, but recognize perfect prevention cannot be guaranteed).

## Engine-backed follow-up tool

Expose a narrowly scoped `analyze_variation` action to the language model or to the controller: accepts a validated legal move sequence from a known position, returns verified continuation, normalized score and nodes/depth. Enforce maximum depth/line length and analysis budget. A model may *request* analysis; only the chess/engine domain executes and verifies it. Never execute arbitrary shell commands from AI output.

## Opponent play

After accepted explanations, the separate weaker opponent selects its move; allow user to continue promptly if an explanation is still streaming by keeping state safe and showing a queue/busy indicator. Whether to overlap opponent thinking and explanation is an implementation decision; never show an opponent move before an uncommitted learner candidate is resolved.

## Acceptance examples

- 1.e4 in an opening position is a reasonable development/central move and should normally be accepted.
- A solid alternative to the top engine opening line should normally be accepted.
- A legal move that simply hangs a queen should be interrupted.
- In the strict phase, a near-identical engine alternative is accepted even if not first in MultiPV.
- A clearly lost tactical opportunity triggers a rejection; repeated attempts are legal and the hint/reveal state behaves deterministically.
- Mate scores and draw rules take precedence over naive cp subtraction.
