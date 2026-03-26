# Trade It MCP Skill

Trade It connects MCP clients directly to the user's brokerage accounts. Use the tools below to search assets, review account balances, and place real trades through natural language.

---

## Available Tools

| Tool | Description |
|---|---|
| `search_assets` | Look up a stock or crypto by ticker or name. Returns current price and metadata. |
| `get_accounts` | List all linked brokerage accounts with balances. Also used to link new accounts. |
| `create_trade` | Create a **draft** equity/crypto buy or sell order for the user to review. |
| `create_options_trade` | Create a **draft** single-leg or multi-leg options order for the user to review. |
| `execute_trade` | Execute a previously created draft trade. Requires explicit user confirmation. |

---

## Safety Model — Read This First

Trade It uses a **draft-first** execution model. Every trade starts as a `draft` and is never placed with the broker until the user explicitly confirms.

**Required flow:**
1. Call `create_trade` or `create_options_trade` → returns a draft order with a `trade_id`
2. Display the order details and inform the user they can execute it
3. Call `execute_trade` **only** when the user explicitly says to execute/confirm/place the trade
4. **Never** call `execute_trade` automatically or immediately after creating a draft

After creating a draft, always tell the user:
> "When you are ready, you can place the order by pressing the Execute button."

---

## Execution Flow

```
User requests trade
       ↓
[Optional] search_assets — confirm ticker, get current price
       ↓
[Optional] get_accounts — identify correct account_id
       ↓
create_trade / create_options_trade → returns draft with trade_id and status: "draft"
       ↓
Display draft details to user, prompt to confirm
       ↓
User confirms → execute_trade(trade_id)
       ↓
Returns updated trade with status: "placed" or "failed"
```

---

## Tool Reference

### search_assets

Look up a stock or crypto by ticker symbol or company/coin name.

**Parameters:**
- `query` *(string)* — ticker symbol or name (e.g., `"TSLA"`, `"Tesla"`, `"bitcoin"`)

**Returns:** Asset info including current price, ticker, exchange, and asset type.

**Example:**
```json
{ "query": "TSLA" }
```

---

### get_accounts

List all linked brokerage accounts. Also used when the user wants to connect a new brokerage.

**Parameters:** *(none)*

**Returns:** Array of accounts, each with `id`, `name`, `brokerage`, `balance`, and `available_cash`.

Use `account.id` as the `account_id` in trade calls when the user specifies a particular account.

---

### create_trade

Create a draft equity or crypto order.

**Parameters:**

| Field | Type | Required | Description |
|---|---|---|---|
| `symbol` | string | ✅ | Ticker symbol, e.g. `"TSLA"` |
| `amount` | number | ✅ | Quantity to trade |
| `unit` | `"dollars"` \| `"shares"` | ✅ | Whether `amount` is in dollars or shares |
| `buy_or_sell` | `"buy"` \| `"sell"` | ✅ | Trade direction |
| `order_type` | `"market"` \| `"limit"` \| `"stop"` \| `"stop_limit"` | ❌ | Defaults to `"market"` |
| `limit_price` | number | Conditional | Required for `limit` and `stop_limit` orders |
| `stop_price` | number | Conditional | Required for `stop` and `stop_limit` orders |
| `time_in_force` | `"day"` \| `"gtc"` \| `"ioc"` \| `"fok"` | ❌ | Omit to use brokerage default |
| `account_id` | number | ❌ | Omit to use user's default account |

**Order Types:**

| Type | When to Use | Price Fields |
|---|---|---|
| `market` | Execute immediately at current price | None |
| `limit` | Execute only at `limit_price` or better | `limit_price` required |
| `stop` | Trigger market order when `stop_price` is hit | `stop_price` required |
| `stop_limit` | Trigger limit order when `stop_price` is hit | Both `stop_price` and `limit_price` required |

**Examples:**

Buy $500 of Apple at market:
```json
{ "symbol": "AAPL", "amount": 500, "unit": "dollars", "buy_or_sell": "buy" }
```

Buy 10 shares of NVDA only if it drops to $800 or below:
```json
{ "symbol": "NVDA", "amount": 10, "unit": "shares", "buy_or_sell": "buy", "order_type": "limit", "limit_price": 800 }
```

Sell 5 shares of Meta if the price falls to $450 (stop loss):
```json
{ "symbol": "META", "amount": 5, "unit": "shares", "buy_or_sell": "sell", "order_type": "stop", "stop_price": 450 }
```

Buy 10 shares of AAPL if it breaks above $200, but pay no more than $202:
```json
{ "symbol": "AAPL", "amount": 10, "unit": "shares", "buy_or_sell": "buy", "order_type": "stop_limit", "stop_price": 200, "limit_price": 202 }
```

Buy $1,000 of Bitcoin:
```json
{ "symbol": "BTC", "amount": 1000, "unit": "dollars", "buy_or_sell": "buy" }
```

Sell all (100 shares) of Tesla, good till canceled:
```json
{ "symbol": "TSLA", "amount": 100, "unit": "shares", "buy_or_sell": "sell", "time_in_force": "gtc" }
```

---

### create_options_trade

Create a draft single-leg or multi-leg options order (spreads, straddles, etc.).

**Parameters:**

| Field | Type | Required | Description |
|---|---|---|---|
| `symbol` | string | ✅ | Underlying ticker, e.g. `"SPY"`, `"TSLA"` |
| `legs` | array | ✅ | One or more legs (see below) |
| `direction` | `"debit"` \| `"credit"` | Multi-leg only | Required for spreads. `"debit"` = you pay; `"credit"` = you receive |
| `order_type` | `"market"` \| `"limit"` \| `"stop"` \| `"stop_limit"` | ❌ | Defaults to `"market"` |
| `limit_price` | number | Conditional | Net debit/credit limit for the spread. Required for `limit` orders. |
| `time_in_force` | `"day"` \| `"gtc"` | ❌ | Omit to use brokerage default |
| `account_id` | number | ❌ | Omit to use default account |

