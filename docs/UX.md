# UI and experience guide

PlyWhy should feel like a focused learning tool, not a crowded chess database.

## Tone and presentation

- Brand: **PlyWhy** — “Understand every move.”
- Clear modern native Android visual style using Material 3, calm neutral surfaces, good contrast and restrained highlights. Adapt to dark/light system theme; don't spend the first milestone on logos or elaborate animations.
- Keep the board and the primary task visible. Favor conversational short feedback (~2–4 sentences) and expandable details rather than lengthy lectures.
- User should never be punished for retrying or seeing hints. Avoid nagging grades, engagement metrics and streaks in v1.

## Screens

1. **Home:** Continue game, New game, Settings. Show useful last-game summary only when available.
2. **New game:** choose side, opponent difficulty, coaching strictness, opening duration. Provide sensible defaults and a single Start action.
3. **Board:** board orientation selected side down; rank/file labels, legal move highlights, last move, turn; actions New Game, Undo, Settings. Coach panel below/alongside board with context-sensitive text and buttons.
4. **Coach intervention:** clear attempted move preview; “Try another move”, “Hint”, “Ask why”, “Reveal”; no solution move displayed before reveal.
5. **Accepted feedback:** concise reason, optional “Why?” / “Compare alternatives” and Continue indicator. Clarify if the move is another excellent choice rather than engine #1.
6. **Coach chat sheet:** focused on the current position; show replies progressively (if model streams), distinguish verified engine facts from interpretive guidance, preserve current-game question context, allow close without losing game.
7. **Settings / Sign-in:** selected model, account/sign-in status, latency/usage note, “Offline mode”, logout, clear errors.
8. **Game end:** result, review moves, New Game; saving is automatic.

## Interaction and accessibility

- Large enough touch targets; all functions usable by tapping without dragging.
- Promotion modal with four options, proper king-in-check display and legal move constraints.
- Content descriptions (piece, square, selected, legal target), TalkBack support, scalable typography, landscape/tablet layout as a graceful enhancement.
- Distinct visual states: selecting piece, evaluating, accepted, intervention, thinking opponent, coach unavailable, finished.
- Clear handling of slow analysis: cancellable spinner; don't freeze the board. If a position is no longer current, discard its late engine/AI response.
