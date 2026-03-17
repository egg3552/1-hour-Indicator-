# High Probability Intraday Setups

A TradingView Pine Script v6 indicator that identifies high-probability intraday BUY and SELL setups using a 4-layer confluence system. Every signal comes with a precise entry, stop loss, and three take-profit levels — no interpretation required.

---

## How It Works

The indicator scores each bar across 4 independent confluence layers. A signal fires only when at least 3 of the 4 layers agree, and only once per trading day.

### Layer 1 — Trend

Requires a full EMA stack alignment plus higher-timeframe confirmation:

- **EMA stack (bull):** EMA 9 > EMA 21 > EMA 50, and price is above EMA 50
- **EMA stack (bear):** EMA 9 < EMA 21 < EMA 50, and price is below EMA 50
- **HTF bias:** The EMA 50 on the next timeframe up must be sloping in the same direction

The higher timeframe is selected automatically based on the chart:

| Chart TF | HTF Used |
|----------|----------|
| 1m       | 5m       |
| 3m       | 15m      |
| 5m       | 15m      |
| 10m      | 30m      |
| 15m      | 1H       |
| 30m      | 1H       |
| 1H       | 4H       |
| Other    | Daily    |

### Layer 2 — Momentum

- **Bull:** RSI(14) between 50 and 65 (sweet spot — above midline, not overbought) + MACD histogram positive
- **Bear:** RSI(14) between 35 and 50 (sweet spot — below midline, not oversold) + MACD histogram negative

### Layer 3 — Volume

- Volume must exceed 1.2× its 20-bar average (confirmed spike)
- Candle direction must align with the signal: bullish candle for longs, bearish candle for shorts

### Layer 4 — Entry Trigger

Two trigger types qualify. Either counts:

**EMA Pullback** — Price touched the EMA 21 within the last 4 bars and is now bouncing away from it with a conviction candle (body > 50% of bar range).

**Structure Breakout/Bounce:**
- *Breakout:* Conviction candle closes above the last pivot high (for longs) or below the last pivot low (for shorts) — fires only on the first bar that crosses the level
- *Bounce:* Conviction candle touches within 0.5 ATR of the last pivot low (for longs) or pivot high (for shorts) and closes beyond it

---

## Signal Types

| Label        | Meaning                                    |
|--------------|--------------------------------------------|
| **BUY**      | 3/4 confluence layers agree — long setup   |
| **STRONG BUY** | All 4 layers agree — highest quality long |
| **SELL**     | 3/4 confluence layers agree — short setup  |
| **STRONG SELL** | All 4 layers agree — highest quality short |
| **★**        | Signal is within 0.5 ATR of a key daily level (PDH, PDL, or PDC) |

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

**SL/TP lines** — drawn forward from the signal bar and extend as new bars print:
- White dotted — Entry
- Red dashed — Stop Loss
- Yellow — TP1
- Orange — TP2
- Lime — TP3

**EMA lines** — EMA 21 (blue) and EMA 50 (orange) plotted on the chart.

**Dashboard** (top-right corner):

| Row      | What it shows                                      |
|----------|----------------------------------------------------|
| SIGNAL   | Current signal (BUY / STRONG BUY / SELL / STRONG SELL / WAITING) |
| SCORE    | Current best confluence score (e.g. ▲ 3 / 4)      |
| TREND    | EMA stack direction (BULL / BEAR / FLAT)           |
| STRENGTH | ADX value with pass/fail indicator                 |
| KEY LVL  | Whether price is near a previous day level         |

---

## Stop Loss Calculation

The SL is placed at the more conservative of two calculations:

1. The 10-bar structure extreme (lowest low for longs, highest high for shorts)
2. Entry price ± 1.5× ATR

An additional 0.1 ATR buffer is added beyond whichever level is more conservative. This prevents SL from being clipped by normal volatility.

---

## Take Profit Calculation

Risk (R) = Entry price − Stop loss price

| Level | Default R:R | Description        |
|-------|-------------|--------------------|
| TP1   | 1R          | First partial exit |
| TP2   | 2R          | Second partial exit |
| TP3   | 3R          | Full runner target  |

Suggested scaling: take 40% off at TP1, 35% at TP2, hold 25% to TP3. After TP1 is hit, move the stop to breakeven.

---

## Settings Reference

