# based-mining-skill

An agent skill for **BASED**, a solo Bitcoin mining pool with a 0% pool fee.

[`skills/based-mining/SKILL.md`](skills/based-mining/SKILL.md) teaches an agent
to use the BASED x402 endpoints on Base: read live pool stats, price hashpower,
calculate solo block odds, check a miner's round status and payout estimate,
buy anytime hashpower in $10 blocks, buy Block Party tickets in $10 slots
(deployed on Bankr but paused — not accepting payments yet), and buy Megapot
lottery tickets. Everything is paid in USDC through x402, so an agent with a
funded wallet can do the live endpoints without a human in the loop.

The skill is a single portable Markdown file with YAML frontmatter. Drop it into
any agent that reads skills.

## What's in it

- Every endpoint with its price, live or paused status, parameters, and — where
  a live call exists — a verbatim example response captured from a real call
- The payout model: what the Bitcoin coinbase transaction enforces on a found
  block, and what it does not
- How to buy anytime hashpower, and what stacking $10 `mine` blocks actually does
- How Block Party tickets work via `party-slot` ($10 per call, stack for more
  power in the next scheduled window) and how to tell paused from live so an
  agent does not pay a closed gate
- How to point physical hardware — Bitaxe, NerdMiner, any SHA-256 miner that
  speaks stratum — at the pool for free

Identifiers in the `worker-status` example are redacted. Every other value in
every example is a verbatim live capture.

## Split

Paid orders split 80/10/10 — see [SPLIT_POLICY.md](SPLIT_POLICY.md).

## More

[basedmining.xyz](https://basedmining.xyz)
