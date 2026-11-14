# Token Radar — pre-token protocols ranked by implied value

A token launch radar for make.fun's `token-radar-01` bounty. It reads the sealed
DefiLlama snapshot (`/input/snapshot.json`), scores every rankable pre-token protocol,
and writes a ranked board, a static dashboard and the judge's dossier.

## The model (radar-2)

The board's implied value is priced by the snapshot's own market, not by one flat
multiple:

```
category multiple = median(mcap_usd / tvl_usd) over the live-token protocols of the
                    same category (at least 3), else the whole-market median,
                    clamped to 0.05x–1.5x
implied = round( tvl_usd x category multiple
                 x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
                 x (0.6 + min(age_days, 1095) / 1095 x 0.4)
                 x breadth )          breadth = 1 + 0.02 per extra chain, at most 1.20x
```

Age is the time since the protocol was first listed (365 days when unlisted). No
protocol name is special-cased. Live-token protocols are set aside with their market
caps, which are facts, not a model's claim.

The board ranks **every** rankable pre-token protocol in the snapshot — all 40
watchlist protocols plus the 157 rankable ones below the floor or left out of the
list — 197 rows in all. A protocol is rankable when it records no token and is not
a wrapped, staked or bridged product of one that already trades: the same rule the
watchlist uses, applied to the whole board, so WBTC, USDT0 and the bridges never
appear as launches still to come.

## The dashboard

`dashboard/` is a static page with no network: a sortable, filterable board
(search, category, chain, minimum TVL, watchlist-only), a detail view for each
protocol (model breakdown, live comparables in its category, farm plan), a
live-token view, and a model sandbox that re-ranks the board live under different
assumptions. Each row carries a seven-step farm plan with costs and times.

## Run it

```
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

Node 22, standard library only. Deterministic: two runs of the same snapshot write
the same bytes. Output: `board.json`, `data.js`, `dashboard/`, `screenshots/`,
`dossier.json`.

## Honesty note

A protocol on this board records no token in the snapshot; that is not proof it
has none. The board does not predict whether or when a token is issued, and its
implied value is this entry's own claim, never one make.fun makes.

## License

MIT — see `LICENSE`.
