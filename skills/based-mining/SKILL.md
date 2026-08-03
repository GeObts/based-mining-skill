---
name: based-mining
description: >-
  Buy Bitcoin hashpower and Megapot lottery tickets through the BASED x402
  endpoints on Base. Use when the user says mine bitcoin, solo mining, based
  pool, buy hashpower, rent hashrate, block odds, block party, my mining
  payout, what would I earn if BASED hits a block, megapot, lottery ticket, or
  jackpot. Covers live pool stats, hashpower quotes, solo block odds, per-miner
  round status, placing $10 mining blocks, and buying $1 lottery tickets.
---

# BASED Mining

BASED is a solo Bitcoin mining pool. This skill lets an agent read pool data,
price hashpower, place mining orders, and buy Megapot lottery tickets, all paid
in USDC on Base through x402.

## What BASED is

BASED is a solo Bitcoin mining pool with a 0% pool fee. Blocks it mines carry
the `/BASED/` tag in the coinbase. Any SHA-256 miner can point at
`stratum+tcp://pool.basedmining.xyz:3333` and start hashing: no signup, no fee,
nothing to pay.

Solo means a block is won by one worker rather than shared proportionally
across everyone at all times. When a BASED worker solves a block, the coinbase
transaction of that block has two outputs:

- **`vout[0]` — 1 BTC to the finder.** This is written into the block itself.
  It pays the wallet whose worker solved the block directly. It does not route
  through the operator and it cannot be withheld. No trust required.
- **`vout[1]` — the rest of the 3.125 BTC subsidy, ~2.125 BTC, plus the
  block's transaction fees, to the operator pool wallet.** Also in the
  coinbase.

What the chain enforces stops there. **Distribution of that 2.125 BTC out to
miners by round-share contribution is operator-run, not chain-enforced.** Never
describe it as automatic or trustless, and never present a round-share estimate
as a payment the chain guarantees.

That enforcement claim is byte-level, on the constructed coinbase and on the
pool running unmodified Parasite Pool. BASED has not found a block yet.

Everything below this section is a paid call. This section is what an agent can
say for free.

## How payment works

Every endpoint below is an x402 resource. Shared payment terms, taken from the
live 402 challenge:

| Field | Value |
| --- | --- |
| x402 version | 2 |
| Scheme | `upto` |
| Network | `eip155:8453` (Base) |
| Asset | USDC `0x833589fcd6edb6e08f4c7c32d4f71b54bda02913` |
| `payTo` | `0x8AEE621035D93Deb3C0C1177fac252dC2dd501a0` |
| Facilitator | `https://api.bankr.bot/facilitator` |
| Max timeout | 60 seconds |

Prices are quoted in atomic USDC units at 6 decimals. `10000` is $0.01,
`1000000` is $1.00, `10000000` is $10.00.

Two rules that matter for how you call these:

1. **The paying wallet is the identity.** `mine` and `megapot-ticket` read the
   payer wallet from the payment itself. Do not ask the user for a wallet
   address to pass in, and do not send one. There is no wallet parameter.
2. **Failures settle $0.** Validation errors, upstream errors, and float limits
   return an error and charge nothing. Only a successful result settles the
   full price. A failed call is safe to report as free.

## Endpoints

Base URL for all six:

```
https://x402.bankr.bot/0xcea5239fdd392e40c2b766375c4de8c991941d87/<name>
```

| Endpoint | Method | Price | Use it when |
| --- | --- | --- | --- |
| `pool-status` | GET | $0.01 | User asks how BASED is doing right now |
| `quote` | GET | $0.01 | Before an order, to price what a block buys |
| `block-odds` | GET | $0.01 | User asks about odds of hitting a block |
| `worker-status` | GET | $0.01 | User asks about their own miner or payout |
| `mine` | POST | $10.00 | User is buying hashpower |
| `megapot-ticket` | POST | $1.00 | User is buying a lottery ticket |

Every example response below is a verbatim capture from one live call at one
moment: read every number in them as a point-in-time snapshot, never as a
typical or steady-state figure.

