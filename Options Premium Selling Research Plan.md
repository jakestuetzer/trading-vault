# Options Premium Selling Research Plan (architected 2026-08-24)

Same "write it down before building anything" discipline as [[Congress Trading Research Plan]] and [[Insider Trading Research Plan]], applied to a genuinely different kind of strategy: harvesting the volatility risk premium by selling options, not predicting price direction. Full prior scoping trail (historical-edge check, data-source search) lives in [[Enhancement Ideas]] under 2026-07-30 — this is the from-scratch architecture that follows it.

## The real edge, already confirmed (not re-derived here)

Checked directly with free data (FRED's VIX history since 1990, SPY price history) on 2026-07-30: implied volatility (VIX) averaged 19.5 vs. subsequent realized volatility of 15.9 over 33 years — a persistent ~3.6-point premium, positive on 82.8% of trading days. This is the real, literature-standard volatility risk premium — a structural payment for insuring other market participants against price swings, not a price-direction bet. Positive almost every year except 2008 and nearly vanishing in 2000/2018 — real, identifiable tail-risk years exist, same shape as funding-arb's risk profile.

## Honest strategic framing -- this is a diversifier, not a growth accelerant

Stated directly before any design work, not glossed over: covered-call/cash-secured-put selling caps upside in exchange for steady income. It is structurally the same shape as every "smoother ride, smaller drawdown" strategy already tested in this codebase (regime-gated strategies, the original 6 price-trend combos) -- all of which lost to raw buy-and-hold in the 2015-2026 bull window. It does **not** serve the "aggressive, beat SPY" goal the leveraged buy-and-hold sleeve (`strategies/buy_and_hold.py`, TQQQ/QLD) already answers. It sits alongside funding-arb as a second real, uncorrelated diversifier -- not a bigger swing.

## Data, checked for real before assuming a price (2026-08-24)

| Source | Result |
|---|---|
| Alpha Vantage (key already on hand from PEAD) | `REALTIME_OPTIONS`/`HISTORICAL_OPTIONS` both confirmed premium-gated by hitting them directly -- real response returns fabricated placeholder data (`symbol: XXYYZZ`, `expiration: 2099-99-99`) with an explicit message naming the gate. |
| Alpha Vantage pricing, checked live | General page implies $49.99/mo unlocks "all premium features," but the options endpoint's own error message specifically named the $199.99/mo (600 req/min) tier -- **treat $199.99/mo as the real number**, not the cheaper headline price, until someone actually pays and confirms which tier unlocks it. |
| Polygon (rebranded Massive.com) | Pricing page is JS-rendered, exact options tier not confirmed, but adjacent tiers (Stocks Advanced $199/mo) suggest a comparable $150-300+/mo range. |
| EODHD | Already known from earlier project work (2026-07-28, PEAD scoping) to gate most useful data behind a $99.99/mo All-in-One tier -- consistent pattern across this vendor's whole catalog, not options-specific. |

**Status: real, ongoing cost, roughly $150-250/month depending on vendor and which tier actually unlocks options chains — not yet paid, not yet confirmed to the dollar.** This is a genuine spend decision, same category as the PEAD Alpha Vantage upgrade or the FMP upgrade path -- not made here.

## Why this needs a genuinely new backtest engine, not a data problem alone

`backtest/run.py`'s `run_backtest()` is vectorbt-based, single-ticker, boolean-signal-on-close-price shaped -- built for "long or flat this ticker," not options. This strategy needs:
- Strike/expiration selection at entry (a real rule, not implied by a close-price crossing)
- Real assignment/exercise mechanics at expiration (ITM vs. OTM outcomes are fundamentally different payoffs, not a continuous return)
- Collateral tracking (cash-secured put: strike x 100 x contracts held in cash; covered call: 100 x contracts shares of the underlying held)
- Optional early-roll logic when a position moves deep ITM before expiration

Same "genuinely needs new backtest logic" situation `funding_arb.py`'s module docstring already established for a different reason (leverage/margin, not cross-sectional ranking or options mechanics) -- likely a **larger** lift than funding-arb's original build, which only needed an always-on accrual-and-liquidation loop. Real new module, `options_income.py`, momentum_rotation.py/funding_arb.py-shaped (standalone, not a `Strategy` subclass, not routed through `walk_forward()`/DSR -- those assume a single continuous-return series, not a discrete per-cycle assignment outcome).

## Fixed design decisions (not grid-searched, so this doesn't quietly reintroduce the exact selection bias DSR exists to correct for)

- **Structure: covered calls only, not cash-secured puts or the full wheel, for v1.** Covered calls compose directly with an existing real position (own the underlying, sell calls against it) -- cash-secured puts require holding idle cash as collateral, a capital-efficiency question this project already treats carefully (funding-arb's isolated-margin scoping). Puts/the wheel are a real, legitimate v2 extension, not attempted here.
- **Underlying: SPY or QQQ, not TQQQ/QLD.** Selling calls against a 3x-leveraged ETF's own violent daily swings carries real, disproportionate assignment/gap risk relative to the premium collected -- flagged explicitly as a real design decision, not defaulted into. SPY/QQQ also already have full real price history in this platform (`ohlcv_daily`), no new equity data needed.
- **Strike selection: target ~30-delta (a standard, real-world retail/institutional convention for covered-call income, not invented for this project), expiration ~30-45 days out.** A fixed, pre-registered rule, same discipline as `regime.py`'s ER window/threshold being fixed rather than searched.
- **Risk gate: reuse `vix_term_structure.py`'s own contango/backwardation signal, don't invent a new one.** That module's docstring already names the exact real risk this strategy faces: "Backwardation... shows up around real market stress (2008, 2018's Volmageddon, COVID) -- exactly when a short-vol position gets crushed." Selling calls only while VIX/VIX3M is in contango (the market's calm, normal state) directly reuses an already-built, already-reasoned-about signal rather than re-deriving risk management from scratch.
- **Transaction costs: modeled explicitly, not ignored** -- per-contract commission (a small, real retail figure, e.g. ~$0.65/contract) plus bid/ask spread cost on entry and exit, same "cost modeling is inside the backtest, not bolted on after" principle `backtest/run.py`'s own docstring states for `DEFAULT_FEES`/`DEFAULT_SLIPPAGE`.

## Validation plan

1. Full real historical options-chain backtest, once data is paid for -- covering enough years to include the known real stress windows already identified in `scripts/bear_market_test.py` (2018 Q4, COVID crash, 2022 bear) -- these are exactly the periods a short-vol strategy most needs to be tested against, not incidentally included.
2. Compare gated (contango-only) vs. ungated (always sell) directly, the same way `vix_term_structure.py`'s own `VixRegimeGatedStrategy` was built to test "does sitting out stress regimes actually help" rather than assuming it does.
3. Report total return, Sharpe, and max drawdown against plain SPY/QQQ buy-and-hold over the identical window -- same standard every other strategy in this platform is held to, not a lighter bar because the mechanism is different.

## Real open decisions, not made here

- Which data vendor and tier, and the exact confirmed price -- needs someone to actually engage a vendor, not assumed further from a pricing page.
- Whether to build the engine now (unpaid, using synthetic/limited data to prove the mechanics) and pay for real data only once the mechanics are proven, vs. paying first -- a real sequencing choice, not decided here.
- Cash-secured puts / the full wheel, as a v2 extension once covered calls are validated.

**Status: architected, not built.** No data has been purchased, no code written yet -- this document is the complete real design so building it is a mechanical exercise once the data decision is made, not an open-ended one.
