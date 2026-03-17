# 1H Intraday Setups

A TradingView Pine Script v6 indicator built specifically for the **1-hour chart**. It identifies high-probability BUY and SELL setups using a 4-layer confluence system. Every signal comes with a precise entry, stop loss, and three take-profit levels — no interpretation required.

Use this on the 1H chart to find the strongest intraday candles with the least noise.

---

## How It Works

The indicator scores each bar across 4 independent confluence layers. A signal fires only when at least 3 of the 4 layers agree, and only once per trading day.

### Layer 1 — Trend

Requires a full EMA stack alignment plus 4H confirmation:

- **EMA stack (bull):** EMA 9 > EMA 21 > EMA 50, and price is above EMA 50
- **EMA stack (bear):** EMA 9 < EMA 21 < EMA 50, and price is below EMA 50
- **4H bias:** Two conditions must both pass on the 4H chart:
  1. The EMA 50 must be sloping in the same direction (compared one full 4H period back — 4 bars on the 1H chart — to avoid a single indecisive 4H candle flipping the bias)
  2. The 4H close must be on the correct side of the 4H EMA 50 — slope alone is not enough if price has already crossed back through the EMA

The 4H is always used as the higher timeframe. If either 4H condition fails, the trend layer fails.

### Layer 2 — Momentum

- **Bull:** RSI(14) between 50 and 65 (above midline, not overbought) + MACD histogram positive
- **Bear:** RSI(14) between 35 and 50 (below midline, not oversold) + MACD histogram negative

### Layer 3 — Volume

- Volume must exceed 1.5× its 20-bar average (confirmed spike)
- Candle direction must align with the signal: bullish candle for longs, bearish candle for shorts

### Layer 4 — Entry Trigger

Two trigger types qualify. Either counts:

**EMA Pullback** — Price touched the EMA 21 within the last 4 bars and is now bouncing away from it with a conviction candle (body > 65% of bar range). The 0.5 ATR proximity zone accounts for the wider range of 1H candles overshooting the EMA before reversing.

**Structure Breakout/Bounce:**
- *Breakout:* Conviction candle closes above the last pivot high (for longs) or below the last pivot low (for shorts) — fires only on the first bar that crosses the level
- *Bounce:* Conviction candle touches within 0.75 ATR of the last pivot low (for longs) or pivot high (for shorts) and closes beyond it. The 0.75 ATR zone reflects the tendency of 1H wicks to probe deeper into support/resistance before rejection

---

## Signal Types

| Label | Meaning |
|---|---|
| **BUY** | 3/4 confluence layers agree — long setup |
| **STRONG BUY** | All 4 layers agree — highest quality long |
| **SELL** | 3/4 confluence layers agree — short setup |
| **STRONG SELL** | All 4 layers agree — highest quality short |
| **★** | Signal is within 0.5 ATR of a key daily level (PDH, PDL, or PDC) |

**One signal per day maximum.** If multiple setups qualify during a session, only the first one fires. This enforces quality over quantity.

---

## What Gets Drawn on the Chart

**Signal label** — appears below the bar (BUY) or above the bar (SELL). Large and unmissable. Hover over it for the full trade plan tooltip.

**Tooltip (on hover)** shows:
- Setup type (EMA Pullback / Structure Breakout / Multi-Confluence)
- Entry price
- Stop loss price and risk in points
- TP1, TP2, TP3 with their R:R ratios
- Suggested scaling: 40% at TP1, 35% at TP2, 25% at TP3
- Reminder to move SL to breakeven after TP1
- **Trailing stop suggestion:** trail distance in points (1× ATR) and as a percentage — activate once TP1 is reached to automatically protect remaining open position

**SL/TP lines** — drawn forward from the signal bar and extend as new bars print:
- White dotted — Entry
- Red dashed — Stop Loss
- Yellow — TP1
- Orange — TP2
- Lime — TP3

**EMA lines** — EMA 21 (blue) and EMA 50 (orange) plotted on the chart.

**Dashboard** (top-right corner):

