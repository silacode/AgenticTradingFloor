# AgenticTradingFloor
A multi-LLM trading simulation platform where four autonomous agents analyze real-time market data and make coordinated trading decisions. Built with MCP for seamless tool integration and data-driven experimentation in a realistic trading environment.

## Highlights
- Autonomous, multi-agent trading loop with alternating trade/rebalance cycles
- Live market data via Polygon (or free EOD fallback)
- Built-in MCP servers for Accounts, Market, Push; optional Search + Memory servers
- Gradio dashboard with live PnL, charts, holdings, transactions, logs
- SQLite persistence for accounts, logs, and cached market data

## Quickstart
- Install uv: see `https://docs.astral.sh/uv/`
- From the project root:
```bash
cd /Volumes/SpaceHD/Learn/AgenticTradingFloor
uv sync
```

## Run the dashboard
- Visualize all traders in a single view:
```bash
uv run src/app.py
```

## Run the trading floor (scheduler)
- Runs every N minutes; alternates between trading and rebalancing:
```bash
# Optional environment knobs
export RUN_EVERY_N_MINUTES=60
export RUN_EVEN_WHEN_MARKET_IS_CLOSED=false
export USE_MANY_MODELS=false

uv run src/trading_floor.py
```

## Data sources: Live vs Free EOD
- Set Polygon credentials to use live or delayed intraday data; otherwise falls back to EOD or random placeholder prices.

Required for live/deferred intraday:
```bash
export POLYGON_API_KEY=your_key
# Options: "paid" (no real-time trades/quotes), "realtime" (full real-time), or leave unset for EOD
export POLYGON_PLAN=realtime
```

## Seamless tool integration (MCP)
Agents use MCP tools that are auto-wired at runtime:
- Accounts: `get_balance`, `get_holdings`, `buy_shares`, `sell_shares`, `change_strategy`
- Market (local or Polygon MCP): `lookup_share_price` (local) or rich Polygon tools (snapshot, last trade, fundamentals)
- Push notifications: `push`
- Researcher toolbox: web fetch, Brave Search, and persistent memory

These are orchestrated via stdio MCP clients (no manual wiring needed to run the platform).

## Manual MCP servers (optional)
Run any server by hand for debugging or external client connections.

- Local Market (EOD or fallback):
```bash
uv run src/market_server.py
```

- Polygon Market MCP (live/delayed intraday):
```bash
export POLYGON_API_KEY=your_key
uvx --from git+https://github.com/polygon-io/mcp_polygon@v0.1.0 mcp_polygon
```

- Accounts:
```bash
uv run /Volumes/SpaceHD/Learn/AgenticTradingFloor/src/accounts_server.py
```

- Push notifications (Pushover):
```bash
export PUSHOVER_USER=your_user
export PUSHOVER_TOKEN=your_token
uv run src/push_server.py
```

### Manual “Search + Memory” MCP servers (for research workflows)
- Web fetch:
```bash
uvx mcp-server-fetch
```

- Brave Search:
```bash
export BRAVE_API_KEY=your_key
npx -y @modelcontextprotocol/server-brave-search
```

- Memory (LibSQL, local db per trader):


The app will also auto-start these internally when running the agents; manual startup is only needed if you prefer explicit processes or want to attach external MCP clients.

## Environment variables
- Trading/agents:
  - `RUN_EVERY_N_MINUTES` (default 60)
  - `RUN_EVEN_WHEN_MARKET_IS_CLOSED` (default false)
  - `USE_MANY_MODELS` (default false; toggles multiple vendor models)
  - Optional LLM keys: `OPENROUTER_API_KEY`, `DEEPSEEK_API_KEY`, `GROK_API_KEY`, `GOOGLE_API_KEY`
- Market data:
  - `POLYGON_API_KEY`
  - `POLYGON_PLAN` = `paid` | `realtime` | unset (EOD)
- Search:
  - `BRAVE_API_KEY`
- Push:
  - `PUSHOVER_USER`, `PUSHOVER_TOKEN`

## Reset demo accounts
- Resets four themed strategies and clears balances/holdings history:
```bash
uv run src/reset.py
```

## Data storage
- SQLite database: `src/accounts.db`
  - Tables: `accounts`, `logs`, `market` (cached grouped daily aggs)

## Troubleshooting
- No prices or zero quotes:
  - Check `POLYGON_API_KEY` and `POLYGON_PLAN`, or expect EOD/free fallback
- Brave Search errors:
  - Rate limited or missing `BRAVE_API_KEY`; the fetch server can still retrieve web content
- Push not received:
  - Verify `PUSHOVER_USER` and `PUSHOVER_TOKEN`
- UI not updating:
  - Keep the scheduler running; the UI polls and reads from the database

## What’s inside (at a glance)
- Core loop: `/src/trading_floor.py`
- Agents and tools: `/src/traders.py`, `/src/templates.py`, `/src/tracers.py`
- Market data: `/src/market.py`, `/src/market_server.py`
- Accounts: `/src/accounts.py`, `/src/accounts_server.py`
- Push server: `/src/push_server.py`
- MCP wiring: `/src/mcp_params.py`
- Dashboard: `/src/app.py`