# Enhancement Ideas

*Additions layered on top of [[Vision & Mission]]. Log new ideas here with a date as they come up — treat this as a living note, not a finished spec.*

## 2026-07-21 — Initial profitability & robustness additions

### Fighting overfitting (biggest risk given the design)
The platform will generate and test hundreds of strategy variants — that's a multiple-testing problem, and some will look great by chance alone.
- Use **Deflated Sharpe Ratio / Probabilistic Sharpe Ratio** so promotion thresholds account for how many variants were tried.
- Use **Combinatorial Purged Cross-Validation** (Lopez de Prado) instead of plain walk-forward — plain walk-forward leaks information across nearby time folds.
- Track "number of trials per strategy family" so the Quant Agent's bar rises automatically as the Evolution Agent explores more variants of the same idea (Bonferroni-style correction).

### Alpha diversification — the real profitability lever
- Portfolio Sharpe scales roughly with `sqrt(number of uncorrelated bets)` (Grinold's Fundamental Law of Active Management). A mediocre but uncorrelated strategy is worth more than a great but correlated one.
- Bias the Research Agent toward *uncorrelated* new ideas, not just high-backtest-Sharpe ones. Argues for expanding across asset classes (equities/futures/FX/crypto) sooner rather than perfecting a single strategy family.
- **Meta-labeling**: separate "should I take this signal" (a secondary filter model) from "which direction" (the primary strategy). Often the single biggest realized-edge improvement in ML-driven systems — reduces false positives without touching the primary signal logic.

### Realistic cost modeling
- Model slippage, spread, fees, and borrow cost *inside* the backtest, not applied after the fact. Most common reason backtests overstate real returns.
- Track **capacity** per strategy — the AUM level at which market impact eats the edge. Matters even at small account sizes for less-liquid strategies.

### Smarter capital allocation than static weights
- Consider Hierarchical Risk Parity or an online bandit-style allocator that shifts weight toward recently-outperforming strategies, with priors that prevent overreacting to noise.
- Give each strategy an explicit "edge decay curve" / expected half-life so retirement is a principled forecast rather than a reactive rolling-Sharpe trigger.

### Institutional memory, not just a graveyard
- When a strategy is retired, write a structured post-mortem: why did it die (regime shift, crowding, cost-structure change, data revision)? Feed this back into idea generation so the Evolution Agent doesn't regenerate dead-end variants.

### Data integrity
- Point-in-time discipline: store data "as known at time T," not restated/current values. Economic and fundamentals data get revised after the fact; using today's revised value in a historical backtest is a subtle but real source of phantom edge.

### Governance
- Keep a human-approval gate before capital moves from paper to *live* trading, at least in the early phase, plus a dead-simple kill switch.
- Full autonomy in research/paper trading; a human checkpoint before real money. Not a compromise on the vision — it's what keeps the system survivable while it's still being trusted.

## 2026-07-21 — Research Agent capabilities: what's realistic and when

User asked for an agent "constantly researching, backtesting, coming up with, and dumping old strategies" plus a 24/7 news-monitoring agent — the Research/Evolution/Performance Agent roles from [[Vision & Mission]]. Split into what's tractable now vs. genuinely later, see [[Platform Status]] item 18 for what actually got built from this (a parameter-search + promotion-tracking loop over the *existing* 3 strategy families).

### LLM-driven / novel strategy generation — deliberately deferred
The "coming up with" part of the Research Agent — inventing genuinely new strategy shapes, not just searching known ones' parameter space — was scoped out of the v1 research loop on purpose. Reasons: executing LLM-generated trading logic automatically is a real safety/quality concern (bad generated code, subtle look-ahead bugs, or a strategy that "backtests well" by exploiting a data quirk), and there's no established pattern yet in this codebase for *validating* an arbitrary generated strategy the way `Strategy.generate_signals()` is validated today (unit-testable interface, but no automatic check that a wholly new strategy doesn't cheat). If pursued later: probably start with LLM-*assisted* ideation (a human reviews and codes up the suggestion, same as SMA/Bollinger/breakout were each manually written) before ever letting generated code run through the pipeline unsupervised.

### 24/7 news monitoring — deferred, and worth real skepticism
Public news is priced into liquid tickers (this project's whole universe) very fast — professional firms with direct feeds and colocated infrastructure typically act before a retail-scale system could. A news signal is not an assumed edge just because it's real-time or LLM-analyzed; it must clear the *same* backtest/walk-forward/DSR/risk-gate bar as everything else before being trusted enough to paper trade. Practical blockers: needs a real data-source decision (good real-time financial news feeds are usually paid, not something to sign up for without asking first) and genuinely continuous infrastructure (shouldn't run on hardware that sleeps — sequence after the Mac mini). Worth treating as its own project with its own validation pipeline, not an assumed extension of the price-based one.

### Formalizing "automation proposes, a human decides"
This has been the practical rule for every automation feature built so far (see [[Platform Status]]'s "Governing principle" note) — it's the Governance section above, applied at every layer, not just the paper→live boundary it was originally written for. Worth keeping explicit as the project adds more autonomous-sounding pieces: a new promoted combo joining paper trading, a demoted combo being pulled, a research cycle running on a schedule — none of these should become automatic without a deliberate decision to make them so, even once the infrastructure to do it exists.

## 2026-07-22 — Can this actually beat the S&P 500?

Prompted by the results dashboard artifact ([[Platform Status]] item 23), which put an S&P 500 buy-and-hold line next to the 3 paper-traded strategies for the first time: SPY buy-and-hold returned +264.3% over 2015–2026 (worst drop -34.1%) versus the strategies' +95.9%/+85.2%/+70.3% (worst drops -24.6%/-11.0%/-12.3%). User asked directly whether the platform could be made to beat that. Full discussion logged in [[Platform Status]] item 24 — summary here since it's the kind of forward-looking idea this note is for.

**Why it's unlikely with more of the same**: single-asset, once-daily trend/mean-reversion/breakout strategies are structurally built to sit out some of the market's up days in exchange for shallower drawdowns — they're not shaped to out-earn a fully-invested index, they're shaped to be steadier. That tradeoff is visible directly in the numbers above: every strategy has a smaller worst-case drop than SPY, none has a bigger total return.

**Four real levers, not yet scoped or built:**
1. **Leverage on already-validated signals** (2x ETF, margin) — fastest, no new infrastructure, but amplifies drawdowns too and leveraged ETFs carry volatility-decay risk in choppy markets. More risk on the same edge, not a smarter one.
2. **Stack more genuinely uncorrelated strategies, then leverage the combined portfolio** to a similar risk level as SPY — the textbook answer (same Grinold's Law logic as the "Alpha diversification" section above), and the more legitimate long-term path. Needs a portfolio-level backtest engine that doesn't exist yet (today's engine only ever tests one ticker in isolation) — a real build, not a quick add.
3. **Meta-labeling** (already listed above under "Alpha diversification") — size up on higher-confidence signals instead of flat-sizing every trade equally.
4. **New asset classes with different return shapes** (futures, options premium-selling, eventually crypto) — real diversification, real new risk types, already understood as separate, deferred, bigger-project work elsewhere in this note.

**A trap worth naming explicitly**: optimizing specifically to "beat SPY over this historical window" is exactly the kind of target that invites curve-fitting a strategy to have looked good in hindsight — the same selection-bias problem DSR exists to correct for, just aimed at a benchmark instead of a Sharpe ratio. The goal stays "genuinely validated, resilient edges," not "beat one benchmark on one backtest."

## 2026-07-30 — Short-interest signal: data access checked, real hypothesis ambiguity flagged

Picked as the next idea to scope after volatility selling hit a real data wall (see below) — chosen over post-earnings drift specifically because FINRA short interest is a straight regulatory disclosure (same category as the Congress-trading data already built against), a safer bet against another paywall than FMP's earnings-surprise coverage, which hasn't been verified yet.

**Worth flagging before building anything**: "heavily-shorted names with rising short interest" (this idea's original framing above) can mean two different, contradictory trades — (1) a squeeze bet: shorts are wrong, crowded short positioning plus a price move up forces covering and pushes price higher, a bullish signal; or (2) the more academically robust finding: short sellers tend to be sophisticated, informed traders, so heavy short interest actually predicts *underperformance*, a bearish signal. These aren't a matter of framing — they're opposite trades. Next step is testing which one the real data actually supports before picking a direction.

**Data access, checked directly (not trusted from docs alone):**
FINRA's real API — `api.finra.org/data/group/otcMarket/name/ConsolidatedShortInterest`, free, no key — confirmed via a live AAPL query. The docs page's URL naming ("otcMarket") misleadingly suggests OTC-only, but the actual data includes real exchange-listed stocks (AAPL came back tagged `NNM`/NASDAQ) — caught by testing directly rather than trusting the WebFetch summary of FINRA's own catalog page, which had claimed no exchange-listed dataset existed. Confirmed current (AAPL's 2026-07-15 settlement already published) and deep (207 bi-monthly records for AAPL alone, ~8+ years).

**Status**: data access confirmed, real. Next: test whether rising short interest historically predicts outperformance or underperformance before deciding which direction to build.

### 2026-07-31 — Short-interest signal tested against full history: no robust edge, not building it

First pass used only 6 months of 2026 data (13 settlement dates) and showed the most-shorted quintile beating the least-shorted by +1.32% (21-day forward return) — the squeeze hypothesis, opposite of the "informed shorts" hypothesis flagged as more academically robust going in. Treated that as too short a window to trust, same lesson as the funding-rate check's first pass, and pulled FINRA's full real history before concluding anything: 206 real settlement dates (2017-12-29 → 2026-07-15, dates shift for weekends/holidays, extracted from AAPL's own history rather than guessed), full universe each date, 3.88M rows, 48,618 symbols.

**Pooled result across the full 8.6 years: no edge.** Most-shorted quintile averaged +0.69% forward 21-day return, least-shorted averaged +0.66% — a +0.03% spread, indistinguishable from noise. Middle quintiles actually outperformed both extremes (a hump shape, not a trend).

**By year, the sign flips constantly and doesn't track any obvious pattern**: positive 2017-2020, **negative in 2021** (notable — the GameStop/meme-stock squeeze era, but a handful of spectacular individual squeezes don't move the average of the whole most-shorted quintile, which is thousands of stocks), negative again 2022 and 2024, positive 2025-2026. 70% of years had a positive spread, but sizes are small and inconsistent — nothing like funding-arb's persistent, repeatable pattern across regimes.

**Conclusion: not building this.** Unlike the volatility-selling wall (blocked on data access) or funding-arb's thin-current-period question (data was fine, just needed more of it), this is a case where deeper history didn't rescue the signal — it dissolved it. A simple rank-by-short-interest-level sort doesn't show a robust edge in either direction once tested properly, so neither hypothesis flagged at the start holds up.

**Not ruled out, just not pursued**: a more refined construction (change in short interest rather than level, combined with size/sector/liquidity controls, or focused on extreme-tail percentiles rather than quintiles) might show something this simple test can't see — but that's a materially deeper investigation than what was scoped here, not a quick next step. Worth revisiting only with a specific, motivated refinement in mind, not as a default next check.

### 2026-07-31 — Post-earnings drift (PEAD): data access checked, real and usable

Moved to this idea next since short-interest didn't hold up. Checked whether the existing FMP subscription (already paid for, used for Congress/House-disclosure data) actually includes earnings-surprise data — confirmed via direct testing, not assumed, same discipline as everywhere else in this doc (this project's own `stock_news.py` already warns "having a working FMP_API_KEY does not mean [a given] access is included").

**Real restriction found, and a real way around it**: `/stable/earnings?symbol=AAPL` works on the current key, but passing a `limit` parameter hits a hard cap ("limit must be between 0 and 5 based on your current subscription"). Dropping `limit` entirely (with or without `from`/`to` date filters) returns the *full* available history in one call, unrestricted — 164 real actual-vs-estimated EPS/revenue reports for AAPL back to 1985. The legacy `/v3/earnings-surprises/` endpoint and the `/stable/earnings-surprises-bulk` endpoint are both dead/paywalled on this tier (403 and 402 respectively) — the per-symbol `/stable/earnings` endpoint is the one that actually works, and it wasn't the first or most obviously-named thing tried.

**Status**: data access confirmed, real, sufficient depth. Next: pull a real universe and test whether earnings surprises actually predict subsequent drift before building anything — not yet done.

### 2026-07-31 — PEAD historical test blocked on a real FMP quota, not a data-existence problem

Built a test universe from the actual S&P 500 constituents (pulled fresh from a public GitHub mirror of the official list, since FMP's own `/stable/sp500-constituent` endpoint is paywalled on the current plan) — all 503 tickers already had price history stored, no new price data needed.

First pull attempt (no pacing) got 22 of 503 tickers before hitting `402`/`429` errors. Rewrote with retry-and-backoff (same pattern already used elsewhere in this codebase for DB lock contention) and re-ran — this is where a real mistake happened: the retry loop had no circuit breaker, so when literally every remaining call started failing, it kept retrying all 503 tickers with escalating backoff instead of aborting early, and ran unsupervised in the background for **~12 hours** before finishing with zero successful rows. Confirmed directly afterward with a single fresh test call: `"Limit Reach. Please upgrade your plan..."` — a real subscription quota exhausted, not a transient rate limit; the 12-hour retry window proved waiting alone doesn't clear it.

**This is a different kind of wall than the other two ideas hit**: volatility selling had no free data anywhere; short-interest had fine data but no real edge. Here, the right data and the right endpoint both exist and are already paid for (confirmed working for a handful of calls) — the plan's usable quota is just too small to pull a broad universe's history in one sitting, and testing alone (plus one bulk attempt) used it up.

**Status**: blocked on quota, not resolved. Real path forward: check the FMP account dashboard for the actual quota and its reset cadence (daily vs. monthly) before trying again, and pace any future pull far more conservatively with an early-abort circuit breaker this time. Whether upgrading the FMP plan is worth it is a deliberate spend decision, not made here.

### 2026-07-31 — Quota partially reset overnight, real capacity measured: ~13 tickers/day

Retried with a real circuit breaker this time (abort after 3 consecutive failures, not blind full-budget retries) — 32 seconds total, not 12 hours. 13 of 20 tickers succeeded with substantial real history (63-165 records each) before the cap hit again.

**Real number now known**: roughly 13 tickers/day on the current plan. Pulling the full 503-stock S&P 500 universe this way would take over a month of daily, spaced-out pulling — impractical to do incidentally, and not something to leave running unattended (that's what caused yesterday's 12-hour incident).

**Status**: confirmed impractical at the current plan's pace without either (a) a deliberate multi-week trickle-pull, done carefully and manually paced, or (b) upgrading the FMP plan. Same real spend-decision shape as volatility selling's conclusion — not something to force around for free.

## 2026-07-30 — Volatility selling (covered calls / cash-secured puts): the honest edge exists, but real data doesn't

Checked as a candidate alongside funding-arb, for the same "return source that isn't predicting price direction" reason.

**The core edge is real** — checked directly with free data (FRED's VIX history since 1990, SPY price history), not assumed: implied volatility (VIX) averaged 19.5 vs. subsequent realized volatility of 15.9 over 33 years — a persistent ~3.6-point premium, positive on 82.8% of trading days. Broken out by year, this premium was positive almost every year except 2008 (the financial crisis) and nearly vanished in 2000/2018 — the same shape as funding-arb's risk profile: reliable most of the time, with real, identifiable tail-risk years where realized volatility suddenly overshoots what was priced in.

**Where this stalled**: turning that into an actual backtestable strategy needs real historical option prices (specific strikes/expirations), not just the VIX index. Checked directly, not assumed: Alpha Vantage's options endpoint needs a real signup and has limited coverage even then; the providers with real depth (Polygon, EODHD, CBOE DataShop) are paid; Deribit's crypto options API is free and live (940 real BTC contracts confirmed) but its own historical-volatility endpoint only covers ~2 weeks, no deep archive via the live API. Unlike funding rates, options chain history is a deliberately monetized data product industry-wide (CBOE sells it themselves), not a free side-effect of an exchange's trading engine — a structurally different situation, not just "hasn't been searched hard enough."

**Considered and rejected**: reconstructing historical option prices synthetically from VIX + a pricing formula (Black-Scholes). Rejected because a flat-volatility model can't represent skew (real markets price OTM puts richer than calls, due to crash-insurance demand) — and skew is concentrated in exactly the strikes a covered-call/cash-secured-put program trades. That's not a minor simplification, it's a blind spot pointed directly at what the backtest is trying to measure, risking a false-positive "validated" result. Given the platform's own risk philosophy (never trust a single backtest, protect capital first), building on that foundation was judged worse than not building at all.

**Status**: shelved, not abandoned. Real path forward if revisited: pay for real options data as a deliberate small spend, matching the platform's own cost philosophy ("spend only after the platform proves itself, prioritizing... better market data"). Not pursued further for now — redirected effort to the short-interest signal instead (see above), which has confirmed free real data.

**Status**: discussed, not decided. Conversation paused (user going to sleep) before picking a direction — pick this back up next session rather than re-deriving the options.

## 2026-07-30 — Non-directional / alternative-data ideas (asked for "out of the box")

Checked against what's already built (per git log: cross-sectional momentum, relative-value pairs, volume breakout, turn-of-month, gap fade, Copy-Congress backtest, House disclosures, Research Agent, expanded universe to stocks/crypto/futures) — everything below is genuinely new, not a restatement of existing items above.

**The common thread**: every strategy built so far profits from correctly predicting price *direction* on one asset at a time. All four ideas below are return sources that don't depend on direction at all — the highest-leverage move per the "Alpha diversification" section above (Grinold's Law: portfolio Sharpe scales with number of *uncorrelated* bets, not with how good any one bet is).

1. **Crypto perpetual funding-rate arbitrage** — now that the universe includes crypto, perp futures pay a periodic fee between longs and shorts depending on crowding. Long spot / short perp (or reverse) harvests that fee with price risk hedged out — market-neutral, unlike everything else built so far.
2. **Post-earnings announcement drift (PEAD)** — stocks that beat earnings tend to keep drifting in that direction for weeks; a well-documented, persistent anomaly driven by earnings-surprise data rather than price patterns. FMP (already used for House disclosures) also has earnings-surprise data, so most of the plumbing already exists.
3. **Short-interest / squeeze signal** — FINRA publishes free short-interest data twice a month. Heavily-shorted names with rising short interest, same "smart money may be wrong" spirit as the existing Congress/insider-trading work, just a different unconventional data source.
4. **Sell volatility instead of only betting on direction** — covered calls / cash-secured puts on existing paper positions profit from time decay and calm volatility, not price direction. Already flagged as a future asset class in the "Can this beat the S&P 500?" section above (lever #4); this reframes it as startable now, on top of positions already running, with minimal new infrastructure.

**Status**: proposed, not decided or scoped. User to pick a direction (or none) next session.

### 2026-07-30 — Funding-rate arb (idea #1 above): checked real history before deciding

First pass used OKX's public API, which only serves ~3 months of free history (Apr–Jul 2026) — that window showed a thin edge (~3%/yr annualized, barely above trading-fee breakeven), which looked unattractive. But that window happened to land in an unusually quiet stretch; a single quarter isn't enough to judge a strategy this cyclical, so pulled the full history before concluding anything.

Binance's *public data archive* (`data.binance.vision` — static files, not the geo-blocked trading API) has funding-rate history back to Jan 2020. Full picture, realized annual return from always holding long-spot/short-perp:

| Year | BTC | ETH |
|---|---|---|
| 2020 | 17.2% | 27.5% |
| 2021 | 30.6% | 37.5% (bull mania) |
| 2022 | 4.2% | 0.8% (crash year) |
| 2023 | 7.9% | 8.3% |
| 2024 | 12.0% | 13.0% |
| 2025 | 5.1% | 4.9% |
| 2026 YTD (6mo, annualized) | ~1.1% | ~0.4% |
| **6.5-yr average** | **~11.9%/yr** | **~14.2%/yr** |

Full history reverses the initial read: averaged across a real market cycle, this clears trading fees easily and beats cash/T-bills by a wide margin — the 3-month snapshot just landed in one of the thinnest stretches in six years. But the longer window also surfaces a risk the short window couldn't show: single-period funding has spiked sharply negative during stress (BTC -0.30% in one 8h window in 2020; ETH -0.30% to -0.36% during the 2022 crash) — exactly the scenario where the short-perp leg's liquidation risk (flagged when this idea was first proposed) actually bites, even though the position is "market neutral" on average.

**Conclusion: worth building**, not shelved. Open design question before building the backtest engine: always-on, or sit out when current funding drops below some threshold (current environment is a thin ~1%/yr run-rate, a bad stretch to launch into blind). That threshold decision changes both the expected return and the risk profile — needs to be made deliberately, not defaulted.

### 2026-07-30 — Threshold question, resolved empirically: always-on wins

Tested against the full 2020–2026 history: a naive last-period threshold (skip the trade when the prior period's rate was negative/below X) lost badly (-64% to -51% total return vs. +77.5%/+92.2% for always-on, BTC/ETH) purely from transition-fee bleed — funding oscillates around small thresholds constantly (964+ flips over the period at 0.15%/transition). Smoothing the signal (trailing 1-day through 3-week moving average) reduces the flip count a lot but every window tested still underperforms always-on; longer smoothing just gets closer, never crosses over.

Also checked whether gating could have at least dodged the worst single-period shocks (BTC -0.30% Mar 2020, ETH -0.36% May 2021, ETH -0.30%/-0.22%/-0.18%/-0.17% cluster Sept 2022): mostly no — several arrived abruptly from calm/positive territory the period immediately before, not a multi-day warning decline a threshold rule could react to in time.

**Decision: always-on, not threshold-gated.** This also reframes the real open design question — it's not "when to be in the trade" (settled: always), it's **position sizing / margin buffer on the short-perp leg**, sized to survive a repeat of one of those single-period shocks without liquidation. That's the next thing to scope if this gets built.

### 2026-07-30 — Margin-buffer/liquidation sizing, scoped

Corrected the risk mechanism first: position is long spot + short perp, so liquidation risk on the (isolated-margin) short-perp leg comes from price *rising* sharply, not falling — the funding-crisis dates found earlier (all deep-negative-funding, crash-associated) are actually the *safe* direction for that leg's margin health. Pulled hourly BTC/ETH price history (2020–2026, `data.binance.vision` klines) to measure the real risk directly instead of reasoning about it.

Two raw "worst case" numbers (BTC 8h=40.5%, ETH 8h=217.7%) turned out to be single-bar data glitches in Binance's historical klines (verified by inspecting neighboring bars — isolated spikes with no surrounding confirmation). After despiking:

| | BTC | ETH |
|---|---|---|
| Worst 8h adverse move (short) | 32.2% | 45.7% |
| Worst 24h | 38.5% | 48.8% |
| Worst 72h | 38.5% | 62.2% |

Both cleaned worst-cases are real, verified events — BTC 2020-03-13 (violent bounce right after the Black Thursday crash bottom) and ETH 2021-05-19 (the flash-crash-to-$1960-then-V-recovery day). Notably, both are the *same dates* flagged earlier for the worst negative-funding shocks — the crash phase costs funding, the bounce right after is where perp-leg liquidation risk lives. Same storm, two compounding risks back to back, not independent tail events.

**Sizing implication**: liquidation happens at roughly `1/leverage` price rise. Surviving the worst 72h move with a real safety margin needs roughly **1.5–2x leverage** on the short-perp leg (i.e. 50–65% of position value held as margin) under naive isolated margin. That's a lot of idle capital stacked on top of the 100%-committed spot leg, and it meaningfully eats into the ~12–14%/yr average edge found in the first pass — worse capital efficiency than it looks on paper.

**Real fork this surfaces**: isolated margin (simple, but this capital-inefficient) vs. portfolio/cross margin (exchange nets spot against the perp short, shrinking liquidation risk to basis divergence instead of raw price moves — much better capital efficiency, but needs a specific account tier/feature, not a given). Next scoping step if this moves forward: check whether cross-margin is realistically available before deciding the trade is worth building at all.

### 2026-07-30 — Cross-margin availability, checked: not yet, for U.S. retail

Important context surfaced along the way: crypto perpetual futures were effectively unavailable to U.S. persons until very recently (explains why Binance/Bybit geo-blocked us in step 1) — Coinbase launched the first CFTC-regulated version (nano BTC/ETH perpetuals, up to 10x leverage) in July 2025; Kraken followed with its own CFTC-regulated US perpetuals in June 2026. These two are the actual compliant venues for a US person, not the offshore exchanges pulled from for the historical data analysis above.

**Kraken US perpetuals**: confirmed via their support docs — launched **USD-only collateral**. Spot crypto holdings cannot margin a perp position on this product. Kraken does have a more flexible multi-collateral margin system that supports exactly this (spot offsetting a short perp) — but that's a *different*, non-US Kraken product; not what's live for US retail. Kraken's docs note collateral options may expand later.

**Coinbase US perpetuals (Coinbase Financial Markets)**: live since July 2025, 10x leverage on nano BTC/ETH. Margin mechanics for this specific US product weren't directly confirmable (page blocked fetch), but the closest documented comparison — Coinbase *International* Exchange's portfolio margin — explicitly nets only within the same margin account/instrument type, not spot vs. futures. Reasonable signal (not proof) the domestic product works similarly.

**Conclusion: design around isolated margin, not cross-margin.** Nothing indicates spot-collateralizing-a-perp-short is available on either compliant US venue today. The ~1.5–2x leverage sizing from the margin-buffer scoping step is the realistic near-term constraint, not a conservative fallback — cross-margin is a future capital-efficiency upgrade to revisit as these very-new (both <1yr old) products mature, not something to design the v1 backtest around.

### 2026-07-30 — Backtest/paper-trading build, scoped against the real codebase

Read `pairs.py`, `momentum_rotation.py`, `strategies/base.py`, `backtest/run.py`, `paper_trading.py`, `store/paper.py`, `data/fetch.py`, `db.py` before scoping, rather than guessing at the architecture.

**Key finding**: the codebase has two existing precedents for "doesn't fit the standard single-ticker `Strategy` interface." `pairs.py` collapses a two-leg position into one synthetic `close` series and reuses the entire existing pipeline (`Strategy`/`run_backtest()`/walk-forward/DSR) completely unchanged — works because that trade carries no leverage/margin mechanic. `momentum_rotation.py` instead is a standalone module with its own bespoke backtest function, explicitly *not* a `Strategy` subclass, and openly documents that it skips the full walk-forward/DSR validation for a lighter single train/test split. Funding-rate arb belongs in the `momentum_rotation.py` camp — the pairs.py trick collapses everything into a net-P&L number, which is exactly what makes it blind to the liquidation risk sized two steps ago (a hedged net position can still get its isolated perp leg wiped out by a raw price spike). That mechanic doesn't exist anywhere in this codebase yet.

**Pieces needed, roughly by effort:**
1. Data layer (small) — new `funding_rates` table, same pattern as `ohlcv_daily`, plus hourly-or-finer perp price. Open question: should source from Kraken/Coinbase (the real venue) going forward, not Binance (used for research only) — not yet checked whether they expose funding history publicly.
2. Cost/margin constants (trivial) — new shared module, following the existing `DEFAULT_FEES`/`DEFAULT_SLIPPAGE` single-source-of-truth pattern (a past bug here already taught this lesson once).
3. Backtest engine (the real lift) — standalone module, momentum_rotation.py-shaped: always-on entry, per-period funding accrual, liquidation check against intra-period high/low. Genuinely new math, not a reuse of anything existing.
4. Validation (small) — explicit lighter validation like momentum_rotation.py's, not the full 5-gate pipeline (that machinery assumes single-ticker `Strategy`). Mostly reuses conclusions already reached (always-on, ~1.5–2x leverage) rather than re-deriving them.
5. Paper trading (medium) — can't reuse `check_and_update()` (daily signal-timing state machine; this position has no daily entry/exit decision). Two real gaps: `paper_trade_events` is entry/exit only, no way to represent an ongoing funding payment while a position stays open (schema extension needed, not just logic); and daily-cadence checking (the existing convention) can't honestly represent the liquidation risk already sized, since that risk is intra-day — needs to check at least every funding period against intra-period highs to actually simulate what was scoped.

**Status**: scoped, not started. Before writing code: check whether Kraken/Coinbase expose funding-rate history via public API the way Binance does.

### 2026-07-30 — Kraken/Coinbase funding-rate API access, checked

**Kraken: yes, real public API.** The actual regulated venue underneath Kraken's new US perpetuals is Bitnomial Exchange (the CFTC-designated contract market Kraken acquired to launch this product). Bitnomial has a free, no-auth REST endpoint confirmed live: `GET https://bitnomial.com/exchange/api/v1/funding-rates/?base_symbol=BTCUC&begin_time=...`. Real BTC data from the product's actual launch (2026-05-17) through today, ~2.5 months, genuinely variable rates (not a placeholder). Good for going-forward paper trading against the real venue, not deep enough to backtest against — Binance's 6.5-year history stays the right source for that, as a reasonable proxy (same underlying economics, not the exact tradeable instrument). ETH's exact Bitnomial symbol code isn't "ETHUSD" (came back empty) — it's one of their other cryptic 4-letter codes, needs identifying at build time, minor.

**Coinbase: no easy path.** Funding-rate data is published via FIX and SBE — institutional binary market-data protocols, not a simple public REST endpoint. Materially higher integration bar than Bitnomial's plain HTTP API.

**Conclusion: build against Kraken/Bitnomial.** It's the compliant venue with the low-friction public API; Coinbase would be its own separate integration project if ever pursued.

**Scoping thread complete** (mechanics → historical edge → always-on decision → margin sizing → cross-margin availability → codebase architecture → data source). Ready to move to actual implementation whenever the user wants to start.

### 2026-07-30 — First implementation pass: built, and two real bugs found running it for real

Built the scoped v1: `data/funding_rates.py` + `data/perp_klines.py` (Binance research-history ingestion, `funding_rates`/`perp_klines_1h` tables), `funding_arb.py` (the always-on backtest engine, momentum_rotation.py-shaped as scoped), `store/funding_arb.py`, and `scripts/funding_arb_backfill.py` / `scripts/funding_arb_backtest.py`. Ran against the real `market.duckdb`, not just read back.

Two real bugs surfaced by actually running it, both fixed and worth remembering:

1. **DuckDB silently converts `TIMESTAMP` to local session time on insert.** This machine's DuckDB session timezone defaults to `America/New_York`. A plain `TIMESTAMP` column took incoming UTC values and silently stored them as local wall-clock time — on 2020-11-01, US DST's fall-back makes local 1am happen twice (once EDT, once EST), so two genuinely different UTC hours collided into one stored value, surfacing as a spurious primary-key violation during backfill. The crash was the only reason this got caught — without it, every timestamp in these tables would have been silently shifted off UTC. Fixed by using `TIMESTAMPTZ` instead. Worth checking any other future table that stores sub-daily timestamps in this codebase for the same issue — `ohlcv_daily` was never exposed to this because it only stores `DATE`, not `TIMESTAMP`.

2. **Liquidation check pinned one entry price for the entire 6.5-year backtest.** First run flagged BTC "liquidated" on 2020-07-27 (a 60.2% adverse move) and ETH on 2020-02-06 (71.8%) — both worse than, and on different dates than, the worst-case figures already found during margin-buffer scoping (BTC 38.5%/72h, ETH 62.2%/72h). Root cause: the simulation measured every period's price move against the *original* Dec-2019 entry price, so ordinary multi-month price drift (BTC rose ~40% from Dec 2019 to Jul 2020) masqueraded as a sudden shock — no real operator leaves 6.5 years of unrealized P&L unmanaged. Fixed by rebalancing margin to the target leverage at the start of every funding period (the tightest, most conservative of the three horizons checked during scoping) — an explicit, stated assumption, not a silent one. After the fix: **no maintenance-margin breach at BTC 2x / ETH 1.5x over the full 6.5-year history**, consistent with those leverage choices having been sized with a real safety margin during scoping.

**Validated results** (after both fixes, stored in `funding_arb_backtests`):
- BTC: +51.6% total return, Sharpe 9.04, max drawdown -1.0%, no liquidation breach at 2x leverage.
- ETH: +55.3% total return, Sharpe 8.45, max drawdown -0.8%, no liquidation breach at 1.5x leverage.

Note these are *lower* than the raw ~77.5%/92.2% "sum of funding rates" figures found during the historical-edge scoping — correctly so: those numbers assumed 100% of capital earns funding, but at 2x/1.5x leverage only ~67%/60% of capital is deployed as tradeable notional, the rest sits as idle margin buffer. This is the capital-inefficiency concern flagged during margin-buffer scoping, now quantified for real rather than just argued qualitatively.

**Status**: backtest engine built and validated. Paper trading built next — see entry below.

### 2026-07-30 — Paper trading built and verified against real Bitnomial data

Built `store/funding_arb_paper.py` (new `funding_arb_paper_events` table — entry/funding_accrual/liquidation, append-only, deliberately not sharing `paper_trade_events` since that table is entry/exit-only and built for a different, signal-timed strategy shape) and `funding_arb_paper.py` (orchestration: `check_and_update()`), plus `scripts/paper_trade_funding_arb.py` and a `funding_arb_paper_trade.sh` wrapper (not registered with launchd — scheduling a real recurring job is left as a deliberate decision, not auto-enabled).

Two data sources, kept split exactly like the backtest: Bitnomial (the real venue) for actual funding accrual, Binance hourly klines as the monitoring proxy for the liquidation check — Bitnomial's own public price history (their Charts API, confirmed via docs) is daily-only, too coarse to catch an intra-8h-period spike.

Verified against real data, not just read back: opened a real BTC position (notional $6,666.67, margin $3,333.33 at 2x), confirmed the idempotent no-op path (`NO_NEW_PERIODS` when nothing new has settled — correctly distinguished from a bug by checking Bitnomial's raw data directly: real UTC time was mid-period, nothing new had actually settled yet). Exercised the full accrual+liquidation-check loop by temporarily backdating a test entry 5 days, which pulled 14 real settled Bitnomial periods, correctly computed $9.33 cumulative funding P&L, and correctly found no liquidation breach — then cleared the test data and left a clean, real, freshly-opened position in place.

**Status**: v1 complete (backtest + paper trading), both validated against real data. Not yet run on a real recurring schedule — that's a separate decision, not made here.

### 2026-08-01 — Short-interest, refined: change-in-SI + liquidity control tested, still no edge

Followed up on the "not ruled out, just not pursued" note above — tested the refinement directly instead of leaving it as a hypothetical. Used the same full FINRA history already on disk (206 settlement dates, 2017-2026), no new data pull needed. Two changes from the original test: (1) signal is **change** in short interest (FINRA's own `changePercent` field, period-over-period) rather than the raw level, ranked into quintiles per settlement date; (2) added a liquidity control (avg daily dollar volume, computed from FINRA's average-volume field × price) since the original test pooled liquid mega-caps and illiquid microcaps together, which could mask a real signal in either direction.

**Pooled result, all liquidity: no edge.** Spread between "short interest grew the most" (Q4) and "shrank the most" (Q0) was -0.01% over 21 days — flat, and the sign flipped roughly at random by year (50% of years positive, a coin flip).

**Split by liquidity, one real but weak pattern found**: in the top liquidity tercile only (largest, most-traded names), the spread was +0.17% and positive in 8 of 10 years — more consistent than anything in the original test. But the direction is backwards from the classic "informed shorts" hypothesis: stocks whose short interest *increased* the most subsequently did slightly *better*, not worse. In the bottom liquidity tercile the pattern disappeared again (-0.05%, 60% of years, no consistency) — likely just microstructure noise in thinly-traded names, consistent with why pooling everything together produced murkier results before.

**Conclusion: still not building this.** +0.17% over 21 days, pre-costs, non-risk-adjusted, in a direction that doesn't match any economic story worth trusting, isn't a real edge — it's the kind of small, directionless number that's easy to talk yourself into and expensive to trade. This closes the refinement out rather than leaving it open: both the level-based and change-based versions of this signal, with and without liquidity controls, land on the same honest answer.

### 2026-08-01 — PEAD: yesterday's "~13 tickers/day" diagnosis was wrong — real root cause found

Picked the PEAD pull back up tonight, expecting the 2026-07-31 "quota, ~13 tickers/day" conclusion to hold and just needed patience. It didn't. A paced, resumed run hit `402` on the first 4 tickers tried (`A`, `ABNB`, `ABT`, `ACGL`) and aborted via the circuit breaker in 25 seconds — inconsistent with "quota reset overnight, 13/day available." Checked the actual response body instead of just the status code this time (the earlier diagnosis never did this): `"Premium Query Parameter: ... This value set for 'symbol' is not available under your current subscription"`.

That's not a quota message — it's a **permanent, per-symbol paywall** on the `/stable/earnings` endpoint's free tier. Confirmed empirically, not just from the error text: probed a random sample of 40 S&P 500 tickers directly, 37 came back `402` and only 3 passed (`CVX`, `BAC`, `HOOD`). Combined with the tickers that worked in earlier runs (`AAPL`, `MSFT`, `ABBV`, and the mega-caps from the original 20-ticker test), the free plan appears to allow a small, fixed, curated list of well-known symbols on this endpoint — not a quota that recovers over time. Also checked both realistic workarounds: the legacy `/v3/earnings-surprises/{symbol}` endpoint is fully dead (`403`, discontinued for non-legacy subscribers), and `/stable/earnings-calendar` with a `from`/`to` date range is also `402`-gated. The one thing that *is* free and unrestricted — `/stable/earnings-calendar` with no parameters — only returns a rolling ~3-month window (77 rows, 74 symbols) with no way to page further back; nowhere near enough for a real multi-year drift test.

**This corrects, not just updates, yesterday's conclusion.** The "13 tickers/day, pace it out over a month" framing was a real mistake — extrapolated from one lucky batch (13/20 succeeded) without checking *why* the other 7 failed. They weren't quota-exhausted, they were never going to work regardless of pacing, because they weren't on the allowed list. No amount of overnight patience or careful scheduling fixes a symbol-level paywall.

**Status: dead end on the current FMP plan, for real this time.** A genuine path forward exists but requires a decision, not more engineering: either upgrade the FMP plan to one that includes broad earnings-surprise access, or find a different free data source (e.g. Alpha Vantage's `EARNINGS` endpoint, untested — would need its own new signup and API key, not something done unilaterally). Not attempting either without checking first. Stopped the pull rather than keep retrying a wall that's now confirmed structural.

### 2026-08-02 — Adding ETH to funding-arb paper trading uncovered a real price-scaling bug in the Bitnomial fetcher (fixed) — BTC's own recorded prices were wrong too

Asked to add ETH to the live funding-arb paper-trading schedule. `BITNOMIAL_SYMBOLS` only had BTC (`"BTCUC"`) — ETH's code was an open gap noted since the original build. Found it the direct way: querying Bitnomial's funding-rates API with an intentionally-wrong `base_symbol` returns an error message listing every valid code on the exchange. `ETHUI` was in that list.

**Sanity-checking `ETHUI` before trusting it surfaced something much bigger.** Its live price ($9,225–9,589) didn't match Ethereum by any plausible measure. Pulled up Bitnomial's own live markets page (a JS-rendered page WebFetch can't execute, so used the browser tool directly) — real-time ETH perpetual was trading at **$1,854–1,866**, nowhere close. Same check against `BTCUC` — the symbol **already live in production since 2026-07-30** — found the identical problem: its API price (~$12,550) vs. the real live BTC perpetual price shown on-site (~$63,300), roughly 5x off.

**Root-caused, not just flagged.** Found the actual API call the live markets page makes under the hood (`/exchange/api/v1/prod/product/specs?active=true`, visible in the browser's network log) — Bitnomial's real product catalog, 615 products, each with a `price_increment` field. `BTCUC` ("Perpetual Bitcoin US Dollar Centi Future") has `price_increment: 5`; `ETHUI` ("Perpetual Ethereum US Dollar Deci Future") has `price_increment: 0.2`. Multiplying the funding-rates endpoint's raw value by its product's increment lines up almost exactly with the live site: 12,550 × 5 = $62,750 (vs. real $63,300); 9,225 × 0.2 = $1,845 (vs. real $1,854–1,866), both within ~1%. **The funding-rates API returns price in tick counts, not raw USD** — `fetch_bitnomial_history()` had been storing the raw tick value directly as `mark_price` this whole time, for every Bitnomial row ever pulled, both symbols.

**Confirmed `ETHUI` genuinely is Ethereum** (not a coincidental name collision, which was a real worry given how far off the raw price looked) — the increment-corrected value matches real ETH cleanly. This was worth checking properly rather than assuming: getting it wrong would have meant paper-trading against something that isn't ETH at all under a misleading label.

**Impact assessment, done before calling this "just cosmetic"**: checked every place `mark_price` actually flows through the strategy. Position sizing (`notional`/`margin_target`) is a fixed dollar split of `init_cash`, never multiplied by price. Funding P&L accrual is `funding_rate × notional`, never touches price. The liquidation check compares Binance klines prices against each other (`period_start_price`, `worst_high`), never Bitnomial's mark_price. **None of the strategy's actual economics were affected** — the backtest (BTC +51.6%, ETH +55.3%) never touches Bitnomial's mark_price at all (it uses Binance's own funding-rate history), and live paper trading's P&L/margin math never used the buggy field either. The blast radius was genuinely confined to the `price` column recorded in the audit-trail events table and the raw `funding_rates` table — real numbers, just mislabeled by a constant factor, not a strategy-corrupting bug. Still fixed properly rather than left as a known cosmetic issue: added `PRICE_INCREMENT = {"BTC": 5.0, "ETH": 0.2}` to `funding_rates.py`, applied it in `fetch_bitnomial_history()`, and corrected all 229 already-stored `bitnomial`-exchange rows plus the 8 already-logged BTC paper-trading events (`price * 5.0`) via a one-time `UPDATE`, rather than leaving old rows wrong going forward.

**ETH opened for real** via the same `check_and_update()` path BTC already uses (entry $1,845, notional $6,000, margin $4,000 at 1.5x) and added to `funding_arb_paper_trade.sh`'s existing `launchd` schedule — no new plist needed, the existing 3x-daily job now checks both coins in one run. Verified end-to-end by running the actual wrapper script, not just the Python entry point.

### 2026-08-02 — Confirmed the maintenance margin rate with Bitnomial directly: the real number is 30x the placeholder, and the strategy still survives it

Asked directly whether liquidation risk was actually zero or just unlikely. Answering that honestly required a real number the code had never actually confirmed: `MAINTENANCE_MARGIN_RATE = 0.005` (0.5%) was flagged in its own comment as "a reasonable real-world estimate... not confirmed against Bitnomial's own... margin schedule" — borrowed from Kraken Futures' base tier during the original margin-buffer scoping, not from Bitnomial itself.

**Found the real, live figure.** Bitnomial's `/exchange/api/v1/prod/product/data?active=true` endpoint (the same one their live markets page calls) returns a `product_group_margin_rate` field — currently **0.15 (15%)** for both BTCUC and ETHUI. Cross-checked this wasn't some unrelated field by reading Bitnomial's own published margin methodology (bitnomial.com/clearinghouse/margin-methodology): "margins are set to cover 99% of expected price movement over a historical time period," a SPAN-style volatility-calibrated approach — consistent with a live, per-product, market-data field rather than a fixed rulebook constant. Real number, not a guess, but also not a permanent one — it can move with realized volatility, so this is today's reading.

**30x higher than what the backtest actually used.** That's not a rounding difference — it meaningfully raises the real liquidation threshold's proximity to a big single-window move (BTC's real threshold: a ~35% adverse move within one 8-hour window, down from the previously-assumed ~49.5%; ETH's: ~51.7%, down from ~66.2%).

**Re-ran the full 6.5-year backtest with the corrected rate before saying anything reassuring.** Result: **zero liquidation events for both BTC and ETH, unchanged from the wrong-rate version.** The "survives every real crash in the data" conclusion (item 93) holds under the actual, confirmed margin requirement — it just hadn't been tested against the real number until now. Updated `MAINTENANCE_MARGIN_RATE` to 0.15 in `funding_arb.py`, re-ran and re-recorded the official backtest (`funding_arb_backtests` table unchanged in outcome, now backed by a real assumption instead of a borrowed one).

### 2026-08-02 — PEAD data-access decision researched, not made: Alpha Vantage (free, untested rate limit) vs. FMP upgrade (~$22-29/mo, instant)

Followed up on the 2026-08-01 dead-end finding. Checked two real paths rather than leaving it as an abstract "needs a decision" note. **Alpha Vantage's `EARNINGS` endpoint tested directly** with their public demo key (against `IBM`, their own canned example) — returns exactly the right shape for PEAD: `reportedEPS`/`estimatedEPS`/`surprise`/`surprisePercentage` per quarter, 122 quarters of real history back to 1996. No sign of FMP's symbol-level paywall in anything checked (demo-key restriction to IBM-only is a demo-key artifact, not evidence of a curated list on the real free tier). Real free-tier limit is either 25 or 500 requests/day depending on which bucket this specific endpoint falls into — couldn't confirm which without a real key, and didn't sign up for one on the user's behalf. **FMP's paid tiers**: their pricing page blocks automated fetches (403), but search-sourced figures put the cheapest tier around $22-29/month; unconfirmed whether that specific tier lifts the earnings-endpoint symbol restriction found yesterday.

**Recommended, not decided**: try Alpha Vantage first since it's free and already checks out structurally — consistent with this project's standing pattern of proving something free before spending money (see the Quiver/Congress-copy decision, item 87: "reaching this conclusion cost $0"). **Status: still open, needs the user's signup + decision, not resolved here.**

### 2026-08-02 — Insider-copy conviction filter (item 89's flagged follow-up), finally run through the real DSR/FDR pipeline — still doesn't survive

Item 89 flagged this explicitly as the next step: "buy every qualifying purchase" isn't fundable, and a real selection layer — which subset of purchases to actually act on — needed to be pre-registered and tested before insider-copying could be called dead. `insider_exec_conviction_backtest.py` (large $250k+ purchases, top-executive titles only, params fixed 2026-07-25) already existed for exactly this purpose but had **never been run through the portfolio-level DSR/FDR pipeline** — checked the DB directly (`insider_copy_evaluations` only ever had I1/I6 rows) and the promotion run log (only ever grid-searched 2 hypotheses) to confirm this wasn't just an assumption.

**Wired it in as a third pre-registered hypothesis (I7_conviction) and re-ran all three together**, not I7 alone with a stale FDR correction from I1/I6's earlier pass — running it separately would have under-corrected for the extra test, exactly the mistake this pipeline exists to prevent. The conviction filter does cut candidate volume dramatically (54,786 → 1,974, a ~96% reduction) — a real, structural attack on the capital-starvation problem, not a cosmetic one.

**Result: still HOLD.** Best trial (lag_days=2, position_fraction=0.01, 597 closed legs — well-powered, not a thin sample): +28.7% return vs. SPY's +76.8%, Sharpe 0.58 vs. 1.57, max drawdown -26.9% (breaches the -25% risk limit), DSR p=0.248 (not significant). Fails all three gates: risk, benchmark, and significance. Even at the smallest tested position size (100 concurrent slots), **70% of conviction-tier purchases still got skipped for insufficient cash** (1,377 of 1,974) — cutting the population by 96% wasn't enough on its own; qualifying large-executive purchases still arrive faster than a 126-day hold can turn over even a much smaller book. DSR (0.75) also isn't meaningfully better than I6's (0.74) — the higher-quality population didn't translate into a more statistically credible signal either.

**This closes out item 89's flagged follow-up with a real answer instead of leaving it open.** Insider-copying has now been tested at three different levels of selectivity (everything, repeat-buyers, high-conviction-only) and none survive contact with a fundable portfolio. The per-trade edge (item 88) was real and held up under scrutiny — but every attempt to turn it into something a real account could actually execute has failed, for the same underlying reason each time: too many qualifying signals arrive too fast for any account size to meaningfully participate in more than a random slice of them.

### 2026-08-02 — Signed up for a free Alpha Vantage key, confirmed it for real, built the PEAD pull as a permanent part of the codebase (not another scratchpad script)

User signed up for a free Alpha Vantage key (no credit card, confirmed before recommending it). Tested it directly against 40+ real S&P 500 tickers, including several FMP had specifically paywalled (`A`, `ABNB`, `CRL`, `PG`, `CVX`) — every one worked, confirming no symbol-level restriction on this source. **Found the real daily limit empirically**: 23 consecutive successes, then a hard wall that held even after slowing request pacing down (ruling out a burst-limit explanation) — and a later run's error message confirmed it explicitly: "our standard API rate limit is 25 requests per day." Real, resets daily, unlike FMP's structural dead end.

**Built `data/earnings_surprises.py` and `scripts/earnings_surprises_backfill.py`** as real, permanent parts of the codebase this time — the three earlier FMP-era pull attempts (2026-07-31) all lived in scratchpad and were thrown away when FMP turned out to be a dead end. New DuckDB table `earnings_surprises` (symbol, fiscal_date_ending, reported/estimated EPS, surprise, surprise_percentage), matching this project's established store/load pattern (funding_rates.py, sec_insider_trades.py). Resumable via the table itself (`stored_symbols()`) rather than a separate progress file — a second run never re-spends quota on an already-pulled ticker. Real circuit breaker (abort after 3 consecutive failures), same lesson as the FMP 12-hour incident.

**Verified both paths for real**: a live run against real tickers (before quota ran out from manual testing) confirmed the fetch/store round-trip works; a second run after quota was genuinely exhausted confirmed the circuit breaker aborts correctly in 3 seconds with a clear, honest message, not a silent hang or blind retry.

**Real math, decided honestly**: at 25/day, the full 503-ticker universe takes ~20 trading days of patient, once-daily pulling. Two ways to go faster, both real spend decisions not made here: Alpha Vantage's cheapest paid tier ($49.99/mo, removes the daily cap entirely — confirmed via their own docs) or the FMP upgrade path already scoped (~$22-29/mo, cheaper but less certain it fixes the specific restriction hit). Deliberately did **not** suggest creating multiple free accounts to stack quotas — that's evading the rate limit on purpose, not a real path.

**Status: infrastructure built and verified, data collection not started.** `earnings_surprises` table is empty (0/503) — today's quota was spent on verification (confirming the endpoint, the limit, and the circuit breaker), not on real ticker data yet. `scripts/earnings_surprises_backfill.sh` is ready to run once/day; not registered with launchd (a recurring-schedule decision left to the user, same governance principle as every other automation here).

### 2026-08-02 — Checked Finnhub as a second free source — real, but doesn't actually solve this problem

Asked whether a second, genuinely different free provider (not multiple accounts on the same one, which is quota-evasion and was ruled out earlier) could add throughput alongside Alpha Vantage. Finnhub looked promising on paper: 60 requests/**minute** free tier vs. Alpha Vantage's 25/**day** — over 3,000x more permissive by the numbers.

**User signed up (confirmed free, no card, same as Alpha Vantage) and tested it directly.** `stock/earnings?symbol=AAPL` works on the free tier — but returns only **4 quarters (1 year) of history**, not the 122 quarters (30 years) Alpha Vantage returns for the same ticker. Checked this wasn't a fluke: same 4-quarter cap for MSFT, and `from`/`to`/`limit` query parameters have no effect — a hard, real free-tier depth limit, not a pacing issue.

**Honest verdict: this doesn't actually solve the problem, despite the much better rate limit.** PEAD needs *years* of surprise history per ticker to build a statistically meaningful drift test (matching how every other hypothesis in this project — Congress trades, insider trades, short-interest — used years of data, not one snapshot). A fast source that only returns the trailing year can't substitute for Alpha Vantage's depth; it could only ever supplement the single most recent quarter, which doesn't move the needle on the actual constraint (backtest history, not current-quarter freshness). **Not integrated — a real, useful negative result, not a build.**

**Widened the search after Finnhub — checked 4 more candidates for a free source of historical *consensus estimates* (the actual bottleneck: any source can give reported EPS, since that's a public SEC filing, but the "estimate" side of a surprise is a proprietary analyst-consensus product almost every provider gates):**
- **Twelve Data**: ruled out definitively without even needing a key — `/earnings` returns `403 "available exclusively with grow or pro or ultra or venture or enterprise plans"`. Not on the free tier at all.
- **EODHD**: earnings-calendar endpoint returns `403 Forbidden` even with their public demo token — consistent with this project's earlier finding (2026-07-28) that EODHD gates most useful data behind their $99.99/mo All-in-One tier.
- **Tiingo**: fundamentals (which would include EPS) is a separate paid add-on requiring a sales conversation, not part of the free price-data tier.
- **Nasdaq Data Link's Zacks Street Earnings Estimates**: exactly the right dataset (consensus estimates for 5,000+ US/Canada tickers) — but premium, á la carte paid.

**Pattern, stated plainly**: every free tier checked either omits consensus estimates entirely or gates them behind a paid plan — Alpha Vantage giving this away free, with real multi-decade depth, is the exception, not the norm. This closes the "look for a faster free alternative" search with a real answer: **there isn't one**, at least not among the providers checked. The tradeoff stays what it was — patient (~20 days on Alpha Vantage) or pay (Alpha Vantage's own $49.99/mo uncapped tier, or the FMP upgrade path).
