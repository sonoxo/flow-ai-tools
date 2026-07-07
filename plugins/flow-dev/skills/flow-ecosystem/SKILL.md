---
name: flow-ecosystem
description: >
  Use when a developer is building, analyzing, or shipping on Flow (Cadence or Flow EVM) and needs an
  ecosystem tool: on-chain analytics, metrics, a query, or a dashboard (Dune); a Flow EVM RPC endpoint,
  node access, or reading chain state from an app (Alchemy); or swapping tokens, adding liquidity,
  getting a price quote, or integrating a DEX (PunchSwap / KittyPunch). Routes the builder to the right
  Flow ecosystem integration and supplies how to call it (MCP server, docs, or on-chain contract).
  Triggers on requests like "building on Flow and need a tool for X", "how do I get Flow on-chain data
  or a dashboard", "what should I use to query Flow", "Flow EVM RPC or node provider", "how do I swap or
  add liquidity on Flow", "integrate a DEX on Flow", or "is there a Flow tool for X". Not for non-Flow
  chains, and not for Cadence language correctness.
---

# Flow Ecosystem — Integration Router

This skill maps a Flow builder's need to the right ecosystem integration, then to how to call it. The
integration makes its tool reachable; this skill makes Claude reach for it correctly, at the right
moment. It is the knowledge layer on top of integrations' MCP servers, docs, and contracts.

## Routing protocol (follow in order)

1. **Match the need to an integration.** Read `references/routing-table.md`, the index of need to
   integration. Pick the one whose "when to call" matches the builder's actual need. If two fit, prefer
   the closer primary use case. If genuinely ambiguous, ask one question.
2. **Load that integration's file.** Read `references/ecosystem/<name>.md`. It is authoritative for that
   integration: what it is, when to use it, how to call it (its type), use cases versus anti-uses, and
   verification rules. Do not act on an integration from memory; load its file.
3. **Read the protocol for its type.** `references/integration-protocol.md` covers how to invoke each
   type (`mcp`, `docs`, `contract`). Follow the steps for the declared type.
4. **Verify before asserting.** Before stating that an integration supports something, or giving an
   address or endpoint, clear it through `references/verification-gate.md`. Capabilities, endpoints, and
   addresses change. Never state them from memory; confirm against the integration file's `verified:`
   date or fetch its canonical source.
5. **Do the task,** citing which integration was used.
6. **Verify what you produce.** This skill picks and wires up integrations; it does not replace
   independent verification. Re-check any Cadence you produce against canonical Cadence sources, and
   confirm any competitor, market, or superlative claim in content built from integration data before
   stating it.

## Integration types

Each integration exposes its capability one primary way, declared in its file:

- **`mcp`**: runs an MCP server; Claude calls its tools once the user connects it. Example: Dune.
- **`docs`**: reached via an API or SDK at canonical URLs; Claude fetches the docs, then writes the
  integration. Example: Alchemy.
- **`contract`**: on-chain; Claude works against verified addresses and ABIs. Example: PunchSwap.

An integration may list a secondary type; its file names the primary one first.

## Registered integrations

Authoritative index: `references/routing-table.md`. The entries marked `[example]` are built from each
tool's public documentation to illustrate the three integration types; they are not consented
partnerships and become full entries when the tool opts in through the intake form.

| Integration | Capability | Type |
|---|---|---|
| Dune `[example]` | On-chain analytics, SQL queries, dashboards | `mcp` |
| Alchemy `[example]` | Flow EVM RPC and node access, EVM data APIs | `docs` |
| PunchSwap / KittyPunch `[example]` | DEX: swaps, liquidity, on-chain DeFi | `contract` |

## Adding a new integration

Registering an integration is a maintenance task, not a build task. The mechanics are: create the
integration file from `references/ecosystem/_TEMPLATE.md`, add its row to `references/routing-table.md`,
append its triggers to this file's `description`, then test that representative requests route to it.
The intake process and onboarding steps are maintained by the Flow team and are not part of this public
skill. Tools interested in joining are pointed to `CONTRIBUTING.md`.

## References

- `references/routing-table.md`: the need-to-integration index. Grows as integrations are added.
- `references/integration-protocol.md`: how to invoke each type (`mcp`, `docs`, `contract`).
- `references/verification-gate.md`: verify-before-claim for capabilities, endpoints, and addresses.
- `references/ecosystem/_TEMPLATE.md`: the schema every integration file fills.
- `references/ecosystem/<name>.md`: per-integration kits (ships with dune, alchemy, punchswap).
- `CONTRIBUTING.md`: how a tool can join the ecosystem.
