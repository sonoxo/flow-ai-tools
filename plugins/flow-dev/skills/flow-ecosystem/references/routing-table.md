# Routing Table: Flow Integration Index

The trigger-to-integration index. This is the file that grows as integrations are added; the SKILL.md
router stays small. To route a builder, find the row whose "When to call" matches their actual need,
then load that integration's file and follow its integration type.

If a need matches no row, say so plainly and do not force-fit an integration. A missing capability is a
signal for a future integration, not a reason to misroute.

The current rows are examples built from public documentation to illustrate each integration type, not
consented partnerships. Each becomes a full entry when the tool opts in through the intake form.

## Index

| Integration | Capability area | When to call (trigger conditions) | Type | File |
|---|---|---|---|---|
| **Dune** | On-chain analytics and dashboards | Builder wants Flow on-chain metrics, TVL, volume, wallet or holder counts, DEX activity, a query, a chart, or a shareable dashboard. Anything framed as "show me, track, or measure the data" | `mcp` | `ecosystem/dune.md` |
| **Alchemy** | RPC and node access, EVM data | Builder needs a Flow EVM RPC endpoint, a node provider, to read chain state from Solidity or JS (balances, blocks, logs, `eth_*` calls), or EVM data APIs (token, NFT, transfers) | `docs` | `ecosystem/alchemy.md` |
| **PunchSwap / KittyPunch** | DEX: swap, liquidity, DeFi | Builder wants to swap tokens, add or remove liquidity, read a price or quote, integrate a DEX, or build DeFi on Flow EVM (spot AMM, concentrated liquidity, routing) | `contract` | `ecosystem/punchswap.md` |

## Disambiguation notes

These are the overlaps most likely to cause a misroute. Resolve them this way:

- **"Flow on-chain data": Dune versus Alchemy.** If the builder wants analysis, aggregation, history,
  metrics, or a dashboard (decoded, SQL-queryable, charted), route to Dune. If they want live raw chain
  reads inside their app (a balance, a block, a contract call, a log subscription), route to Alchemy.
  Analytics and reporting is Dune; runtime and app data is Alchemy.
- **"DEX data or price": Dune versus PunchSwap.** Historical or aggregate DEX analytics (volume over
  time, TVL, top pairs) is Dune. A live quote or an actual swap or LP action is PunchSwap contracts.
- **Cadence versus Flow EVM.** Alchemy and PunchSwap operate on Flow EVM (Solidity, JSON-RPC). If the
  builder is in Cadence, note that explicitly: Flow EVM tools are reached from Cadence via the `EVM`
  core contract, and Cadence correctness is out of scope for this skill.

## Adding a row

When onboarding an integration, add one row here (capability area, a sharp "when to call", type, and
file) and create `ecosystem/<name>.md`. Keep the "when to call" phrased as the builder's need, not
the integration's feature list. Routing is need-first.
