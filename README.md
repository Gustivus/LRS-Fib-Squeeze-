# LRS Fib Squeeze+ Indicator

## Overview
The **LRS Fib Squeeze+** is a volatility compression and momentum expansion indicator built on **Linear Regression Slope (LRS)** analysis, Fibonacci-projected slope levels, and adaptive squeeze detection logic. It is designed to detect when price action is in a **low-volatility “squeeze” regime** and when it transitions into an **expansion phase** with directional bias.

By combining **trend strength**, **volatility compression**, and **momentum impulse analysis**, this tool helps traders anticipate high-probability breakout or breakdown scenarios.

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

## License
MIT License — use freely, share improvements!
