# Absorption Candle Detector - Tuning Guide

## Overview

This indicator detects **absorption candles** - situations where aggressive buyers or sellers are absorbed by passive limit orders on the opposite side, creating a divergence between price direction and actual order flow (delta).

### What is Absorption?

| Type | Candle | Delta | Meaning |
|------|--------|-------|---------|
| **Bearish Absorption** | Red/Down | Positive (+) | Buyers were aggressive, but sellers absorbed all buying pressure |
| **Bullish Absorption** | Green/Up | Negative (-) | Sellers were aggressive, but buyers absorbed all selling pressure |

---

## Quick Start for RTY (Russell 2000 Futures)

### Recommended Settings for RTY 1-Minute Chart

```
Delta Settings:
├── Lower Timeframe: "1" (1-minute, same as chart)
├── Delta Mode: "Bar Delta"
└── Delta Threshold: 0

Absorption Filters:
├── Wick Multiplier: 1.5
├── Close Position Low: 0.25
├── Close Position High: 0.75
├── Require Wick OR ClosePos: "Either"
└── Min Body % of Range: 0.10

Volume Filter:
├── Volume SMA Length: 20
├── Volume Multiplier: 1.0
└── Enable Volume Filter: ✓
```

### Recommended Settings for RTY 5-Minute Chart

```
Delta Settings:
├── Lower Timeframe: "1" (1-minute sub-bars)
├── Delta Mode: "Bar Delta"
└── Delta Threshold: 0

Absorption Filters:
├── Wick Multiplier: 1.2  (slightly looser)
├── Close Position Low: 0.30
├── Close Position High: 0.70
├── Require Wick OR ClosePos: "Either"
└── Min Body % of Range: 0.10

Volume Filter:
├── Volume SMA Length: 20
├── Volume Multiplier: 1.2  (require above-avg volume)
└── Enable Volume Filter: ✓
```

---

## Parameter Guide

### Delta Settings

#### Lower Timeframe for Delta (`deltaLowerTF`)

This is the **most important parameter** for delta accuracy.

| Your Chart TF | Recommended Lower TF | Notes |
|---------------|---------------------|-------|
| 1 minute | `"1"` or `"15S"` | 1-min uses simple direction; 15S more granular if supported |
| 5 minute | `"1"` | 5 sub-bars per chart bar |
| 15 minute | `"1"` or `"5"` | 15 or 3 sub-bars per chart bar |
| 1 hour | `"5"` or `"15"` | 12 or 4 sub-bars per chart bar |
| Daily | `"60"` or `"240"` | 24 or 6 sub-bars per chart bar |

**TradingView Timeframe Codes:**
- Minutes: `"1"`, `"5"`, `"15"`, `"30"`, `"60"`, `"240"`
- Seconds: `"5S"`, `"15S"`, `"30S"` (limited exchange support)
- Daily: `"D"`, Weekly: `"W"`

**Note:** If the lower timeframe isn't available for your symbol/exchange, the script falls back to a simple heuristic (up bar = buy, down bar = sell). Check the debug table to see "Sub-bars" count - if it shows "(FALLBACK)", your lower TF isn't supported.

#### Delta Mode

| Mode | Description | Best For |
|------|-------------|----------|
| **Bar Delta** | Raw delta for the single bar | Precise, responsive signals |
| **CVD Slope** | Change in CVD over N bars | Smoother, trend-following signals |

For RTY scalping, use **Bar Delta**. For swing trading, consider **CVD Slope** with 3-5 bar length.

#### Delta Threshold

Minimum absolute delta to qualify as absorption. Set to 0 for any divergence, or increase to filter out weak signals.

---

### Absorption Filters

#### Wick Multiplier

The rejection wick must be at least this multiple of the body size.

| Value | Strictness | Use Case |
|-------|------------|----------|
| 1.0 | Loose | More signals, higher noise |
| 1.5 | Moderate | **Default - good balance** |
| 2.0 | Strict | Fewer, higher-quality signals |
| 3.0+ | Very Strict | Only extreme rejection wicks |

**For RTY:** Start with 1.5, adjust based on signal frequency.

#### Close Position Thresholds

Where the close must be within the bar's range:
- **Bearish absorption:** Close should be near the LOW (buyers pushed price up, but it closed low)
- **Bullish absorption:** Close should be near the HIGH (sellers pushed price down, but it closed high)

| Setting | Value | Meaning |
|---------|-------|---------|
| Close Position Low | 0.25 | Close in bottom 25% of range |
| Close Position High | 0.75 | Close in top 25% of range |

Tighter thresholds (0.15/0.85) = fewer signals, stricter quality.

#### Require Wick OR ClosePos

| Option | Behavior |
|--------|----------|
| **Either** | Wick OR close position qualifies (more signals) |
| Wick Only | Only large rejection wick qualifies |
| ClosePos Only | Only close position qualifies |
| Both | Wick AND close position required (strictest) |

**Recommendation:** Start with "Either" for RTY.

#### Min Body % of Range

Filters out dojis and spinning tops. A value of 0.10 means the body must be at least 10% of the total range.

---

### Volume Filter

#### Volume SMA Length

Period for calculating average volume. 20 is standard.

#### Volume Multiplier

