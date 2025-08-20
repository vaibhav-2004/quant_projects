# quant_projects
Great progress. Here’s what’s left in **Phase 1** and exactly how to approach each item (no code—just the what/why/how + acceptance checks).

# 1) Add Trading Costs & Execution Frictions

**Why:** Make results realistic; fast, whipsaw-y strategies can die once costs are included.

**What to decide (put in a config):**

* **Commissions:** pick one model

  * Flat per trade (e.g., `$1.00`), or
  * Per share (e.g., `$0.005/share`), or
  * % of notional (e.g., `0.01%`).
* **Slippage:** price impact at execution

  * Start with `5–10 bps` (0.05–0.10%) of price.
  * Apply **+bps** for buys, **-bps** for sells.
* **Execution timing:** keep **next-day open** (done).

**How to apply (conceptually):**

* On **buy**: effective price = `ExecPrice * (1 + slippage)`; subtract commissions from cash.
* On **sell**: effective price = `ExecPrice * (1 - slippage)`; subtract commissions from proceeds.

**Acceptance checks:**

* Report shows: total trades, total commissions paid, average slippage per trade.
* Strategy metrics degrade moderately vs zero-cost run (not catastrophically unless the signal is too noisy).

---

# 2) Add Position Sizing (not all-in)

**Why:** Control risk/exposure; reduce sensitivity to signal noise.

**Options to try (enable via config):**

1. **Fixed fraction**: e.g., 50% of equity on each entry.
2. **Volatility targeting**: target an annualized vol (e.g., 10%); position size = target\_vol / recent\_realized\_vol.
3. **Capital cap**: never allocate more than X% (e.g., 80%) to a single position.

**Acceptance checks:**

* Equity curve changes smoothly when you vary the sizing knob.
* Exposure (%) and turnover are reported.

---

# 3) Add Risk Controls: Stops / Targets

**Why:** Prevent large losses and reduce time in bad trades.

**Rules to test (independently toggled):**

* **Stop-loss:** exit if price falls `X%` below entry (try 5–10%).
* **Take-profit:** exit if price rises `Y%` above entry (try 10–20%).
* **Trailing stop:** exit if price drops `Z%` from post-entry peak (try 5–10%).
* **Exit precedence:** stop/target exits override “opposite signal” exits, or vice versa—decide and document.

**Acceptance checks:**

* Trade log shows exits labeled with the reason (stop, target, opposite signal).
* Drawdowns improve (or not); record the impact on hit rate and profit factor.

---

# 4) Expand Performance Metrics

**Why:** Sharpe + MaxDD aren’t enough to understand behavior.

**Add these to your report:**

* **Sortino Ratio** (downside-only risk).
* **Calmar (or MAR)** = CAGR / MaxDD.
* **Hit Rate** (win %) and **Avg Win / Avg Loss**.
* **Profit Factor** (gross profits / gross losses).
* **Turnover** (annualized sum of notional traded / avg equity).
* **Exposure** (% time in the market).

**Rolling/periodic views:**

* **Rolling 6- or 12-month Sharpe/Sortino**.
* **Year-by-year returns**.

**Acceptance checks:**

* One “tear sheet” shows a complete picture (CAGR, risk ratios, drawdown curve, rolling stats).

---

# 5) Better Visuals for Diagnosis

**Why:** Quickly see regime dependence and pain points.

**Add plots:**

* **Equity curve vs Buy & Hold** on the **same scale** (not rescaled/normalized separately).
* **Underwater (drawdown) plot**.
* **Rolling Sharpe/Sortino** timeline.
* (Optional) **Trade markers** colored by profit/loss on the price chart.

**Acceptance checks:**

* From the plots alone, you can tell where/when the strategy suffers (e.g., choppy sideways periods).

---

# 6) Trade Log Enhancements (auditability)

**Why:** To verify correctness and debug.

**Track per trade:**

* `signal_date`, `action`, `exec_date`, `exec_price_raw`, `exec_price_after_slippage`, `commission`, `shares_delta`, `position_after`, `cash_after`, `equity_after`, `exit_reason`.

**Optional: per-trade P\&L (round trips):**