### pool-status

`GET /pool-status`, $0.01.

Live pool stats. No input.

Example response (`GET /pool-status`):

```json
{"pool_tag":"/BASED/","connect":"stratum+tcp://pool.basedmining.xyz:3333","hashrate":{"1m":491000000000000,"1h":813000000000000,"1d":10700000000000000,"unit":"H/s"},"worker_count":52,"user_count":20,"best_share":4244834753081,"block_count":0,"network_difficulty":126231507121868.2,"about":"Solo Bitcoin pool, hybrid payout, agents mine via x402."}
```

Returns `pool_tag`, `connect`, `hashrate`, `worker_count`, `user_count`,
`best_share`, `block_count`, `network_difficulty`, and `about`.

`hashrate` is a nested object, not a flat number: it carries `1m`, `1h`, `1d`,
and a `unit` key. Read the unit rather than assuming — in the capture above it
is `H/s`, so `1d` of `10700000000000000` is 10.7 PH/s.

Those hashrate figures are a snapshot, and the `1d` value in particular is
elevated by a recent event rather than being a steady-state figure. Quote the
pool's current hashrate as what it is right now, never as what the pool
normally runs.

`connect` is the live stratum string, and it is the authority for how a miner
points at BASED. `pool_tag` is the `/BASED/` marker that appears in the
coinbase of blocks the pool mines.

Call it as the opening move when someone asks about BASED generally, or to
ground a mining pitch in current numbers before quoting.

### quote

`GET /quote`, $0.01.

Prices hashpower. It has two modes, and **they return different field sets.**
Know which one you called before you read the response.

- **Menu mode** — omit `amount_usdc`. Returns the tier table.
- **Priced mode** — pass `amount_usdc`. Prices that one amount, and returns a
  split breakdown and an expiry that menu mode does not have.

**The $10 block is the unit of purchase.** `mine` is fixed at $10, so the $25
and $50 tier rows are not something an agent can buy in one call — a $50
decision is five `mine` calls.

Those two are not the same purchase, and this is the thing to get right in this
section:

- **A tier row prices one single order of that size.** You rent one rig, and a
  rig has a fixed hashrate, so a bigger amount buys *more hours* at roughly the
  same TH/s. That is why all three tier rows below read 122 TH/s and differ
  only in `duration_hours`.
- **Five $10 blocks are five separate concurrent rentals**, all pointed at the
  same worker name for the paying wallet. That is roughly **5× the hashrate**
  for the ~37 hour duration one block buys — not one rental running five times
  as long.

Both are true of their own path. The total work bought is nearly identical
either way; what differs is the shape — one rig for a long time, or five rigs
at once. When a user stacks blocks through `mine`, describe it as more
hashrate, not more hours.

Treat the tier table as indicative pricing, not a fixed rate card.

#### Menu mode

Example response (`GET /quote`, no parameters):

```json
{"product":"based_hashpower_menu","tiers":[{"amount_usdc":10,"hashrate_ths":122,"duration_hours":37,"price_usd_per_th_day":0.0532,"summary":"$10 → ~122 TH/s for 37 hours"},{"amount_usdc":25,"hashrate_ths":122,"duration_hours":93,"price_usd_per_th_day":0.0529,"summary":"$25 → ~122 TH/s for 93 hours"},{"amount_usdc":50,"hashrate_ths":122,"duration_hours":187,"price_usd_per_th_day":0.0526,"summary":"$50 → ~122 TH/s for 187 hours"}],"btc_usd":63748,"split_policy":"80/10/10 — 80% hashpower, 10% operator, 10% MINR buyback to the rewards wallet","note":"BASED sells hashpower in $10 blocks — the $10 tier is one block; stack additional $10 orders for more hashpower."}
```

Returns `product`, `tiers` (each with `amount_usdc`, `hashrate_ths`,
`duration_hours`, `price_usd_per_th_day`, `summary`), `btc_usd`,
`split_policy`, and `note`. There is no `expires_at` here.

