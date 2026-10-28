# Sign in with ChatGPT, model access and AI security

## Key assumption to validate FIRST

OpenAI currently documents open-source and locally hosted **Sign in with ChatGPT / ChatGPT plan usage** with OAuth dynamic client registration and a **127.0.0.1 HTTP loopback** redirect. The official example is not an Android-specific implementation. Validate it on an Android emulator and (where available) a real Android device before depending on it. Do not assert that every ChatGPT plan qualifies: actual permission, model catalog and account eligibility must be discovered using signed-in credentials. See `REFERENCES.md`.

## Preferred integration

- A separate `CoachProvider`/`AiAuthProvider` interface prevents authentication-specific logic from leaking into chess code.
- Use the official dynamic registration, required `ext_agent_host_id`, system-browser authorization, PKCE S256, fresh state and nonce, exact loopback callback `/auth/callback`, token endpoint exchange and ID-token validation. Protect against unsolicited callbacks, token substitution, wrong workspace or missing granted scopes. Do not substitute custom-scheme deep links if unsupported by OpenAI OAuth registration.
- Generate/persist an opaque stable per-host ID as documented; do not use account email/PII. Reuse the issued `client_id` for subsequent sign-ins to the same account/workspace.
- Use KeyStore-backed encryption, safe refresh token rotation and explicit logout/revoke semantics. Do not embed secrets or log credentials. Follow platform backup rules.
- `GET https://api.openai.com/v1/models` with the OAuth token: list models using the current documented response structure (`models` with `slug`, `display_name`, `visibility`). Prefer `gpt-6.1-sol` if actually available to this account. `reasoning.effort: low` by default; allow fallback to another listed model if the user explicitly chooses it.
- `POST https://api.openai.com/v1/responses` with OAuth access token. For this preview flow, explicitly send `store: false`, `stream: true`, and `input` array with every required conversation turn. Handle terminal `response.completed`; treat `response.failed`, `response.incomplete` and interrupted streams as failures.
- The OAuth token is **not** permission to access existing ChatGPT conversation history or ChatGPT memory.

## IMPORTANT: restricted SIWC Responses parameters (as of 2026-10-28)

Do not use `previous_response_id` over HTTP; store only necessary history locally and include it in `input`. Use `instructions` or developer messages, not explicit system-role `type:message` items. The SIWC preview documentation forbids many normal API arguments (including `temperature`, `max_output_tokens`, `metadata` and `conversation`). Supported client-side function/custom tools must follow current restrictions; hosted tool search, file search, code interpreter etc. are not available in this flow. **Consult official limitations on each implementation pass.**

## Fallback and failures

- Missing OAuth, declined consent, no eligible plan, unavailable model, quota exceeded, token expiry or no network: board and local Stockfish still function. Display a calm, accurate error: “AI coaching is unavailable. You can keep playing and see the engine's line.”
- No automatic transition to paid billing. A **separately billed OpenAI API provider** is optional *future* work, and should be implemented only if the user expressly opts in. If added, use secure server-side storage of provider credentials for general distribution; no provider-owned key in the APK. A user-supplied key on a personal device is a separate security decision and not the MVP default.
- Do not send private local game database or unrelated user content to the model; send minimal FEN, moves, relevant analysis context and explicit questions only.
- Keep network timeouts, cancellation, backoff and bounded context size. Never spam retries against plan limits.

## Quality and trust

- Stockfish determines move quality. The LLM interprets data; it may not overrule move acceptance.
- AI explanations should be generated only after receiving validated engine data. Test spoiler policies and checking legality of quoted variations.
- If the AI response cannot be verified or violates the spoiler policy, fall back to a safe engine-backed message; never let an unsafe explanation block the game.

## Authentication test to close milestone 0

1. App starts system browser and registers/authenticates per official flow.
2. Correct account/host ID saved securely; granted permission and ID-token identity checked.
3. Models endpoint lists selected-account models; only show available choices.
4. A `gpt-6.1-sol` low-reasoning response, **if available**, streams successfully and ends with `response.completed`.
5. Logout, token refresh, denied consent, network failure and account unavailability are exercised.
6. If Android OAuth is unsupported or cannot be established, document raw evidence and continue with an offline/mock provider in a way that still permits a real build; do not assert OAuth is working.
