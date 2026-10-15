# Repository guide

This is the user-facing Trade It MCP/plugin distribution repository. Read [CONTRIBUTING.md](CONTRIBUTING.md) for source locations, packaging, and publishing conventions.

- Keep developer SDK/API integration guidance in `trade-it-inc/skills`; keep personal account and trading workflows here.
- Use the current application `ToolDefinitions` and authenticated tool schemas as authority. Keep `SKILL.md` concise and load references only when needed.
- Preserve explicit approval before live trading, user account ownership, both OAuth and API-key authentication, and accurate order status reporting.
- Keep all four plugin manifest versions aligned and the portable/native MCP endpoints equivalent. Do not advertise the old HTTP+SSE transport.
- Registry metadata versions are separate and immutable after publication. Changing `server.json` on `main` triggers publication; do not push, dispatch, or publish without user authorization.
- Keep changes focused and legible. Do not add a build system, dependency, test suite, or executable helper merely to validate documentation. Respect the user's testing instructions.
- Never commit credentials, account data, or publisher authentication artifacts. Use [SECURITY.md](SECURITY.md) for private reports.
