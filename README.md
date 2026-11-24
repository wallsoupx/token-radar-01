[![token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/c/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/wallsoupx/token-radar-01/rank.svg)](https://make.fun/c/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/wallsoupx/token-radar-01/reward.svg)](https://make.fun/c/token-radar-01)

# Token Radar

A token launch radar: it ranks pre-token protocols by implied value and shows a reader
how to farm each one. Built for the [token-radar-01](https://make.fun/c/token-radar-01)
bounty on [make.fun](https://make.fun).

## What it does

`main.ts` reads the sealed DefiLlama snapshot (`input/snapshot.json`, as of 2026-11-14),
scores every pre-token protocol with the radar-2 model, and writes:

- `board.json` — the ranked board (198 rows: all 40 watchlist protocols plus every other
  pre-token protocol that could plausibly launch a token of its own)
- `data.js` + `dashboard/` — a static, no-network dashboard: sortable and filterable board,
  a detail view per protocol with the model's own inputs and a farm plan
- `dossier.json` — the judge's summary, built with the vendored make.fun library

## The model (radar-2)

```
implied = round(tvl_usd × category_multiple
              × (1 + clamp(change_7d, -50, 150) / 100 × 0.6)
              × (0.7 + min(age_days, 1460) / 1460 × 0.5)
              × (1 + 0.05 × min(chains - 1, 4)))
```

- **category_multiple** — the median market-cap-to-TVL ratio of the live-token protocols
  in the same category (at least 4 of them, else the global median), clamped to 0.05–1.5.
  It is learned from the snapshot itself, not hard-coded.
- **growth** — the protocol's seven-day TVL change, clamped and weighted.
- **maturity** — how long it has run with no token (age unknown: 365 days).
- **breadth** — how many chains it runs on.

No protocol name is special-cased. Wrapped, staked and bridged products, exchanges and
versioned pools are kept out of the ranking — the same exclusion rule the watchlist uses.
Live-token protocols are set aside and shown with their market caps, never mixed into
the ranking. Every farm plan is derived from the protocol's category, chains, TVL and age,
with a cost and a time on each step.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

Requires Node.js 22.18 or later, standard library only, no network. The program is
deterministic: two runs write the same bytes.

## Layout

- `main.ts` — entrypoint: reads the snapshot, writes the board, data, dashboard, screenshots and dossier
- `lib/model.ts` — the radar-2 model, farm plans, snapshot parsing and board validation
- `vendor/makefun-board.ts` — make.fun's dossier library (vendored from the kit)
- `dashboard/` — the static page (`index.html`, `app.css`, `app.js`)
- `screenshots/` — board, detail and filtered views of the dashboard

## License

MIT — see [LICENSE](LICENSE).
