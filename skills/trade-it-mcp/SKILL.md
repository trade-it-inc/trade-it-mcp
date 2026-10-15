---
name: trade-it-mcp
description: Use Trade It's connected MCP tools to inspect your brokerage accounts, holdings, and orders, look up assets, create trade links, and review or explicitly approve trades. Use for personal brokerage workflows, not building SDK or API integrations.
---

# Trade It

Work with the user's own brokerage accounts through the existing Trade It MCP connection. For development of SDK, API, OAuth, or webhook integrations, use [Trade It Developer](https://github.com/trade-it-inc/skills) instead. The two packages work independently.

## Connect and discover

The remote MCP endpoint is `https://mcp.tradeit.app/mcp`. Native plugin installation supplies the connection configuration; standalone skill installation supplies instructions only. Follow the client's connection flow and [installation guide](https://docs.tradeit.app/installation) when setup is needed.

Use Streamable HTTP; the legacy HTTP+SSE transport is unsupported. Read [tool workflow notes](references/tools.md) when choosing contracts, preparing orders, paging results, or interpreting execution outcomes. Discover actual schemas before making calls.

- Authenticate through OAuth after an MCP/API `401`, following [MCP authentication](https://docs.tradeit.app/mcp.html#authentication).
- If the client supports bearer headers, a [user-created API key](https://tradeit.app/account/api-keys) is another supported path; follow [API authentication](https://docs.tradeit.app/api.html#authentication). Store it in the client's credential settings, not in source files or prompts.
- Discover the connected server's actual tools and input schemas. Availability depends on the authenticated client and permissions. Do not assume trading tools exist or work around missing permissions through another API.
- Use [supported brokerages](https://docs.tradeit.app/brokerages) and current server responses for availability. Do not maintain a static brokerage list or promise capabilities based on another brokerage.

## Account and market questions

| Task | Tools, when available | Handling |
| --- | --- | --- |
| Accounts and balances | `get_accounts` | Use returned account IDs and distinguish cash, buying power, and total value. |
| Positions | `get_holdings` | Select the intended account and use the returned quantities and costs. |
| Order history or status | `get_trades` | Follow returned pagination cursors when more results are needed. |
| Asset lookup | `search_assets` | Resolve ambiguous symbols before using them in an order. |
| Options contracts | `get_option_chain` | Use actual contracts; delta requests need absolute decimal bounds and `greeks: true`. |

Use returned timestamps to explain data freshness; cached balances and quotes are not necessarily live. Request fresh data only when needed and supported, and respect returned cooldowns and plan restrictions. If an account is disconnected, use the provided connection action or direct the user to Trade It; do not ask for brokerage credentials in chat.

## Trade review and approval

Prefer a reviewable draft. Before creating an order, resolve material ambiguity in the account, asset, side, size/unit, order type, prices, and option legs. Use the tool's supported defaults only when they fit the user's request; ask about missing choices together.

1. Use `create_trade` for stock/crypto or `create_options_trade` for options with concrete contracts and quantities. Do not substitute one tool's schema for another.
2. Creation can submit immediately under account auto-execution settings. If creation could submit, obtain approval for the exact order before that call. Inspect the returned status and show the actual account, order details, and estimated cost or credit. Do not describe creation as universally harmless or change the user's execution settings.
3. For a returned draft, wait for the user to explicitly approve that exact reviewed order, then call `execute_trade` with its returned trade ID. An instruction to prepare or review an order is not approval to execute it. If material details change, obtain approval for the changed order.
4. Report the returned status accurately. Submitted or accepted does not mean filled. If a submitting call times out or returns an uncertain result, inspect order history before attempting another submission.

Show an Execute control only if the current client actually provides one; otherwise explain how the user can confirm in chat. For options, distinguish paying a debit from receiving a credit, preserve each leg's open/close effect, and use current contract expirations rather than copied examples.

If trading tools are unavailable, offer supported read operations or a `create_trade_link` when available. A Trade Link opens Trade It for the recipient's own review and execution; creating it does not place an order.

## Cancellation

Use `cancel_trade` only for the order the user requests to cancel. A cancellation request can race with a fill. Report the returned order state, and check again when the outcome remains uncertain; do not promise cancellation from an HTTP success alone.

## Keep results grounded

Use tool results for balances, prices, positions, and order status. Explain missing information instead of inventing portfolio data or guarantees. Do not execute a trade to test the connection. Keep brokerage credentials, access tokens, and API keys out of shared outputs.
