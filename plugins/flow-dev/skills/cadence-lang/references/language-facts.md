# Cadence Facts That Break Habits From Other Languages

Cadence differs from the languages an author or a model is likely to be carrying habits from.
These are the differences that produce wrong code most often.
Check code against this list before assuming a construct behaves the way it would elsewhere.

## No `defer`

There is no post-hoc cleanup construct.
A resource that must be moved or destroyed on a failure path has to be handled
before the operation that can fail, not after it.
Order the body correctly instead.

## No if-expressions

`if` is a statement, so it cannot produce a value.
The ternary `cond ? a : b` is the expression form —
using an `if` to assign the same variable in both branches forces that variable to be `var`.

## No initial values in field declarations

Every field is initialized in `init`.
`access(all) let fee: UFix64 = 0.05` is not valid;
declare the field and assign it in the initializer.
This is also why a named constant lives as a `let` field assigned in `init`.

## All arithmetic is checked

Overflow and underflow abort the transaction; nothing wraps.
`UInt8.max + 1` aborts, and `0 - 1` on an unsigned type aborts.
A subtraction that could go negative needs a pre-condition, not a clamp afterwards.

## All numeric conversions are checked

An out-of-range conversion aborts: `UInt8(300)` does not truncate.
Every conversion is therefore an assertion about range,
and belongs where a message can be produced rather than deep in a call chain.

## An unannotated fixed-point literal is `UFix64` / `Fix64`

It never infers to a 128-bit type.
`let x = 0.0` is `UFix64` with 8 decimal places;
`let x: UFix128 = 0.0` is what gives you 24.

## Owning a value grants every method on it

Entitlements gate access *through references*, not ownership.
Whoever owns a struct or resource can call every method on it regardless of entitlements.
Never hand out ownership of a resource whose methods are the authority —
hand out a reference carrying exactly the entitlements the holder needs.

## Fields are assignable only inside their declaring type

`item.gated = 1` is a compile error from outside `item`'s own declaration,
whatever the field's access modifier and whether or not the value is owned.
Mutation from outside always goes through a method,
which is the thing that can be entitled — so design the method, not the field.

## Indexing a container of structs yields a copy

Mutating what `dict[key]` or `array[i]` handed you changes nothing in the container,
silently and with no error anywhere:

```cadence
// ❌ Mutates a copy; the stored pool is untouched
if let pool = self.pools[side] {
    pool.absorb(shares: shares)
}

// ✅ Reference into the container
let pool = &self.pools[side] as &Pool?
    ?? panic("absorb: no pool for side \(side)")
pool.absorb(shares: shares)
```

## Storage references and capabilities resolve like symbolic links

The value behind a path can be replaced between two resolutions,
so checking through one `borrow` and acting through another is a hole.
Borrow once, bind it, and work through that binding.

## A struct proves nothing — anyone can construct one

A struct passed as an argument is not evidence of anything:
its initializer is callable by any code that can name the type,
so a `struct OperatorProof { let operator: Address }` is forgeable by definition.
Authority in Cadence is a reference you hold, not a value you present.

```cadence
// ❌ Anyone can build a `proof` naming any address
access(all)
fun mint(proof: OperatorProof, amount: UFix128) { }

// ✅ The authority is the entitled reference the caller had to obtain
access(Mint)
fun mint(amount: UFix128) { }
```

Where a struct is something *the contract reports* rather than something a caller may
manufacture, restrict its initializer with `access(contract) view init` —
then the struct in hand really did come from the contract that vouches for it.

## Resources cannot be stored in fields as references

`&T` cannot be stored; a field that must outlive the current scope holds
`Capability<&T>` instead and borrows on each use.
