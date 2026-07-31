# Cadence Security Best Practices

Protocol-specific rules for contracts that price assets or gate a live market —
oracle sourcing, foreign-contract assumptions, and emergency stops —
live in the `flow-defi` skill's `protocol-safety.md`.

## Access Control

### Practice 1: Default to Private
Start with `access(self)`, expand only when necessary. Only use `access(all)` for view functions, intentionally public APIs, and interface implementations.

### Practice 2: Protect Composite-Typed Fields
Resources, structs, and capabilities stored as fields MUST be `access(self)`:
```cadence
// ❌ CRITICAL: Anyone can copy this capability
access(all) let adminCapability: Capability<&Admin>

// ✅ CORRECT
access(self) let adminCapability: Capability<&Admin>
access(Admin) fun performAdminAction() {
    if let admin = self.adminCapability.borrow() { admin.doSomething() }
}
```

Widening is a promise that is hard to take back:
a contract update can add access, while removing it breaks every caller that had it.
This applies to fields as much as functions.
A public `var` field reads as a public write surface even though Cadence forbids the write —
a field is assignable only inside the type that declares it —
so prefer `access(self) var` plus a `view` accessor,
which puts the read path in public and leaves the write path internal by construction.

### Practice 3: Use Entitlements for Privileged Operations
Never use `access(all)` for state-modifying or privileged operations.

Authority in Cadence is a reference you hold, not an address you check.
A privileged operation belongs on a resource whose method is entitled,
so that holding a plain reference to that resource grants nothing:

```cadence
access(all) entitlement Update

access(all)
resource Updater {

    access(Update)
    fun update(price: UFix64) {
        // ...
    }
}
```

An accidentally published `Capability<&Updater>` is then inert,
where an `access(all) fun update` would have made it a public write path.

Two rules of the model that are easy to get wrong:

- **Owning a value bypasses entitlements entirely.**
  Entitlements gate access *through references*.
  Whoever owns the struct or resource can call every method on it regardless.
  So never hand out ownership of a resource whose methods are the authority — hand out a
  reference with exactly the entitlements the holder needs.
- **Fields can only be assigned inside the type that declares them.**
  `item.gated = 1` is a compile error from outside `item`'s own declaration,
  whatever the field's access modifier and whether or not the value is owned.
  Mutation therefore always goes through a method,
  which is the thing that can be entitled — design the method, not the field.

### Practice 3b: A Struct Proves Nothing — Anyone Can Construct One

A struct passed as an argument is not evidence of anything:
its initializer is callable by any code that can name the type,
so a `struct OperatorProof { let operator: Address }` is forgeable by definition.

```cadence
// ❌ Anyone can build a `proof` naming any address
access(all)
fun mint(proof: OperatorProof, amount: UFix128) { }

// ✅ The authority is the entitled reference the caller had to obtain
access(Mint)
fun mint(amount: UFix128) { }
```

Where a struct is something *the contract reports* rather than something a caller may
manufacture, restrict its initializer with `access(contract) view init`.
Then the struct in hand really did come from the contract that vouches for it.

### Practice 3c: Enforce Least Authority Across Component Boundaries

Each contract, resource, and transaction should hold the narrowest authority
that lets it do its job, and no more:
a reference with one entitlement rather than a resource,
one capability rather than an account reference,
a read view rather than a mutable reference.

Where a contract function would need `auth(AddContract)` or similar,
leave that call in the transaction, where the account access already lives,
and keep in the contract only the part worth centralizing and testing.

## Capability Security

### Practice 4: Issue Capabilities Sparingly
Check if capability already exists before issuing new ones.

### Practice 5: Publish with Verification
Check for existing published capabilities before publishing.

### Practice 6: Validate Before Borrowing
```cadence
// ✅ Handle optional
if let receiver = getAccount(address)
    .capabilities.borrow<&{FungibleToken.Receiver}>(/public/receiver) {
    receiver.deposit(from: <-vault)
} else {
    destroy vault
}

// ❌ Force unwrap
let receiver = cap.borrow()!  // Panics if invalid
```

### Practice 7: Never Expose Capabilities in Public Fields
Capabilities in public fields can be copied by anyone.

### Practice 8: Never Expose Capabilities in Public Arrays/Dictionaries
```cadence
// ❌ Anyone can access all capabilities
access(all) let capabilities: [Capability<&Admin>]

// ✅ Private storage
access(self) let capabilities: {Address: Capability<&Admin>}
```

