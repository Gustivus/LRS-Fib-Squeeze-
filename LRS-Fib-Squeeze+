describe_indicator('LRS Fib Squeeze+', 'lower', { decimals: 'by_symbol_+4', shortName: 'LRS+' });

/* =========================  Inputs  ========================= */
const length   = input.number('Length', 20, { min: 2, max: 199 });
const priceSrc = input.select('Price source', 'close', constants.price_source_options);
const price    = market[priceSrc];

// Inversion detection (chop-tolerant)
const invLen     = input.number('Inversion Lookback', 8,  { min: 3, max: 50, step: 1 });
const invMinPk   = input.number('Inversion Min |Hist|', 0.40, { min: 0.0, max: 10, step: 0.05 });
const invMinTurn = input.number('Inversion Min Turn (abs)', 0.08, { min: 0.0, max: 10, step: 0.01 });
const invTolPct  = input.number('Inversion Peak Tolerance %', 0.15, { min: 0.0, max: 0.5, step: 0.01 });
const invMaxAge  = input.number('Inversion Max Age (bars)', 3, { min: 1, max: 10, step: 1 });
const invSmoothN = input.number('Inversion Smooth EMA', 3, { min: 1, max: 10, step: 1 });

// Fibonacci levels (on slope range)
const fib236 = input.number('Fib 23.6%', 0.236, { min: 0, max: 1, step: 0.001 });
const fib382 = input.number('Fib 38.2%', 0.382, { min: 0, max: 1, step: 0.001 });
const fib500 = input.number('Fib 50.0%',  0.5,   { min: 0, max: 1, step: 0.001 });
const fib618 = input.number('Fib 61.8%',  0.618, { min: 0, max: 1, step: 0.001 });
const fib786 = input.number('Fib 78.6%',  0.786, { min: 0, max: 1, step: 0.001 });

// Squeeze + viz
const squeezeThreshold   = input.number('Squeeze Threshold (fixed)', 0.30, { min: 0, max: 1, step: 0.01 });
const slopeLineThickness = input.number('Slope Line Thickness', 3, { min: 1, max: 5, step: 1 });

// Bands / variability
const volLen = input.number('Slope Vol Lookback', 20, { min: 5, max: 300, step: 1 });
const bandK  = input.number('Slope Band Mult', 2.0, { min: 0.5, max: 5, step: 0.1 });
const bandType = input.select('Band Type', 'StdDev', ['StdDev', 'MAD']);
const madLen   = input.number('MAD Len', 20, { min: 5, max: 300, step: 1 });

// Threshold mode
const thresholdMode = input.select(
  'Threshold Mode',
  'Adaptive (z-score)',
  ['Fixed', 'Adaptive (z-score)', 'Adaptive (percentile)']
);
const zLen = input.number('Adaptive Lookback (z)', 100, { min: 20, max: 500, step: 1 });
const zCut = input.number('Adaptive Cut (z)', -0.5, { min: -3, max: 3, step: 0.1 });

const prLen  = input.number('Percentile Lookback', 100, { min: 30, max: 500, step: 5 });
const pctCut = input.number('Percentile Cut (0..1)', 0.20, { min: 0.05, max: 0.5, step: 0.01 });

// Cross / impulse debouncing (for "strong" expansions)
const minImpulse = input.number('Min Hist Impulse', 0.15, { min: 0, max: 5, step: 0.01 });

// Histogram compression (visual only)
const histScale = input.number('Hist Scale', 0.20, { min: 0.01, max: 5, step: 0.01 });
const histCap   = input.number('Hist Cap (abs)', 3.0, { min: 0.1, max: 50, step: 0.1 });

// Anti-chop on histogram (visual & optional logic)
const histDead       = input.number('Hist Deadzone', 0.05, { min: 0, max: 2, step: 0.01 });
const histSmooth     = input.number('Hist Smooth EMA', 3, { min: 1, max: 20, step: 1 });
const signStreak     = input.number('Min Sign Streak', 2, { min: 0, max: 5, step: 1 }); // 0=off
const paintSmoothSel = input.select('Paint Smoothed Histogram', 'On', ['On','Off']);
const paintSmoothed  = (paintSmoothSel === 'On');
const useSmoothSel   = input.select('Use Smoothed Hist For Signals', 'On', ['On','Off']);
const useSmoothedForSignals = (useSmoothSel === 'On');
const signVoteLen = input.number('Sign Vote Window', 3, { min: 1, max: 5, step: 1 });     // N
const signVotePct = input.number('Sign Vote Majority %', 0.60, { min: 0.5, max: 1, step: 0.05 }); // e.g. 60%

