# Trade It MCP Server

## [👉 Full Documentation Here 👈](https://docs.tradeit.app)
Now available through the [_Official MCP Registry_](https://registry.modelcontextprotocol.io/?q=app.tradeit%2Fmcp)

## Table of contents

- [Overview](#overview)
- [Getting started](#getting-started)
- [Connecting](#connecting)
- [Tools](#tools)
  - [Safety model (draft-first)](#safety-model-draft-first)
  - [`search_assets`](#search_assets)
  - [`get_accounts`](#get_accounts)
  - [`create_trade`](#create_trade)
  - [`create_options_trade`](#create_options_trade)
    - [OCC option symbol format](#occ-option-symbol-format)
    - [Options JSON examples](#options-json-examples)
  - [`execute_trade`](#execute_trade)
- [Trade status reference](#trade-status-reference)
- [Brokerage IDs (API helpers)](#brokerage-ids-api-helpers)
- [Disclaimers](#disclaimers)

## Overview

The Trade It MCP Server brings stock, crypto, and options trading support to agents. It enables natural-language interaction with stock and crypto brokerages—execute trades, query portfolio performance, and surface market insights by sending plain-English requests through the MCP protocol.

**Endpoints:**  
- Streamable HTTP: `https://mcp.tradeit.app/mcp` 
- SSE: `https://mcp.tradeit.app/sse`

**Brokerage Support:**
- <img height="14" width="14" src="https://images.tradeit.app/brokerages/robinhood-logo.svg" alt="Robinhood Logo" /> **[Robinhood](https://robinhood.com)**
- <img height="14" width="14" src="https://images.tradeit.app/brokerages/charles_schwab-logo.svg" alt="Charles Scwhab Logo" /> **[Charles Schwab](https://schwab.com)**
- <img height="14" width="14" src="https://images.tradeit.app/brokerages/etrade-logo.svg" alt="ETrade Logo" /> **[E*Trade](https://etrade.com)**
- <img height="14" width="14" src="https://images.tradeit.app/brokerages/webull-logo.svg" alt="Webull Logo" /> **[Webull](https://webull.com)**
- <img height="14" width="14" src="https://images.tradeit.app/brokerages/public-logo.svg" alt="Public Logo" /> **[Public](https://public.com)**
- <img height="14" width="14" src="https://images.tradeit.app/brokerages/tastytrade-logo.svg" alt="Tastytrade Logo" /> **[Tastytrade](https://tastytrade.com)**

**Crypto Exchange Support:**
- <img height="14" width="14" src="https://images.tradeit.app/brokerages/coinbase-logo.svg" alt="Coinbase Logo" /> **[Coinbase](https://coinbase.com)**
- <img height="14" width="14" src="https://images.tradeit.app/brokerages/kraken-logo.svg" alt="Kraken Logo" /> **[Kraken](https://kraken.com)**

More to be added soon!

This server is **remote** so you don't need to run anything locally to connect. Just point your MCP-compatible agent platform to the URL above.

---

## Getting Started

1. First, create an account at https://tradeit.app.
2. Sign up for the Pro plan's free trial.
3. Connect your brokerage of choice.

## Connecting
1. Connect your MCP client to `https://mcp.tradeit.app/mcp` or `https://mcp.tradeit.app/sse`.
2. Authenticate through the browser-based OAuth flow.
3. You're now ready to start trading!

---

## Tools

MCP tools connect your agent to linked brokerages: search symbols, list accounts, create **draft** orders, then execute only after confirmation.

| MCP tool | What it does |
| --- | --- |
| `search_assets` | Look up a stock or crypto by ticker or name; returns price and metadata. |
| `get_accounts` | List linked accounts and balances; also used when linking a new brokerage. |
| `create_trade` | Create a **draft** equity/crypto buy or sell for review. |
| `create_options_trade` | Create a **draft** single- or multi-leg options order for review. |
| `execute_trade` | Submit a previously created draft to the broker **after** explicit user confirmation. |

### Safety model (draft-first)

Trades start as `draft` orders and are **not** sent to the broker until the user clearly confirms.

**Intended flow:**

1. Call `create_trade` or `create_options_trade` → you get a draft with a `trade_id`.
2. Show the user the full order details and how to proceed.
3. Call `execute_trade` **only** when the user explicitly asks to execute, confirm, or place the trade.
4. Do **not** call `execute_trade` automatically or immediately after creating a draft.

After creating a draft, make sure the user knows they can place the order when ready (e.g. via your client’s Execute control, if available).

**Optional steps before creating a draft:**

- `search_assets` — confirm ticker and context.
- `get_accounts` — pick the right `account_id` when the user cares which account to use.

**Execution flow:**

```
User requests trade
       ↓
[Optional] search_assets — confirm ticker, get current price
       ↓
[Optional] get_accounts — identify correct account_id
       ↓
create_trade / create_options_trade → draft with trade_id, status: "draft"
       ↓
Show draft details; user confirms
       ↓
execute_trade(trade_id)
       ↓
Status: "placed" or "failed" (with details)
```

**Account / order defaults:** If the user omits amount, account, or order type, Trade It applies their default amount, default account, and **market** orders where applicable. If **auto-execute** is enabled in Trade It settings, behavior may skip the manual execute step in some setups; when in doubt, still treat execution as user-confirmed.

---

### `search_assets`

Look up a stock or crypto by ticker or name.

- **Parameter:** `query` (string) — e.g. `"TSLA"`, `"Tesla"`, `"bitcoin"`.
- **Returns:** Price, ticker, exchange, asset type, and related metadata.

**Example:**

```json
{ "query": "TSLA" }
```

**Natural-language examples:** *"How's Apple doing?"* · *"What's the price of TSLA?"*

---

### `get_accounts`

List all linked brokerage accounts (and use this flow when the user wants to connect a new brokerage).

- **Parameters:** none.
- **Returns:** Accounts with `id`, `name`, `brokerage`, `balance`, `available_cash`. Use `account.id` as `account_id` in trade calls when a specific account is required.

**Natural-language example:** *"Show my accounts."*

---

### `create_trade`

Create a **draft** equity or crypto order.

**Parameters:**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `symbol` | string | Yes | Ticker, e.g. `"TSLA"`. |
| `amount` | number | Yes | Size to trade. |
| `unit` | `"dollars"` or `"shares"` | Yes | Unit for `amount`. |
| `buy_or_sell` | `"buy"` or `"sell"` | Yes | Direction. |
| `order_type` | `"market"`, `"limit"`, `"stop"`, `"stop_limit"` | No | Defaults to `"market"`. |
| `limit_price` | number | If limit / stop_limit | Max or min price per share as applicable. |
| `stop_price` | number | If stop / stop_limit | Stop trigger price. |
| `time_in_force` | `"day"`, `"gtc"`, `"ioc"`, `"fok"` | No | Omit for brokerage default. |
| `account_id` | number | No | Omit for default account. |

**Order types:**

| Type | Use when | Price fields |
| --- | --- | --- |
| `market` | Fill at current market | None |
| `limit` | Only at `limit_price` or better | `limit_price` |
| `stop` | Market order triggers at `stop_price` | `stop_price` |
| `stop_limit` | Limit order triggers at `stop_price` | `stop_price` and `limit_price` |

**JSON examples:**

Buy $500 of Apple at market:

```json
{ "symbol": "AAPL", "amount": 500, "unit": "dollars", "buy_or_sell": "buy" }
```

Buy 10 shares of NVDA only if it drops to $800 or below:

```json
{ "symbol": "NVDA", "amount": 10, "unit": "shares", "buy_or_sell": "buy", "order_type": "limit", "limit_price": 800 }
```

Sell 5 shares of Meta if the price falls to $450 (stop):

```json
{ "symbol": "META", "amount": 5, "unit": "shares", "buy_or_sell": "sell", "order_type": "stop", "stop_price": 450 }
```

Buy 10 AAPL if it breaks above $200, paying at most $202/share:

```json
{ "symbol": "AAPL", "amount": 10, "unit": "shares", "buy_or_sell": "buy", "order_type": "stop_limit", "stop_price": 200, "limit_price": 202 }
```

Buy $1,000 of Bitcoin:

```json
{ "symbol": "BTC", "amount": 1000, "unit": "dollars", "buy_or_sell": "buy" }
```

Sell 100 shares of Tesla, good till canceled:

```json
{ "symbol": "TSLA", "amount": 100, "unit": "shares", "buy_or_sell": "sell", "time_in_force": "gtc" }
```

**Natural-language examples:** *"Buy $1000 of Tesla"* · *"Buy $1000 of Tesla only if the price drops to $150 or lower"* · *"Sell 10 shares of Apple if the price falls to $140"* · *"Buy a share of Apple if it hits $200"* · *"Buy 10 shares of Apple if it rises to $140, but don't pay more than $142"*

---

### `create_options_trade`

Create a **draft** single-leg or multi-leg options order (spreads, straddles, etc.).

**Parameters:**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `symbol` | string | Yes | Underlying ticker, e.g. `"SPY"`. |
| `legs` | array | Yes | One or more legs (see below). |
| `direction` | `"debit"` or `"credit"` | Multi-leg | `"debit"` = you pay; `"credit"` = you collect. |
| `order_type` | `"market"`, `"limit"`, etc. | No | Defaults to `"market"`. |
| `limit_price` | number | For limit | Net debit/credit limit for the package. |
| `time_in_force` | `"day"` or `"gtc"` | No | Omit for default. |
| `account_id` | number | No | Omit for default account. |

**Each leg:**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `type` | `"option"` or `"equity"` | Yes | Leg type. |
| `action` | `"buy"` or `"sell"` | Yes | Side of the leg. |
| `position_effect` | `"open"` or `"close"` | Options | Open a new position or close an existing one. |
| `occ` | string or `null` | Options | OCC string (below); `null` for equity legs. |
| `quantity` | number | Yes | Contracts (options) or shares (equity). |

#### OCC option symbol format

OCC strings follow: `YYMMDD` + `C` or `P` + 8-digit strike (strike × 1000, zero-padded).

| Description | OCC |
| --- | --- |
| Jun 20, 2025 $250 call | `250620C00250000` |
| Jun 20, 2025 $260 call | `250620C00260000` |
| Mar 21, 2025 $500 put | `250321P00500000` |
| Dec 19, 2025 $1,500 call | `251219C01500000` |
| Jan 16, 2026 $50 put | `260116P00050000` |

Strike encoding: multiply dollars by 1,000 and pad to 8 digits (e.g. $250 → `00250000`; $50.50 → `00050500`).

#### Options JSON examples

**Single call** — buy 1 SPY $520 call exp Jun 20, 2025:

```json
{
  "symbol": "SPY",
  "legs": [
    { "type": "option", "action": "buy", "position_effect": "open", "occ": "250620C00520000", "quantity": 1 }
  ]
}
```

**Bull call spread (debit)** — buy $250 call, sell $260 call, same expiry:

```json
{
  "symbol": "TSLA",
  "direction": "debit",
  "legs": [
    { "type": "option", "action": "buy", "position_effect": "open", "occ": "250620C00250000", "quantity": 1 },
    { "type": "option", "action": "sell", "position_effect": "open", "occ": "250620C00260000", "quantity": 1 }
  ]
}
```

**Bear put spread (debit):**

```json
{
  "symbol": "SPY",
  "direction": "debit",
  "legs": [
    { "type": "option", "action": "buy", "position_effect": "open", "occ": "250620P00520000", "quantity": 1 },
    { "type": "option", "action": "sell", "position_effect": "open", "occ": "250620P00510000", "quantity": 1 }
  ]
}
```

**Bull put spread (credit):**

```json
{
  "symbol": "SPY",
  "direction": "credit",
  "legs": [
    { "type": "option", "action": "sell", "position_effect": "open", "occ": "250620P00510000", "quantity": 1 },
    { "type": "option", "action": "buy", "position_effect": "open", "occ": "250620P00500000", "quantity": 1 }
  ]
}
```

**Spread with limit** — net debit $3.50 or better:

```json
{
  "symbol": "TSLA",
  "direction": "debit",
  "order_type": "limit",
  "limit_price": 3.50,
  "legs": [
    { "type": "option", "action": "buy", "position_effect": "open", "occ": "250620C00250000", "quantity": 1 },
    { "type": "option", "action": "sell", "position_effect": "open", "occ": "250620C00260000", "quantity": 1 }
  ]
}
```

**Close a long call** — sell to close 2 AAPL $200 calls exp Mar 21, 2025:

```json
{
  "symbol": "AAPL",
  "legs": [
    { "type": "option", "action": "sell", "position_effect": "close", "occ": "250321C00200000", "quantity": 2 }
  ]
}
```

**Straddle** — long $250 call and $250 put, same expiry:

```json
{
  "symbol": "TSLA",
  "direction": "debit",
  "legs": [
    { "type": "option", "action": "buy", "position_effect": "open", "occ": "250620C00250000", "quantity": 1 },
    { "type": "option", "action": "buy", "position_effect": "open", "occ": "250620P00250000", "quantity": 1 }
  ]
}
```

**Natural-language examples:** *"Buy 1 AAPL $300 call expiring next month"* · *"Covered call on MSFT at $500 strike"* · *"TSLA call spread: buy $475 / sell $485, next week"* · *"ATM straddle on SPY this Friday"* · *"2 AMZN puts, limit $3.50"* · *"Sell AMZN260130P00200000"*

---

### `execute_trade`

Send a **draft** to the brokerage after the user has reviewed it.

- **Parameter:** `trade_id` (number) — the draft’s `id` from `create_trade` or `create_options_trade`.
- **Returns:** Updated trade; status `"placed"` or `"failed"` (with error details).

**Call only when** the user clearly confirms (e.g. execute, confirm, place it, go ahead). Confirm the trade that matches what they just reviewed.

**Do not** call automatically right after creating a draft, without showing order details, or when status is not `"draft"`.

---

### Trade status reference

| Status | Meaning |
| --- | --- |
| `draft` | Created; not yet sent to broker |
| `pending` | Submitted; awaiting broker ack |
| `placed` | Accepted; awaiting fill |
| `partially_filled` | Partially filled |
| `complete` | Fully filled |
| `canceled` | Canceled |
| `failed` | Rejected — check errors |
| `disconnected` | Brokerage connection issue |

### Brokerage IDs (API helpers)

| Brokerage | ID | Options |
| --- | ---: | --- |
| Robinhood | 1 | Yes |
| E\*TRADE | 2 | Yes |
| Coinbase | 3 | Crypto only |
| Kraken | 5 | Crypto only |
| Charles Schwab | 7 | Yes |
| Webull | 8 | Yes |
| Public | 11 | Yes |
| Tastytrade | 12 | Yes |

**Clarification:** Ask once, with everything you need, when: order type is ambiguous (e.g. “buy TSLA at $200” — limit vs stop), options are missing expiry/strike, multiple accounts apply and none is chosen, or a symbol could mean more than one asset. Skip redundant questions when defaults are clear (default amount, market order, primary account).

### Disclaimers

- Investing involves risk, including possible loss of principal.
- Trade It is not a financial advisor and does not provide investment advice.
- Options involve substantial risk and are not appropriate for all investors.
- Trade It cannot withdraw funds, transfer assets, or take custody — it can only place trades through your linked brokerages.

---