### Trend
| Setting | Default | Description |
|---------|---------|-------------|
| Fast EMA | 9 | Short-term EMA period |
| Slow EMA | 21 | Mid-term EMA period (also used for pullback detection) |
| Trend EMA | 50 | Long-term EMA period |
| ADX Length | 14 | Period for ADX calculation (displayed only, not a gate) |
| Min ADX | 20 | ADX threshold shown in dashboard ✓/✗ |

### Momentum
| Setting | Default | Description |
|---------|---------|-------------|
| RSI Length | 14 | RSI period |
| RSI Long Max | 65 | Upper RSI bound for longs (above this = overbought, skip) |
| RSI Short Min | 35 | Lower RSI bound for shorts (below this = oversold, skip) |

### Volume
| Setting | Default | Description |
|---------|---------|-------------|
| Volume Spike (×avg) | 1.2 | Minimum volume multiple vs its moving average |
| Volume MA Length | 20 | Period for the volume moving average |

### Risk Management
| Setting | Default | Description |
|---------|---------|-------------|
| ATR Length | 14 | ATR period used for SL, TP, and proximity checks |
| SL ATR Multiplier | 1.5 | Minimum SL distance in ATR units |
| TP1 R:R | 1.0 | Risk:reward ratio for TP1 |
| TP2 R:R | 2.0 | Risk:reward ratio for TP2 |
| TP3 R:R | 3.0 | Risk:reward ratio for TP3 |

### Filters
| Setting | Default | Description |
|---------|---------|-------------|
| Min Bars Between Signals | 4 | Cooldown bars after a signal fires |
| Pivot Lookback/Forward | 5 | Bars used to confirm pivot highs and lows |
| Enable Session Filter | Off | Restrict signals to a time window |
| Active Session | 09:30–16:00 | Exchange-local session hours (used only if filter is enabled) |

### Display
| Setting | Default | Description |
|---------|---------|-------------|
| Show EMAs (21 + 50) | On | Toggle EMA lines on the chart |
| Show SL/TP Lines | On | Toggle entry/SL/TP lines |
| Show Prev Day Levels | Off | Draw previous day H/L/C as step lines |
| Show Signal Dashboard | On | Toggle the top-right dashboard table |

---

## Alerts

Four alert conditions are available. Set them up via TradingView's Alert dialog (select the condition from the dropdown):

| Alert | Fires when |
|-------|-----------|
| BUY Signal | A BUY or STRONG BUY signal fires |
| SELL Signal | A SELL or STRONG SELL signal fires |
| BUY at Key Level | A BUY signal fires near a previous day level |
| SELL at Key Level | A SELL signal fires near a previous day level |

All alerts are transition-based — they fire once when the signal appears, not on every bar it is valid.

---

## Recommended Usage

**Best timeframes:** 1H (primary), 15m (secondary)

**Workflow:**
1. Wait for the dashboard SIGNAL row to show BUY or SELL (not WAITING)
2. Check the SCORE row — 4/4 is stronger than 3/4
3. Check KEY LVL — a ★ means the setup has additional confluence from daily structure
4. Enter at the close of the signal bar at the price shown in the label
5. Place your stop at the SL level
6. Scale out: 40% at TP1, 35% at TP2, 25% at TP3
7. Move SL to breakeven once TP1 is reached

**Note:** ADX is shown in the dashboard for context but does not gate signals. Use it as a secondary confirmation — an ADX ✓ alongside a 4/4 STRONG signal is the highest-conviction setup.

---

## Key Design Decisions

- **3/4 threshold (not 4/4)** — requiring all 4 layers simultaneously is too restrictive for intraday use. 3/4 maintains quality while producing actionable frequency.
- **1 signal per day** — suppresses lower-quality follow-on signals within the same session. The first qualifying setup of the day is typically the highest-probability one.
- **ADX not in gate** — ADX measures trend strength but is slow-responding. Including it in the gate was eliminating valid momentum moves. It is displayed as context only.
- **HTF bias via slope, not crossover** — the higher-timeframe EMA slope check is simple and fast, avoids look-ahead bias, and correctly captures trend direction without lag.
- **Structure breakout crossover guard** — the breakout trigger (`close[1] <= pivot`) ensures the signal fires only on the first bar that crosses a pivot, not on every bar thereafter.
# 1-hour-Indicator-