**Leg Fields:**

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | `"option"` \| `"equity"` | ✅ | Leg type |
| `action` | `"buy"` \| `"sell"` | ✅ | Buy or sell this leg |
| `position_effect` | `"open"` \| `"close"` | Option legs only | Whether this opens a new position or closes an existing one |
| `occ` | string | Option legs only | OCC format string (see below). Set to `null` for equity legs. |
| `quantity` | number | ✅ | Contracts (options) or shares (equity) |

---

### OCC Format

Options are identified using the OCC (Options Clearing Corporation) standard format:

```
YYMMDD[C|P]STRIKE
```

| Part | Description | Example |
|---|---|---|
| `YY` | 2-digit year | `25` for 2025 |
| `MM` | Month, zero-padded | `06` for June |
| `DD` | Day, zero-padded | `20` for the 20th |
| `C` or `P` | Call or Put | `C` |
| `STRIKE` | Strike × 1000, 8 digits, zero-padded | `00250000` for $250 |

**OCC Examples:**

| Description | OCC String |
|---|---|
| Jun 20, 2025 $250 Call | `250620C00250000` |
| Jun 20, 2025 $260 Call | `250620C00260000` |
| Mar 21, 2025 $500 Put | `250321P00500000` |
| Dec 19, 2025 $1,500 Call | `251219C01500000` |
| Jan 16, 2026 $50 Put | `260116P00050000` |

**Strike price formula:** Multiply dollar strike by 1000, then zero-pad to 8 digits.
- $250 → 250 × 1000 = 250000 → `00250000`
- $1,500 → 1500 × 1000 = 1500000 → `01500000`
- $50.50 → 50.5 × 1000 = 50500 → `00050500`

---

### Options Examples

**Single call option** — Buy 1 SPY call at $520 strike, expiring Jun 20, 2025:
```json
{
  "symbol": "SPY",
  "legs": [
    { "type": "option", "action": "buy", "position_effect": "open", "occ": "250620C00520000", "quantity": 1 }
  ]
}
```

**Bull call spread (debit)** — Buy TSLA $250 call, sell TSLA $260 call, both expiring Jun 20, 2025:
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
*Max loss = net debit paid. Max gain = $10 spread width minus net debit.*

**Bear put spread (debit)** — Buy SPY $520 put, sell SPY $510 put, expiring Jun 20, 2025:
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

**Bull put spread (credit)** — Sell SPY $510 put, buy SPY $500 put, expiring Jun 20, 2025:
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

**Bull call spread with limit price** — Same spread, but only if net debit is $3.50 or less:
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

**Close an existing long call** — Sell to close 2 AAPL $200 calls expiring Mar 21, 2025:
```json
{
  "symbol": "AAPL",
  "legs": [
    { "type": "option", "action": "sell", "position_effect": "close", "occ": "250321C00200000", "quantity": 2 }
  ]
}
```

**Straddle** — Buy 1 TSLA $250 call and 1 TSLA $250 put, both expiring Jun 20, 2025:
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

---

### execute_trade

Execute a draft trade after the user has reviewed and confirmed it.

**Parameters:**
- `trade_id` *(number)* — The `id` from the draft trade returned by `create_trade` or `create_options_trade`

**Returns:** Updated trade with status `"placed"` (or `"failed"` with error details).

**Call this only when:**
- The user explicitly says "execute", "confirm", "place it", "go ahead", "do it", or similar
- The trade being confirmed is the most recently reviewed draft

**Never call this:**
- Automatically after creating a draft
- Without the user seeing the order details first
- If the trade status is not `"draft"`

---

## Trade Status Reference

| Status | Meaning |
|---|---|
| `draft` | Created, not yet submitted to broker |
| `pending` | Submitted, awaiting broker acknowledgment |
| `placed` | Accepted by broker, awaiting fill |
| `partially_filled` | Some contracts/shares have filled |
| `complete` | Fully filled |
| `canceled` | Canceled by user or broker |
| `failed` | Rejected — check error details |
| `disconnected` | Brokerage connection issue |

---

## Supported Brokerages

| Brokerage | ID | Supports Options |
|---|---|---|
| Robinhood | 1 | ✅ |
| E*TRADE | 2 | ✅ |
| Coinbase | 3 | Crypto only |
| Kraken | 5 | Crypto only |
| Charles Schwab | 7 | ✅ |
| Webull | 8 | ✅ |
| Public | 11 | ✅ |
| Tastytrade | 12 | ✅ |

---

## Clarification Guidelines

Ask for clarification (all at once, not one field at a time) when:
- The order type is ambiguous (e.g., "buy TSLA at $200" — limit or stop?)
- The user requests an options trade but doesn't specify expiry or strike
- Multiple accounts are linked and the user hasn't specified which to use
- The symbol is ambiguous (e.g., ticker collision between stock and crypto)

Do not ask for clarification on fields with clear defaults (amount defaults to the user's default amount, order_type defaults to market, account defaults to the user's primary account).

---

## Disclaimers

- All investing involves risk, including the possible loss of principal
- Trade It is not a financial advisor and does not provide investment advice
- Options trading involves substantial risk and is not appropriate for all investors
- Trade It cannot withdraw funds, transfer assets, or take custody — it can only place trades
