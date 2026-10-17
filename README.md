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

## `rb_ob_entries.pine`: Rejection Block × Order Block Entries (Pine Script v6)

Short setup (long is the mirror image):
1. A strong bearish **displacement** candle (body ≥ 1.2× ATR) leaves an upper wick.
2. The wick, from the candle's **open to its high**, is the **rejection block**.
3. The wick must cover the **order block directly to its left**: the most recent bullish candle within 3 bars, with ≥ 70% of its body inside the wick.
4. **Entry:** limit at the rejection block **midpoint** when price retraces into it. The entry candle is painted bright yellow.
5. **Stop:** just above the rejection block high (2-tick buffer).
6. **Target:** the **next liquidity**, meaning the most recent swing low that formed after the displacement. The setup is skipped if reward:risk is below 2.

Drawn on the chart: the rejection block box with a dashed midpoint line, red stop and green target boxes once filled, an entry label, and a ✔/✖ result label when the trade closes. A table shows win/loss and net R.

Alerts:
- **Setup armed:** a new rejection block, with the limit and stop prices.
- **Entry filled.**

## `nq_trend_pullback_strategy.pine`: NQ Trend Pullback, Sweep & Reclaim (Pine Script v6 **strategy**)

A backtestable strategy built for a high win rate. Run it in TradingView's **Strategy Tester** on NQ1! or MNQ1!, 5-minute chart. Commission ($2.50/contract/side) and 1 tick of slippage are included.

**Long rules** (shorts are the mirror image):
1. **Trend:** the 1h 50 EMA is rising, price is above it, and price is above RTH VWAP.
2. **Flush:** the 2-period RSI dropped below 15 within the last 3 bars.
3. **Pullback:** price sweeps the 10-bar low or tags VWAP, and the candle closes back above it as a green candle. The signal candle is painted yellow.
4. **Stop:** signal candle low − 0.1 ATR. The trade is skipped if the stop is < 0.3 or > 1.5 ATR away.
5. **Exits:** half the position comes off at 1R. The runner's stop then moves to breakeven, and the runner targets 2R.
6. **Limits:**
   - Entries only 09:45–11:30 and 13:30–15:30 ET.
   - Max 3 trades/day, and no more trades after 2 losing trades.
   - Flat by 15:55.

Each leg (L1/L2) shows as its own trade in the Strategy Tester's list of trades.

## `first_candle_930_strategy.pine`: 9:30 First 5-Min Candle (Pine Script v6 **strategy**)

- **Direction:** after the 09:30–09:35 ET five-minute candle closes, go **long if it's green** and **short if it's red**. A doji means no trade. Entry is at the next bar's open.
- **Stop:** the far end of the candle's wick (the low for longs, the high for shorts), but at least **15 points** from entry.
- **Target:** the **next level of liquidity** in the trade direction. It's the nearest of: prior-day RTH high/low, overnight high/low, prior-week high/low, and swing highs/lows price hasn't taken out yet.
  - The level must be at least 1R away; otherwise the next level out is used.
  - With no level available, the target falls back to 2R.
- **Trade limits:** one trade per day, flat by 15:55 ET.
- **Bracket:** every trade is sent as a stop + target bracket, where one fill cancels the other. It's drawn on the chart as a red stop box and a green target box with a label showing stop, target and R:R.
- **Results table** (top-right): trades, wins/losses, **win rate**, profit factor, average win/loss and net P&L. The Strategy Tester tab shows the same numbers in full.
- **Chart:** works on 1–5 minute charts. The 9:30 candle is painted yellow.
- **Costs:** $2.50/contract/side commission and 1 tick of slippage are included.

## `first_candle_930_15m_strategy.pine`: 9:30 First 15-Min Candle (Pine Script v6 **strategy**)

The same strategy as above, using the **09:30–09:45 ET fifteen-minute candle**. Entry is at 09:45 on the 15-minute chart.

- **Unchanged:** the stop rule (far end of the wick, minimum 15 pts), the liquidity targets, the brackets and the results table.
- **Chart:** built for the 15-minute chart. It also runs on 1/3/5-minute charts, rebuilding the 15-minute candle from the smaller bars.
