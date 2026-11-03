# AlgoGuard

A rule-based algorithmic trading engine for NSE equities with an enforceable risk layer.

Research question: **does a pre-committed, automatically enforced risk layer reduce tail risk (max drawdown, worst days, loss clusters) for common retail strategies, and what does it cost in returns?**

This grows out of last semester's TradeGuardian idea. A third-party app cannot stop a person from placing a manual trade in their broker's app. But when an algorithm places the orders, the risk engine sits in the execution path, so enforcement is real. That is the pivot.

## Run it (Windows, Anaconda/Miniconda Prompt)

```
cd algoguard
conda create -n algoguard python=3.11 -y
conda activate algoguard
pip install -r requirements.txt
python -m pytest -q tests          # 7 checks incl. no look-ahead
python run.py --download           # fetch NSE data (once, needs internet)
python run.py                      # all experiments -> results\
python paper_trade.py --days 60    # paper-trading replay through the broker layer
streamlit run app.py               # live dashboard for the demo
```

Or double-click `run_all.bat` from an Anaconda Prompt. Offline? Add `--demo` to `run.py` / `paper_trade.py` to use synthetic data (pipeline test only, not market results).

Everything runs on a normal laptop CPU in well under a minute. No GPU needed.

## What is inside

| Module | Purpose |
|---|---|
| `algoguard/data.py` | NSE daily OHLCV, NIFTY 50 and India VIX via yfinance, cached to CSV |
| `algoguard/strategies.py` | EMA crossover (trend), Donchian breakout (momentum), RSI-2 (mean reversion). Long-only |
| `algoguard/risk.py` | The guardrail engine: ATR stops, daily loss limit, losing-streak cooldown, drawdown circuit breaker, India VIX filter, volatility-scaled sizing. Every intervention logged with its reason |
| `algoguard/backtest.py` | Event-driven daily portfolio backtester. Signals at close, fills at next open, stops intraday, 0.15% cost per side |
| `algoguard/metrics.py` | CAGR, Sharpe, Sortino, max drawdown, Calmar, worst day, VaR, profit factor |
| `algoguard/broker.py` | Execution layer built around SEBI's retail algo framework: 10 orders/s limiter, MPP-protected limit orders, daily login, strategy tags. SimBroker + Kite Connect mapping |
| `run.py` | Experiments: raw vs guarded, in/out-of-sample, ablation, parameter grid, cost sensitivity, intervention counts |
| `app.py` | Streamlit dashboard: toggle each rule and see the effect live; audit log |
| `paper_trade.py` | Day-by-day replay through the broker layer |
| `tests/` | No look-ahead, rules fire, costs bite, no leverage, rate limiter, MPP pricing |

## Outputs (`results/`)

`report.md` (all tables), `chart_1_equity_drawdown.png`, `chart_2_ablation.png`, `chart_3_robustness_grid.png`, per-strategy trade lists and audit logs as CSV.

## Design choices worth defending

- **No look-ahead**: decisions for day t use data up to close of t-1; tested by perturbing future prices.
- **Parameters fixed before testing** (textbook values), with a 2022+ hold-out period, so results are not curve-fitted.
- **Ablation** shows which rule actually does the work instead of claiming the whole bundle helps.
- **Realistic costs**: STT 0.1% per side on delivery is the dominant cost in Indian equities.

## Limitations

Survivorship bias (today's large caps), daily bars only, unofficial Yahoo data, long-only cash segment. See the end of `results/report.md`.