The `note` field is about the `mine` path, and it is correct: stacking $10
blocks does give more hashpower, because each block is its own rental. The
`tiers` above it price single orders instead. The two sit next to each other
and can read like a contradiction — they are not. Read the note as describing
`mine`, and the tier rows as describing one-shot orders of that size.

#### Priced mode

Example response (`GET /quote?amount_usdc=10`):

```json
{"amount_usdc":10,"hashrate_ths":122,"duration_hours":37,"price_usd_per_th_day":0.0532,"btc_usd":63802,"split":{"policy":"80/10/10 — 80% hashpower, 10% operator, 10% MINR buyback to the rewards wallet","hashpower_usdc":8,"operator_usdc":1,"buyback_usdc":1},"as_of":"2026-08-03T14:57:02.732242+00:00","expires_at":"2026-08-03T15:07:02.732242+00:00","human_summary":"$10 gets you ~122 TH/s for 37 hours on BASED right now (80/10/10 split, 10% MINR buyback to the rewards wallet)."}
```

Returns `amount_usdc`, `hashrate_ths`, `duration_hours`,
`price_usd_per_th_day`, `btc_usd`, `split`, `as_of`, `expires_at`, and
`human_summary`. There is no `tiers`, `product`, or `note` here.

The two modes describe the split differently. Menu mode gives a flat string in
`split_policy`. Priced mode gives an object in `split`, with `policy`,
`hashpower_usdc`, `operator_usdc`, and `buyback_usdc`, so the 80/10/10 arrives
as actual dollar amounts: $8 hashpower, $1 operator, $1 buyback on a $10 block.
Do not assume one shape and read the other.

`human_summary` is a ready-made sentence. Prefer it over composing your own.

#### Quotes expire

Treat `expires_at` as real. In the capture above the window was ten minutes
(`as_of` 14:57:02, `expires_at` 15:07:02). That is one observation, not a
guaranteed contract, so read `expires_at` off the response rather than assuming
ten minutes holds. If a quote is past its `expires_at`, requote before calling
`mine` instead of paying against a stale price.

A quote is a live market reading and it moves. `btc_usd` was 63748 in the menu
capture and 63802 in the priced capture roughly twenty minutes later — that is
what a live reading looks like. Requote if the user takes a while to decide.

### block-odds

`GET /block-odds`, $0.01.

Requires `hashrate_ths` and `duration_hours`. Returns
`probability_at_least_one_block`, `odds_one_in`, `expected_blocks`,
`expected_time_to_block_seconds`, `expected_time_to_block_human`,
`network_difficulty`, `inputs`, `framing`, and `jackpot` (which carries
`finder_reward_btc: 1` and its USD value).

Example response (`GET /block-odds?hashrate_ths=100&duration_hours=24`):

```json
{"inputs":{"hashrate_ths":100,"duration_hours":24},"network_difficulty":126231507121868.2,"probability_at_least_one_block":0.00001593612227235308,"odds_one_in":62750.02254782582,"expected_blocks":0.00001593624925374068,"expected_time_to_block_seconds":5421601948.132151,"expected_time_to_block_human":"172 years","framing":"At 100 TH/s you would expect ~1 block every 172 years. Over 24h your chance of finding at least one is 0.0016%.","jackpot":{"finder_reward_btc":1,"finder_reward_usd":63677,"note":"BASED is a solo pool — whoever's worker solves the block gets the 1 BTC finder bonus."}}
```

Use it after `quote` to turn TH/s into a probability the user can judge. The
endpoint's own `framing` string is already honest — pass it through rather than
softening it. State the odds plainly. Solo mining is a low probability, high
payout bet, and the reply should read that way.

### worker-status

`GET /worker-status`, $0.01.

Requires one of `evm_wallet` or `btc_address` as a query param. Passing neither
returns a 400 and settles $0.

Both address types resolve. A base58 BTC address is matched and mapped to its
EVM wallet, which comes back as `mapped_to_evm`, so a user who only knows their
BTC address gets the same answer as one who supplies an EVM wallet.

