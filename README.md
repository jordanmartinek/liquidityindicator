# Liquidity Radar

A focused **TradingView Pine Script v6** indicator for trading liquidity. It maps the key resting liquidity levels, rates their confluence, shows which way price is being drawn, and marks the order blocks and fair value gaps around them — nothing more, so it stays fast and stable.

> Single file: [`LiquidityRadar.pine`](./LiquidityRadar.pine). Copy its contents into a new indicator in the TradingView Pine Editor and **Add to chart**.

> **Publishing?** A ready-to-paste **“About this indicator”** write-up is included — a full how-to-use guide, an explanation of every setting, how to read the DOL banner, and a complete Draw-on-Liquidity trading workflow. Copy it into the TradingView description box when you publish. Two formats: [`DESCRIPTION.md`](./DESCRIPTION.md) (Markdown) and [`DESCRIPTION.txt`](./DESCRIPTION.txt) (plain text — pastes cleanest into the TradingView box).

---

## Features

Settings are organized into numbered groups in the indicator's inputs.

### ① Previous-Period Levels
Previous day/week/month highs and lows (PDH/PDL, PWH/PWL, PMH/PML) and the daily open. Removed once swept.

### ② Sessions
Asian / London / New York session highs and lows, with a daily reset, sweep-removal, and reclassification of unswept session extremes into swing liquidity.

### ③ Structure
- **Swing highs/lows** kept as historical unswept liquidity, plus optional **higher-timeframe swings** for major structure far back.
- **Equal highs/lows** via rolling-buffer detection (catches non-consecutive equals).
- **Fair Value Gaps** with selectable invalidation (body close through / wick touch / full fill).
- **Sweep + reversal (SFP)** markers: a wick pierces a tracked level then the candle closes back on the origin side (a stop-run), optionally volume-confirmed.
- **Order Blocks**: after a sweep + displacement, the last opposing candle is marked as a demand/supply zone until mitigated.

All swing / equal / previous-period levels are removed once price sweeps them.

### ④ Countdown
Per-level "time-to-reach" estimate based on price velocity (clock time or bars), optionally direction-aware.

### ⑤ Clustering & Confluence
Nearby levels merge into a single **strength-rated zone**, color-coded by side — **BSL** (buy-side, above price) vs **SSL** (sell-side, below price) — with tiered strength emojis and a min-strength filter. Each zone gets a **confluence score (0–100)** rolling up stacked levels + order-block overlap + **OTE-fib proximity + premium/discount extreme** (computed from the live dealing range internally — not drawn on the chart) + round-number proximity, and the zone shades from its side color toward hot amber as the score climbs.

### ⑥ Draw-on-Liquidity Gauge
A corner banner showing the side price is most likely being drawn toward. Each level's pull is `distance-decay × type × age × freshness × confluence`, summed per side and blended with a momentum tilt. The banner shows direction, conviction %, a conviction meter, an optional **tug-of-war bar** (BSL vs SSL balance), and an optional primary-magnet **target** row (price + distance in ATR / %).

**Accuracy model:**
- **Confluence-weighted pull** — each level's pull is amplified by its own confluence score, so a level sitting in an OB + OTE + round number pulls far harder than a lone level.
- **Nearest-pool emphasis** — a sharpened distance decay so the closest untapped pool dominates (price hunts the nearest liquidity first).
- **Freshness premium** — untested levels pull harder; a level price is currently sitting on is discounted.
- **Equal highs/lows rank top** — engineered liquidity is the textbook draw target.
- **Balance-aware conviction** — a near-50/50 tug-of-war reads as low confidence even when the winning side's absolute pull is large.
- **Flip stability (deadband + hysteresis)** — the arrow no longer flips the instant one side edges out the other. The committed direction is held until the opposing side wins *decisively* (the bias clears an adjustable **deadband**); inside the band it keeps its prior direction. Optionally the flip is only committed **on bar close** (a single intrabar spike through a level can't turn it — a faint **⟳** hint shows a pending turn intrabar). This removes the random flip near a key level, which matters most on lower timeframes (e.g. the 5-minute chart). Set the deadband to 0 for the legacy instant-flip behavior.

**Per-level confidence:** every drawn level/zone shows a **0–100 % target-likelihood** (its share of total liquidity pull), and the single most-likely target is flagged **🎯** — so you can see at a glance which level price is most likely to head toward next.

### ⑦ Style
Theme (Refined Dark/Light, Midnight, Slate, Classic), line/label styling, FVG opacity, age-scaled line extension (older = longer), and a compact mode to hide far-from-price levels.

### ⑧ Emoji Tags
Configurable emoji tags per level class.

### ⑨ Tested Levels
Flags levels a wick has already **probed** into (reacted off) but not fully swept — "used" liquidity vs a virgin, untouched level. Tested levels get a **✓** and a dotted/dimmed outline; untouched ones stay solid. (Fully swept levels are removed already.) Optionally **discount tested levels in scoring** so a probed level is treated as a slightly weaker DOL magnet. Adjustable probe depth, discount amount, and max tracked.

---

## Alerts

`PD level touched`, `Liquidity swept`, `Session level touched`, `SFP fired`, `SFP long`, `SFP short`.

---

## Notes

- Written for **Pine Script v6**; `max_bars_back = 1000`.
- A **MASTER** toggle (in ⑤) turns all drawings on/off at once.
- Levels removed once swept keep the chart honest — only *live, untapped* liquidity is shown.

*Not financial advice. For research and educational use.*
