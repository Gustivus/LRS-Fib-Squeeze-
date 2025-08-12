# LRS Fib Squeeze+ Indicator

## Overview
The **LRS Fib Squeeze+** is a volatility compression and momentum expansion indicator that combines:

- **Linear Regression Slope (LRS)** for trend velocity
- **Adaptive volatility compression detection** for squeeze states
- **Momentum histogram with direction bias**
- **Variability bands (StdDev or MAD)**
- **Fibonacci-projected slope levels** for slope retracements

This indicator is designed to **identify moments when volatility contracts (“squeeze”)** and prepare for **explosive breakouts or breakdowns** when the compression ends.

---

## Methodology

### 1. Linear Regression Slope (LRS)
The **LRS** measures the slope of a least-squares regression line fitted to recent price data. This slope serves as a **trend velocity proxy** — positive slope means upward bias, negative slope means downward bias.

### 2. Normalized Slope Range
The rolling **highest** and **lowest** slopes in the lookback window define a slope range. This range is **normalized** by the ATR of price to make it comparable across instruments and volatility regimes.

### 3. Squeeze Detection
A squeeze state is detected when the **normalized slope range** is below a threshold:
- **Fixed Threshold:** Compares directly to a static level.
- **Adaptive (z-score):** Measures how many standard deviations below the mean the slope range is.
- **Adaptive (percentile):** Measures percentile rank within the recent window.

When the slope range is compressed enough, the indicator enters **“Squeeze On”** mode, often preceding expansion.

### 4. Momentum Direction & Histogram
A momentum proxy is computed from **slope - EMA(slope, 2)**.
- **Magnitude:** Strength of compression
- **Sign:** Direction of momentum
- The histogram shows this signed magnitude, scaled and optionally smoothed.

When the histogram crosses zero, momentum is flipping from positive to negative or vice versa — often an early warning of trend changes.

### 5. Variability Bands
Around the slope mean, variability bands (StdDev or MAD) are plotted. When the slope pushes outside these bands after a squeeze, it often signals sustained expansion.

### 6. Fibonacci Slope Levels
Fibonacci retracement levels (23.6%, 38.2%, 50%, 61.8%, 78.6%) are calculated on the slope range. These can serve as **reference zones** for slope pullbacks or acceleration points.

---

## Use Cases
- **Breakout Trading:** Identify pre-breakout compressions and expansion triggers.
- **Swing Entries:** Use strong expansion signals to time entries aligned with trend.
- **Reversal Detection:** Look for histogram inversions and band breaches as reversal clues.
- **Volatility Regime Shifts:** Anticipate changes in volatility structure.

### 1. Volatility Compression & Breakout Theory
Markets tend to **cycle between contraction and expansion**:
- **Compression:** Volatility dries up, ranges get smaller, order books tighten. This often represents balance between buyers and sellers.
- **Expansion:** A new imbalance occurs, volatility spikes, and a directional move begins.

Statistically, volatility clustering means that **low volatility periods are more likely to be followed by high volatility** than vice versa. Catching the **transition** can offer low-risk/high-reward trades.

---

### 2. Why Linear Regression Slope?
Traditional squeeze indicators (like **Bollinger Bands / Keltner Channels**) measure volatility based on **price ranges** alone.  
LRS instead measures the **trend velocity** of price movement, not just its dispersion.  

**Advantages:**
- Captures directional bias earlier than BB/KC squeezes.
- Filters sideways chop better because slope changes precede band expansion.
- Can identify squeezes inside trending moves, not just range-bound ones.

---

### 3. Why Fibonacci Levels?
Fibonacci ratios (23.6%, 38.2%, 50%, 61.8%, 78.6%) are widely used in trading to measure **retracements and extensions**. Here they are applied to the **slope range**, not price:
- The slope retracing to a fib level can indicate a “healthy pullback” before trend continuation.
- Helps set **context** for whether the slope is approaching overextension or support levels.

---

### 4. Adaptive Thresholds vs. Fixed Thresholds
While a fixed compression threshold works, volatility regimes change:
- Adaptive modes (z-score or percentile) adjust thresholds dynamically.
- Makes the squeeze detection robust to different instruments, timeframes, and market phases.

---

### 5. Histogram Direction & Smoothing
The histogram represents **signed compression magnitude**:
- Height = Strength of squeeze or expansion
- Sign = Direction bias (positive = bullish, negative = bearish)

Smoothing logic (Sign Vote, Deadzone, Reversal Block) prevents noisy one-bar flips that plague other squeeze methods.
---

## Tunable Inputs & Instructions

