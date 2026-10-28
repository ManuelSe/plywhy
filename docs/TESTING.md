# Testing and verification contract

## Test pyramid

1. **Pure unit tests (required in CI/local build)**: chess state, legal moves, SAN/UCI, castling, en passant, promotion, mate, stalemate, draw types, opening phase boundaries, mate-vs-cp scoring, tolerance policy, hint reveal rules, rejected candidates do not commit, accepted candidates commit once, undo restores FEN and repetition state, stale response cancellation, save/reload.
2. **Engine integration tests**: actual Stockfish engine starts on Android; MultiPV and candidate-specific analysis are compared at fixed node budget; engine-perspective normalization; alternative acceptance and major blunder rejection; opponent strength independent of analysis strength.
3. **AI contract tests**: deterministic fake coach providers validate prompt context, request shape, streaming completion/failure, session auth errors and guardrails, no hidden move leaks at spoiler levels 0–3. Validate verified FEN/variation legality and reject ungrounded content.
4. **Android instrumentation / smoke tests**: full new-game workflow, promote a pawn, wrong candidate→retry, hints→reveal, accepted explanation, game end, resume app, offline mode, orientation/rotation, denied sign-in and available-account sign-in when credentials/devices exist.
5. **Human AI quality review**: 20+ realistic engine-evaluated positions sampled across openings, tactics, endgames and alternatives; review clarity, accuracy, no-spoiler compliance. This is required to call explanation quality validated, but do not fake API results.

## Reference positions (valid intended fixtures; verify via chosen chess library before promoting into tests)

- **Initial chess position:** `rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1`; reasonable `e2e4` should be accepted, at least in opening mode.
- **After 1.e4 e5 2.Nf3 Nc6 3.Bc4 Bc5:** `r1bqk1nr/pppp1ppp/2n5/2b1p3/2B1P3/5N2/PPPP1PPP/RNBQK2R w KQkq - 4 4`; verify legal moves and correct rights, avoid asserting a unique opening solution.
- **Fool's mate tactical chance after 1.f3 e5 2.g4:** `rnbqkbnr/pppp1ppp/8/4p3/6P1/5P2/PPPPP2P/RNBQKBNR b KQkq g3 0 2`; Stockfish should identify a forced mate (`d8h4` is checkmate), and no positional tolerance should treat it as a minor preference.

### Generated golden position set

At milestone 2, Codex should create `testdata/positions.json` containing at least 24 positions from **verified legal** move sequences (generate them with the same rules library), spanning:

- 4 reasonable opening alternatives and 3 opening blunders;
- 4 tactical positions including material loss and simple checkmates;
- 4 near-equal move choices;
- 3 meaningful positional/strategic advantages;
- 3 endgame cases (promotion, stalemate/draw, mating threats);
- 3 positions after special moves (castling/en passant/promotion) and at least one black-to-move case.

These are target categories, not pre-certified judgments. For each entry store the verified FEN, legal candidate moves, an engine-evidence snapshot/version/nodes, expected assessment **category** and a short human teaching note. For near-equal alternatives use engine observations to select candidates; do not manufacture equivalence. Allow engine-version-dependent score margins but preserve stable mate and rule outcomes. Set deterministic node budgets in CI; flag unstable positions for review rather than asserting invalid precise cp expectations.

## Behavioral acceptance checks

- `AC-01`: legal candidate submitted in first 8 full moves with small advantage loss → accepted, no routine interruption.
- `AC-02`: clear tactical blunder in opening → provisional intervention, committed FEN unchanged.
- `AC-03`: strict phase move ranked second but effectively equal → accepted.
- `AC-04`: strict phase move is substantially worse → rejected and retryable indefinitely.
- `AC-05`: first three hints do not explicitly state recommended UCI/SAN or reveal best-move destination; reveal requires explicit action.
- `AC-06`: accepted move explanation refers to the played move, and does not incorrectly call an equally good alternative the uniquely best move.
- `AC-07`: failing AI stream or missing auth does not corrupt/lock the board.
- `AC-08`: local saved game resumes with correct current player, castling/en-passant and counters.
- `AC-09`: opponent plays legally at chosen difficulty while evaluator remains strong.
- `AC-10`: auth integration only passes on successful device and server tests, otherwise marked blocked/unverified.

## Required build evidence

Codex should report commands it actually ran and outcomes (Gradle assembleDebug, unit tests, lint, available instrumented tests); APK exact path and SHA-256; real device/emulator model if tested; Stockfish binary/version and source provenance; account/model used for successful OAuth inference **without revealing credentials or identifying information**.

At final review classify each criterion **PASS**, **FAIL**, or **NOT VERIFIED** with a one-line rationale. Stop and repair failed mandatory offline/engine checks; report externally blocked network/account checks honestly.