* Pair BUY→SELL to compute `$` and `%` P\&L, holding period (days), MAE/MFE (max adverse/favorable excursion).

**Acceptance checks:**

* You can answer: “Why did we exit this trade?” and “What was the realized P\&L?”

---

# 7) Data & Parameter Hygiene

**Why:** Avoid hidden changes that pollute comparisons.

**Decide & lock:**

* **`auto_adjust`** in yfinance (True recommended for consistency).
* **Parameter registry:** fast/slow MA windows, stop/target percentages, costs/slippage all live in one config section.
* **Random seeds** (if you later bootstrap).

**Acceptance checks:**

* Rerunning with the same config reproduces the same metrics and the same trade log.

---

## Suggested order to implement (fastest path)

1. **Costs:** add commissions + slippage; rerun and record before/after metrics.
2. **Sizing:** add fixed-fraction sizing; compare 100% vs 50% vs 25%.
3. **Stops:** add a simple fixed stop-loss; test 5%, 7.5%, 10%.
4. **Metrics & plots:** add Sortino/Calmar + underwater + rolling Sharpe; generate a one-page tear sheet.
5. **Trade log fields:** add reasons + per-trade P\&L.


Awesome — here’s a **clear, staged roadmap** from your current MA crossover backtest to a solid, “near-pro” research stack. No code, just what to do and how to validate each step.

# Phase 0 — Lock In Baseline (You’re here)

**Goal:** Make your current results reproducible and audit-able.

* **Freeze inputs:** Fix ticker(s), date range(s), and data vendor. Save raw CSVs to disk.
* **Config > constants:** Move parameters (MAs, starting\_cash) into a single config dict or YAML.
* **Deterministic runs:** Seed any random processes (later, for bootstraps).
* **Trade log:** Record each trade with: date, signal type, price, shares, fees, P\&L, cash, equity.
* **Acceptance check:** Running twice yields identical metrics; trade count & dates match.

---

# Phase 1 — Make It Realistic

**1. Costs & Execution**

* **Add commissions** (e.g., per share) and **slippage** (e.g., ±bps of price, or volume-based).
* **Execution timing:** Signals at close → trade at next open (avoid look-ahead).
* **Partial fills:** Cap trade size by a % of daily volume (e.g., 5%) to approximate liquidity.
* **Acceptance:** Report shows lower returns than zero-cost version; trade log includes fees; no trades exceed liquidity cap.

**2. Position Sizing & Risk**

* **Sizing rules:** Fixed fraction (e.g., 50% of equity), volatility targeting (e.g., 10% annualized), or risk per trade (ATR-based stop distance).
* **Stops/Targets:** Add stop-loss (e.g., 2× ATR), trailing stop, and take-profit variants. Test “exit on opposite signal” vs. stop/target.
* **Acceptance:** You can toggle sizing/stop variants via config and see expected changes in drawdown and turnover.

**3. Metrics Upgrade**

* Add **Sortino**, **Calmar**, **MAR (CAGR/MaxDD)**, **rolling Sharpe/Sortino**, **hit rate**, **profit factor**, **average win/loss**, **exposure (%)**, **turnover**.
* **Charts:** Equity curve vs Buy\&Hold (same scale), rolling 6-/12-mo Sharpe, drawdown curve, underwater plot.
* **Acceptance:** A one-page “tear sheet” summarizes the strategy cleanly.

---

# Phase 2 — Avoid Fooling Yourself (Robustness)

**4. Parameter Robustness**

* **Grid/Random sweep** of MA windows (e.g., fast=5–30, slow=40–200), with heatmaps of total return, Sharpe, and MaxDD.
* **Stability tests:** Report not just the best combo but how *many* nearby combos work (parameter insensitivity).
* **Acceptance:** The “good” region is a plateau, not a sharp spike.

**5. Train/Test Discipline**

* **Walk-forward:** Split time into sequential folds (e.g., 3–5). Optimize on fold k, test on k+1. Aggregate OOS results.
* **Anchored OOS:** Expand training window over time and always test on the next chunk.
* **Acceptance:** You can report OOS CAGR/Sharpe distinct from IS, with similar behavior (no collapse).

**6. Resampling & Stress**

