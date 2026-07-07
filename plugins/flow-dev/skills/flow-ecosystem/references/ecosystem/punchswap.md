# Integration Kit: PunchSwap / KittyPunch

The `contract`-type exemplar: an on-chain integration on Flow EVM, worked against verified addresses.

```
integration: punchswap
capability: DEX swaps, liquidity, and on-chain DeFi integration on Flow EVM
type: contract
verified: 2026-06-23
canonical_source: https://kittypunch.gitbook.io/kittypunch-docs/protocols-and-products-flow/punchswap
```

## 1. What it is

PunchSwap is the primary spot DEX on Flow EVM, built by KittyPunch. It is a fork of Uniswap V2 and V3,
using WFLOW as the base trading pair across the ecosystem. KittyPunch's broader app also includes
StableKitty (stablecoin-optimized swaps) and AggroKitty (an on-chain DEX aggregator), but for a builder
integrating a DEX on Flow, PunchSwap's V2 and V3 contracts are the entry point.

## 2. When to call it (trigger conditions)

Route here when the builder wants an on-chain DeFi action or integration on Flow EVM:

- "Swap token A for token B on Flow", or "integrate swaps into my app"
- "Add or remove liquidity", or "create a pool"
- "Get a live price or quote" for a Flow EVM pair
- "Build DeFi on Flow EVM" needing an AMM (V2) or concentrated liquidity (V3)
- "Which DEX or router do I use on Flow EVM?"

## 3. How to call it (type `contract`)

Fork: Uniswap V2 and V3, so the standard Uniswap V2 and V3 router, factory, pair, and pool interfaces
apply. Re-confirm any mainnet address from the canonical source before a user transacts. A wrong address
can lose funds.

**PunchSwap V2, Mainnet (Flow EVM)** (verified 2026-06-23, source: KittyPunch docs)
| Contract | Address |
|---|---|
| V2 Router | `0xf45AFe28fd5519d5f8C1d4787a4D5f724C0eFa4d` |
| V2 Factory | `0x29372c22459a4e373851798bFd6808e71EA34A71` |
| WFLOW (base pair) | `0xd3bF53DAC106A0290B0483EcBC89d40FcC961f3e` |

**PunchSwap V2, Testnet (Flow EVM)**
| Contract | Address |
|---|---|
| V2 Router (testnet) | `0xeD53235cC3E9d2d464E9c408B95948836648870B` |
| V2 Factory (testnet) | `0x0f6C2EF40FA42B2F0E0a9f5987b2f3F8Af3C173f` |
| WFLOW (testnet) | `0xd3bF53DAC106A0290B0483EcBC89d40FcC961f3e` |

**PunchSwap V3, Mainnet (concentrated liquidity)**
| Contract | Address |
|---|---|
| V3 Factory | `0xf331959366032a634c7cAcF5852fE01ffdB84Af0` |
| Quoter | `0x7eCc830bd3c172d6eD42651b7781D1eE7b1e98B2` |
| QuoterV2 | `0x48Ae9ED61c6ECe580Af86F140AcEC3DDB4A7367E` |
| NonfungiblePositionManager | `0xDfA7829Eb75B66790b6E9758DF48E518c69ee34a` |
| InterfaceMulticall | `0x5eF9Cf25D4D5B599Db370790D3c7BA4F65082C09` |
| TickLens | `0xb1Bd4c83ECBda0C19f8FB7567E4fE7c9e92aBa7d` |
| V3Migrator | `0x2CC66363F4ff296f8ba3B6f05CC92DD778a63e59` |

- **ABI source:** standard Uniswap V2 and V3 ABIs apply (router, factory, pair, and V3 factory, quoter,
  position manager). For exact verified bytecode or ABI, use the Flow EVM explorer (`evm.flowscan.io`
  for mainnet, `evm-testnet.flowscan.io` for testnet) on the address above. KittyPunch also publishes a
  `punchswap-v2-sdk`.
- **Transport:** you reach these contracts via a Flow EVM RPC, that is, Alchemy
  (`ecosystem/alchemy.md`) is the runtime transport and PunchSwap is the DEX logic. The two compose.
- **Read (quote), V2:** `Router.getAmountsOut(amountIn, [tokenIn, tokenOut])`.
- **Write (swap), V2:** `approve` the router on the input token, then
  `swapExactTokensForTokens(amountIn, amountOutMin, path, to, deadline)`. Always set `amountOutMin` from
  a quote minus slippage, and a real `deadline`.
- **Liquidity, V2:** `addLiquidity` and `removeLiquidity` (or the `ETH` or WFLOW variants for native
  FLOW).
- **V3:** quotes via `QuoterV2`, positions via `NonfungiblePositionManager`, the standard Uniswap V3
  flow.

## 4. Use cases (do use it for)

- Integrating swaps into a Flow EVM app (V2 router), or routing through concentrated liquidity (V3).
- Programmatic LP provisioning and position management.
- Reading live on-chain prices and quotes for Flow EVM pairs.
- Any DeFi primitive that needs an AMM on Flow EVM.

## 5. Anti-uses (do not use it for)

- **Historical DEX analytics** (volume over time, TVL trends, top pairs). That is Dune. PunchSwap
  contracts give you the current state, not history.
- **Just an RPC connection.** That is Alchemy. PunchSwap is the DEX, not the transport.
- **Stablecoin-to-stablecoin optimized swaps.** KittyPunch's StableKitty is purpose-built for that. For
  best-execution routing across all Flow EVM venues, AggroKitty (the aggregator) may be the better
  answer. Flag these sibling products when the need is specifically stable-swap or routing.
- **Cadence-native DEX logic.** These are Flow EVM (Solidity) contracts; reach them from Cadence via the
  `EVM` core contract, and independently verify any such Cadence.

## 6. Verification and guardrails

- **High stakes:** addresses here are funds-bearing. Always re-confirm the mainnet router or factory
  address from the canonical KittyPunch docs (or the verified explorer page) before a user transacts.
  Never present an address you have not confirmed current.
- **Version-sensitive:** addresses (contracts can be redeployed), fee splits, and which sibling products
  exist. **Stable:** that PunchSwap is a Uniswap V2 and V3 fork using WFLOW as the base.
- Fee: every V2 trade charges 0.3% (0.05% to the KittyPunch treasury, 0.25% to LPs). Confirm current.
- Always use a quote-derived `amountOutMin` (slippage protection) and a `deadline` on swaps.
- Re-verify cadence: addresses every 3 months or so, and before any guide that a user will transact
  against.
