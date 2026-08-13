# Vision & Mission

*Last updated: 2026-07-21*

## Mission

Build a long-term software platform — an **autonomous quantitative research organization**, not a "trading bot" — capable of:

- Collecting and storing massive amounts of market data
- Detecting changing market regimes
- Generating new trading ideas automatically
- Backtesting new strategies
- Validating strategies with rigorous statistical testing
- Paper trading successful strategies
- Deploying only proven strategies to live trading
- Continuously monitoring live performance
- Reducing capital allocation to underperforming strategies
- Retiring strategies that no longer have a statistical edge
- Generating and testing new strategies to replace them

The platform should improve over time while avoiding overfitting and maintaining strict risk management.

## Design Philosophy

Never assume any strategy will work forever. Markets evolve; the software should expect that and adapt.

Emphasis on:
- Statistical evidence over intuition
- Scientific experimentation (hypothesis → test → validate → retire)
- Modular software architecture
- Automation
- Continuous learning
- Risk management
- Explainable decisions
- Long-term maintainability

## Core Architecture

**Data Layer** — OHLCV, tick data, order book (future), economic calendar, news, options data (future), market internals, VIX, treasury yields, correlation data.

**Database Layer** — store everything: every candle, feature, prediction, trade, strategy, model version, backtest, performance metric. Nothing discarded.

**AI Research Agents**
- **Research Agent** — generates strategy ideas, identifies patterns, forms hypotheses
- **Quant Agent** — statistical validation: walk-forward, cross-validation, Monte Carlo; rejects weak strategies
- **Risk Manager** — position sizing, exposure, drawdown limits, correlation limits, capital preservation
- **Portfolio Manager** — dynamic capital allocation across strategies by performance and regime
- **Execution Agent** — order placement, entries/exits, slippage minimization
- **Performance Agent** — evaluates every trade, tracks degradation, measures long-term edge
- **Evolution Agent** — retires poor strategies, creates variations, promotes candidates
- **Supervisor Agent** — coordinates all components, makes deployment decisions

**Strategy Framework** — every strategy is a plug-in competing for capital; none are permanent. Examples: Opening Range Breakout, Trend Following, Mean Reversion, Momentum, VWAP, Liquidity Sweeps, Market/Volume Profile, Statistical Arbitrage, ML models.

## Continuous Learning Pipeline

```
Collect Data → Engineer Features → Detect Regime → Generate Ideas → Backtest
→ Validate → Paper Trade → Deploy Small → Monitor → Adjust Allocation
→ Retire Weak Strategies → Generate New Strategies → Repeat
```

## Risk Philosophy

Protect capital above everything else.
- Max drawdown limits, daily loss limits, portfolio exposure limits
- Dynamic, volatility-adjusted position sizing
- Automatic shutdown conditions
- Strategy health monitoring
- Never deploy without statistical validation; never trust a single backtest
- Always separate train / validation / out-of-sample

## Technical Stack (preferred)

Python, Polars, DuckDB, PostgreSQL, Redis, FastAPI, Docker, Git/GitHub, vectorbt, Backtrader, PyTorch, LightGBM, XGBoost, Plotly.

## Development Philosophy

Multi-year project, built like a professional software company: clean code, full documentation, reusable modules, automated testing, versioned models and strategies, logged decisions. No shortcuts.

## Infrastructure

- **Primary server**: Mac mini, 24/7 — data collection, DB, agents, research jobs, backtesting, paper trading, live execution, dashboard.
- **Dev workstation**: MacBook.
- Future: expansion across multiple machines / cloud.

## Cost Philosophy

Minimize expenses early — free software, local hardware, paper trading, free data where viable. Spend only after the platform proves itself, prioritizing: professional market data → better infra → more compute → larger datasets.

## Long-Term Goal

A professional-grade autonomous quant research platform that continuously adapts to changing markets by discovering, validating, deploying, monitoring, and retiring strategies automatically — prioritizing statistical robustness, disciplined risk management, and modular engineering over chasing short-term profits.

---

**Note to future Claude sessions**: this document is the foundation for all architecture decisions, coding guidance, and implementation plans on this project. See [[Enhancement Ideas]] for additions layered on top of this original vision, and [[Platform Status]] for what's actually been built so far vs. what's still just planned.
