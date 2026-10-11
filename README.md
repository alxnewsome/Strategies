# Strategies

## `trendline_3touch_breakout.pine`: 3-Touch Trend Lines + Volume Breakouts (Pine Script v6)

Overlay indicator for TradingView. It finds trend lines automatically and gives entry signals.

### What counts as a valid trend line
- It is built from confirmed swing pivots: lows for an uptrend support line, highs for a downtrend resistance line.
- It needs **at least 3 touches**, meaning pivots within `Touch tolerance × ATR` of the line.
- **A false break does not invalidate the line.** A close past the line (by more than `Break buffer × ATR`) is fine if price closes back inside within `Max closes outside` bars. Wicks through the line never count against it.
- The line is invalidated by a **volume-confirmed break**, or by more than `Max closes outside` consecutive closes past it.

### Signals
| Marker | Side | Meaning |
|---|---|---|
| `TL` ▲ | Long | Price tags uptrend support and closes back above it (trend continuation) |
| `RC` ▲ | Long | Support reclaimed after a false break (trend continuation) |
| `BO` | Long | Close above downtrend resistance **with a volume spike** (`volume ≥ Volume spike × SMA(volume)`) |
| `TL` ▼ | Short | Price tags downtrend resistance and closes back below it |
| `RC` ▼ | Short | Price falls back below resistance after a false break |
| `BD` | Short | Close below uptrend support **with a volume spike** |
| grey × | none | Close past the line with no volume spike (unconfirmed, could be a false break) |
| grey ● "no vol" | none | Line invalidated because price stayed outside with no volume spike. No signal is given |

Alerts: *Any long/short entry*, *trend continuation* (long/short) and *volume breakout/breakdown*.
The `Long entry` / `Short entry` series (1/0) are exposed in the data window for use by other scripts.

### Notes
- Pivots are confirmed `Pivot right bars` after they print, so a line becomes active with that delay. Signals on the active line are real-time and do not repaint after the bar closes.
- Symbols with no volume data, such as some FX/index feeds, will never confirm a breakout.

## `nq_key_levels.pine`: NQ Key Levels, ranked by confluence (Pine Script v6)

Overlay indicator for day trading NQ / MNQ. It calculates every widely watched intraday reference level, gives each one a weight, and merges levels that sit within a few points of each other into one zone. A zone's score is the sum of its weights. Stronger zones are drawn heavier and get more stars. Only zones near price that reach `Minimum score to draw` (default 2) are shown, so lone minor levels only appear when they line up with something else.

| Group | Levels (weight) |
|---|---|
| Higher timeframe | Prior month H/L (4), prior week H/L (3), week open (2) |
| Prior day | RTH high/low (3), RTH close / gap fill (2), prior Globex H/L (1) |
| Overnight & opens | Overnight high/low (3), RTH open (2), Globex 18:00 open (1), midnight open (1) |
| Volume profile (RTH) | Prior POC (3), prior VAH/VAL (2), naked (untested) POCs (3), developing POC (2) |
| Initial balance | IB high/low (2), IB ±50% / ±100% extensions (1), opening range H/L (1) |
| Sessions | London H/L (2), Asia H/L (1) |
| VWAP | VWAP (2), ±2σ bands (1). Anchored at RTH open or Globex open |
| Volatility | VXN-implied 1σ daily move from prior close (2), ADR exhaustion targets (1) |
| Round numbers | 1000s (3), 500s (2), 100s (1) |

The table in the bottom-right lists the 3 nearest zones above and below price, with their distance in points.

**Use a 1–5 minute chart.** The volume profile is built from the chart's bars, so lower timeframes give a more accurate POC and value area. All times are New York.

## `rejection_block_signals.pine`: Rejection Block Confluence Signals (Pine Script v6)

Gives buy and sell signals when a **rejection block** lines up with another setup. The signal candle is painted **bright yellow (#FFFF00)**, and its label lists the confluences, e.g. `BUY OB + RB + GC`.

**Rejection block (RB):** the wick zone at a swing, measured from the lowest wick to the lowest candle body at a swing low (mirrored at a swing high). A *rejection candle* is one with a long wick (≥ 40% of its range and ≥ its body) that closes back in the trade direction.

| Signal | Fires when |
|---|---|
| **GC / DC + RB** | A rejection candle retests an RB, OB or FVG while a golden cross is in force (50/200 SMA, cross within the last 100 bars by default). A death cross is used for sells |
| **OB + RB** | A rejection candle taps an order block, so an RB forms inside the OB. OB = the last opposite candle at the extreme of the leg that broke the prior swing |
| **FVG → RB** | Price taps a fair value gap and rejects out of it, forming an RB |
| **RB retest** (optional, off by default) | A rejection candle retests a prior swing RB with no other confluence |

- Raise `Minimum confluences` to 2+ to only get signals where several setups stack.
- A zone is removed once price closes through its far side, or after it expires.
- By default each zone can trigger only one signal.
- Signals are final at bar close. Use the alerts, which fire once per bar close.

## `chase_buy_sell_add_enhanced.pine`: Chase's BUY/SELL + ADD, enhanced (Pine Script v6)

An improved version of Chase's indicator. It keeps the same defaults (SMA 70 cross, RSI 14 at 80/20, ADD in its own pane). Changes:

- **ADD confirms signals.** Buys need ADD above its 10-period MA; sells need it below. Other modes: also rising/falling, zero line, or off. Reversals need ADD turning. A reversal is graded `REV+` when ADD was beyond ±1000.
- **Bug fixes.**
  - The cross is checked on the correct bar. The original compared `close[1]` against the *current* SMA.
  - Buy and sell are symmetric. Sell used `low` while buy used `close`.
  - Down reversals need a red candle, matching up reversals.
- **Fewer false crosses.** There's an ATR buffer around the SMA, and the indicator only registers a new cross after price has closed clearly on the other side. The cross candle's high or low must then be broken within 3 bars (not only the next bar).
- **RTH-only filter** (on by default). ADD only exists 09:30–16:00 ET.
- **Layout.** Signals, the SMA and bar colors draw on the price chart via `force_overlay`. ADD stays in its pane with zero and ±extreme lines.
- **Scorecard.** Each signal is graded on whether it hit a 2×ATR target before a 1×ATR stop. Win rate is shown per signal type, along with the breakeven win rate, so you can tune settings with real feedback.
- **Optional filters.** SMA slope, and above-average volume.
