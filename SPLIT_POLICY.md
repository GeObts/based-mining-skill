# Split Policy

Every paid mining order on BASED splits three ways:

- **80% — hashpower rental.** Buys the hashrate the order is for.
- **10% — operator.**
- **10% — MINR buyback** to the rewards wallet.

## Buyback

**The buyback is not active yet.** It is gated behind a `TOKEN_NOT_LAUNCHED`
flag until MINR exists. Until then the 10% is accounted for, but no buy is
executed.

Once MINR launches, the buyback will execute from the BASED agent wallet after
rental confirmation. Bought-back MINR will go to the community rewards wallet.
It will **not be burned and not sent to a dead address** — it will be held and
deployed as community incentives, contest prizes, and rewards.

Both the buy transaction and the transfer to the rewards wallet will be logged
publicly.

## Scope

This is current policy, not an immutable promise. It can change.
