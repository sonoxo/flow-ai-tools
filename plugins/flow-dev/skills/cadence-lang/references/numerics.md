# Cadence Numeric Types and Arithmetic

## Prefer the 128-bit fixed-point types

`Fix128` and `UFix128` carry 24 decimal places;
`Fix64` and `UFix64` carry 8, and cap out around `9.2e10` / `1.8e11`.
Protocol values that are multiplied and divided — prices, share counts, funds, ratios —
lose meaningful precision at 8 decimals, and the loss compounds over a position's lifetime.
Declare those as `[U]Fix128`.

```cadence
// ❌ 8 decimals for a value that is repeatedly multiplied and divided
access(all) var price: UFix64

// ✅ 24 decimals
access(all) var price: UFix128
```

The exception is a boundary fixed by a standard —
`FungibleToken` balances and amounts are `UFix64`, for example.
Convert at that boundary; do not widen the standard's own types.

## Use number literals with contextual types, not conversion functions

An annotated declaration already tells the compiler which numeric type the literal is,
so write the literal directly.
Wrapping it in a conversion function adds a runtime conversion
and hides the type behind a call.

```cadence
// ❌ Conversion function around a literal
let start = UFix128(0.0)
let leverage = UInt8(10)

// ✅ Literal with a contextual type
let start: UFix128 = 0.0
let leverage: UInt8 = 10
```

An unannotated fixed-point literal infers to `UFix64` (or `Fix64` if signed),
not to the 128-bit type — so the annotation is what makes `let start: UFix128 = 0.0` correct,
and omitting it silently gives you 8 decimals.
Reserve the conversion functions for converting an existing *value* of another type.

## Multiply before dividing

Integer and fixed-point division truncates, and every truncation is permanent.
`a / b * c` throws away the remainder of `a / b` before `c` can restore any of it,
so it drifts from `a * c / b` by up to `c` units of the last decimal place.

```cadence
// ❌ Truncates before the multiplication can restore the remainder
let share = funds / totalShares * shares

// ✅ Multiply first
let share = funds * shares / totalShares
```

The exception is an intermediate product that would overflow.
Arithmetic is checked, so an overflowing multiply aborts the transaction
rather than wrapping — when the product genuinely cannot fit,
divide first and say in a comment that precision is being traded for range.

## Know what aborts: arithmetic and conversions

Two Cadence behaviours turn a silent bug in other languages into a hard abort here.
Both are worth designing around rather than discovering in a failed transaction.

**Every arithmetic operation is checked.**
`UInt8.max + 1` aborts; there is no wrapping.
Underflow on unsigned types is the same:
`0 - 1` on a `UInt64` aborts, so a subtraction that could go negative
needs a pre-condition, not a clamp after the fact.

```cadence
// ❌ The clamp never runs — the subtraction already aborted
let remaining = self.total - amount
let safe = remaining < 0 ? 0 : remaining

// ✅ Guard before subtracting
pre {
    amount <= self.total:
        "Vault.withdraw: amount \(amount) exceeds total \(self.total)"
}
let remaining = self.total - amount
```

**Every numeric conversion is checked.**
`UInt8(a)` for a `UInt32` value above `UInt8.max` aborts.
A conversion is therefore an assertion about range; validate at the boundary where the
value enters, where you can produce a message, rather than letting the conversion abort
deep in a call chain.

```cadence
// ✅ The range assertion carries a message, at the point the value arrives.
// `UInt8.max` rather than `255`: the bound is the target type's own limit,
// so it stays correct if the field's type ever widens.
pre {
    leverage <= UInt64(UInt8.max):
        "Market.open: leverage \(leverage) exceeds the maximum of \(UInt8.max)"
}
let stored = UInt8(leverage)
```

Two details the compiler enforces here.
The comparison is `<=`, not `<`: `UInt8.max` is itself a legal `UInt8`,
so `< UInt8.max` would reject a value the conversion accepts.
And both operands must have the same type — `leverage <= UInt8.max` for a `UInt64`
`leverage` fails with `cannot apply binary operation <= to types UInt64 and UInt8`,
which is why the constant is widened to `UInt64(UInt8.max)` rather than the value narrowed.
