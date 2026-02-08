# Trade It MCP Server
Now available through the [_Official MCP Registry_](https://registry.modelcontextprotocol.io/?q=app.tradeit%2Fmcp)

<a href="https://glama.ai/mcp/servers/@trade-it-inc/trade-agent-mcp">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@trade-it-inc/trade-agent-mcp/badge" />
</a>



**Endpoints:**  
- Streamable HTTP: `https://mcp.tradeit.app/mcp` 
- SSE: `https://mcp.tradeit.app/sse`  
**Mode:** Remote-only (no local deployment required)

## Overview

The Trade It MCP Server brings stock, crypto, and options trading support to agents. It enables natural-language interaction with stock and crypto brokerages—execute trades, query portfolio performance, and surface market insights by sending plain-English requests through the MCP protocol.

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

## Tools

- 💬 **Create Trade**
  Creates a trade order to buy or sell an asset.

  ORDER TYPES:
  - **market** (default) → Executes immediately at current market price. No price fields required.
  - **limit** → Executes only at a specific limit_price or better. Requires `limit_price`.
  - **stop** → Triggers a market order when stop_price is reached. Requires `stop_price`.
  - **stop_limit** → Triggers a limit order when stop_price is reached. Requires BOTH `stop_price` and `limit_price`.
 
  EXAMPLES:
  - "Buy $1000 of Tesla"
  - "Buy $1000 of Tesla, but only if the price drops to $150 or lower"
  - "Sell 10 shares of Apple if the price falls to $140 or lower"
  - "Buy a share of Apple if it hits $200"
  - "Buy 10 shares of Apple if the price rises to $140, but don't pay more than $142 per share"

  DEFAULTS:
  - If no amount is given, your default amount is used.
  - If no account is given, your default account is used. 
  - If no order type is given, the trade is a market order. 
  - If auto-execute is enabled in settings, the trade will execute immediately. Otherwise, it gets created in draft state and requires a call to `Execute Trade` to complete. This allows you to review and confirm trades.

- 💬 **Create Option Trade (Beta)**
  Creates a trade order to buy or sell an options contract.
 
  EXAMPLES:
  - "Buy 1 call option on Apple with a $300 strike price expiring next month"
  - "Sell a covered call on my Microsoft shares at $500 strike"
  - "Open a call spread: buy 1 TSLA $475 call and sell 1 TSLA $485 call, both expiring next week"
  - "Buy an ATM straddle on SPY, expiring this Friday"
  - "Buy 2 AMZN 200 1/30 P, limit price $3.50"
  - "Sell AMZN260130P00200000"

- 💬 **Execute Trade**
  Execute the trade on your brokerage.

- 💬 **Show Account Details**
  List your linked brokerages along with their current value and cash balance.
  Example: `"Show my accounts"`

- 💬 **Search Asset**
  Get current price and metadata for any stock or cryptocurrency.
  Example: `"How's Apple doing?"` or `"What's the price of TSLA?"`

---

## Getting Started

1. First, create an account at https://tradeit.app.
2. Sign up for the Pro plan's free trial.
3. Connect your brokerage of choice.

## Connecting
1. Connect your MCP client to `https://mcp.tradeit.app/mcp` or `https://mcp.tradeit.app/sse`.
2. Authenticate through the browser-based OAuth flow.
3. You're now ready to start trading!