| Value | Meaning |
|-------|---------|
| 0.5 | At least half of average volume |
| 1.0 | At least average volume |
| 1.5 | 50% above average |
| 2.0 | Double average volume |

**For RTY:** 1.0-1.2 works well. Higher values reduce noise but may miss valid signals.

---

## Interpreting Signals

### Bearish Absorption (Red Triangle Above Bar)

**What happened:** The bar closed red (bearish), but delta was positive (more buying volume).

**Interpretation:**
- Buyers were aggressive during this bar
- Sellers absorbed all buying pressure AND pushed price down
- Sellers demonstrated **strength** - they can absorb buyers and still close the bar lower
- Often appears at resistance levels or during downtrends

**Potential Action:** Look for short entries or continuation of downtrend.

### Bullish Absorption (Green Triangle Below Bar)

**What happened:** The bar closed green (bullish), but delta was negative (more selling volume).

**Interpretation:**
- Sellers were aggressive during this bar
- Buyers absorbed all selling pressure AND pushed price up
- Buyers demonstrated **strength** - they can absorb sellers and still close the bar higher
- Often appears at support levels or during uptrends

**Potential Action:** Look for long entries or continuation of uptrend.

---

## Debug Mode

Enable Debug Mode to see detailed condition breakdown:

1. **Info Table (top-right):** Shows current bar's metrics
2. **Label Details:** Each absorption label shows which conditions passed (✓) or failed (✗)

### Debug Table Fields

| Field | Description |
|-------|-------------|
| Delta Mode | Current delta calculation mode |
| Lower TF | Timeframe used for sub-bars |
| Sub-bars | Number of sub-bars found (FALLBACK if none) |
| Bar Delta | Calculated delta for current bar |
| CVD | Cumulative Volume Delta |
| Vol Ratio | Current volume ÷ average volume |
| Close Pos | Where close is in range (0-1) |
| Wick Ratio | Rejection wick size ÷ body size |

---

## Limitations & Important Notes

### Delta Approximation

**This indicator uses an APPROXIMATION of true delta volume.**

True delta (order flow) requires:
- Tick-by-tick trade data
- Aggressor side identification (market buy vs. market sell)
- Usually requires specialized data feeds (CQG, Rithmic, etc.)

Our approximation:
- Uses lower timeframe OHLC data
- Classifies sub-bar volume as buying (close > open) or selling (close < open)
- Accuracy depends on how many sub-bars we can fetch

### When Approximation Works Well

- High-liquidity markets (ES, NQ, RTY, CL)
- Lower timeframe data available (not in fallback mode)
- Trending markets where direction is clear

### When Approximation Struggles

- Low-liquidity markets or off-hours
- Fallback mode (no sub-bar data)
- Consolidating/ranging markets with small bars
- Very fast moves where all sub-bars go one direction

### TradingView Limitations

- Maximum 100,000 sub-bars per request
- Some symbols/exchanges don't support second-level data
- Historical second data limited on many feeds

---

## Tuning Workflow

### Step 1: Check Sub-bar Availability

1. Enable Debug Mode
2. Look at the info table - check "Sub-bars" field
3. If it says "(FALLBACK)", your Lower TF isn't supported
4. Try a higher Lower TF (e.g., "1" instead of "15S")

### Step 2: Calibrate Signal Frequency

Too many signals:
- Increase Wick Multiplier (1.5 → 2.0)
- Increase Volume Multiplier (1.0 → 1.5)
- Tighten Close Position thresholds (0.25 → 0.20)
- Change "Require Wick OR ClosePos" to "Both"

Too few signals:
- Decrease Wick Multiplier (1.5 → 1.2)
- Decrease Volume Multiplier (1.0 → 0.8)
- Loosen Close Position thresholds (0.25 → 0.30)
- Keep "Either" for structural condition

### Step 3: Validate on Historical Data

1. Scroll back through chart history
2. Check if absorption signals align with reversals or continuations
3. Note which parameter combinations work best for your instrument

### Step 4: Set Alerts

Once tuned, create alerts:
1. Right-click the indicator
2. Add Alert → Condition: "Absorption Candle Detector"
3. Select alert type (Bearish, Bullish, or Any Absorption)

---

## Example Configurations

### Aggressive Scalping (More Signals)

```
Wick Multiplier: 1.0
Close Position: 0.30 / 0.70
Volume Multiplier: 0.8
Require: Either
```

### Conservative Swing (Fewer, Higher Quality)

```
Wick Multiplier: 2.0
Close Position: 0.20 / 0.80
Volume Multiplier: 1.5
Require: Both
```

### High-Volatility Sessions (RTY Open)

```
Wick Multiplier: 1.5
Close Position: 0.25 / 0.75
Volume Multiplier: 1.5  (higher bar due to high volume)
Require: Either
```

---

## Combining with Other Analysis

Absorption signals work best when combined with:

1. **Key Levels:** Support/resistance, VWAP, POC
2. **Market Structure:** Higher highs/lows, trend direction
3. **Time of Day:** RTY most active 9:30-11:30 ET and 2:00-4:00 ET
4. **Multiple Timeframe:** Confirm on higher TF before entering on lower TF

**Example Setup:**
- 5-min chart shows bullish absorption at prior day's VAL (Value Area Low)
- 1-min chart confirms with second bullish absorption
- Enter long with stop below the absorption candle low