| Row | What it shows |
|---|---|
| SIGNAL | Current signal (BUY / STRONG BUY / SELL / STRONG SELL / WAITING) |
| SCORE | Current best confluence score (e.g. ▲ 3 / 4) — shows RANGING when the EMA separation gate is blocking signals |
| TREND | EMA stack direction (BULL / BEAR / FLAT) |
| STRENGTH | ADX value with pass/fail indicator |
| KEY LVL | Whether price is near a previous day level |

---

## Stop Loss Calculation

The SL is placed at the more conservative of two calculations:

1. The 10-bar structure extreme (lowest low for longs, highest high for shorts)
2. Entry price ± 1.5× ATR

An additional 0.1 ATR buffer is added beyond whichever level is more conservative. This prevents SL from being clipped by normal volatility.

---

## Take Profit Calculation

Risk (R) = Entry price − Stop loss price

| Level | Default R:R | Description |
|---|---|---|
| TP1 | 1R | First partial exit |
| TP2 | 2R | Second partial exit |
| TP3 | 3R | Full runner target |

Suggested scaling: take 40% off at TP1, 35% at TP2, hold 25% to TP3. After TP1 is hit, move the stop to breakeven.

---

## Settings Reference

### Trend
| Setting | Default | Description |
|---|---|---|
| Fast EMA | 9 | Short-term EMA period |
| Slow EMA | 21 | Mid-term EMA period (also used for pullback detection) |
| Trend EMA | 50 | Long-term EMA period |
| ADX Length | 14 | Period for ADX calculation |
| Min ADX | 20 | ADX threshold — when ADX ≥ this value signals require 3/4 layers; below it requires 4/4 |

### Momentum
| Setting | Default | Description |
|---|---|---|
| RSI Length | 14 | RSI period |
| RSI Long Max | 65 | Upper RSI bound for longs (above this = overbought, skip) |
| RSI Short Min | 35 | Lower RSI bound for shorts (below this = oversold, skip) |

### Volume
| Setting | Default | Description |
|---|---|---|
| Volume Spike (×avg) | 1.5 | Minimum volume multiple vs its moving average |
| Volume MA Length | 20 | Period for the volume moving average |

### Risk Management
| Setting | Default | Description |
|---|---|---|
| ATR Length | 14 | ATR period used for SL, TP, and proximity checks |
| SL ATR Multiplier | 1.5 | Minimum SL distance in ATR units |
| TP1 R:R | 1.0 | Risk:reward ratio for TP1 |
| TP2 R:R | 2.0 | Risk:reward ratio for TP2 |
| TP3 R:R | 3.0 | Risk:reward ratio for TP3 |

### Filters
| Setting | Default | Description |
|---|---|---|
| Pivot Lookback/Forward | 5 | Bars used to confirm pivot highs and lows |
| Enable Session Filter | Off | Restrict signals to a time window |
| Active Session | 09:30–16:00 | Exchange-local session hours (used only if filter is enabled) |

### Display
| Setting | Default | Description |
|---|---|---|
| Show EMAs (21 + 50) | On | Toggle EMA lines on the chart |
| Show SL/TP Lines | On | Toggle entry/SL/TP lines |
| Show Prev Day Levels | Off | Draw previous day H/L/C as step lines |
| Show Signal Dashboard | On | Toggle the top-right dashboard table |

---

## Alerts

Four alert conditions are available. Set them up via TradingView's Alert dialog:

| Alert | Fires when |
|---|---|
| BUY Signal | A BUY or STRONG BUY signal fires |
| SELL Signal | A SELL or STRONG SELL signal fires |
| BUY at Key Level | A BUY signal fires near a previous day level |
| SELL at Key Level | A SELL signal fires near a previous day level |

All alerts are transition-based — they fire once when the signal appears, not on every bar it is valid.

---

## Workflow

