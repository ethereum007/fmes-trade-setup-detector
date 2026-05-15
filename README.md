# FMES Trade Setup Detector

A rule-based H4 trade setup detector for TradingView (Pine v6) that automates the FMES / HTF Mystery strategy by Marcell (Flipping Markets).

The detector stacks five criteria to identify high-conviction reversal setups, then plots the full trade plan — entry, SL, TPs, and order block — directly on the chart.

## What it detects

| # | Criterion | Output |
|---|-----------|--------|
| 1 | **Break of Structure** (BOS) — body close past most recent swing | (debug layer) |
| 2 | **Liquidity sweep** — wick pierces a swing, opposing BOS within window | Protected High / Low zones |
| 3 | **Volume gap** — 3-bar pattern: top of 1st < bottom of 3rd (or inverse) | Optional filter on every BOS |
| 4 | **Order Block** — supply / demand zone at the swept extreme | Yellow box |
| 5 | **Fibonacci 0.71 entry** — deep retracement confluence | Green entry line |
| +1 | **LQ+ patterns** — double / triple tops & bottoms | DT / TT chips on pivots |

Per detected setup, the script auto-plots:

- **Entry** at Fib 0.71 (green, 2.45R target)
- **SL** at Fib 1 (red)
- **TP** at Fib 0 (cyan)
- **TP1** at Fib -0.27 (cyan dashed, partial / extension target)
- **OB Entry** at order-block level (purple, **HWR dual-limit** method — 1:2 R:R)
- **OB TP** for the second limit
- **Protected zone** (purple) and Order Block (yellow) reference boxes

Alert conditions: `Short Setup Ready` / `Long Setup Ready` — set in TV to be notified the moment a new setup is detected.

## How to use in TradingView

1. Open TradingView → any chart (H4 forex majors recommended: USDJPY, EURUSD, GBPUSD, XAUUSD)
2. Pine Editor → paste contents of [`pine/fmes_setup_detector.pine`](pine/fmes_setup_detector.pine) → Save → Add to chart
3. Right-click chart → Add alert → Condition: `FMES — Trade Setup Detector` → choose `Short Setup Ready` or `Long Setup Ready`
4. When alert fires, the levels are pre-plotted. Place limits at:
   - Entry (green line) — Fib 0.71
   - OB Entry (purple line) — deeper retracement, 1:2 R:R
5. Both share SL (red line). Suggested risk: 0.5–1% per limit, total 1–2% per setup.

## Settings worth tuning

| Input | Default | What to change for what effect |
|-------|---------|--------------------------------|
| `Pivot lookback` | 5 | Lower (2–3) for more setups, higher (7–10) for stricter / fewer |
| `Max bars between sweep & BOS` | 20 | Shorter for tighter reversal pattern |
| `Require volume gap on BOS` | true | Turn off to relax filtering |
| `Equal-level tolerance (× ATR)` | 0.3 | Tighter (0.1–0.2) for stricter DT/TT detection |
| `OB limit target R:R` | 2.0 | Adjust per your prop firm / RR profile |

## Module breakdown

The detector is built in 5 incremental modules, each in its own file for debugging / understanding:

| File | What it adds |
|------|-------------|
| [`pine/fmes_m1_structure.pine`](pine/fmes_m1_structure.pine) | Pivots, body-close BOS, wick sweeps (foundation) |
| [`pine/fmes_m2_protected.pine`](pine/fmes_m2_protected.pine) | Protected High / Low zones + volume gap filter |
| [`pine/fmes_m3_setup.pine`](pine/fmes_m3_setup.pine) | Fib 0/0.71/1 + OB + Entry / SL / TP / TP1 lines |
| [`pine/fmes_setup_detector.pine`](pine/fmes_setup_detector.pine) | **Final** — adds M4 (LQ+) + M5 (HWR dual-limit) |

For production use, only `fmes_setup_detector.pine` is needed. The Mx files are kept for reference / step-by-step debugging.

## Setup density (sample, last ~250 H4 bars)

| Pair | PSH | PSL | LQ+ patterns | Notes |
|------|-----|-----|--------------|-------|
| USDJPY | 4 | 1 | many DTs | Choppy — more setups |
| EURUSD | 1 | 1 | 2 DT | Clean trend — fewer |
| GBPUSD | 1 | 2 | DT + TT | Best confluence example (TT + PSH) |

## Known limitations (planned improvements)

- **No setup state tracking** — once a setup invalidates (SL hit) or completes (TP hit), the lines stay drawn. User must mentally archive.
- **Fib drawn at BOS confirmation** uses BOS-bar high/low as impulse extreme. If the impulse extends further before pullback, Fib will be slightly off.
- **No trendline-LQ detection** — only DT / TT for now. Trendline pattern recognition is complex and unreliable.
- **Single-symbol** — doesn't scan across multiple pairs in parallel.

## License & credit

Strategy design: Marcell (Flipping Markets) — see [flippingmarketsfx.com](https://www.flippingmarketsfx.com/) and the original FMES Blueprint / HTF Mystery PDFs.

Pine implementation: this repo. Educational use only — not financial advice. Trading carries risk; past setups do not guarantee future results.
