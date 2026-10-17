# 0.618 Day Trade: Break & Retrace

Script: `fib_618_daytrade_strategy.pine` (TradingView strategy, Pine v6)

## The idea in one sentence
Wait for price to **prove** a direction by breaking structure. Then buy (or sell) the **0.618 pullback** of the move that did the breaking, with your stop under the start of that move.

Why 0.618 works as a day-trade entry:
- A healthy trend rarely moves in a straight line. After a strong push, early buyers take profit and price pulls back.
- Shallow pullbacks (0.382) often go nowhere useful, and pullbacks past 0.786 usually mean the move is failing.
- 0.618 sits in between. It's deep enough to give you a good price and a tight stop, and shallow enough that the trend is still intact.
- So many traders watch it that it often becomes a reaction point by itself.

## The rules (long; short is the mirror image)

| Step | Rule |
|---|---|
| 1. Trend | 15-minute 21 EMA rising, price above it, **and** price above VWAP |
| 2. Break | A candle **closes** above the last swing high (a break of structure) |
| 3. Leg | 1.0 = the lowest low between that swing high and the break. 0 = the high of the move. The leg must be ≥ 1.5× ATR |
| 4. Entry | Buy limit at **0.618** |
| 5. Stop | A few ticks below **1.0** (risk = 0.382 of the leg) |
| 6. Target 1 | Half off at **0** (retest of the high), about **1.6R** |
| 7. Target 2 | Runner to the **−0.272 extension**, about **2.3R**. Its stop moves to breakeven after target 1 |
| 8. Cancel | Price closes below 1.0, or makes a new high after starting to pull back (you missed it), or 40 bars pass |

While the leg is still pushing up (before any pullback to 0.382), the 0 level keeps moving up with each new high, so the fib always measures the full move.

## What you see on the chart
- **One fib at a time.** Only the current setup's fib is drawn. It's deleted when the setup is broken (close beyond 1.0), expires, is missed (price makes a new high or low after pulling back), or the trade finishes. Turn on *Keep fibs of completed trades* to keep traded ones for review.
- **Higher-timeframe fibs** (15m in purple, 1h in blue by default). Each shows the **current** break-and-retrace leg on that timeframe, with its 0 / 0.5 / 0.618 / 0.786 / 1 levels labelled, e.g. `15m ▲ 0.618`. An HTF fib disappears when it's disrespected or completed, so what's on the chart is always live. When a 5m 0.618 lines up with a 15m or 1h 0.618, that's a stronger entry.

## How to use it day to day
1. **Chart:** 5-minute for setups. If you want tighter fills, run the same settings on a 3-minute or 2-minute chart. Keep the HTF filter at 15 minutes.
2. **Times (New York):** trades only from **9:45–11:30** and **13:30–15:45**. The first 15 minutes are too erratic, and lunch is too thin. Everything is flat by 15:55.
3. **Before the open:** know the prior-day high/low/close, the overnight high/low and where VWAP is. The script checks these automatically. When one sits within ¼ ATR of the 0.618, the entry label shows **★** and the level's name. **★ setups are your A+ trades.** If you only want those, turn on *Require a key level near the 0.618*.
4. **When the setup arms** (alert "Long setup"), the gold 0.618 line is your order. Let the limit sit; don't chase.
5. **Prefer the rejection-candle entry** (`Entry type`) when price is falling into the level fast with big candles. You'll miss a few trades but avoid catching the falling knives.
6. **After target 1 fills,** you're in a free trade. Leave the runner alone.
7. **Daily limits:** max 3 trades and stop after 2 losses. Both are built in. Respect them; the worst days come from forcing trades after losses.

## When NOT to use it
- **Choppy, sideways days.** Price keeps crossing VWAP and the 15m EMA is flat, so breaks of structure don't follow through. If VWAP is flat and price has crossed it 4+ times by 10:30, skip the day.
- **Into major news.** CPI, FOMC, NFP, big earnings for your stock, and for indexes the mega-cap earnings (NVDA, AAPL, MSFT…). Fibs mean nothing when the market jumps 50 points in a second. Stay flat 15 minutes before and after.
- **When price slices through 0.618 with huge candles.** That's not a pullback, it's a reversal starting. Use the rejection-candle entry or skip.
- **When the leg is tiny or made of one giant candle.** A one-candle spike has no structure behind it. The minimum leg size filter handles most of these.
- **Against the higher timeframe.** A 5-minute break of structure against a falling 15-minute trend is usually just a pullback in that downtrend. The trend filter blocks these; don't turn it off.
- **Third retest of the same level.** Each touch eats the orders waiting there. The first pullback after a break is the best; later ones get weaker.
- **When 0.618 is right below a major level** (like PDH, for a long whose leg broke just under it). The 0 target sits right into resistance, so the reward is likely smaller than it looks.
- **Last 30 minutes, holidays, half days.** Liquidity and behaviour change.
- **After 2 losses in a day.** Done for the day.

## Reading the results
Open the **Strategy Tester** (commission and slippage are included). Each half of a trade is a separate line in the list of trades, so "trades" counts legs.
- A **breakeven runner** shows as a tiny loss after commission. Judge the system by **profit factor** (aim for > 1.3) and **net P&L**, not by win rate alone.
- With target 1 at about 1.6R, the strategy breaks even at about **38%** win rate on the first half. A typical healthy result is 45–60% on the first half, with the runners adding the extra profit.

## Tuning, in order
1. `Min impulse leg` (1.5 ATR): raise it to 2.0 for fewer, cleaner legs.
2. `Swing left / right bars` (5 / 2): use larger values for bigger, more significant structure.
3. `Require a key level near the 0.618`: fewer trades, higher quality.
4. `Entry type → Rejection candle`: compare the win rate against the limit entry.
5. Windows: if one window loses consistently, turn it off.
