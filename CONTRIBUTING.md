# Contributing

This repository contains the user-facing Trade It plugin, MCP connection metadata, and workflow guidance. SDK/API integration guidance belongs in [`trade-it-inc/skills`](https://github.com/trade-it-inc/skills). Hosted server changes belong in [`trade-it-inc/trade-it`](https://github.com/trade-it-inc/trade-it/tree/main/workers/mcp).

## Sources and scope

Use current [ToolDefinitions](https://github.com/trade-it-inc/trade-it/blob/main/lib/types/src/types/tool.types.ts), the MCP worker's registration and client policies, and first-party docs when changing guidance. The authenticated tool list is authoritative for a particular client. Source changes must be deployed before documentation promises their availability on the hosted endpoint.

Keep the skill entry point short and route tool-specific details to `references/tools.md`. Do not duplicate full parameter schemas, fixed brokerage-ID lists, prices, or plan matrices. Read supported brokerages from the current first-party docs/server metadata; the application derives them from its shared brokerage constants.

The README's brokerage table displays the active list with logos. Refresh it from the application's generated brokerage guide, which derives from `Brokerages` in `lib/types/src/constants/brokerage.ts`; keep inactive brokerages out and include active paper-trading connections. Keep the skill linked to the current guide rather than copying this table into its instructions.

Keep OAuth and user-created API keys documented. Preserve explicit approval for the exact order before a submitting call, account ownership boundaries, and the distinction between a submitted order and a fill. Do not place a trade or rotate credentials to check documentation.

## Plugin packaging

- Root `plugin.json` is the portable manifest; portable clients discover `skills/` and `mcp.json`.
- `.codex-plugin/plugin.json`, `.claude-plugin/plugin.json`, and `.cursor-plugin/plugin.json` provide native compatibility and presentation. Keep all four plugin names and versions synchronized.
- Keep `.mcp.json` and `mcp.json` aligned on server name and URL. The native transport is `http`; the portable transport is `streamable-http`.
- Keep the skill name `trade-it-mcp`, logo paths, `agents/openai.yaml`, and assistant-specific invocation examples consistent.
- Codex and Claude Code install from the shared `trade-it` marketplace in the skills repository. The `trade-it` entry points here; `trade-it-developer` points to the developer package.

Plugin releases are GitHub source releases. They do not deploy the server or update public plugin directory listings automatically. Follow the shared [release instructions](https://github.com/trade-it-inc/skills/blob/main/RELEASING.md).

## MCP Registry publishing

`server.json` describes the existing `app.tradeit/mcp` registry identity. Keep that name stable. Its metadata version is independent of the plugin version and protocol revision. Published registry versions are immutable; choose a new version before changing and publishing metadata. See [registry versioning](https://modelcontextprotocol.io/registry/versioning).

The next prepared metadata release is `1.0.1`, following the existing `1.0.0` listing. It updates the schema and capabilities and removes the unsupported legacy SSE transport. Only Streamable HTTP at `https://mcp.tradeit.app/mcp` is advertised.

The **Publish to MCP Registry** workflow runs when `server.json` changes on `main`, or when manually dispatched. It downloads a pinned `mcp-publisher` binary, checks its SHA-256 digest, validates metadata, authenticates, and publishes. Updating only the skill or README does not trigger registry publication.

Existing GitHub configuration is retained:

| Setting | Purpose |
| --- | --- |
| Secret `MCP_PUBLISH_KEY` | Base64-encoded PEM private key matching the hosted registry ownership proof. |
| Variable `MCP_SERVER_DOMAIN` | Verified domain for HTTP namespace authentication, `tradeit.app`. |

The `app.tradeit` namespace uses domain ownership authentication. Do not switch it to GitHub OIDC without changing registry identity and its consumers. Keys and ownership proofs do not need rotation for this metadata refresh. The workflow keeps decoded key material in a private temporary file and removes it after login. Never commit keys, tokens, or publisher authentication files.

Before publishing, review the metadata against the [remote server requirements](https://modelcontextprotocol.io/registry/remote-servers), confirm the referenced hosted capabilities are deployed, and select an unpublished version. After publishing, inspect the [registry listing](https://registry.modelcontextprotocol.io/?q=app.tradeit%2Fmcp). To update the publisher, change the pinned version and checksum together using the official release assets.

## Review and reporting

Review changed schemas, manifest paths, links, examples, and versions. Run proportionate validation when requested, and report which checks were actually performed. Do not turn a documentation edit into a live financial operation or publication.

Open an issue for incorrect guidance using sanitized reproduction details. Follow [SECURITY.md](SECURITY.md) for vulnerabilities or exposed credentials.
