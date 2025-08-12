# LRS Fib Squeeze+

> **Volatility-compression → momentum-expansion** indicator based on Linear Regression Slope (LRS), with adaptive squeeze detection, anti-chop histogram logic, slope variability bands, Fibonacci slope levels, and inversion detection/shading.

---

## Summary

- **Measures trend velocity** via **Linear Regression Slope (LRS)** over `Length`.
- Tracks the **range of slope** and **normalizes** it by ATR → a scale-free compression gauge.
- Detects **Squeeze On** when normalized slope variability is **below** a fixed or **adaptive** threshold.
- Builds a **directional histogram** from compression magnitude × slope momentum sign (using anti-chop logic).
- Draws **variability bands** (StdDev or MAD) around the slope mean as context.
- Projects **Fibonacci levels** on the slope range for visual reference.
- Detects **inversions** (peak→turn down, trough→turn up) on a smoothed histogram and **shades** those zones on the histogram panel.
- Provides **expansion** and **inversion** signals; also computes **reversal block** masks (cool-down windows) without altering visuals.
- Code for use as **trendspider** indicator.

---

## Theoretical Foundation

1. **Volatility clustering**: Markets cycle between **compression** and **expansion**. Low volatility often precedes a large move.
2. **LRS instead of raw price range**: Using **slope** (trend velocity) helps distinguish directional expansion from sideways volatility changes (a common weakness of pure BB/KC squeezes).
3. **Normalization (ATR)**: Makes slope-range comparable across instruments/timeframes.
4. **Adaptive thresholds**: z-score/percentile modes scale with regime shifts; fixed mode stays simple.
5. **Anti-chop histogram**: Deadzone + EMA magnitude smoothing + majority sign vote + sticky sign → fewer 1–2 bar flips.
6. **Inversion detection**: Finds **recent local maxima/minima** in momentum that **turn** with a minimum magnitude; these areas are shaded for focus.

---

## Use Cases

- **Pre-breakout timing**: Trade as the squeeze releases into expansion with directional bias.
- **Trend continuation**: Use positive/negative histogram with band context and Fib slope levels.
- **Exits/Pauses**: Inversion shading can warn of a top/bottom forming (momentum cresting).
- **Filter for systems**: Use Squeeze On/Off and Expansion signals as **logic builder** conditions.

---

## Methodology

1. **LRS**: Linear regression slope of selected `Price source` over `Length`.
2. **Slope Range**: Rolling `highest(slope, Length)` − `lowest(slope, Length)`.
3. **Normalize** by ATR of price (same `Length`) → **normalizedRange**.
4. **Squeeze Detection**  
   - **Fixed**: `normalizedRange < squeezeThreshold`  
   - **Adaptive (z-score)**: `z = (normalizedRange - mean) / stdev`; squeeze when `z < zCut`  
   - **Adaptive (percentile)**: squeeze when rank ≤ `pctCut`
5. **Histogram (anti-chop)**  
   - **Magnitude** = |squeezeBase| (compression/expansion magnitude)  
   - **Sign** = sign of `slope - EMA(slope, 2)`  
   - **Deadzone** near 0, **EMA** magnitude smooth, **majority vote** over `N` bars, **sticky** last non-zero decision.
6. **Bands**: Slope mean ± `bandK` × (StdDev or MAD).  
7. **Fibs on slope range**: 23.6/38.2/50/61.8/78.6%.
8. **Inversions (chop-tolerant)** on a **smoothed histogram** (`invSmoothN`):  
   - Detect **peak → turn down** (Inversion Down) and **trough → turn up** (Inversion Up) if:  
     recent extreme within **tolerance %**, **recent** (≤ `invMaxAge`), **min height** (|hist| ≥ `invMinPk`), and **min turn** (|Δ| ≥ `invMinTurn`).  
   - **Shade** the next `invShadeBars` bars above/below zero line on histogram panel.
9. **Reversal Blocking (computed)**: Cool-down windows that **suppress opposite-direction signals** for `revBlock` bars after an event (does **not** change visuals).

---

## Signals

- **Squeeze On** / **Squeeze Off**
- **Expansion Up** (histogram flips `≤ 0 → > 0`)
- **Expansion Down** (histogram flips `≥ 0 → < 0`)
- **Expansion Up (strong)** (flip up **and** |hist| ≥ `Min Hist Impulse`)
- **Expansion Down (strong)**
- **Inversion Up** (trough → turn up)
- **Inversion Down** (peak → turn down)

> **Note on reversal blocking**: The script computes blocked versions internally (cool-downs), but **does not register** them as separate signals. Use the raw signals above in the logic builder, and apply blocking in your strategy rules if desired.

---

## Tunable Inputs

### Core & Visualization
| Input | Default | What it Does | Notes |
|---|---:|---|---|
| **Length** | 20 | LRS lookback and slope range window | Larger = smoother, slower |
| **Price source** | close | Price series for LRS | e.g., close/hl2/etc. |
| **Slope Line Thickness** | 3 | Visual width | Cosmetic |