## Reference Security

### Practice 9: Use Capabilities for Persistence (Not References)
References cannot be stored — use `Capability<&T>` instead of `&T` in fields.

### Practice 10: Minimize Entitlements on References
Grant only necessary entitlements when creating authorized references.

## Account Security

### Practice 11: Never Trust User Storage
Users control their own storage completely. Use capabilities with explicit types instead.

### Practice 12: Avoid Passing Authorized Account References
```cadence
// ❌ DANGEROUS
access(all) fun dangerous(account: auth(Storage) &Account) { }

// ✅ Pass specific capabilities
access(all) fun safe(vaultCap: Capability<auth(Withdraw) &Vault>) { }

// ✅ BEST: Perform operations in transaction prepare block
```

### Practice 13: Minimal Transaction Entitlements
```cadence
auth(BorrowValue) &Account                          // Read-only
auth(BorrowValue, SaveValue) &Account                // Read + write
auth(IssueStorageCapabilityController, PublishCapability) &Account              // Cap issuance + publish
```

## Type Safety

### Practice 14: Match Type Specificity to Intent

Use the most specific type your function actually requires. At open interface boundaries (standards implementations, marketplaces, vaults that hold multiple types), interface types are correct — pair them with an internal force-cast to verify the concrete type.

```cadence
// ✅ Correct for internal/single-type functions
access(all) fun processMyNFT(nft: @MyNFT) { }

// ✅ Correct for standards-conforming APIs (marketplace, collection deposit, FT receiver)
// Accept any conforming type, cast internally to verify
access(all) fun deposit(token: @{NonFungibleToken.NFT}) {
    let nft <- token as! @MyNFT  // Panics cleanly if wrong type — this is intended
    // ...
}

// ❌ Wrong: accepting an interface type without casting, then using it as concrete type
access(all) fun deposit(token: @{NonFungibleToken.NFT}) {
    // Using token as if it's @MyNFT without verifying — silent type confusion
}
```

**Rule:** Use `@ConcreteType` for functions that must only accept that exact type. Use `@{Interface}` at standard API boundaries that genuinely accept any conforming value, and always cast (`as!`) internally to your expected concrete type.

### Practice 15: Cast Less-Specific Types
Verify and cast to expected concrete types when receiving generic types. A failing cast panics cleanly — this is the intended security boundary, not a problem to avoid.

## Resource Safety

### Practice 16: Handle All Resources in Every Code Path
```cadence
// ✅ Both paths handle resource
if save {
    account.storage.save(<-vault, to: /storage/vault)
} else {
    destroy vault
}
```

### Practice 17: Resource Cleanup in Error Cases
```cadence
// ✅ Handle resource before panicking
if !someCondition() {
    destroy vault
    panic("Condition not met")
}
```

### Practice 17b: Always Handle Resources Before Panic-Prone Code
Cadence does NOT have `defer`. Explicitly move or destroy resources before any operation that could panic:
```cadence
fun process(vault: @{FungibleToken.Vault}) {
    let balance = vault.balance
    if balance != 0.0 {
        destroy vault
        panic("process: transfer incomplete, balance is \(balance)")
    }
    doWork()
    destroy vault
}
```

## Transaction Security

### Practice 18: Audit Transactions Like Contracts
Transactions can contain arbitrary code — review entitlements carefully.

### Practice 19: Users Should Understand Requested Entitlements
Verify what entitlements are requested, what resources are accessed, and where resources are moved.

## Events

### Practice 20: Emit Events for Significant Actions
```cadence
access(all) event TokensWithdrawn(amount: UFix64, from: Address?)
access(all) event TokensDeposited(amount: UFix64, to: Address?)
```

## State Changes and Foreign Calls

### Practice 21: Follow Checks — Effects — Interactions

Order every state-changing function: validate, then update your own state,
then call out to anything you do not own.
A call into foreign code — depositing to a `Receiver`, withdrawing through a `Provider`,
minting or burning through an interface someone else implements — can re-enter your contract,
and it must find your state already consistent when it does.
Resolving a capability with `borrow` runs no foreign code,
which is why it belongs with the checks rather than with the interactions.