// Reversal suppression (cool-down between opposite signals) — computed, not registered
const revBlock = input.number('Reversal Block Bars', 0, { min: 0, max: 5, step: 1 }); // 0 = off

/* =========================  Helpers  ========================= */
const rollingStd = (series, win) =>
  sliding_window_function(series, win, vals => {
    const n = vals.length; if (n === 0) return 0;
    let tot = 0; for (let i = 0; i < n; i++) tot += vals[i];
    const mean = tot / n;
    let vari = 0; for (let i = 0; i < n; i++) { const d = vals[i] - mean; vari += d * d; }
    return Math.sqrt(vari / Math.max(1, n - 1));
  });

const lrSlope = values => {
  let xSum = 0, ySum = 0, xySum = 0, xxSum = 0;
  const n = values.length;
  for (let i = 0; i < n; i++) { xSum += i; ySum += values[i]; xySum += i * values[i]; xxSum += i * i; }
  const num = n * xySum - xSum * ySum;
  const den = n * xxSum - xSum * xSum;
  return den === 0 ? 0 : num / den;
};

const fibOnRange = (h, l, f) => l + (h - l) * f;

/* =========================  Core series  ========================= */
const slope = sliding_window_function(price, length, lrSlope);

const rollHi = highest(slope, length);
const rollLo = lowest(slope, length);
const slopeRange = sub(rollHi, rollLo);

const fib236S = for_every(rollHi, rollLo, (h, l) => fibOnRange(h, l, fib236));
const fib382S = for_every(rollHi, rollLo, (h, l) => fibOnRange(h, l, fib382));
const fib500S = for_every(rollHi, rollLo, (h, l) => fibOnRange(h, l, fib500));
const fib618S = for_every(rollHi, rollLo, (h, l) => fibOnRange(h, l, fib618));
const fib786S = for_every(rollHi, rollLo, (h, l) => fibOnRange(h, l, fib786));

// Normalize slope-range by ATR of price
const myAtr = atr(high, low, close, length);
const normalizedRange = div(slopeRange, myAtr);

/* =========================  Adaptive thresholds  ========================= */
const nrMean = sma(normalizedRange, zLen);
const nrStd  = rollingStd(normalizedRange, zLen);
const nrZ    = div(sub(normalizedRange, nrMean), nrStd);

const nrRank = sliding_window_function(normalizedRange, prLen, vals => {
  const last = vals[vals.length - 1];
  let cnt = 0; for (let i = 0; i < vals.length; i++) if (vals[i] <= last) cnt++;
  return cnt / vals.length; // 0..1
});

let squeezeBase; // signed compression/expansion proxy
let squeezeOn;   // boolean compressed?

if (thresholdMode === 'Adaptive (z-score)') {
  squeezeBase = nrZ;                                // negative => compressed
  squeezeOn   = for_every(nrZ, z => z < zCut);
} else if (thresholdMode === 'Adaptive (percentile)') {
  squeezeBase = for_every(nrRank, r => r - pctCut); // <0 => compressed
  squeezeOn   = for_every(nrRank, r => r <= pctCut);
} else {
  squeezeBase = sub(normalizedRange, squeezeThreshold); // <0 => compressed
  squeezeOn   = for_every(normalizedRange, v => v < squeezeThreshold);
}

/* =========================  Direction & histogram (anti-chop, majority+sticky)  ========================= */

// Momentum proxy: slope - EMA(slope, 2)
const slopeMom = sub(slope, ema(slope, 2));

// Raw signed histogram = |compression| * sign(momentum)
const magRaw     = for_every(squeezeBase, v => Math.abs(v));
const signSeries = for_every(slopeMom, m => (m >= 0 ? 1 : -1));
const histRaw    = for_every(magRaw, signSeries, (sz, sg) => sz * sg);

// 1) Dead-zone near zero (kills tiny flips)
const histDZ   = for_every(histRaw, v => (v == null ? null : (Math.abs(v) < histDead ? 0 : v)));

// 2) Smooth magnitude (EMA), keep sign separate
const magDZ    = for_every(histDZ, v => Math.abs(v));
const magSm    = ema(magDZ, histSmooth);
const signRawS = for_every(histDZ, v => (v > 0 ? 1 : (v < 0 ? -1 : 0)));

// 3) Majority vote over last N bars, then sticky last decided sign
const N   = Math.max(1, signVoteLen);
const TH  = Math.ceil(N * Math.max(0.5, Math.min(1.0, signVotePct))); // threshold bars agreeing

// Sum of signs over last N bars (-N..+N)
const signSumN = sliding_window_function(signRawS, N, vals => {
  let s = 0; for (let i = 0; i < vals.length; i++) s += (vals[i] || 0); return s;
});