* **Block bootstraps** of daily returns to see dispersion of outcomes.
* **Regime stress tests:** Evaluate performance in distinct regimes (bear, bull, sideways; high/low vol).
* **Acceptance:** You can show confidence intervals and know where the strategy fails.

**7. Bias Checks**

* **No look-ahead:** Only use info available at trade time; trade next bar.
* **Survivorship bias:** If you add more tickers, ensure delisted names are included (use index constituents by date if possible).
* **Data snooping:** Limit how many times you iterate parameters without a holdout.
* **Acceptance:** A brief “assumptions & biases” section accompanies results.

---

# Phase 3 — Broaden Scope (Portfolio & Filters)

**8. Multi-Asset / Multi-Ticker**

* Apply same logic to a **basket** (e.g., AAPL, MSFT, AMZN, GOOG, NVDA) or sector ETFs.
* **Weighting:** Equal weight, vol-scaled, or risk parity; rebal at fixed intervals or on signals.
* **Acceptance:** Portfolio metrics/tear sheet + per-asset contrib to return and drawdown.

**9. Signal Filters**

* **Confirmations:** Only act on MA cross if RSI is oversold/overbought, or price above/below 200-day MA, or volume spike confirms.
* **Regime filter:** Trade only when market (SPY) is above its 200-day EMA, or VIX below X.
* **Acceptance:** Filters reduce whipsaws (lower turnover, higher profit factor) without killing returns.

**10. Capital Efficiency**

* **Cash yield:** Earn short-rate on idle cash (changes results meaningfully over long windows).
* **Leverage:** If using, apply vol targeting to keep risk consistent.
* **Acceptance:** Exposure and risk stay within policy; drawdowns don’t balloon disproportionately.

---

# Phase 4 — Industrialize the Research Loop

**11. Research Framework**

* **Configs & runners:** One command per experiment; outputs go to timestamped folders.
* **Artifact logging:** Save config, metrics JSON, figures, and trade logs per run.
* **Versioning:** Git tag data snapshots and parameter sets. Keep a RUNS.md changelog.

**12. Comparative Reporting**

* **Leaderboard:** A small table comparing top runs by OOS Sharpe/Calmar with notes on filters/sizing.
* **Diagnostics:** Per-year returns, worst month, average drawdown length, time to recovery.
* **Acceptance:** You can reproduce and rank any past result in minutes.

**13. Libraries (when ready)**

* Migrate to **vectorbt**, **backtrader**, or **zipline** for speed and correctness.
* Add **quantstats/pyfolio** for pro-level reports.
* **Acceptance:** Your custom logic matches library results on a small validation case.

---

# Phase 5 — Stretch (Only if you want to explore more)

* **Intraday data:** Test on 30m/60m bars; revisit slippage/liquidity models.
* **Position optimization:** Kelly-style scaling with risk caps; CVaR constraints.
* **Ensemble strategies:** Combine MA crossover with a mean-reversion sleeve; allocate by recent efficacy.
* **Light ML:** Regime classification (e.g., vol/return features) to turn strategy on/off; still keep simple, interpretable rules.

---

## Deliverables Checklist (per experiment/run)

* ✅ Config file (parameters, assets, dates, costs, sizing, filters)
* ✅ Trade log CSV (entries, exits, size, fees, P\&L, equity)
* ✅ Metrics JSON (CAGR, Sharpe, Sortino, Calmar, MaxDD, turnover, exposure)
* ✅ Plots (equity vs B\&H, drawdown, rolling Sharpe, parameter heatmap)
* ✅ Notes on assumptions/bias, key observations, next hypothesis

---

## Common Pitfalls to Avoid

* Trading at the same bar as the signal (look-ahead).
* Ignoring fees/slippage (inflates Sharpe).
* Optimizing parameters on all history and reporting that as “results”.
* Declaring victory after one period — always show OOS and multiple regimes.
* Over-filtering until you have almost no trades (great Sharpe, zero capacity).

---

If you want, I can turn this into a **step-by-step checklist for just your next 2–3 sessions** (e.g., “add costs & next-bar execution → add trade log → add rolling metrics → produce a tear sheet”), so you ship improvements quickly.


