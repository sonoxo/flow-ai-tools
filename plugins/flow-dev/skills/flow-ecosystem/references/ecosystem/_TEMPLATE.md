# Integration Kit: _TEMPLATE

Copy this file to `references/ecosystem/<name>.md` and fill every field. Delete this top line and the
parenthetical guidance once filled. Keep it tight; this file loads into context when the integration is
routed to.

```
integration: <name>
capability: <one-line capability area, e.g. "On-chain analytics and dashboards">
type: <mcp | docs | contract>   # primary first; list a secondary after if any
verified: <YYYY-MM-DD>          # last date the facts below were confirmed
canonical_source: <the integration's own docs, contracts, or explorer URL, the source of truth>
```

## 1. What it is

One or two sentences. What the integration is, for a Flow builder. No marketing fluff.

## 2. When to call it (trigger conditions)

The builder needs that should route here. Write them as needs, not features. These are the phrases that
also go into the SKILL.md description. Be specific enough not to collide with other integrations; see
the routing-table disambiguation notes.

## 3. How to call it (integration)

Follow the shape for the declared `type`. Include only verified, current specifics.

- **For `mcp`:** server URL, auth method (header or OAuth, plus exact header name), the key tools and
  what each is for, connection steps for the user, and any timeout or credit notes.
- **For `docs`:** canonical doc URLs, the exact Flow or Flow EVM endpoints (mainnet and testnet), chain
  ID, auth scheme, supported libraries, and rate-limit or pricing notes.
- **For `contract`:** which fork or standard, verified addresses (mainnet and testnet, labeled), ABI
  source, key read methods, key write methods, fees, and explorer links.

## 4. Use cases (do use it for)

3 to 6 concrete builder tasks this integration is the right answer for.

## 5. Anti-uses (do not use it for)

Explicit "don't reach for this when..." cases that should route elsewhere. This is what prevents
misrouting; fill it honestly.

## 6. Verification and guardrails

- What is stable versus version-sensitive, so `verification-gate.md` knows what to re-check.
- Auth, rate-limit, and cost constraints a builder must know.
- Any safety note (for example, "always confirm the mainnet router address before a user transacts").
- Re-verify cadence: how often this file's facts should be re-confirmed.