### Slope Variability Bands
| Input | Default | What it Does | Notes |
|---|---:|---|---|
| **Slope Vol Lookback** | 20 | Lookback for bands | StdDev or MAD |
| **Slope Band Mult** | 2.0 | Multiplier for band width | Wider = looser |
| **Band Type** | StdDev | Variability method | MAD is robust to outliers |
| **MAD Len** | 20 | EMA len for MAD | Only used in MAD mode |

### Squeeze Detection
| Input | Default | What it Does | Notes |
|---|---:|---|---|
| **Threshold Mode** | Adaptive (z-score) | Fixed / z-score / percentile | Choose method |
| **Squeeze Threshold (fixed)** | 0.30 | Fixed cutoff for normalizedRange | Only in Fixed mode |
| **Adaptive Lookback (z)** | 100 | z-score window | Larger = smoother |
| **Adaptive Cut (z)** | −0.5 | z-score cutoff for squeeze | Lower = stricter |
| **Percentile Lookback** | 100 | Rank window | Percentile mode only |
| **Percentile Cut (0..1)** | 0.20 | Rank cutoff | Lower = stricter |

### Histogram (Anti-Chop) & Expansion
| Input | Default | What it Does | Notes |
|---|---:|---|---|
| **Hist Deadzone** | 0.05 | Zeroes tiny values | Removes micro-flips |
| **Hist Smooth EMA** | 3 | Smooths magnitude | Not sign |
| **Sign Vote Window** | 3 | Majority window (N) | 1–5 recommended |
| **Sign Vote Majority %** | 0.60 | Fraction of bars needed to flip | 0.6–0.8 for calm |
| **Paint Smoothed Histogram** | On | Use smoothed series for paint | Cosmetic |
| **Use Smoothed Hist For Signals** | On | Use smoothed series in logic | Stability ↑ |
| **Min Hist Impulse** | 0.15 | “Strong” expansion threshold | Filter weak flips |
| **Hist Scale** | 0.20 | Visual scale | Cosmetic |
| **Hist Cap (abs)** | 3.0 | Visual clamp | Cosmetic |

### Inversion Detection & Shading
| Input | Default | What it Does | Notes |
|---|---:|---|---|
| **Inversion Smooth EMA** | 3 | Smooths histogram for detection | Noise reduction |
| **Inversion Lookback** | 8 | Window to find recent extreme | Local peak/trough |
| **Inversion Min \|Hist\|** | 0.40 | Min height at extreme | Strength filter |
| **Inversion Min Turn (abs)** | 0.08 | Min change from extreme to next bar | Turn filter |
| **Inversion Peak Tolerance %** | 0.15 | Extreme proximity tolerance | Within X% of max/min |
| **Inversion Max Age (bars)** | 3 | Max bars since extreme | Recency constraint |
| **Inversion Shade Bars** | 2 | Shade duration after inversion | Visual |
| **Inversion Shade Height** | 0 | Shade height (0 → use `Hist Cap`) | Visual |

### Reversal Block (Computed Cool-Down)
| Input | Default | What it Does | Notes |
|---|---:|---|---|
| **Reversal Block Bars** | 0 | Suppresses opposite-direction signals for N bars | Does **not** change visuals or registered signals; apply masks in strategy rules |

### Fibonacci Slope Levels
| Input | Default | What it Does | Notes |
|---|---:|---|---|
| **Fib 23.6% / 38.2% / 50% / 61.8% / 78.6%** | 0.236–0.786 | Visual reference levels on slope range | Context only

---

## How to Read It

1. **Squeeze On (red slope)** → watch for **Expansion** signals.
2. **Expansion Up/Down** → momentum flips and leaves deadzone; “Strong” when magnitude ≥ threshold.
3. **Bands** → Context for mean reversion vs acceleration.
4. **Inversion shading** → Crimson (peak→down), Seagreen (trough→up). Use as **exit caution** or countertrend probe, especially after extended runs.

---

## Differences vs BB/KC Squeeze

| Aspect | LRS Fib Squeeze+ | BB/KC Squeeze |
|---|---|---|
| **Driver** | Trend velocity (LRS) | Price dispersion (StdDev/ATR) |
| **Direction** | Integrated via slope momentum | Needs extra momentum filter |
| **Squeeze Threshold** | Fixed or Adaptive (z/percentile) | Usually fixed ratio |
| **Noise Handling** | Deadzone + EMA + Sign Vote + Sticky | Typically none |
| **Extras** | Fib slope levels, inversion shading | Not typical |

---

## Caveats

- **Not a standalone system**: Use with structure/volume and risk management.
- **Parameters are regime-sensitive**: Intraday vs daily may need different Deadzone/Vote/Impulse.
- **Inversion ≠ immediate reversal**: It flags **momentum crests**, not guaranteed price tops/bottoms.
- **Reversal Block** doesn’t change what you see: It’s a **logic-side** guardrail for backtests/bots.

---

## Workflow (Example)

1. Scan for **Squeeze On**.
2. Wait for **Expansion Up/Down** (strong preferred) with price confirmation.
3. Manage with bands/Fibs; **exit or reduce** on inversion shading or opposite expansion.

---

## License

MIT — use, modify, and share freely. Please credit if you fork or publish variants.
