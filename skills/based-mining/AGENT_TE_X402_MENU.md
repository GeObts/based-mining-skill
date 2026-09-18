# TE x402 approve / deny menu

This is the BASED **TE** (EVM / AI-agent) mining-pool menu: what an agent may
call today, what is only a name on the roadmap, and what this skill must never
teach.

**Docs only.** This file does not ship handlers, deploy anything, or make an
endpoint live. The companion `bankr-mining` repo is where handlers would land.
Until that repo ships a name **and**
[`https://basedmining.xyz/.well-known/x402`](https://basedmining.xyz/.well-known/x402)
lists it `live` (or a probe returns HTTP 402), treat the name as not callable.

Authority, in order:

1. Live discovery: `https://basedmining.xyz/.well-known/x402` (alias `/x402`).
2. An unpaid probe of that exact Bankr resource URL: **402** means a priced
   challenge exists; **404** `{"error":"Endpoint not found"}` means closed —
   do not pay.
3. Call instructions in [`SKILL.md`](SKILL.md), only for names that file
   already documents.

Never invent a path, method, price, body, or query param for a name that is
not `live` in discovery. A 404 is not a glitch and not a reason to try another
host.

## How to read `status`

TE planning buckets:

| Status | Meaning for this skill |
| --- | --- |
| `live` | In discovery as `live`. An unpaid probe returns 402. Callable only with the call instructions in `SKILL.md` (or, for catalog-only live names, only the fields discovery actually publishes — do not invent extra behavior). |
| `build` | Intended for the TE push. **Not live.** Do not teach agents to call it until `bankr-mining` ships it **and** discovery lists it `live`. |
| `deferred` | Not in this TE push. Do not document a call flow. |
| `out-of-scope` | Deny. Do not document, clone, or teach. |

`paused` is **not** a planning bucket. It is the catalog's own status for
`party-slot`: the name is listed, the price is published, payments are **not**
accepted. Do not relabel it `live`. Probe before any pay; 404 → do not pay.

`free` / `paid`: every x402 resource in discovery is paid (USDC on Base).
`$0.01` is still paid. Pointing physical hardware at stratum is free and is
**not** an x402 endpoint — see `SKILL.md`. Unpublished prices are `—`; do not
guess.

Catalog snapshot used to fill the `live` / `paused` rows:
`https://basedmining.xyz/.well-known/x402` on 2026-09-18. Re-read discovery
before treating this table as current.

## Menu

| name | method | price | free/paid | status | why |
| --- | --- | --- | --- | --- | --- |
| `mine` | POST | $10.00 | paid | live | Anytime hashpower; one call is one $10 block. Live in discovery. |
| `megapot-ticket` | POST | $1.00 | paid | live | One on-chain Megapot ticket to the paying wallet. Live in discovery. |
| `party-slot` | POST | $10.00 | paid | paused | Block Party ticket. Catalogued at $10 but **paused** — probe-before-pay; HTTP 404 `Endpoint not found` → do not pay. |
| `pool-status` | GET | $0.01 | paid | live | Live pool stats. Live in discovery. |
| `quote` | GET | $0.01 | paid | live | Hashpower price / tier menu. Live in discovery. |
| `worker-status` | GET | $0.01 | paid | live | Per-miner round status / payout estimate. Live in discovery. |
| `block-odds` | GET | $0.01 | paid | live | Solo odds for a given TH/s and window. Live in discovery. |
| `hashprice-oracle` | GET | $0.01 | paid | live | Network-wide hashprice. Live in discovery. |
| `btc-basis` | GET | $0.01 | paid | live | cbBTC / WBTC basis vs BTC spot on Base. Live in discovery. |
| `coinbase-decode` | GET | $0.01 | paid | live | Pool identity from a block's coinbase tag. Live in discovery; this skill does not yet teach a call flow — do not invent one beyond discovery. |
| `party-status` | — | — | — | build | Intended party-window read. **Not live.** Do not call; 404 is not a price. Wait for `bankr-mining` + discovery. |
| `order-status` | — | — | — | build | Intended order-lifecycle read. **Not live.** Agents already poll `status_url` from a paid `mine` / `party-slot` receipt — that is not this endpoint. Wait for `bankr-mining` + discovery. |
| `webhook-subscribe` | — | — | — | build | Intended push subscription. **Not live.** Do not invent callbacks, secrets, or a POST body. Wait for `bankr-mining` + discovery. |
| `mrr-summary` | — | — | — | deferred | Revenue rollup. Not in this TE push. |
| `leaderboard` | — | — | — | deferred | Leaderboard **x402 resource**. Not in this TE push. Do not confuse with `leaderboard_url` on a `mine` receipt (a public `basedmining.xyz` page, not a paid endpoint). |
| `stratum-connect` | — | — | — | deferred | Paid/connect wrapper. Not in this TE push. Free stratum pointing is already documented in `SKILL.md` and is not this name. |
| `hashpower-nft-status` | — | — | — | deferred | NFT-tier status. Not in this TE push. |
| `block-found` alerts | — | — | — | deferred | Block-found push/alert product. Not in this TE push. |
| `agent-mining-board` | — | — | — | deferred | Agent mining board UI/API. Not in this TE push. |
| LF / Lightning wallet ops | — | — | — | out-of-scope | Deny. No Lightning wallet clone, no LF wallet endpoints. |
| `ask-human` | — | — | — | out-of-scope | Deny. Do not document or teach. |
| fortune / fun / LLM utilities | — | — | — | out-of-scope | Deny. No fortune, trivia, or other fun/LLM utility endpoints. |

## Approve

- Call **live** discovery names that `SKILL.md` already documents, at the
  pinned price, after validating the 402 challenge.
- For `party-slot`: probe unpaid `POST` first. Pay only on HTTP 402 after
  field-by-field challenge validation at exactly $10.00. See
  [Paused: do not pay yet](SKILL.md#paused-do-not-pay-yet).
- Show `leaderboard_url` / `status_url` / `hashrate_url` from a receipt only
  after the host allowlist in `SKILL.md`. Those URLs are not new x402 names.

## Deny

- Do not pay `party-slot` while the probe is 404 `Endpoint not found`.
- Do not call `party-status`, `order-status`, or `webhook-subscribe`. They are
  `build`. This skill must not teach a call until `bankr-mining` ships them
  and discovery lists them `live`.
- Do not invent method, price, or input for any `build`, `deferred`, or
  `out-of-scope` row (`—` means unpublished, not free).
- Do not treat a 404 on a `build` name as an alternate path into `party-slot`
  or `mine`.
- Do not document `ask-human`, fortune, Lightning / LF wallet ops, or other
  fun/LLM utilities as BASED x402 resources.

## Probe-before-pay (paused `party-slot`)

Unpaid `POST` to the catalog resource URL, no payment header:

| Probe result | Meaning | What you do |
| --- | --- | --- |
| HTTP 404 `{"error":"Endpoint not found"}` | Paused / closed gate | Stop. Do not pay. Do not retry with a payment. Do not try another path or host. |
| HTTP 402 Payment Required | Accepting payments | Validate the challenge against the pinned terms in `SKILL.md` at exactly `10000000` ($10.00). Preview, confirm, then pay. |

A 404 on `party-status` / `order-status` / `webhook-subscribe` is the same
closed gate for those names. It does **not** authorize a paid call anywhere.