// Decided sign by majority vote
const decidedSign = for_every(signSumN, sum => {
  if (sum >= TH)  return 1;
  if (sum <= -TH) return -1;
  return 0; // indecisive this bar
});

// Sticky sign: if indecisive, hold the last non-zero decision from a short lookback
const stickySign = sliding_window_function(decidedSign, N + 1, vals => {
  // scan backward for last non-zero decision (including current bar)
  for (let i = vals.length - 1; i >= 0; i--) {
    const v = vals[i];
    if (v === 1 || v === -1) return v;
  }
  return 0;
});

// 4) Reconstruct histogram with smoothed magnitude and sticky sign
const histSmoothSeries = for_every(magSm, stickySign, (m, s) => m * s);

// Which histogram to paint / use for signals? (reuse your toggles)
const histForPaint  = paintSmoothed ? histSmoothSeries : histRaw;
const histForLogic  = useSmoothedForSignals ? histSmoothSeries : histRaw;

// Scale & cap for painting
const histScaled = for_every(histForPaint, v => v * histScale);
const squeezeHist = for_every(histScaled, v => {
  if (v == null) return null;
  const cap = histCap;
  return v >  cap ?  cap :
         v < -cap ? -cap : v;
});

/* =========================  Variability bands (StdDev or MAD)  ========================= */
const slopeMean = ema(slope, volLen);
const slopeStd  = rollingStd(slope, volLen);
const upperStd  = for_every(slopeMean, slopeStd, (m, s) => m + bandK * s);
const lowerStd  = for_every(slopeMean, slopeStd, (m, s) => m - bandK * s);

const slopeEMA  = ema(slope, madLen);
const absDev    = for_every(slope, slopeEMA, (s, m) => Math.abs(s - m));
const mad       = ema(absDev, madLen);
const upperMAD  = for_every(slopeEMA, mad, (m, d) => m + bandK * d);
const lowerMAD  = for_every(slopeEMA, mad, (m, d) => m - bandK * d);

const usingMAD  = (bandType === 'MAD');
const upperBand = usingMAD ? upperMAD : upperStd;
const lowerBand = usingMAD ? lowerMAD : lowerStd;

/* =========================  Paints  ========================= */
const slopeColor = for_every(squeezeOn, s => (s ? 'red' : 'blue'));
paint(slope,     { name: 'LRS+ Slope', color: slopeColor, width: slopeLineThickness });
paint(upperBand, { name: 'LRS+ Slope Upper Band', color: 'gray', style: 'line', width: 1 });
paint(lowerBand, { name: 'LRS+ Slope Lower Band', color: 'gray', style: 'line', width: 1 });

// Smoothed/Raw (per toggle), scaled & capped
paint(squeezeHist, { name: 'LRS+ Squeeze Histogram', style: 'histogram' });

// Optional Fibs on slope range
paint(fib236S, { name: 'LRS+ Fib 23.6%', color: 'silver', width: 1 });
paint(fib382S, { name: 'LRS+ Fib 38.2%', color: 'silver', width: 1 });
paint(fib500S, { name: 'LRS+ Fib 50.0%', color: 'silver', width: 1 });
paint(fib618S, { name: 'LRS+ Fib 61.8%', color: 'silver', width: 1 });
paint(fib786S, { name: 'LRS+ Fib 78.6%', color: 'silver', width: 1 });

/* =========================  Cross detection & debounced signals  ========================= */
const prevHist = sliding_window_function(histForLogic, 2, vals => vals[0]);

const expansionUp   = for_every(histForLogic, prevHist, (c, p) => (p <= 0) && (c > 0));
const expansionDown = for_every(histForLogic, prevHist, (c, p) => (p >= 0) && (c < 0));

const strongUp   = for_every(histForLogic, prevHist, (c, p) => (p <= 0 && c > 0) && Math.abs(c) >= minImpulse);
const strongDown = for_every(histForLogic, prevHist, (c, p) => (p >= 0 && c < 0) && Math.abs(c) >= minImpulse);

/* =========================  Inversion detectors (chop-tolerant)  ========================= */
const histDet = ema(histForLogic, invSmoothN);

