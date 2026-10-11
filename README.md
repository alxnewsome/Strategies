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
