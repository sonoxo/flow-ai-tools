# DeFi Protocol Safety

Code-level rules for protocol contracts that price assets, integrate foreign token
contracts, or gate a live market.
These sit on top of the general Cadence security rules —
access control, capabilities, resource handling, reentrancy ordering, and denial of service
live in the `cadence-lang` skill's `security-best-practices.md`.

## Never use an AMM spot price as an oracle

A pool's spot price is a function of its reserves, and reserves are movable within a
transaction, so a spot price is an attacker-chosen input.
Price from a feed, not from a pool.
Do not trade on an AMM without a target obtained off-chain or from a feed,
and sanity-check whatever the feed says before acting on it:
bound the deviation from the previous reading, require a fresh timestamp and an unconsumed
sequence number, and refuse a zero price.

Build the feed so that a stalled reading is distinguishable from a repeated one:
take the timestamp from the block rather than from the writer,
take the sequence number from the contract rather than from the caller,
and let readers compose an age check with an identity check.

## Check your assumptions about what other contracts do and return

A conforming interface is not a promise about behaviour.
Another contract's `deposit` may take a fee, its `balance` may be computed,
its `withdraw` may return less than asked for, and any of it may change under a contract update.
Assert what you depend on rather than assuming it:
measure the balance before and after, compare the received vault's `balance` to the amount
requested, check the concrete type when the type matters.

## Provide an emergency stop

Any contract holding value or gating a live market needs a way to stop
without a contract update.
Make the switch a field the critical paths read in their `pre` blocks,
and put the authority to flip it behind an entitlement:

```cadence
access(all) entitlement Administrate

access(all) contract MyContract {

    access(self) var paused: Bool

    access(all)
    view fun isPaused(): Bool {
        return self.paused
    }

    access(all)
    resource Administrator {

        access(Administrate)
        fun setPaused(_ paused: Bool) {
            MyContract.paused = paused
        }
    }

    init() {
        self.paused = false
    }
}
```

Two things this must not become.
It must not be a back door: pausing may stop new risk from being taken,
but users must still be able to withdraw what is theirs,
or the switch is a freeze on other people's funds.
And it must not be a second authority to protect:
the `Administrator` is as sensitive as the minter, and belongs in the same place.