| **Input Name** | **Type** | **Default** | **Description** | **How to Use** |
|----------------|----------|-------------|------------------|----------------|
| **Length** | Number | 20 | Lookback for slope calculation and range analysis. | Larger values = smoother signals, slower response; smaller values = more sensitive. |
| **Price Source** | Select | Close | Which price series to measure slope on. | Typical = Close; can use HL2 or other averages for smoothing. |
| **Fib Levels (23.6%–78.6%)** | Number | 0.236–0.786 | Fibonacci levels on slope range. | Mostly visual guides for slope retracements. Adjust rarely. |
| **Squeeze Threshold (Fixed)** | Number | 0.30 | Static compression cutoff for “Squeeze On” in Fixed mode. | Lower = more squeezes, higher = fewer but stronger. |
| **Slope Line Thickness** | Number | 3 | Width of LRS slope plot. | Visual preference only. |
| **Slope Vol Lookback** | Number | 20 | Lookback for band variability calculations. | Higher = wider bands, slower reaction. |
| **Slope Band Mult** | Number | 2.0 | Multiplier for variability bands. | Larger = looser bands, smaller = tighter. |
| **Band Type** | Select | StdDev / MAD | Type of variability calculation. | StdDev = more responsive; MAD = more robust to outliers. |
| **MAD Len** | Number | 20 | Lookback for MAD calculation if MAD mode selected. | Only affects MAD band mode. |
| **Threshold Mode** | Select | Fixed / Adaptive (z-score) / Adaptive (percentile) | How squeeze compression is measured. | Adaptive modes self-adjust to volatility shifts. |
| **Adaptive Lookback (z)** | Number | 100 | Window for z-score calculation in Adaptive mode. | Longer = slower threshold adaptation. |
| **Adaptive Cut (z)** | Number | -0.5 | Z-score cutoff for “Squeeze On.” | Lower = fewer squeezes, higher = more frequent. |
| **Percentile Lookback** | Number | 100 | Window for percentile-based threshold. | Only in percentile mode. |
| **Percentile Cut (0..1)** | Number | 0.20 | Percentile rank cutoff for “Squeeze On.” | Lower = fewer squeezes. |
| **Min Hist Impulse** | Number | 0.15 | Minimum histogram magnitude for “strong” expansion signals. | Filters out weak/noisy flips. |
| **Hist Scale** | Number | 0.20 | Multiplier for histogram magnitude. | For visual sizing only. |
| **Hist Cap (abs)** | Number | 3.0 | Max absolute histogram height. | Clamps extreme values. |
| **Hist Deadzone** | Number | 0.05 | Ignores small histogram values to reduce noise. | Increase to smooth more aggressively. |
| **Hist Smooth EMA** | Number | 3 | EMA smoothing length for histogram magnitude. | Higher = smoother, slower. |
| **Sign Vote Window** | Number | 3 | Number of bars to consider for sign voting. | Higher = more stable histogram direction. |
| **Sign Vote Majority %** | Number | 0.60 | Fraction of bars in the vote window that must agree to flip sign. | Higher = stricter flips. |
| **Reversal Block Bars** | Number | 0 | Prevents flips within X bars after last flip. | Useful to avoid whipsaws. |

---

## Signals Generated
- **Squeeze On** — Volatility compression detected.
- **Squeeze Off** — Compression ended.
- **Expansion Up** — Histogram flips from ≤0 to >0.
- **Expansion Down** — Histogram flips from ≥0 to <0.
- **Expansion Up (Strong)** — Expansion Up with magnitude ≥ Min Hist Impulse.
- **Expansion Down (Strong)** — Expansion Down with magnitude ≥ Min Hist Impulse.

---

## Practical Trading Tips
1. **Squeeze → Expansion:** Look for price breakouts coinciding with **Squeeze Off + Strong Expansion** signals.
2. **Avoid Chop:** Increase **Sign Vote Window** or **Deadzone** to avoid one-bar flips in choppy conditions.
3. **Combine With Price Structure:** Use horizontal levels, order blocks, or VWAP as confirmation.
4. **Band Breaches Matter:** A slope break beyond upper/lower bands post-squeeze often signals trend continuation.

---

## Example Workflow
1. Wait for **Squeeze On** → indicates low-volatility compression.
2. When histogram flips to positive/negative and **Squeeze Off** occurs → watch for **Strong Expansion**.
3. Enter trade in direction of expansion if price action supports it.
4. Use opposite histogram flip or band re-entry as exit criteria.

---
## FAQ & Caveats

### **Q1: Why not just use Bollinger/Keltner squeeze?**
BB/KC squeezes work well in many contexts, but they:
- Rely solely on volatility bands
- Need a separate momentum filter
- Struggle to detect squeezes during ongoing trends  
**LRS Fib Squeeze+** solves this by integrating trend velocity, momentum, and adaptive thresholds into one tool.

---

### **Q2: What’s the benefit of adding Fibonacci slope levels?**
Fib levels act as **contextual zones** for slope retracements:
- If slope pulls back to a fib and re-expands, it’s often a healthy continuation.
- If slope breaks through deeper fibs, it may signal trend exhaustion.

---

### **Q3: How do I interpret the histogram?**
- **Near Zero:** Neutral / low momentum
- **Strong Positive:** Bullish expansion
- **Strong Negative:** Bearish expansion
- **Compression + Flip:** Often a pre-breakout signal

---

### **Q4: Can I use this intraday?**
Yes. Works from 1-min to weekly:
- On lower timeframes, **tighten thresholds** to avoid excessive noise.
- On higher timeframes, **widen thresholds** for fewer but higher-quality signals.

---

### **Q5: Why are there still one-bar flips?**
If histogram flips back and forth quickly:
- Increase **Sign Vote Window** or Majority %
- Use **Reversal Block Bars** to lock direction for X bars after a flip
- Increase **Hist Deadzone** to filter small changes

---

### **Q6: Is this a standalone entry/exit tool?**
No. It’s best used with:
- Structure (support/resistance, order flow)
- Volume confirmation
- Risk management rules

---

### **Q7: How do I avoid false breakouts?**
Combine with:
- Market context (trend direction on higher timeframe)
- Volume confirmation
- Avoid trading immediately before major news

---

## Final Thoughts
The LRS Fib Squeeze+ takes the **core logic of volatility squeeze trading** and fuses it with **trend slope analysis**, **Fibonacci context**, and **noise reduction mechanisms**.  
Its strength lies in adaptability — whether you want a **fast, aggressive** squeeze trigger or a **slow, high-confidence** one, the tunable parameters let you match the tool to your market and style.

## License
MIT License — use freely, share improvements!