Identifiers below (the BTC address, the EVM wallet, the worker names) are
redacted. Every other value is a verbatim live capture.

Example response (`GET /worker-status?btc_address=3EXAMPLEaddressREDACTEDxxxxxxxxxxx`):

```json
{
  "key": "3EXAMPLEaddressREDACTEDxxxxxxxxxxx",
  "matched": true,
  "address": "3EXAMPLEaddressREDACTEDxxxxxxxxxxx",
  "evm_wallet": "0xEXAMPLE0000000000000000000000000000redact",
  "mapped_to_evm": "0xEXAMPLE0000000000000000000000000000redact",
  "group_total_diff": 69485696053,
  "hashrate": {
    "1m": 0,
    "1h": 0,
    "1d": 13900000000000,
    "unit": "H/s"
  },
  "accepted": {
    "round_diff": 31986096821,
    "share_count": 150434
  },
  "best_share": 1384596662938.696,
  "worker_count": 0,
  "workers": [
    {
      "workername": "3EXAMPLEaddressREDACTEDxxxxxxxxxxx.worker1",
      "hashrate_1m": 0,
      "hashrate_1hr": 0,
      "last_share": 1785682424
    },
    {
      "workername": "3EXAMPLEaddressREDACTEDxxxxxxxxxxx.worker2",
      "hashrate_1m": 0,
      "hashrate_1hr": 0,
      "last_share": 1785665506
    },
    {
      "workername": "3EXAMPLEaddressREDACTEDxxxxxxxxxxx.worker3",
      "hashrate_1m": 0,
      "hashrate_1hr": 0,
      "last_share": 1785516667
    }
  ],
  "pool_share_pct": 0.7513886643583234,
  "est_split_btc": 0.015967009117614374,
  "est_split_usd": 1016.7312395823304,
  "finder_bonus_btc": 1,
  "pool_total_diff": 4256930978366,
  "as_of": "2026-08-03T14:38:10.182910+00:00",
  "qualifier": "estimate based on share of round work so far",
  "reason": null
}
```

`worker_count` is the count of currently active workers, which is why it can be
`0` while `workers[]` still lists known workers — each with a `last_share`
timestamp showing when it was last seen. A zero `worker_count` alongside a
non-zero `hashrate.1d` means the miner was hashing earlier in the day and is
idle right now. Read it that way rather than reporting the user has no workers.

The estimate is a snapshot of the current round based on share of work so far.
It moves as shares accumulate. Always carry that qualifier into the reply.

Reply shape for a payout question:

> If BASED finds a block right now, your split is about X BTC (about $Y) on
> Z% of round work so far. If your own worker finds it, add 1.0 BTC on top.
> This is a snapshot of the current round and moves as shares accumulate.

## Placing a mining order

`POST /mine`, $10.00 per call. Live.

One call is one $10 block. The price is fixed at $10 and the input body is
empty. There is no amount parameter and no wallet parameter.

### The multi-block flow

More hashpower means more calls, not a bigger call. Each block is placed as its
own rental, and every rental for a given paying wallet is pointed at the same
worker name, so five blocks run concurrently as one worker at roughly five
times the hashrate — for the duration a single block buys, not five times
longer.

When a user asks to place a mining order:

1. **Ask how many $10 blocks they want.** Do not assume one.
2. **Confirm the total before paying.** Quote the dollar total, the hashrate
   and the duration, for example: "That is $50 for 5 blocks, around 610 TH/s
   for roughly 37 hours. Confirm and I will place it."
3. **Call `mine` that many times** once they confirm.
4. **Report the worker once, not five times.** The blocks land on the same
   worker, so surface one worker name, one BTC address, one status URL, and the
   total spent.

Get the per-block hashrate and duration for step 2 from `quote` in priced mode
at `amount_usdc=10`, not from memory. Multiply that hashrate by the number of
blocks; leave the duration as it is. Rates move.

A multi-block sequence takes real time and a quote can expire part way through
it. Check the quote once before starting the sequence, not before each call. If
the sequence runs long, report the actual total paid rather than the rate
originally quoted.

