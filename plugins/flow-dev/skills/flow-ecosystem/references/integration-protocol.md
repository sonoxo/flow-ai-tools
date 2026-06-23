# Integration Protocol: How to Invoke Each Integration Type

Every integration declares a primary integration type in its file. This document is the how-to for each
type. Read the section matching the integration's `type:` before acting.

One rule applies to all three. The integration file's stated capabilities, endpoints, and addresses
carry a `verified:` date. If that date is stale, or the detail is load-bearing (an address a user will
send funds to, an endpoint they will wire into production), re-confirm against the integration's
canonical source before stating it. See `verification-gate.md`.

## Type `mcp`: the integration runs an MCP server

The integration exposes tools through an MCP server. This is the lowest-friction path: once connected,
Claude calls the tools conversationally.

Steps:

1. **Check if it is connected.** The integration's tools may already be available in the session. If
   tool names matching the integration appear, use them directly.
2. **If not connected, guide the user to connect it.** Give the exact endpoint URL and auth method from
   the integration file (for example, an MCP server URL plus an API-key header or OAuth). Do not invent
   connection steps; use the integration file's documented setup. The user connects via their client's
   connector settings; you cannot connect it for them.
3. **Call the right tool for the task.** The integration file lists the key tools and what each is for.
   Pick the minimal tools that satisfy the need. Prefer discovery tools first when you do not yet know
   the data shape (for example, find the table or dataset before querying it).
4. **Handle results and cost.** MCP calls may consume the integration's credits or quota and can be slow
   on large jobs. Respect any rate or timeout notes in the integration file. Return results, citing the
   integration.

Do not hand-roll an HTTP client to hit the integration's REST API when an MCP server exists and the user
can connect it. The MCP path is the supported one. Falling back to `docs` or REST is only correct when
the user cannot use MCP in their environment, and the integration file says so.

## Type `docs`: capability reached via documented API or SDK

The integration's capability lives behind an API or SDK documented at canonical URLs. Claude fetches the
docs and writes the integration.

Steps:

1. **Fetch the canonical doc URLs** listed in the integration file. Do not rely on memory for endpoints,
   method signatures, or parameters; fetch the current page.
2. **Confirm the Flow-specific details:** the exact endpoints for Flow or Flow EVM (mainnet and
   testnet), the chain ID, the auth scheme, and any Flow-specific caveats the integration file flags.
3. **Write the integration code** in the builder's stack. The integration file notes supported libraries
   (for example, viem, ethers, or web3.py for an EVM RPC provider). Use placeholders for secrets
   (`<api-key>`), never a real key.
4. **Note limits:** rate limits, compute-unit pricing, and archive or trace availability, whatever the
   integration file or fetched docs specify, so the builder is not surprised in production.

## Type `contract`: the integration is on-chain

The integration is a set of deployed contracts on Flow EVM (or Cadence). Claude works against verified
addresses and ABIs.

Steps:

1. **Use only verified addresses.** Take addresses from the integration file, which records a
   `verified:` date and a canonical source. For anything a user will transact against, re-confirm the
   address from the integration's canonical docs or explorer before presenting it. A wrong address can
   lose funds. This is the highest-stakes verification in the skill.
2. **Match network to intent.** Use the testnet addresses for build and test guidance, and mainnet only
   when the builder is going live. The integration file lists both; never mix them.
3. **Get the ABI or interface.** For Uniswap-style forks, the standard V2 and V3 interfaces apply
   (router, factory, pair or pool). The integration file says which fork and flags any deviations. For a
   non-standard contract, fetch the verified ABI from the explorer link in the integration file.
4. **Write the interaction.** A read calls a view such as `getAmountsOut` or `quote`. A write calls
   `approve`, then `swapExactTokensForTokens` or `addLiquidity`, with slippage and a deadline. Respect
   the fee and any gotchas the integration file documents.
5. **Re-verify any Cadence output** against canonical Cadence sources if any Cadence is produced (for
   example, calling Flow EVM from Cadence via the `EVM` contract).

## Choosing when an integration has more than one type

Some integrations offer multiple paths (for example, on-chain contracts and a data API). Default to the
primary type listed first in the integration file, which is chosen for the most common builder need.
Switch to a secondary type only when the task specifically calls for it; the integration file says when.
