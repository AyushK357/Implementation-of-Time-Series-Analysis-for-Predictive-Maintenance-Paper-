# Time Series Prediction (TSP) Algorithm for Predictive Maintenance

A Python implementation of the **Time Series Prediction (TSP) Algorithm** proposed in:

> *Lin et al., "Time Series Prediction Algorithm for Intelligent Predictive Maintenance," IEEE Robotics and Automation Letters, Vol. 4, No. 3, July 2019.*

---

## What This Does

Industrial equipment degrades over time. The goal of **predictive maintenance** is to forecast *when* a machine will fail so that maintenance can be scheduled just in time — avoiding both unexpected breakdowns and wasteful over-maintenance.

The classic approach uses an **exponential model** to fit the degradation curve and extrapolate a Remaining Useful Life (RUL). However, it fails in two common real-world scenarios:

- **Flat trend** — when degradation is slow and smooth, the exponential model keeps pushing the predicted failure further into the future, badly underestimating urgency.
- **Sudden rise** — when the aging signal spikes abruptly, the exponential model reacts too slowly to track the rapid deterioration.

This implementation demonstrates the **TSP algorithm**, which replaces the exponential model with a statistically-selected **ARIMA time series model**, and shows side-by-side how TSP handles both failure cases better.

---

## How It Works

### The TSP Pipeline

When the device enters the "sick" state (health index drops below threshold), TSP builds an ARIMA model from the 30 pre-sick data samples using the following steps:

1. **Log transform** — applied if variance grows with time, to stabilise it.
2. **ADF test** — checks if the series is stationary; if not, first-order differencing is applied.
3. **ACF & PACF analysis** — identifies the dominant lag structure to bound the AR and MA orders.
4. **Ljung-Box test** — confirms the series has exploitable autocorrelation before fitting.
5. **BIC grid search** — fits all ARIMA(p, d, q) combinations within the identified bounds and selects the one with the lowest Bayesian Information Criterion (penalises complexity to avoid overfitting).
6. **Rolling forecast** — from sick onset onward, the selected ARIMA model is re-fitted at every new span using all available data, and a one-step-ahead prediction is made.

The exponential baseline is fitted in parallel at each step for direct comparison.

### Synthetic Data

The script generates a four-phase throttle-valve signal that mirrors the paper's Example I:

| Phase | Spans | Behaviour | What it tests |
|---|---|---|---|
| Pre-sick | 267–296 | Gentle rise, low noise | Modelling window for TSP |
| Flat / sick | 297–378 | Slow linear drift | Case A — flat trend |
| Sudden rise | 379–384 | ~1.9 units/span jump | Case B — abrupt spike |
| Oscillation | 385–375 | Sinusoidal near dead spec | Near-failure instability |

---

## Project Structure

```
.
├── tsp_algorithm.py   # Main script (data synthesis + TSP + exponential + plot)
└── README.md
```

---

## Requirements

```
numpy
matplotlib
scipy
statsmodels
```
---

The script will print diagnostic output for each TSP step to the console and display a single plot comparing:

- **Red stars** — actual aging feature $Y_T$
- **Pink line** — TSP one-step-ahead prediction
- **Navy dashed line** — exponential model prediction
- **Gold dashed** / **orange solid** lines — sick and dead specification thresholds

---

## Sample Output (Console)

```
[Step 5] Log transform applied: False
[Step 6] ADF p-value: 0.0023 | Stationary: True
[Step 8] ACF max-lag A=1 | PACF max-lag B=2
[Step 9] Ljung-Box p-value: 0.0312 | White-noise: False
[Step 11-13] Searching ARIMA combinations by BIC ...
Best order: ARIMA(2, 0, 0) | BIC = -142.817
Final TSP model: ARIMA(2, 0, 0)
```

---

## Key Observations from the Plot

- During the **flat phase**, the exponential curve drifts upward while TSP stays close to the actual signal — resolving the paper's Case A problem.
- During the **sudden rise**, TSP reacts within one span; the exponential model lags by several spans — resolving Case B.
- During **oscillation near the dead spec**, both methods become noisy. The paper addresses this with a Pre-Alarm Module (PreAM) and Death Correlation Index (DCI), which are not implemented here.

---




