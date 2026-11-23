# Ridge Runner

[![make.fun · btc-race-24h-20261123-0511](https://make.fun/badge/btc-race-24h-20261123-0511.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)
[![rank](https://make.fun/badge/btc-race-24h-20261123-0511/{owner}/{repo}/rank.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)
[![reward](https://make.fun/badge/btc-race-24h-20261123-0511/{owner}/{repo}/reward.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)

A BTC trading bot for the make.fun race "Race a BTC trading bot live for 24 hours" (btc-race-24h-20261123-0511). It trades CBBTC/USDC on 1-minute bars, long only, starting from 1,000 USDC of paper money.

## Strategy

Volatility-scaled dual-EMA trend riding, implemented in pure integer arithmetic:

- **Entry:** buy the full cash balance when the fast EMA(60) of closes sits above the slow EMA(240) by at least 40 bps and the latest close is above the slow EMA, after at least 240 bars of history. The slow filter and the 40 bps spread keep the bot out of chop smaller than the ~30-35 bps a full-size round trip costs in fees, slippage and price impact, and out of the market entirely while the trend points down.
- **Exit (first rule that fires):** an ATR trailing stop (close below the highest close since entry by max(200 bps, 2x ATR(14))); or a hard stop 250 bps below the entry fill. There is deliberately no exit on an EMA cross back — a slow EMA flip lags the price by hours, while the trailing stop already bounds every decline.
- **Cooldown:** 30 bars after an exit before the next entry, to avoid re-entering the same whipsaw.
- **Determinism:** every price is a bigint and every indicator is fixed-point integer math (scaled by 1e6). No floats, no clock, no environment, no randomness — the same bars always produce the same orders.

## Files

- `main.ts` — the entrypoint the grader runs (`node /bot/main.ts`).
- `vendor/makefun-bars.ts` — the kit's bars-replay SDK shim, copied byte-for-byte.
- `LICENSE` — MIT.

## Local grade

From inside the unpacked kit:

```sh
node cli/makefun.mjs grade-local --kit . --bot <path-to-this-repo> --driver process
```

Dev-window result (grade-local, process driver): net PnL **+$21.44** on 1 trade, max drawdown 2.0% — comfortably above the dev gate (net PnL > 0). The same logic stays flat ($0.00, no trades) on a mirrored down-trend twin of the window and loses only $2.60 on a detrended flat twin, so the worst case is one stopped-out round trip rather than a bleed.

## License

MIT — see `LICENSE`.