1. Load the indicator on the **1H chart**
2. Wait for the dashboard SIGNAL row to show BUY or SELL (not WAITING)
3. Check the SCORE row — 4/4 is stronger than 3/4
4. Check KEY LVL — a ★ means the setup aligns with a key daily level
5. Enter at the close of the signal bar at the price shown in the label
6. Place your stop at the SL level
7. Scale out: 40% at TP1, 35% at TP2, 25% at TP3
8. Move SL to breakeven once TP1 is reached
9. Activate a trailing stop (settings in the tooltip) to automatically protect the remaining position

**Note:** ADX controls the minimum confluence threshold — when ADX ✓ the indicator requires 3/4 layers; when ADX ✗ it requires 4/4. A 4/4 STRONG signal with ADX ✓ is the highest-conviction setup.

---

## Key Design Decisions

- **1H only** — all thresholds (body, ATR zones, HTF) are tuned specifically for hourly candles. Each 1H bar represents a full hour of institutional activity, producing cleaner signals with less noise than shorter timeframes.
- **4H as HTF** — the 4H EMA 50 provides directional bias via two checks: slope (compared one full 4H period back to avoid single-candle flips) and price side (the 4H close must be above/below the 4H EMA 50). Slope alone can be misleading if price has already crossed back through the EMA mid-consolidation.
- **Body 65%** — a 65%+ body on a 1H candle means the entire hour was directionally controlled. Doji and wick-heavy candles that pass a 50% threshold are excluded.
- **3/4 threshold (not 4/4)** — requiring all 4 layers simultaneously is too restrictive for intraday use. 3/4 maintains quality while producing actionable frequency.
- **1 signal per day** — suppresses follow-on signals within the same session. The first qualifying setup of the day on the 1H is typically the highest-probability one.
- **ADX dynamic threshold** — ADX gates the minimum required confluence score rather than being a binary pass/fail. In trending conditions (ADX ≥ threshold) a 3/4 score is sufficient; in weak-trend conditions (ADX < threshold) all 4 layers must agree. This avoids eliminating valid momentum moves while still raising the bar in low-conviction environments.
- **EMA separation gate** — when EMA 21 and EMA 50 are within 0.2 ATR of each other the market is ranging, regardless of what ADX reports (ADX can stay elevated from a prior trend while price consolidates). No signal fires until the EMAs separate enough to confirm a genuine directional move.
- **Structure breakout crossover guard** — the breakout trigger (`close[1] <= pivot`) ensures the signal fires only on the first bar that crosses a pivot, not on every bar thereafter.

---

## Recommended Assets

This indicator is designed for **forex major pairs**. The recommended starting point is **EURUSD**.

### Why forex majors

- **Clean trending behaviour** — majors trend well and respect EMA structures more reliably than indices or crypto, which whipsaw more aggressively
- **Consistent volume patterns** — volume spikes (1.5× average) are meaningful during London and New York sessions, which align naturally with the session filter
- **ATR-based levels work well** — major pairs have predictable volatility on the 1H, so 1.5× ATR stops and 1–3R targets are realistic without being too tight or too wide
- **4H EMA50 alignment is reliable** — institutional participants actively respect higher timeframe EMA levels on major pairs

### Why EURUSD specifically

- Highest liquidity of any forex pair = cleanest price action
- Strong institutional participation = volume spikes are meaningful, not noise
- Trends clearly during the London/New York overlap (the prime 1H window)
- ADX readings are reliable — ranging vs trending is easier to distinguish than on thinner instruments

### What to avoid

| Asset class | Why it underperforms |
|---|---|
| **Crypto** | 24/7 trading weakens the daily gate logic; extreme volatility makes ATR stops very wide; volume spikes are less institutionally driven |
| **Individual stocks** | Earnings, gaps, and thin liquidity produce false breakouts that bypass the confluence filters |
| **Indices (SPX, NQ)** | Tend to grind rather than trend cleanly on 1H; gap behaviour at open creates misleading structure breakouts |

### Recommended session filter setting

Enable the session filter and set it to **12:00–16:00 UTC** to isolate the London/New York overlap — the highest-liquidity window of the trading day and where the strongest 1H moves originate.
