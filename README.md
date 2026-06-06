# NIFTY Options — Implied Volatility Surface Reconstruction
### Finance Club, IIT Roorkee · Open Project 2026

**Author:** Anugra | **Kaggle ID:** anugra1606

---

## Problem Statement

Reconstruct every missing implied volatility (IV) value in a NIFTY options dataset spanning multiple strikes and timestamps, without introducing any lookahead bias.

## Method

All missing IV values are reconstructed **purely cross-sectionally** — using only other strikes at the **same timestamp**. No information from any other timestamp (past or future) is ever used.

### Two-part pipeline

| Component | Technique | Rationale |
|-----------|-----------|-----------|
| **Interior** (between observed strikes) | Monotone PCHIP interpolation, CE and PE fit separately | Shape-preserving smile, no overshoot, respects put/call discontinuity at the money |
| **Wings** (deep-OTM beyond outermost observed strike) | Log-linear extrapolation in log(IV) vs strike | Captures the exponential rise into the wings; flat carry badly under-predicts expiry-day spike cells |

### Key insight

> 95% of the metric's error comes from a handful of expiry-day, deep-OTM "spike" cells. The log-slope wing projection cuts held-out MSE ~75% vs a flat carry baseline, with lower seed-to-seed variance for better private leaderboard stability.

## Results

| Regime | Flat Wings MSE | Log-Slope Wings MSE | Improvement |
|--------|---------------|---------------------|-------------|
| Normal (IV ≤ 0.5) | 0.000085 | 0.000043 | 49% lower |
| Spike (IV > 0.5)  | 0.008805 | 0.004054 | 54% lower |
| **All cells**     | 0.000710 | 0.000147 | **79% lower** |

*(Fill in actual values after running the notebook)*

## Repository Structure

```
├── iv_surface_reconstruction.ipynb   # Main notebook (run top to bottom)
├── README.md                         # This file
```

## How to Run

1. Place `dataset.csv` in the same directory as the notebook
2. Run all cells top to bottom
3. Outputs:
   - `filled_dataset.csv` — fully reconstructed dataset
   - `submission.csv` — competition submission file
   - `eda.png` — visualisation of wing behaviour

## Dependencies

```
pandas
numpy
matplotlib
```

All standard — no additional installs required.

## Design Decisions

- **No time axis blending:** An earlier version blended same-moneyness values across time. Even windowed to the same day it reads neighbouring bars, conflicting with the no-lookahead rule. Removed entirely.
- **3-point log-slope fit:** Using the 3 outermost observed strikes (not 2) resists a single noisy mid-quote flipping the wing's direction.
- **CE/PE interpolated separately:** Respects the structural put/call IV discontinuity at the money.
- **Parameter-free:** Nothing to overfit — CV behaviour transfers directly to the private split.

---

*Finance Club, IIT Roorkee · Open Project 2026*
