# Liquidity Radar

A focused **TradingView Pine Script v6** indicator for trading liquidity. It maps the resting liquidity levels, rates their confluence, shows which level price is most likely heading to next, and marks the order blocks and fair value gaps around them. An optional **Al Brooks microtrendline module** handles execution and entries.

> Single file: [`LiquidityRadar.pine`](./LiquidityRadar.pine). Copy its contents into a new indicator in the TradingView Pine Editor and **Add to chart**.
> Raw copy link: https://raw.githubusercontent.com/jordanmartinek/liquidityindicator/main/LiquidityRadar.pine

> **Publishing?** [`DESCRIPTION.txt`](./DESCRIPTION.txt) is a ready-to-paste "About this indicator" guide (plain text pastes cleanly into the TradingView box).

---

## Features

Settings are organised into numbered groups.

| Group | What it does |
|---|---|
| ① Previous-Period Levels | PDH/PDL, PWH/PWL, PMH/PML, daily / weekly / monthly opens, NY midnight open. Removed once swept. |
| ② Sessions | Asia / London / NY highs and lows, daily reset, remove once swept, relabel unswept extremes as swings. |
| ③ Structure | Swings (+ HTF swings), equal and relative-equal highs/lows, **fair value gaps ranked 0–100** (size, volume, displacement, premium/discount + bonuses for OB overlap, inversion, HTF-gap alignment, round/VWAP/OTE confluence, liquidity pool, sweep precedence; discounts for tested / walled gaps), **SFP** sweep + reversal markers, **order blocks** (scored) and **breakers**. |
| ④ Countdown | Time (or bars) to reach each level from recent velocity. |
| ⑤ Clustering & Confluence | Nearby levels merge into BSL/SSL zones with a 0–100 confluence score. **Chart density** is the single declutter control (Minimal / Balanced / Everything / Custom). |
| ⑥ Level Confidence | Per-level confidence % (share of liquidity pull), 🎯 most-likely target, ⭐ stacked agreement. The draw-on-liquidity model behind it runs on fixed internal settings. |
| ⑦ Style | Dark / light panel theme, line + label style, colors, FVG opacity. |
| ⑧ Tested Levels | Probed-but-not-swept levels get ✓ + dotted line and pull slightly less. |
| ⑨ FVG Execution | Off by default. Arms when price returns into a qualifying gap; draws entry / stop / TP1 / TP2 + liquidity target with R:R gates. Bar-close commits. |
| ⑩ Daily Bias | Sticky structural bias from the last two daily candles + PDH/PDL sweep/reclaim. Can fade counter-bias FVGs and levels. |
| ⑪ Market Condition | One shared **TREND / RANGE / TRANSITION** classifier (efficiency ratio, bar overlap, VWAP crosses, EMA slope, recent microtrendline break). Feeds the FVG execution gate and the Brooks module. |
| ⑫ Opening Range (ORB) | Opening-window box with confirmed break (ATR buffer + N closes) and failed-break markers. |
| ⑬ Brooks Microtrendlines | **Execution + entries** (toggle). Microtrendlines + microchannels, break vs failed breakout, setups A–D (failed breakout, pullback continuation, failed-breakout reversal, High 2 / Low 2), H1/H2/L1/L2, signal-bar quality, 0–100 confluence score, pending → triggered → stop / target / invalidated tracking, dashboard, signal statistics. |

## Alerts

`PD level touched`, `Liquidity swept`, `Session level touched`, `SFP fired / long / short`, `FVG setup approaching / armed / confirmed / invalidated`, `Daily bias flipped`, `ORB break`, `ORB failed break`, `MT 1 … MT 16` (Brooks module), plus optional detailed `alert()` messages from the Brooks module.

## Notes

- Pine Script v6, `max_bars_back = 5000`.
- **No repaint by default:** SFPs, FVG setup states, Brooks signals and Daily Bias commit on bar close; higher-timeframe context uses the last closed HTF bar.
- Scores and percentages are heuristic ratings, not probabilities. Brooks statistics are signal statistics, not a strategy backtest.

*Not financial advice. For research and educational use.*
