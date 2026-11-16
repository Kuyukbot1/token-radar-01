[![make.fun](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Kuyukbot1/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Kuyukbot1/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01)

An entry for the [make.fun bounty](https://make.fun/bounty/token-radar-01).

# Token Radar — token-radar-01

The best token launch radar: a program that reads a sealed snapshot of every
protocol DefiLlama tracks, finds the ones that record no token and are not a
wrapped, staked or bridged product of a token that already trades, ranks them
by what a launch would be worth, and hands in a dashboard a reader can use.

## What it does

- `main.ts` (Node 22, standard library only) reads `/input/snapshot.json`,
  scores the board with the model in `lib/model.ts`, writes
  `/output/board.json` and `/output/data.js`, copies `dashboard/` and
  `screenshots/` to `/output`, and writes `/output/dossier.json` with
  `buildDossier` from `vendor/makefun-board.ts` (shipped unchanged from the kit).
- The board ranks **every** rankable pre-token protocol in the snapshot —
  the 40-protocol watchlist plus every protocol below its floor (and the few
  above it the list leaves out) that passes the same rule. Wrapped, staked
  and bridged products (WBTC, Binance staked ETH, USDT0, bridges,
  liquid-staking wrappers) are excluded before scoring, so a wrapper of a
  token that already trades can never top the board.
- The dashboard (`dashboard/`) is a static page with no network: a sortable,
  filterable board, a top-five spotlight, a detail view for every protocol
  with the model's line-by-line breakdown and a six-step farm plan, a
  live-token view set aside with market caps, and the full methodology
  including the category comparables.

## The model (radar-2)

One function scores every row, from the snapshot's figures alone. No
protocol name is special-cased.

```
category_multiple = median(mcap_usd / tvl_usd) over live-token protocols in
                    the same category (>= 4 peers), else the global median,
                    clamped to [0.05, 1.00]
implied = round( tvl_usd x category_multiple
                 x (1 + clamp(change_7d, -50, 100) / 100 x 0.60)
                 x (0.65 + min(age_days, 1095) / 1095 x 0.35)
                 x (1 + 0.02 x min(chains - 1, 8)) )
```

- **Category multiple:** a category's own market is its own comparable —
  DEX tokens trade at a different multiple of TVL than lending tokens do.
  The peers are the live-token protocols in the same snapshot.
- **Growth:** the seven-day TVL change, clamped so one hot week cannot
  dominate the board.
- **Maturity:** how long the protocol has run with no token on record
  (days since first listed; 365 when unlisted).
- **Reach:** a small premium for operating on more chains.

The farm plan for each protocol is derived from its own record (category,
chains, TVL, age): a fresh wallet, a category-specific core action with a
capital sized to TVL ($50–$250), a weekly qualifying action, a hold of
30–60 days, a points check that never pays to claim, and a farm log with an
exit rule.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

Then open `out/dashboard/index.html` in a browser. The page reads
`out/data.js` beside it; nothing is fetched.

## Honest labels

A protocol "records no token" when the snapshot carries no market cap,
CoinGecko id or symbol for it. That is DefiLlama's record, not proof that
no token exists. An implied value is this entry's own claim about what a
launch would be worth if the protocol's record stayed as it is — never a
price, and never a prediction of whether or when a token is issued.

## License

MIT — see `LICENSE`.
