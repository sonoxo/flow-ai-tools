# Cadence Style and Readability

Rules that make intent obvious to the next reader.
Several of them are also mechanically checked —
`flow cadence lint` reports them as `if-let-hint`, `unnecessary-cast-hint`,
`unused-variable-hint`, `unused-result-hint`, and `replacement-hint`.

## Make the name match the logic

A function's name is a claim about what it does; the body has to honour it.
`getPrice` that also writes a reading, `validate` that mutates state,
or `isFresh` that panics instead of returning `false` are all bugs in waiting —
every caller reasons about the name, not the body.
When the body grows past the name, rename the function or split it;
do not let the name go stale.

## Keep argument labels, and label for the call site

Cadence gives every parameter a label by default,
so the work is not adding labels but declining to remove them:
do not write `_` in front of a parameter unless the call site reads unambiguously without it.
A single obvious argument (`isValidPoolName(_ poolName: String)`) can drop its label;
a function with several parameters — especially several of the same type — must not.

Where the internal name and the call-site word differ, declare both:

```cadence
access(all)
fun vaults(of address: Address): [VaultRecord] {
    let account = getAccount(address)
    // ...
}
```

`vaults(of: operator)` reads as the question it answers,
while the body still calls the parameter `address`.

## Use string template expressions instead of `.concat`

Interpolate values with `\(expr)` rather than chaining `.concat` calls.
The template reads as the sentence it produces,
and it drops the `.toString()` noise that concatenation forces on every non-string value.

```cadence
// ❌ Concatenation chain
panic("position ".concat(id.toString()).concat(" not found for ").concat(owner.toString()))

// ✅ String template
panic("position \(id) not found for \(owner)")
```

This applies everywhere a message is built —
`panic`, `log`, `pre`/`post` condition messages, and event field values.

## Bind optionals with `if let` / `guard let`; never force-unwrap

Force-unwrapping with `!` aborts the whole transaction when the value is `nil`,
at a line that says nothing about what went wrong.
Bind the optional instead and handle the `nil` case explicitly.

Use `if let` when the unwrapped value is only needed inside the branch:

```cadence
if let pool = self.pools[side] {
    total = total + pool.shares
}
```

Read through the binding, not write through it:
indexing a dictionary or array of structs yields a **copy**,
so `if let pool = self.pools[side] { pool.absorb(shares: shares) }` mutates the copy
and leaves the stored pool untouched — silently, with no error anywhere.
Mutation needs a reference into the container:
`let pool = &self.pools[side] as &Pool? ?? panic("…")`.

Use `guard let` when the rest of the function needs the value —
the binding stays available in the enclosing scope after the guard,
and its `else` block must exit (`return`, `break`, `continue`, or `panic`):

```cadence
// ❌ Force-unwrap
let position = self.positions[id]!
return position.funds

// ✅ Guard with a message
fun settle(id: UInt64): UFix128 {
    guard let position = self.positions[id] else {
        panic("position \(id) does not exist")
    }
    // position is available here, unwrapped
    return position.funds
}
```

`?? panic("…")` is the acceptable one-line form of the same idea
when there is nothing to do but fail with a message.

The shape the linter names `if-let-hint` is the worst of the three:
a `nil` check followed by a force-unwrap of the same optional.

```cadence
// ❌ Check and unwrap are two separate acts
let marketID = UInt64.fromString(parts[1])
if marketID != nil && PoolShares.isValidPoolName(parts[2]) {
    return PoolKey(marketID: marketID!, poolName: parts[2])
}

// ✅ Check and unwrap are the same act
if let marketID = UInt64.fromString(parts[1]) {
    return PoolKey(marketID: marketID, poolName: parts[2])
}
```

Nothing between them can then invalidate one but not the other.

## Use a ternary expression for a value that depends on a condition

Cadence has no if-*expression*, so an `if` statement that assigns the same variable
in both branches forces the variable to be `var` and separates it from its value.
The ternary keeps the declaration, the type, and both outcomes on one line.

```cadence
// ❌ `var` that exists only because `if` is a statement
var fee: UFix128 = 0.0
if side == Side.long {
    fee = self.longFee
} else {
    fee = self.shortFee
}

// ✅ Ternary
let fee = side == Side.long ? self.longFee : self.shortFee
```

Stop at one level.
Nested ternaries and ternaries whose branches are long expressions are worse
than the `if` statement they replace — use a `switch` or an early `return` there.

## Prefer `let` over `var`

Declare every field and local `let` unless something reassigns it.
`let` is a proof, checked by the compiler, that the value cannot change after it is bound —
which removes the reader's need to scan the rest of the scope for a write.
Reaching for `var` and never assigning again is the common form of this miss;
so is a `var` that exists only because the value is built in two steps
(bind the finished value instead).

## Replace magic numbers with named constants

A bare literal in an expression is a value with no name and no single place to change.
Give it a contract-level `let` field whose name says what it is,
and prefer a built-in constant when the language already has one —
`UInt128.max`, `UFix128.max`, `UFix64.max`.

```cadence
// ❌ Bare bound
if poolName.length == 0 || poolName.length > 32 {
    return false
}

// ✅ Named bound
access(all) let maxPoolNameLength: Int   // set in init()

// ...
if poolName.length == 0 || poolName.length > PoolShares.maxPoolNameLength {
    return false
}
```

The name is also what makes the bound reviewable:
`> 32` invites no questions, `> maxPoolNameLength` invites the right one.

## Keep operands side-effect-free

`&&` and `||` short-circuit: the right operand runs only when the left does not decide the
result, so anything in it that mutates state, emits an event, or aborts happens conditionally —
on a condition the reader has to derive from the operator rather than read.
Comparisons (`==`, `>=`, …) evaluate both sides, but they are no better a place for an effect:
the effect then hides inside something that looks like a question.

```cadence
// ❌ `isFresh` runs only when `consume` returned true
if self.consume(id: id) && self.isFresh(reading) {
    // ...
}

// ✅ Effects first, then the question
let consumed = self.consume(id: id)
let fresh = self.isFresh(reading)
if consumed && fresh {
    // ...
}
```

Marking the pure side `view` makes the rule mechanical:
a `view` function cannot have side effects, so a condition built from `view` calls is safe by
construction.

## Mark pure functions `view`

If a function does not modify state, declare it `view`.
It is not decoration:
`view` is what lets the function be called from other `view` functions,
from `pre`/`post` conditions, and from scripts that must not write —
and it is the compiler-checked form of "this is safe to call in a condition".
Initializers can be `view` too (`view init(...)`),
which is what keeps a struct constructible from inside a `view` function.
