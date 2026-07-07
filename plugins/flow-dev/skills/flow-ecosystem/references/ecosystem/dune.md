# Integration Kit: Dune (the reference integration)

This is the gold-standard, fully worked integration file. New `mcp` integrations should match its shape
and rigor. Dune is the launch and reference integration because it is a real, in-use integration with an
official remote MCP server.

```
integration: dune
capability: On-chain analytics, SQL queries, dashboards, and visualizations
type: mcp
verified: 2026-06-23
canonical_source: https://docs.dune.com/api-reference/agents/mcp
```

## 1. What it is

Dune is an on-chain analytics platform with a SQL query engine over a data warehouse spanning more than
100 indexed blockchains. Its official remote MCP server lets an agent discover datasets, write and run
DuneSQL queries, inspect results, and build visualizations and dashboards, all conversationally. For a
Flow builder it is the fastest path from "I want to know X about Flow on-chain activity" to a verified
number, chart, or shareable dashboard.

## 2. When to call it (trigger conditions)

Route here when the builder wants analysis, aggregation, history, or reporting of on-chain data:

- "What's the TVL, volume, holder count, or active wallets for [Flow protocol or token]?"
- "Track [metric] on Flow over time" or "show me the trend"
- "Write me a query for ...", "build a dashboard for ...", "make a chart of ..."
- "Compare [protocol A] versus [protocol B] on Flow"
- Anything framed as "show me, measure, or track the data" as analytics, not a live in-app read.

## 3. How to call it (type `mcp`)

- **Server URL:** `https://api.dune.com/mcp/v1`
- **Auth (two modes):**
  - **OAuth 2.0:** recommended for clients with browser access. Add the server by name and URL, and a
    browser login flow completes it.
  - **API key:** header `x-dune-api-key: <DUNE_API_KEY>` (or, for clients that only support URL auth,
    `?api_key=<DUNE_API_KEY>`). Header is preferred.
- **Connect (examples):**
  - Claude Code (OAuth): `claude mcp add --scope user --transport http dune https://api.dune.com/mcp/v1`
  - Claude Code (API key): add `--header "x-dune-api-key: <key>"` to the above.
  - The user connects it in their own client; Claude cannot connect it for them. If Dune tools are
    already present in the session, use them directly.
- **Key tools** (verify the live list with `searchDocs` or the docs if it matters, since Dune updates
  them):
  - *Discovery:* `searchTables` (find tables by protocol, chain, or category),
    `searchTablesByContractAddress` (decoded tables for a contract, useful for a specific Flow
    contract), `listBlockchains`, `getTableSize` (estimate data scanned before running), `searchDocs`.
  - *Query lifecycle:* `createDuneQuery`, `getDuneQuery`, `updateDuneQuery`, `executeQueryById`,
    `getExecutionResults`.
  - *Visualization:* `generateVisualization`, `getVisualization`, `updateVisualization`,
    `listQueryVisualizations`.
  - *Dashboard:* `createDashboard`, `getDashboard`, `updateDashboard`, `archiveDashboard`.
  - *Materialized views:* `createMaterializedView` (plus list, get, refresh, update, delete) for keeping
    a result table fresh on a schedule.
  - *Account:* `getUsage` (check credit usage, since MCP calls spend the user's Dune API credits).
- **Workflow pattern:** discover the table first (`searchTables` or `searchTablesByContractAddress`),
  check cost (`getTableSize`), write and run the query (`createDuneQuery`, then `executeQueryById`, then
  `getExecutionResults`), then visualize or build a dashboard if asked. Do not guess table names;
  discover them.
- **Timeout note:** long-running queries can exceed a client's default MCP tool timeout (around 60
  seconds in some clients). If a poll times out, increase the client's tool timeout and re-run rather
  than assuming failure.

## 4. Use cases (do use it for)

- Flow protocol or token dashboards (TVL, volume, fees, users) for a blog post, investor update, or
  integration brief.
- Ad-hoc on-chain questions answered with a real, reproducible query.
- Decoded-event analysis for a specific Flow contract address.
- Recurring or refreshed metrics via a materialized view.
- Cross-protocol or cross-chain comparisons where Flow is one of the chains.

## 5. Anti-uses (do not use it for)

- **Live in-app chain reads** (a user's current balance, latest block, a contract call your dApp makes
  at runtime). That is Alchemy (`docs`), not Dune. Dune is analytics, not an RPC.
- **Executing a swap or any write.** That is PunchSwap contracts.
- **Cadence language questions.** Out of scope for this skill.
- **Treating a query result as live or real-time.** Dune data is indexed with some lag; note that for
  time-sensitive claims.

## 6. Verification and guardrails

- **Version-sensitive** (re-check per `verification-gate.md`): the exact tool list, endpoint, auth
  header, and credit or pricing terms. **Stable:** that Dune is SQL-over-a-warehouse analytics with an
  official MCP.
- MCP calls consume the user's Dune API credits; mention `getUsage` for heavy work.
- Any number pulled from Dune that ships in content is a claim, so confirm it before stating it
  (with the query and date as the source).
- Re-verify cadence: every 3 months or so, or before relying on the tool list or endpoint in a setup
  guide.
