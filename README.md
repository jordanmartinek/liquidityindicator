# Liquidity Radar

A clean, feature-rich **TradingView Pine Script v6** indicator for trading liquidity. It maps resting liquidity (previous-period levels, session highs/lows, swings, equal highs/lows), clusters overlapping levels into strength-rated zones, estimates *where price is being drawn*, and detects the entry events that liquidity traders actually wait for — sweeps, order blocks, and structure breaks.

> Single file: [`LiquidityRadar.pine`](./LiquidityRadar.pine). Copy its contents into a new indicator in the TradingView Pine Editor and **Add to chart**.

---

## Features

Settings are organized into numbered groups in the indicator's inputs.

### ⓪ Appearance & Theme
A refined, muted palette drives every on-chart panel (DOL banner, dashboards, coach) so the UI looks clean rather than garish. Choose a **theme** (Refined Dark/Light, Midnight, Slate, or Classic), adjust **panel opacity**, **accent softness**, and **slim borders**. Also includes **⚡ Performance mode** — one switch that disables the heaviest modules (Candle Volume Profile, MTF Dashboard, SMT) for faster loading on low-end machines while keeping all on-chart drawing.

### ① Previous-Period Levels
PDH/PDL, PWH/PWL, PMH/PML and the daily open. Swept levels are removed automatically (toggleable).

### ② Sessions
Asian / London / New York session highs and lows, with a daily reset, sweep-removal, and reclassification of unswept session extremes into swing liquidity.

### ③ Structure / SMC
- **Fair Value Gaps** with selectable invalidation (body close through / wick touch / full fill).
- **Swing highs/lows** kept as historical unswept liquidity, plus optional **higher-timeframe swings** for major structure far back.
- **Equal highs/lows** via rolling-buffer detection (catches non-consecutive equals).
- **Premium / Discount** dealing range with OTE fibs (0.705 / 0.788 / 0.886), equilibrium, and range boundaries — anchored live to the nearest swing.

### ④ Countdown
Per-level "time-to-reach" estimate based on price velocity (clock time or bars), optionally direction-aware.

### ⑤ Clustering (anti-clutter)
Nearby levels merge into a single strength-rated zone. Color-coded by side — **BSL** (buy-side, above price) vs **SSL** (sell-side, below price) — with tiered strength emojis and a min-strength filter to cut noise.

### ⑥ Style & Compact Mode
Line/label styling, opacity controls, age-scaled line extension (older = longer), and a compact mode to hide far-from-price history.

### ⑦ Emoji Importance Tags
Configurable emoji tags per level class.

### ⑧ Draw-on-Liquidity Gauge — *Liquidity Gravity model*
A **fixed status banner** pinned to a chart corner (default **Top Right**) showing the side price is most likely being drawn toward. It never floats at the price line. The pull for each level is a multi-factor score:

```
pull = distance × type × age × confluence
```

blended with a **momentum tilt** (EMA trend + ROC). The banner shows the direction emoji, conviction %, a conviction meter, and a momentum-agreement flag; an optional second row shows the primary-magnet **target** price and its distance (ATR / %). The border **pulses green (up) / red (down)**. Position is selectable to any corner or middle side.

### ⑧b Candle Volume Profile
Reconstructs a real intrabar volume profile for the **last closed candle** using lower-timeframe data (`request.security_lower_tf`), drawn as a horizontal histogram with POC / VAH / VAL. Works on recent candles only (LTF data is limited); the intrabar timeframe must be lower than the chart timeframe.

### ⑨ Discipline Coach
A motivational panel with context-aware nudges (premium/discount location), a rotating quote, a discipline streak, and a custom mantra.

### ⑩ Liquidity Grab Engine
- **Sweep + reversal (SFP)** — the core entry trigger: a wick pierces a tracked level then the candle closes back on the origin side. Marks 🎯 SFP▲/▼.
- **Volume confirmation** — only flags grabs on elevated relative volume.
- **Rejection wick-strength grade** — grades each sweep by how hard price rejected the level (rejection wick as a share of the candle range), shown as ✦ / ✦✦ / ✦✦✦ on the marker, with an optional minimum-wick filter.
- **Order blocks** — after displacement following a grab, marks the last opposing candle as a demand/supply zone until mitigated.

### ⑪ Structure Breaks & Voids
- **BOS / CHoCH** markers (continuation break vs. first counter-trend break).
- **Liquidity voids** — runs of consecutive one-sided candles marked with a midpoint magnet that clears when rebalanced.

### ⑫ Killzones & Asian Range
London/NY killzone background shading (sweeps inside a killzone get a 🔥 emphasis) and an Asian-range box (the Judas-swing reference).

### ⑬ Heatmap · MTF · Ranking · R:R · Stats
- **Age heatmap** — old untapped levels glow brighter, fresh ones fade.
- **MTF confluence badge (🔱)** on clusters spanning higher- and lower-timeframe sources.
- **Top-3 draw ranking** table — strongest untapped magnets by gravity score.
- **Auto R:R projection** on SFPs — entry / stop / target lines with a reward:risk label.
- **Backtest stat panel** — rolling sweep → reversal hit rate.

### ⑭ Confluence & Context
- **MTF liquidity dashboard** — a compact table pinned to the middle-right showing, per timeframe (default 15m / 1H / 4H / D): nearest buy-side liquidity above, nearest sell-side below, directional bias **with a conviction %** (EMA separation vs recent range — read higher-timeframe conviction at a glance), and premium/discount location.
- **SMT divergence** — compares this symbol against a correlated one (e.g. ES vs NQ, BTC vs ETH). When one makes a higher high / lower low while the other fails to, it flags an SMT divergence (a strong reversal cue). Set the correlated symbol in the input.
- **Confluence score (0–100)** — rolls up everything overlapping a cluster zone (stacked levels, order block, liquidity void, OTE fib, premium/discount extreme, round number) into a single ⭐ score on the zone label, with an optional filter to hide low-score zones. Zones now also **heat-shade** from their side color toward hot amber as the score climbs.
- **Power-of-three daily bias** — classifies the day into Accumulation → Manipulation → Distribution around the daily open (first decisive sweep = the Judas leg; distribution is the opposite direction), tagged on the daily-open level.
- **Correlation / driver context** — compares this symbol against a driver (e.g. DXY, a lead instrument) and flags whether the driver **supports or conflicts** with your directional bias (invertible for inverse correlations).

### ⑮ Ghost Levels
When a level is **swept** (price trades through it), it leaves behind a faint **ghost** — a very light dotted line marking where liquidity was recently taken. Ghosts fade from the set opacity toward invisible over their lifetime, then disappear. Adjustable **opacity**, **lifetime (bars)**, **color**, and max kept; fully toggleable. Covers swept swing highs/lows, equal highs/lows, and previous-period levels.

---

## Alerts

`SFP fired`, `SFP long`, `SFP short`, `Order block entered`, `BOS`, `CHoCH`, `DOL flipped`, `SMT divergence`, plus previous-day / session / liquidity-sweep touches.

---

## Notes

- Written for **Pine Script v6**.
- A **MASTER** toggle (in ⑤) turns all drawings on/off at once; every element also has its own switch.
- Most heavy drawing happens on the last bar via pooled drawings that are cleared and rebuilt each pass, keeping within TradingView's line/label/box limits.
- The candle volume profile depends on lower-timeframe data availability and is best used on recent candles.

*Not financial advice. For research and educational use.*