If a call in a multi-block sequence fails, the blocks that already succeeded
are placed and paid, and the failed one charged nothing. Tell the user exactly
how many blocks landed rather than reporting the whole order as failed.

### What `mine` returns

`order_id`, `status`, `worker_name`, `btc_address`, `amount_usdc`,
`status_url`, `hashrate_url`, `leaderboard_url`, `quote_summary`, and
`provisioning`.

This response shape is not from a live paid probe. It comes from the endpoint's
402 description plus a confirmed order placed on 30 July 2026. Treat the field
list as reliable in outline and verify against the first real response.

Surface `worker_name`, `btc_address`, and `status_url` to the user every time.

### Fulfillment language

Fulfillment is asynchronous. The order is placed against a live rental book, so
terms are quoted at order time and the rig is picked when the order is filled.
Quoted and delivered hashrate can differ slightly.

After a paid order, say this:

> Order placed. Funding and placing now. Hashrate is typically live within
> 30 minutes. Poll the status URL to watch it come online.

Do not say the worker is hashing until `status_url` reports it. The status
lifecycle runs through provisioning, awaiting_funding, placing, live, failed.

`hashrate_url` is the pool side live hashrate for that worker once mining
starts. It answers a different question from order status, so use `status_url`
for "is my order done" and `hashrate_url` for "how fast is it going".

## Buying a Megapot ticket

`POST /megapot-ticket`, $1.00 per call. Live.

One call buys exactly one ticket. The input body is empty. Sending
`ticket_count` or `quantity` set to anything other than 1 returns an error and
settles $0. For more tickets, make more calls, and confirm the total with the
user first the same way as mining blocks.

**The ticket goes to the paying agent's own wallet.** It is bought on-chain and
delivered to the wallet that paid, not held in custody by BASED. The user keeps
the ticket and any winnings. Say this plainly when offering it, because users
tend to assume it works the other way.

The referrer on every buy is the BASED treasury. That is how BASED earns on the
sale. It does not change the ticket, the odds, or who receives the winnings.

Returns `tx_hash` (Base transaction hash of the purchase), `ticket_count`
(always 1), `drawing_id` (the drawing the ticket is entered in), and
`recipient` (the wallet the ticket went to, which is the paying wallet).

Confirm with the transaction hash, the drawing id, and the recipient wallet.
The $1 settles only on a confirmed on-chain purchase.

## Pointing physical hardware

A user who already owns a miner does not need to buy anything. Bitaxe,
NerdMiner, and any other SHA-256 device that speaks stratum can point at BASED
directly. There is no payment and no x402 call involved in this.

| Setting | Value |
| --- | --- |
| Stratum URL | `stratum+tcp://pool.basedmining.xyz:3333` |
| Port | `3333` |
| Username | `BTC_ADDRESS.workername` |
| Password | `x` |

The BTC address in the username is where a found block's 1 BTC finder output
goes, so it must be the user's own address. The `workername` after the dot is a
free-form label the user picks to tell their devices apart, for example
`bc1qexample.bitaxe1`.

Once it is hashing, that same BTC address is what `worker-status` takes as
`btc_address`.

## Block Party

Block Party is a recurring event window BASED runs daily, from **22:20 to 06:00
UTC** (16:20 to 24:00 in America/Mexico_City, which is fixed UTC-6 with no
daylight saving).

This is awareness only. If a user mentions Block Party, or asks when it runs,
you can tell them the window. There is nothing in this skill to join or buy.

## Reply rules

- Give real numbers from the endpoints. Do not estimate hashrate, odds, or
  payouts from memory.
- Confirm the dollar total before any paid action, and say how many calls it
  will take.
- Solo mining odds are long. State them straight rather than selling them.
- Never claim hashpower is live before the status URL says so.
- Never claim a block payout is owed. Round estimates are estimates until a
  block is found.
- Never describe the per-miner round split as automatic or trustless. The
  coinbase split is chain-enforced, the distribution to individual miners is
  operator-run.
