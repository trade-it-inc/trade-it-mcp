# Brokerage tool workflows

Read the connected tool schemas before making calls. The [public tool documentation](https://docs.tradeit.app/tools/) explains inputs; current [ToolDefinitions](https://github.com/trade-it-inc/trade-it/blob/main/lib/types/src/types/tool.types.ts) define the server's tools. Client policies can expose a subset. Examples here are inputs, not requests to run them or guarantees of particular results.

## Accounts, holdings, and orders

Use `get_accounts` to identify the intended account. Returned account IDs are the values accepted by `account_id`; do not invent one from a brokerage name. Account value, cash, and buying power are different figures.

Use `get_holdings` with that account's ID. `get_accounts`, `get_holdings`, and `get_trades` accept optional `refresh`. Omit it for normal reads; use it when fresh brokerage data is needed and the server permits it. Respect returned eligibility and cooldown errors rather than retrying continuously.

`get_trades` accepts `limit` (up to 100) and `cursor`. For example:

```json
{
  "limit": 20
}
```

Use the returned next cursor for another page when more results are needed. Do not claim to have reviewed all history after reading only the first page. The tool does not expose an `account_id` filter; inspect the returned orders for the relevant account.

MCP results may contain formatted text, structured content, or client-specific UI. Do not assume an SDK/REST response envelope or require an Execute button in clients that do not provide one.

## Assets and option contracts

Use `search_assets` with a ticker or company/coin name, then verify the intended asset from its returned identity and price information. A quote is not a promised execution price.

Use `get_option_chain` with `root_symbol`. Supported filters include `type`, `expiration_date_gte`, `expiration_date_lte`, `strike_price_gte`, `strike_price_lte`, `delta_gte`, and `delta_lte`, plus `greeks` and `limit`.

For a request near 16 delta, filter by absolute decimal delta instead of downloading a broad chain and guessing:

```json
{
  "root_symbol": "SPY",
  "type": "put",
  "greeks": true,
  "delta_gte": 0.15,
  "delta_lte": 0.17,
  "limit": 10
}
```

Select expiration dates from the user's request and current returned contracts. Keep queries bounded to the needed range. The current input schema has no page-token parameter; do not invent one from an output cursor or imply a limited result is the complete chain.

## Stock and crypto orders

`create_trade` requires `symbol`, positive `amount`, `unit` (`dollars` or `shares`), and `buy_or_sell` (`buy` or `sell`). A default account can be used when omitted and appropriate to the request. Resolve material ambiguity before preparing or submitting an order.

Shared optional order fields include `account_id`, `order_type`, `limit_price`, `stop_price`, and `time_in_force`. Stock/crypto orders also support `take_profit` and `stop_loss` when the brokerage permits them. Exit triggers use `value` and `unit` (`$` or `%`); they are not a universal brokerage capability.

| Order type | Price fields |
| --- | --- |
| `market` | No limit or stop price. |
| `limit` | `limit_price`. |
| `stop` | `stop_price`. |
| `stop_limit` | Both `limit_price` and `stop_price`. |

Set the intended order type explicitly when prices matter. Distinguish share quantities from dollar amounts and use defaults supported by the actual tool, not presumed account preferences.

## Options orders

Use `create_options_trade` with the underlying `symbol` and concrete `legs`. Each leg specifies `type`, `action`, `quantity`, and `occ`; option legs also specify their open/close `position_effect`. Equity legs use `occ: null`.

The tool's `occ` value excludes the root symbol: `YYMMDD`, `C` or `P`, then strike × 1000 padded to eight digits. Chain results can include the root; preserve the returned contract's expiration, type, and strike when converting to the tool's expected suffix. Do not reuse an expired example or construct a contract without verifying it exists.

For a multi-leg order, use `direction` consistent with its intended cash flow: `debit` means paying, `credit` means receiving, and `even` means zero net. A bull call debit spread buys the lower-strike call and sells the higher-strike call at the same expiration. Review every leg, quantity, open/close effect, and the net limit before approval. Attached stock/crypto exit fields are not part of this tool's schema.

## Submission and cancellation

Follow the [skill's approval workflow](../SKILL.md#trade-review-and-approval). Creation may submit immediately under account auto-execution settings; approval must precede any potentially submitting call. Do not turn off review settings or treat an account preference as approval for a specific order.

For a returned draft, display the actual order, get approval for that exact order, and call `execute_trade` using its returned ID as `trade_id`. If creation already returned a submitted order, do not execute it again. Recheck recent orders after an uncertain response before attempting another submission.

| Returned status | Meaning |
| --- | --- |
| `draft` | Prepared, not submitted. |
| `pending` | Submission is pending. |
| `placed` | Submitted/accepted; not necessarily filled. |
| `partially_filled` | Only part of the order has filled. |
| `complete` | The order has completed. |
| `canceled` | The order is canceled; inspect any filled quantity. |
| `failed` | Inspect the returned failure details. |
| `disconnected` | Brokerage connectivity needs attention. |

Use `cancel_trade` with the intended `trade_id` only when cancellation is requested. A submitted order may fill before cancellation succeeds. Report the returned state rather than equating a successful call with a canceled order.

## Trade Links

Use `create_trade_link` to return a URL for someone to review, edit, and execute on Trade It using their own account. This tool does not create a brokerage draft or execute an order. Its input is a trade configuration under `config`; it is not the `create_trade` request schema. Follow its actual schema rather than translating fields by guesswork.
