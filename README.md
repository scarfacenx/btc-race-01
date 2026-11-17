# BTC Dip Swing

[![make.fun · btc-race-24h-20261117-0603](https://make.fun/badge/btc-race-24h-20261117-0603.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![rank](https://make.fun/badge/btc-race-24h-20261117-0603/scarfacenx/btc-race-01/rank.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![reward](https://make.fun/badge/btc-race-24h-20261117-0603/scarfacenx/btc-race-01/reward.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)

A cbBTC trading bot for the make.fun "Race a BTC trading bot live for 24 hours" bounty. It trades Coinbase Wrapped BTC against USDC on 1-minute bars from the Orca Whirlpool, paper $1,000, long only.

## Strategy

Dip-buy / take-profit swing trading, full size, one position:

- Track the 90-bar SMA of closes.
- **Buy** all cash when a bar closes at least 0.8% below the SMA(90) — a dip in a range-bound market. The order fills at the next bar's open.
- **Sell** the whole position when its value reaches 1.0% over its cost (fees included).
- **Fallback:** if no trade has been sent by batch 1080 (hour 18), buy and hold to the end, so a quiet window still takes the drift instead of finishing with zero trades.
- Every order caps slippage at 100 bps past the close it was decided on.

## Why this shape

On the published 24h dev window BTC ranged $63.9k–$65.5k with 1-minute noise (sd ~6 bps) far below the ~35 bps full-size round-trip cost, so fast trend systems whipsaw to death (EMA 8/21: -$106 on dev). Wide dip/take bands trade only the few real swings — 4 trades on dev, max drawdown under 1%. Walk-forward (first half / second half, each run starting fresh) and neighbouring parameters (SMA 30–120, dip -0.6%..-1.0%, take +0.8%..+1.2%) all stay profitable, so this is not a knife-edge fit.

## Dev-window result (local replay, exact harness fill model)

- Net PnL: **+$20.86 USDC** (+2.09%)
- Trades: 5 (2 round trips won)
- Max drawdown: 0.7%
- Hourly Sharpe: 0.28
- Fees paid: ~$1.71

Docker is unavailable on the grading VM used for development, so the grade above comes from a faithful Python replay of the harness source (`sim/bars/engine.ts` fill model: next-bar open, 4 bps fee, 2 bps slippage, constant-product impact vs open depth, flatten at last close) driving the bot through the real `agent-fun/bars-replay/v1` protocol.

## Layout

- `main.py` — the entrypoint (Python 3.13, stdlib only). Run by the grader as `python3 -P -s /bot/main.py`.
- `vendor/makefun_bars.py` — byte-identical copy of the kit's Python SDK shim.

Deterministic: pure arithmetic over the bars in order, no randomness, no wall clock, no files, no environment. Two runs produce byte-identical replies.
