# Integration Kit: Alchemy

The `docs`-type exemplar: capability reached through a documented API, on Flow EVM.

```
integration: alchemy
capability: Flow EVM RPC and node access, EVM data APIs
type: docs
verified: 2026-06-23
canonical_source: https://www.alchemy.com/docs/reference/flow-evm-api-quickstart
```

## 1. What it is

Alchemy is a node and RPC infrastructure provider with Flow EVM support. It gives builders a hosted RPC
endpoint and EVM data APIs for Flow EVM, using the standard Ethereum JSON-RPC interface. Anyone familiar
with Ethereum tooling can read Flow EVM chain state and send transactions without running a node.

## 2. When to call it (trigger conditions)

Route here when the builder needs live, runtime, in-app chain access on Flow EVM:

- "What's the RPC URL or node provider for Flow EVM?"
- "How do I read a balance, block, log, or contract state on Flow from my app?"
- "Connect viem, ethers, or web3.py to Flow", or "set up a Flow EVM client"
- "Send a transaction to Flow EVM" (signing and broadcast via RPC)
- EVM data APIs: token balances, NFT data, and transfers for a Flow EVM address.

## 3. How to call it (type `docs`)

- **Canonical docs:** `https://www.alchemy.com/docs/reference/flow-evm-api-quickstart`. Fetch for the
  current method list; the API follows the full Ethereum JSON-RPC spec.
- **Endpoints** (verified 2026-06-23; re-confirm before a builder wires production):
  - Mainnet RPC: `https://flow-mainnet.g.alchemy.com/v2/<api-key>`
  - Testnet RPC: `https://flow-testnet.g.alchemy.com/v2/<api-key>`
  - Testnet WebSocket: `wss://flow-testnet.g.alchemy.com/v2/<api-key>`
- **Auth:** API key embedded in the URL path. Use an `<api-key>` placeholder in any code you write,
  never a real key.
- **Compatibility:** full Ethereum JSON-RPC. Works with viem, ethers, web3.js, and web3.py, plus
  Hardhat and Foundry for deploys. Flow EVM has EVM equivalence, so standard Solidity tooling works
  largely unchanged.
- **Example (viem, testnet):**
  ```js
  import { createPublicClient, http } from "viem";
  import { flowTestnet } from "viem/chains";
  const client = createPublicClient({
    chain: flowTestnet,
    transport: http("https://flow-testnet.g.alchemy.com/v2/<api-key>"),
  });
  const blockNumber = await client.getBlockNumber();
  ```

## 4. Use cases (do use it for)

- Standing up a Flow EVM RPC connection for a dApp or backend.
- Runtime reads: balances, block data, event logs, and `eth_call` against a contract.
- Broadcasting signed transactions to Flow EVM.
- Token, NFT, and transfer data for a Flow EVM address via Alchemy's data APIs.

## 5. Anti-uses (do not use it for)

- **Historical analytics, dashboards, or aggregate metrics.** That is Dune. Alchemy is per-call runtime
  access, not a SQL warehouse.
- **Cadence-side access.** Alchemy's Flow support is Flow EVM (Solidity, JSON-RPC). Native Cadence reads
  use Flow's Cadence access nodes and SDKs, not this. From Cadence, Flow EVM is reached via the `EVM`
  core contract; verifying that Cadence is out of scope for this skill.
- **A swap or LP action.** That is PunchSwap contracts. Alchemy is the transport you would send it
  through, not the DEX logic.

## 6. Verification and guardrails

- **Version-sensitive:** exact endpoint hostnames, supported method list, compute-unit pricing and rate
  limits, and archive or trace availability. **Stable:** that Alchemy provides Flow EVM RPC over
  standard JSON-RPC.
- Note compute-unit and rate-limit pricing to the builder before production reliance (fetch current
  terms).
- Re-verify cadence: endpoints and pricing every 3 months or so, or before a setup guide ships.
- Multiple providers serve Flow EVM RPC. If the builder needs a comparison or a superlative ("cheapest",
  "fastest"), confirm it against a current source; do not assert it from this file.