```cadence
access(all)
fun payOut(
    amount: UFix64,
    from: auth(FungibleToken.Withdraw) &{FungibleToken.Provider},
    to: Capability<&{FungibleToken.Receiver}>
) {
    // 1. checks
    pre {
        !self.paused:
            "Payouts.payOut: payouts are paused"
        amount > 0.0:
            "Payouts.payOut: amount must be positive, got \(amount)"
    }
    let receiver = to.borrow()
        ?? panic("Payouts.payOut: receiver capability does not resolve")

    // 2. effects — our own accounting is settled before foreign code runs
    self.totalOut = self.totalOut + amount

    // 3. interactions — the call that hands over control comes last
    let payment <- from.withdraw(amount: amount)
    receiver.deposit(from: <-payment)
}
```

Cadence has no `defer`, so there is no "clean up afterwards" to fall back on:
if the interaction aborts, the whole transaction reverts,
and anything you were relying on running after it simply does not exist as a construct.
Get the ordering right instead.

### Practice 22: Check and Use the Same Value

Storage references and capabilities behave like symbolic links:
resolving the same path twice can yield two different values,
because the stored value can be replaced in between.
Anything you check and then use must be the *same* resolution —
borrow once, bind it, and work through that binding.

```cadence
// ❌ Two resolutions: the check and the withdrawal may see different vaults
if signer.storage.borrow<&{FungibleToken.Vault}>(from: path)!.balance >= amount {
    // ... code in between, which may run foreign code that swaps the stored value
    let vault = signer.storage.borrow<auth(FungibleToken.Withdraw) &{FungibleToken.Vault}>(from: path)!
    let payment <- vault.withdraw(amount: amount)
    // ...
}

// ✅ One resolution, bound and reused
let vault = signer.storage.borrow<auth(FungibleToken.Withdraw) &{FungibleToken.Vault}>(from: path)
    ?? panic("settle: no vault at \(path)")
assert(
    vault.balance >= amount,
    message: "settle: balance \(vault.balance) below \(amount)"
)
let payment <- vault.withdraw(amount: amount)
```

`signer` here is a transaction's `auth(BorrowValue) &Account`:
storage access needs an authorized account reference,
which is why this code belongs in a `prepare` block rather than in a contract function.

The same applies to a `Capability` argument:
`cap.check()` followed by a later `cap.borrow()` is two resolutions.
Borrow once and validate what you got.

### Practice 23: Never Make Arbitrary Calls From User Input

Do not resolve a caller-supplied address, path, or capability into a call target
in a function that holds authority — that turns your contract's authority into the caller's.
Where a capability must come from the caller, constrain its type,
borrow it once, and do nothing with it but the one narrow operation it is there for.

## Input Validation and Availability

### Practice 24: Validate Every Input at the Trust Boundary

Treat every argument that originates off-chain or from another account as hostile:
amounts (zero, negative, absurdly large), identifiers (unknown, already settled),
strings that become paths or identifiers, capabilities that do not resolve,
timestamps that run backwards.
Validate once, where the value enters, and then pass the validated value inward
rather than re-checking it at every level.

A string that will become a storage path or a contract name is the case most often missed:
`StoragePath(identifier:)` itself validates nothing,
so constrain the string at the boundary with a `view` predicate the whole contract shares.

### Practice 25: Keep Failure Paths Cheap and Loops Bounded

A transaction that can be made to run out of computation is a denial of service
against every user of the path it sits on.
Iterate over caller-supplied collections only with a bound you enforce,
and never make one user's operation walk a structure that another user can grow —
per-account state that grows without limit eventually makes its own owner's
transactions unexecutable.
Where an unbounded set must be processed, make it paginated or caller-chunked,
so the work per transaction stays bounded.

The same reasoning applies to abort conditions:
an `assert` that a hostile caller can trigger in a shared path
(a pooled settlement, a batch update) turns one bad entry into a stall for everyone.
Skip and record the bad entry there instead of aborting the batch.

## Security Checklist

- [ ] All fields use `access(self)` or `access(contract)` by default
- [ ] Privileged operations use entitlements
- [ ] No capabilities in public fields, arrays, or dictionaries
- [ ] Capabilities validated before borrowing
- [ ] Minimal entitlements requested in transactions
- [ ] Resources handled in all code paths
- [ ] Types are as specific as possible
- [ ] User storage not trusted without validation
- [ ] Events emitted for significant actions
- [ ] State-changing functions ordered checks → effects → interactions
- [ ] Every check and its use share one `borrow` — no path resolved twice
- [ ] No call target derived from caller-supplied addresses, paths, or capabilities
- [ ] Inputs validated once, at the boundary where they enter
- [ ] Caller-supplied collections iterated only under an enforced bound
