# External references (verified 2026-10-28; recheck before implementation)

- Codex Goals overview, `/goal` command and >=0.128.0 CLI: https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex
- Long-running development and `AGENTS.md` / plans: https://developers.openai.com/blog/run-long-horizon-tasks-with-codex
- Plan and Goal distinction: https://developers.openai.com/blog/mastering-codex-remote-for-engineering
- Sign in with ChatGPT open-source flow: https://developers.openai.com/siwc/token-sharing-open-source
- SIWC OAuth details (loopback callback, PKCE, scopes): https://developers.openai.com/siwc/token-sharing-open-source/sign-in
- SIWC account-scoped model list and Responses streaming: https://developers.openai.com/siwc/token-sharing-open-source/models-and-inference
- SIWC preview limitations (MUST READ; numerous ordinary Responses fields unsupported): https://developers.openai.com/siwc/token-sharing-open-source/preview-limitations
- GPT-6.1 Sol supports `reasoning.effort: low`: https://developers.openai.com/api/docs/models/gpt-6.1-sol
- Stockfish developer documentation, UCI, lack of move commentary, GPLv3: https://official-stockfish.github.io/docs/stockfish-wiki/Developers.html
- Stockfish homepage / source: https://stockfishchess.org/ and https://github.com/official-stockfish/Stockfish

Unverified assumptions **requiring milestones**: Android compatibility of SIWC's mandated loopback browser callback; eligibility of the specific signed-in ChatGPT plan/workspace; ability to select GPT-6.1 Sol through that account; safe Stockfish engine packaging and startup on the targeted Android versions. Codex must use evidence to resolve these rather than treating the design document as proof.
