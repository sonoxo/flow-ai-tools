---
name: flow-defi
description: |
  Architecture and safety guide for building DeFi protocols on the Flow blockchain. Covers Flow's DeFi-specific advantages (MEV-free EVM, cross-VM atomic transactions, on-chain automation), core DeFi primitives (lending health factors, interest rate kink models, AMM type selection), liquidity bootstrapping strategy (veFLOW, Merkl, CL ranges, bootstrapping benchmarks), the current Flow DeFi ecosystem map (existing protocols, missing primitives, opportunity analysis), and protocol-level safety rules (oracle sourcing, assumptions about foreign token contracts, emergency stops).
  TRIGGER when: designing a lending protocol on Flow, choosing AMM type for a DEX, liquidity bootstrapping strategy, veFLOW mechanics, pricing a protocol from an oracle, pausing a protocol, "how does Flow DeFi work", "AMM types on Flow", "liquidity bootstrapping", "Flow DeFi ecosystem", "cross-VM composability for DeFi", "health factors", "interest rate curves", "collateral design", "DEX TVL", "Merkl integration", "missing DeFi primitives on Flow", "perp DEX on Flow", "launchpad on Flow", "COA pattern", "MEV-free EVM", "spot price as oracle", "price feed staleness", "emergency pause", "circuit breaker", "can I trust this token contract".
  DO NOT TRIGGER when: designing token economics (use flow-tokenomics), asking about Cadence syntax or general contract security — access control, reentrancy ordering, input validation, bounded loops (use cadence-lang).
---

# Flow DeFi Architecture

Design and build DeFi protocols on Flow — covering architectural advantages, core primitives, liquidity strategy, and the ecosystem landscape.

## Navigation Map

| Task | Reference |
|------|-----------|
| Flow DeFi foundations: COAs, MEV-free EVM, cross-VM atomicity, on-chain automation | [protocol-architecture.md](references/protocol-architecture.md) |
| Core primitives: lending models, AMM type selection guide, risk framework | [defi-primitives.md](references/defi-primitives.md) |
| Liquidity bootstrapping: veFLOW, Merkl, CL ranges, failure modes, benchmarks | [liquidity-strategy.md](references/liquidity-strategy.md) |
| Ecosystem map: current state, missing primitives, opportunity analysis | [ecosystem-map.md](references/ecosystem-map.md) |
| Protocol safety: oracle sourcing, foreign-contract assumptions, emergency stop | [protocol-safety.md](references/protocol-safety.md) |

## Key Principles

1. **MEV-free EVM** — Flow's architecture prevents sandwich attacks on the EVM side; LP positions retain more yield
2. **Cross-VM atomicity** — A single Cadence transaction can call EVM contracts atomically (no bridges for Cadence↔EVM)
3. **COA pattern** — Cadence Owned Accounts bridge Cadence logic and EVM contracts within the same address space
4. **On-chain automation** — FlowTransactionScheduler (FLIP 330) enables recurring transactions without keeper bots
5. **Lock-up or lose** — Liquidity programs without lock-ups lose 60–80% TVL within 30 days
6. **Price from a feed, never a pool** — reserves move within a transaction, so a spot price is an attacker-chosen input
7. **Stoppable without an upgrade** — a contract holding value needs a pause switch that still lets users withdraw

## Companion Skills

- **`flow-tokenomics`** — Use alongside when designing the token economics for your DeFi protocol (veFLOW-style governance token, fee distribution, etc.).
- **`cadence-lang`** — Consult for Cadence language rules when implementing protocol contracts. Its `security-best-practices.md` carries the general contract security rules — checks-effects-interactions, resolving a path once, trust-boundary validation, bounded loops — that `protocol-safety.md` here builds on.
- **`cadence-audit`** — Use to audit protocol contracts before deployment.
