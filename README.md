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
- **Fair Value Gaps** with selectable invalidation (body close through / wick touch / full fill) **and a strength filter** that scores every gap **0–100** for how likely price is to *respect* it, then only draws the ones that clear a threshold (weak gaps are dimmed or hidden). The score blends **gap size vs ATR**, the **relative volume / displacement** of the candle that created it, whether it sits **inside an order block or breaker block**, and whether it is an **inversion (IFVG)** of a prior opposite gap. This cuts the flood of trivial gaps on low timeframes (e.g. the 5-minute chart). A **Max FVGs kept** setting controls how many unfilled gaps are retained before the oldest is dropped — raise it to keep more / older gaps far back in history. The 0–100 figure is a **relative quality ranking, not a probability**, computed as a **normalized weighted average** of the factors (so it never saturates at 100 and always discriminates between strong gaps; raising a factor weight re-proportions its influence rather than inflating the total). A **poor-location (trapped) discount** cuts the rank when a gap is walled against opposing structure in its intended direction (a bullish gap with major resistance just above it, or a bearish gap with support just below), and SFP signals have a **per-level cooldown** so a wick repeatedly poking one level doesn't re-fire the signal every bar. Each kept gap can print its strength % with `inv` / `blk` tags, in a selectable **label style** (default: the % sits **centered inside the gap box itself**, so it's tied to its box and never blends into the level/cluster callouts at the right edge; also a left-edge pointer tag, plain borderless text, or the legacy right-edge callout). The strength score also **feeds the rest of the model**: it weights the gap's contribution to the confluence score, scales how hard the gap walls the **path-clearance / resistance** read on the DOL banner (a strong gap blocks harder than a trivial one), and gates whether a gap counts toward the high-conviction **stacked-agreement ⭐** flag.
  - **Min gap size (× ATR)** — a hard noise floor: gaps smaller than this many ATRs are ignored entirely (never scored, drawn, or tracked), killing micro-gaps at the source.
  - **CE (50%) line** — optional dashed line at each gap's midpoint (the *consequent encroachment* — the 50% level price often reacts to instead of the full fill).
  - **Higher-timeframe (HTF) FVG alignment** — detect gaps on the HTF timeframe (reuses the `HTF timeframe` input you already set for HTF swings — edit it to pick 60 / 240 / D). HTF gaps are **not drawn** on the chart; instead, a chart-TF gap that aligns with an HTF gap gets a large **score boost** (the single biggest accuracy signal — a gap sitting inside an HTF imbalance is defended by far heavier flow), and HTF gaps still feed the DOL model (confluence, path-clearance / resistance, ⭐). Aligned gaps are tagged `htf`. **This factor only applies on low timeframes** (default: chart TF ≤ 5 min, configurable). On higher chart timeframes the gap score reverts to the original full-weight model (plus the premium/discount, displacement, confluence and freshness factors), so HTF charts keep scoring gaps the way they always did.
  - **Expanded strength model** — the gap score now also factors in **premium/discount location** (a bullish gap in discount or bearish gap in premium scores higher), **displacement quality** (the creating candle's body dominance + whether it broke structure), **confluence** (midpoint near a round number / VWAP / OTE), a **freshness discount** (a gap already probed by price is treated as weaker), **liquidity-pool confluence** (the gap sits on or just beyond a tracked pool — equal highs/lows, previous-period levels, opens, sessions, swings, or volume-profile POC/VAH/VAL — graded by the most important nearby pool), and **sweep precedence** (the gap formed right after a same-direction liquidity sweep / SFP — the ICT sweep → displacement → FVG sequence). Every factor has its own weight slider, and drawn gaps are tagged (`htf` / `liq` / `swp` / `inv` / `blk`) to show what reinforced them.
- **Sweep + reversal (SFP)** markers: a wick pierces a tracked level then the candle closes back on the origin side (a stop-run), optionally volume-confirmed.
- **Order Blocks**: after a sweep + displacement, the last opposing candle is marked as a demand/supply zone until mitigated.

All swing / equal / previous-period levels are removed once price sweeps them.

### ④ Countdown
Per-level "time-to-reach" estimate based on price velocity (clock time or bars), optionally direction-aware.

### ⑤ Clustering & Confluence
Nearby levels merge into a single **strength-rated zone**, color-coded by side — **BSL** (buy-side, above price) vs **SSL** (sell-side, below price) — with tiered strength emojis and a min-strength filter. Each zone gets a **confluence score (0–100)** rolling up stacked levels + order-block overlap + **FVG overlap (weighted by the gap's own strength score)** + **OTE-fib proximity + premium/discount extreme** (computed from the live dealing range internally — not drawn on the chart) + round-number proximity, and the zone shades from its side color toward hot amber as the score climbs.

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

### ⑪ Daily Bias
A **sticky, structural daily bias** meant to be a directional *anchor* rather than a reactive arrow. It's built from **resolved higher-timeframe structure** — the last 1–2 completed **daily candles** (higher-high/higher-low vs lower-high/lower-low, plus the last daily's close position) combined with the most recent **PDH/PDL sweep + close-confirmed reclaim/reject** (PDL swept then reclaimed → bullish; PDH swept then rejected → bearish). Because it uses daily structure and confirmed sweeps (no intraday proximity, no momentum tilt), it **leads** rather than follows, and **holds** between real events (a new daily candle or a confirmed sweep) instead of flickering on 5-minute noise.
- **Fast flip** (default): a confirmed counter-direction PDH/PDL sweep flips the bias immediately. **Conservative:** a counter sweep only raises a *caution* flag; the bias flips only when daily structure itself breaks.
- Shown as a clean **"TODAY: LONG / SHORT / NEUTRAL"** banner in its own corner (default top-left, away from the DOL gauge), with a one-line *why*.
- **Bias-aligned declutter:** counter-bias FVGs are faded (or hidden), and optionally the counter-bias side's levels are faded — so the chart visually agrees with your one directional thesis for the day.
- The FVG execution engine can **require Daily Bias agreement** to arm (NEUTRAL = both directions allowed).

**Momentum-exhaustion dynamic weighting** (in group ③): a *live* layer that nudges each gap's effective score as momentum exhausts, driven primarily by **RSI divergence**. If price pushes to a new low while RSI makes a higher low (the down-push is tiring), **bullish gaps below price gain a rising bonus** (likely reversal/support) while bearish continuation gaps below are discounted — and the mirror for an exhausting up-push; overbought/oversold adds a secondary nudge. Unlike the static scoring factors (fixed at gap creation), this one **re-evaluates every bar**, so the on-chart % visibly breathes (⛽▲ / ⛽▽) and the execution engine's arming score responds live. (It does not alter the DOL gauge.) A **dedicated RSI gauge banner** (its own corner) displays pure RSI information — the live RSI value, its overbought/oversold/neutral state, and whether an RSI divergence is active — with no directional-bias framing (bias lives only in the Daily Bias banner).

### ⑩ FVG Execution (setups)
Turns high-quality gaps into actionable, *wait-for-the-setup* trade plans (off by default; this is a planning/signalling overlay — it never places orders). The engine stays quiet until price **retraces back into a qualifying gap**, then runs a small state machine: **watching → armed → confirmed**.
- **Qualification gates:** a gap can only arm if its strength ≥ a threshold, (optionally) it sits in the correct **premium/discount** half, and (optionally) its direction **agrees with the DOL bias** — so only A+, with-the-draw setups arm.
- **Arm:** when price taps the entry (the **CE/50%** by default, or the proximal edge), the setup arms and draws **entry · stop · TP1 · TP2** with the **risk:reward**. The stop sits beyond the far edge by an ATR buffer; targets are R-multiples.
- **Confirm:** optionally wait for a candle to **close rejecting back out** of the gap before flagging *confirmed*.
- Only the nearest *N* setups draw lines (keeps the chart clean); all qualifying gaps still fire alerts: **FVG setup approaching / armed / confirmed / invalidated**.
- **Lifecycle:** an armed/confirmed setup is marked **dead** (stops drawing, no more alerts) the moment price breaches its stop level, or after a configurable **expiry** in bars — so stale setups don't linger. The approach alert fires **once per gap** (no per-bar repeats).

---

## Alerts

`PD level touched`, `Liquidity swept`, `Session level touched`, `SFP fired`, `SFP long`, `SFP short`, `FVG setup approaching`, `FVG setup armed`, `FVG setup confirmed`, `FVG setup invalidated`, `Daily bias flipped`.

---

## Notes

- Written for **Pine Script v6**; `max_bars_back = 1000`.
- A **MASTER** toggle (in ⑤) turns all drawings on/off at once.
- Levels removed once swept keep the chart honest — only *live, untapped* liquidity is shown.
- **No-repaint by default:** SFP markers/alerts and FVG execution setup states (arm/confirm) commit on **bar close**, so they can't flash on the forming bar and then vanish. Each has an opt-in intrabar-preview toggle. The Daily Bias daily feed uses non-repainting `lookahead_off`.

*Not financial advice. For research and educational use.*