const invFlag = sliding_window_function(histDet, Math.max(invLen, 3), vals => {
  const n = vals.length; if (n < 3) return null;
  const prev = vals[n - 2], curr = vals[n - 1];
  if (prev == null || curr == null) return null;

  let maxV = -Infinity, minV = Infinity, idxMax = -1, idxMin = -1;
  for (let i = 0; i < n - 1; i++) {
    const v = vals[i]; if (v == null) continue;
    if (v > maxV) { maxV = v; idxMax = i; }
    if (v < minV) { minV = v; idxMin = i; }
  }
  if (idxMax < 0 && idxMin < 0) return null;

  const tolUp   = Math.abs(maxV) * invTolPct;
  const tolDown = Math.abs(minV) * invTolPct;
  const ageMax  = (idxMax >= 0) ? (n - 1 - idxMax) : Infinity; // bars since max (0 => prev)
  const ageMin  = (idxMin >= 0) ? (n - 1 - idxMin) : Infinity;

  const prevAbs = Math.abs(prev);
  const turnedDown = (prev - curr) >= invMinTurn;
  const turnedUp   = (curr - prev) >= invMinTurn;

  const nearMax   = (maxV > 0) && (prev >= (maxV - tolUp));
  const recentMax = ageMax <= invMaxAge;
  const invDown   = (prev > 0) && nearMax && recentMax && (prevAbs >= invMinPk) && turnedDown;

  const nearMin   = (minV < 0) && (prev <= (minV + tolDown));
  const recentMin = ageMin <= invMaxAge;
  const invUp     = (prev < 0) && nearMin && recentMin && (prevAbs >= invMinPk) && turnedUp;

  if (invDown) return 1;
  if (invUp)   return -1;
  return null;
});

const inversionDown = for_every(invFlag, f => (f === 1 ? 1 : null));
const inversionUp   = for_every(invFlag, f => (f === -1 ? 1 : null));

/* ===== Inversion highlight bands (length-safe) ===== */
const invShadeBars   = input.number('Inversion Shade Bars', 2, { min: 1, max: 10, step: 1 });
const invShadeHeight = input.number('Inversion Shade Height', 0, { min: 0, max: 50, step: 0.1 });
const H = invShadeHeight > 0 ? invShadeHeight : histCap;

const invDnNum = for_every(inversionDown, v => (v ? 1 : 0));
const invUpNum = for_every(inversionUp,   v => (v ? 1 : 0));

const invDnShade = for_every(highest(invDnNum, invShadeBars), v => (v > 0 ? 1 : null));
const invUpShade = for_every(highest(invUpNum, invShadeBars), v => (v > 0 ? 1 : null));

const zero = series_of(0);
const posH = series_of(H);
const negH = series_of(-H);

const topDn = for_every(invDnShade, posH, (s, ph) => (s ? ph : null));  // 0 → +H
const botDn = for_every(invDnShade, zero, (s, z)  => (s ? z  : null));

const topUp = for_every(invUpShade, zero, (s, z)  => (s ? z  : null));  // −H → 0
const botUp = for_every(invUpShade, negH, (s, nh) => (s ? nh : null));

const pTopDn = paint(topDn, { hidden: true });
const pBotDn = paint(botDn, { hidden: true });
fill(pTopDn, pBotDn, 'crimson');

const pTopUp = paint(topUp, { hidden: true });
const pBotUp = paint(botUp, { hidden: true });
fill(pTopUp, pBotUp, 'seagreen');

/* =========================  Reversal blocking (computed only)  ========================= */
const toNum = s => for_every(s, v => (v ? 1 : 0));
const expUpNum  = toNum(expansionUp);
const expDnNum  = toNum(expansionDown);
const invUpNumB = toNum(inversionUp);
const invDnNumB = toNum(inversionDown);

const makeRecent = (numSeries, n) =>
  (n > 0)
    ? for_every(highest(numSeries, n), v => (v > 0 ? 1 : 0))
    : series_of(0);

const recentExpUp = makeRecent(expUpNum,  revBlock);
const recentExpDn = makeRecent(expDnNum,  revBlock);
const recentInvUp = makeRecent(invUpNumB, revBlock);
const recentInvDn = makeRecent(invDnNumB, revBlock);

const expansionUp_blk   = for_every(expansionUp,   recentExpDn, (sig, opp) => (sig && opp === 0 ? 1 : null));
const expansionDown_blk = for_every(expansionDown, recentExpUp, (sig, opp) => (sig && opp === 0 ? 1 : null));
const inversionUp_blk   = for_every(inversionUp,   recentInvDn, (sig, opp) => (sig && opp === 0 ? 1 : null));
const inversionDown_blk = for_every(inversionDown, recentInvUp, (sig, opp) => (sig && opp === 0 ? 1 : null));

/* =========================  Signals (original registrations)  ========================= */
register_signal(squeezeOn,                        'Squeeze On');
register_signal(for_every(squeezeOn, v => !v),    'Squeeze Off');
register_signal(expansionUp,                      'Expansion Up');
register_signal(expansionDown,                    'Expansion Down');
register_signal(strongUp,                         'Expansion Up (strong)');
register_signal(strongDown,                       'Expansion Down (strong)');
register_signal(inversionUp,                      'Inversion Up (trough → turn up)');
register_signal(inversionDown,                    'Inversion Down (peak → turn down)');
