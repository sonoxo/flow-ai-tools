# Verification Gate: Verify Before Claiming

Verify before you claim, applied to integration facts: never state an integration's current
capability, endpoint, pricing, or contract address from memory. Integration
products move fast. MCP tool lists change, endpoints get versioned, and contracts get redeployed. A
confident but stale integration fact wastes a builder's time at best and loses funds at worst.

## What requires verification before you state it

Always confirm, by fetching the canonical source or relying on a fresh `verified:` date in the
integration file:

- A contract address the user will transact against. Highest stakes; always re-confirm from the
  integration's canonical docs or explorer. A wrong address can lose funds.
- An RPC or API endpoint URL the builder will wire into production.
- A claim that an integration does or does not support a specific feature, chain, or method.
- Pricing, credits, rate limits, and quotas.
- The list of tools an MCP server exposes (tool names and what they do).
- Any superlative or comparison ("the only Flow DEX with X", "cheapest RPC"). Confirm it against a
  current source before stating it.

Does not require a fetch:

- Stable, structural facts the integration file records with a recent `verified:` date that are not the
  load-bearing detail of the answer (for example, "Dune is an analytics platform", "PunchSwap is a
  Uniswap fork").
- Generic protocol mechanics that are not integration-specific (how a Uniswap V2 `getAmountsOut` works).

## The `verified:` date contract

Every integration file header carries `verified: YYYY-MM-DD`. Treat it as the freshness stamp:

- **Under 3 months old:** trust the file for structural facts. Still re-confirm any address the user
  will transact against and any production endpoint.
- **3 to 12 months old:** re-fetch endpoints, tool lists, and pricing before stating them. Structural
  facts likely still hold.
- **Over 12 months old:** treat the whole file as a lead, not a source. Re-verify capabilities,
  endpoints, and addresses from canonical sources, and update the file's `verified:` date.

When you re-verify a detail, update the integration file (the value and the `verified:` date) so the
next session benefits. The registry should get more trustworthy with use.

## The protocol

1. **Identify the load-bearing integration facts** in your intended answer: the address, endpoint, tool,
   capability, or number the builder will act on.
2. **Check the integration file's `verified:` date** against the freshness contract above.
3. **If a fetch is required,** hit the integration's canonical source named in the integration file (its
   docs URL, its contracts page, its explorer), not a third-party aggregator and not search snippets.
4. **State only confirmed facts,** and say where each came from (for example, "per Alchemy's Flow EVM
   quickstart, fetched today", or "PunchSwap V2 Router per KittyPunch docs, verified 2026-06-23"). If
   you could not confirm, say so and do not present the detail as current.

## Failure mode this prevents

A builder asks how to integrate a given tool. Claude gives a remembered endpoint or address. It has
changed. The builder ships against a dead endpoint or sends funds to a wrong or old contract. Trust is
destroyed, and funds may be lost.

When the stakes are an address or a production endpoint, fetching is never the slow path. It is the only
path.
