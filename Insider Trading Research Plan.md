# Insider Trading Research Plan (pre-registered 2026-07-28)

Same discipline as [[Congress Trading Research Plan]], applied to a different population: corporate insiders (SEC Form 4 filings), not politicians. Written down **before running anything**.

## Why insiders are a genuinely different test, not a repeat of the congress exercise

- **The mechanism is narrower and more direct.** A congressional committee assignment gives broad, sector-wide access. A corporate insider knows one specific company's real numbers before the public does. If any "informed trading" signal is going to show up in disclosure data, this is the tighter, more plausible case.
- **The data is current.** `sec_insider_trades` runs 2023-05 through 2026-06 — unlike the congress archive, this isn't a dead historical dataset. If something real is found here, it doesn't have item 84's forward-actionability problem.
- **The scale is much larger**: 321,183 rows (63,405 purchases, 257,778 sales), 12,444 distinct insiders, 4,317 distinct tickers — vs. Congress's ~3,000 purchases and 18 testable people. That means the member-clustered test (the one that actually decided the congress question) will have real statistical power here instead of n=13-18.
- **Filing is fast and tight**: median disclosure lag is 2 days, max ~8 for the bulk of filings (SEC's 2-business-day deadline), vs. Congress's legal 45-day window. Less room for a "slow filers are informed" story, noted honestly below.

## Data verified on hand (2026-07-28)

| Check | Result |
|---|---|
| Rows | 321,183 (Purchase 63,405 / Sale 257,778) |
| Disclosure lag | 0 negative-lag rows (unlike Congress's 7) — median 2 days, max ~8 for the bulk, a tail out to 45 |
| `amount`/`shares`/`price_per_share` | 320,463 / 321,183 / 320,463 populated (~99.8%) |
| `owner_name` | Clean, structured SEC EDGAR strings — spot-checked, no spelling-variant problem the way Congress's free-text names had |
| `owner_title` | Free text, highly variable ("CEO", "Chief Executive Officer", "President and CEO", "COB and CEO", ...) — needs normalization before any role-based test, done below |
| Price data | 3,685 of 4,317 traded tickers already stored; 632 need a fresh fetch |
| Outlier check | A few (owner, ticker) pairs trade extremely often (top: 4,331 purchases by one person in one ticker) — flagged for a robustness re-check excluding the heaviest traders, same check that ruled out a Perdue artifact in the congress work |

## Fixed design (not searched, same as the congress exercise)

- Holding period: 126 trading days (~6 months), matching `build_purchase_hold_portfolio()`'s existing default and the congress exercise's own constant, for direct comparability.
- Entry: 1 trading day after `disclosure_date` (never `transaction_date` — same lookahead rule).
- Returns: excess over SPY across each trade's identical calendar window, not raw.
- FDR correction at n=6 (the full list below).

## The 6 pre-registered hypotheses

### I1 — Purchases beat sales
**Claim:** Disclosed insider purchases outperform disclosed insider sales.
**Mechanism:** Same as congress H1 — a purchase is a deliberate commitment; a sale is routine (diversification, taxes, 10b5-1 plans). This is also the most heavily studied claim in the real academic insider-trading literature, so it's the natural first test here.

### I2 — Seniority: C-suite vs everyone else ★ the marquee hypothesis
**Claim:** Purchases by CEOs/CFOs/other C-suite officers outperform purchases by directors, VPs, and other lower-visibility insiders.
**Mechanism:** This is this exercise's version of H7 — the closest thing to a direct causal story. A CEO or CFO sees the whole company's real numbers before anyone; a director sees far less, far less often.
**Needs:** normalizing `owner_title` into a `is_c_suite` flag (CEO/CFO/COO/President/Chairman-with-CEO-role variants) vs everything else — fixed by pattern-matching title keywords **before** running, same "no post-hoc tuning" rule as the congress committee crosswalk.

### I3 — Unusually large trades (for that insider)
**Claim:** Purchases above an insider's own median purchase size outperform their typical purchase.
**Mechanism:** Same as congress H3 — conviction, measured relative to that person's own baseline, not absolute dollars (which would just select for wealthy insiders).

### I4 — Slow disclosers
**Claim:** Purchases disclosed near the legal deadline outperform promptly-filed ones.
**Mechanism:** Same story as congress H2, but the honest caveat is the window here is compressed (median 2 days, most within a week) — real spread exists (some tail out to 45 days) but the effect, if any, has much less room to show up than in Congress's 45-day window.

### I5 — Multiple insiders buying together ★ the strongest theoretical prior on the list
**Claim:** A stock purchased by 2+ distinct insiders within a short window outperforms one bought by a single insider.
**Mechanism:** This is the single most replicated finding in the actual academic insider-trading literature — cluster buying by multiple company insiders is a stronger signal than any one person's trade, because several people with direct, independent knowledge of the same company reaching the same conclusion is far less likely to be noise. Congress's version of this (H5) tested corroboration across *different companies and different information domains*; this tests it within *the same company*, which is the mechanistically tighter and more defensible version of the same idea.
**Window:** same ±30 calendar days as the congress exercise, for comparability.

### I6 — Repeat accumulation
**Claim:** An insider's repeat purchase of a ticker they already hold outperforms their first purchase of it.
**Mechanism:** Same as congress H6 — sustained conviction across multiple decision points.

## Robustness checks planned alongside the main run

1. **Member-clustered test is primary, not a follow-up.** Given how badly the pooled/clustered gap mattered in the congress work, the clustered (one-insider-one-vote) test is treated as decisive from the start here, not run only if the pooled stage passes.
2. **Heaviest-trader exclusion**, same check that ruled out Perdue as the explanation for congress H6 — re-run any surviving hypothesis excluding the top 1% most active (owner, ticker) pairs, since a few extreme repeat traders (one with 4,331 purchases in a single ticker) could otherwise dominate a pooled average.

## What would make this worth acting on

Even a hypothesis that survives both stages needs to clear a bar beyond "statistically real" before it means anything: the congress exercise's own H1/H6 topped out around +1.7% excess over six months *before costs* even at face value — not a tradeable margin. Given this data is current (unlike Congress's), a genuine survivor here would be the first candidate in this whole project with both a real statistical basis *and* no data-staleness problem blocking it from being forward-tradeable.

---

# RESULTS (run 2026-07-28, same day as pre-registration)

**Different outcome from the congress exercise: two hypotheses survive everything, including a check the congress work never needed.** Panel: 241,350 usable trades of 321,183 raw (24,092 dropped for no price history, 55,741 for lacking a full 126-day forward window).

## All six, pooled and clustered

| # | Hypothesis | Pooled p | Clustered n | Clustered p | Verdict |
|---|---|---|---|---|---|
| I1 | Purchases beat sales | <0.0001 | 326 | 0.0002 | **CONFIRMED** |
| I2 | C-suite vs other | 0.2661 | 26 | 0.0046 | **not confirmed** (see below) |
| I3 | Large-for-insider trades | 0.0002 | 1,446 | 0.9241 | collapses |
| I4 | Slow disclosers | 0.0128 | — | n/a | unconfirmable (no clustered form) |
| I5 | Multiple insiders together | 0.0001 | 355 | 0.3045 | collapses |
| I6 | Repeat accumulation | <0.0001 | 89 | 0.0043 | **CONFIRMED** |

## Why I2 doesn't count despite p=0.0046

The clustered test requires the same person to have both C-suite-labeled and non-C-suite-labeled purchases (≥3 each) to contribute — but C-suite status is mostly a fixed trait of a person, not something that varies trade to trade. Only 146 of 12,444 insiders ever have both (mostly mid-dataset promotions or inconsistent title strings across filings), and only 26 cleared the ≥3-per-side bar. That's not a test of "C-suite people vs. everyone else" — it's "does the same person's trades do better under one label vs. another," on a tiny, weird slice. The heaviest-trader-excluded check confirms the suspicion: excluding the 100 most active (owner, ticker) pairs collapses it from diff +20.52% (p=0.0046) to diff +0.22% (p=0.40). **Verdict: not confirmed** — the properly-powered pooled test (n=13,486 vs 32,631) already agreed, showing no evidence (p=0.27).

## The two real survivors

**I1 — Purchases beat sales.** Purchases: +1.42% excess over 6 months (n=46,117). Sales: −1.45% (n=195,233). Survived all three checks: pooled (p<0.0001), person-clustered on a real sample of 326 people (p=0.0002, 180/326 positive), and holds after excluding the 100 heaviest traders (diff +2.06%, still p<0.0001). This also matches the most replicated finding in the actual academic insider-trading literature — a real mechanism, not a data-mined curiosity.

**I6 — Repeat accumulation.** An insider buying a ticker they already hold again: +2.06% excess vs. −0.90% for a first purchase. Survived all three checks: pooled (p<0.0001), person-clustered on 89 people (p=0.0043, 58/89 positive), holds after heaviest-trader exclusion (diff +2.42%, p=0.0011).

## What this means, and the honest caveats before getting excited

- **This is the first result in the whole project (congress or price-technical) to survive dependence correction on a real sample.** Different outcome from the clean 0-for-7 on congress trading.
- **The data is current** (2023-2026) — no item-84-style staleness problem blocking forward use.
- **Still a modest edge, not a proven trade.** +1.4% to +2.1% excess over six months is real but not large, and this is before any transaction costs, taxes, or slippage. The congress exercise's own numbers (which turned out to be noise) were in a similar range — magnitude alone isn't proof, the survival through clustering + exclusion is what's different here.
- **Not yet a portfolio backtest.** These are per-trade split tests establishing *whether a mechanism exists* — the next step, if pursued, is the same treatment congress-copy got in Platform Status.md item 84: run `build_purchase_hold_portfolio()` (already built, reused from congress_copy.py) at real scale, with the platform's own DSR/FDR/risk-gate pipeline, before this is anything close to tradeable.

---

# PORTFOLIO-LEVEL RESULT (run 2026-07-29): does NOT survive

Ran I1 and I6 through the exact treatment congress-copy got in item 84 — see Platform Status.md item 89 for the full writeup, including a real capital-starvation bug found and fixed along the way (the pre-existing `insider_purchase_hold_backtest.py` run from 2026-07-24 had funded only 136 of 54,786 candidate purchases; re-run here with a searched, DSR-corrected position-size grid instead of one fixed value). Even after that fix, both hypotheses still HOLD, not PROMOTE:

| Hypothesis | Portfolio return | SPY return | Sharpe (portfolio vs SPY) | Max drawdown | DSR p-value | Verdict |
|---|---|---|---|---|---|---|
| I1 — all purchases | +60.1% | +70.9% | 0.88 vs 1.48 | -25.5% | 0.151 | HOLD |
| I6 — repeat accumulation | +39.3% | +70.9% | 0.61 vs 1.48 | -29.1% | 0.259 | HOLD |

Both underperform buy-and-hold SPY, both breach the -25% drawdown limit, and neither is statistically significant once corrected. The underlying reason: even at the smallest tested position size (1-2% per trade), the sheer volume of daily insider purchases means well over 98% of eligible trades still get skipped for lack of free cash — a portfolio built this way (buy everything, first-come-first-funded) ends up holding essentially a random, chronologically-biased slice of the real signal, not a deliberately curated one. **The per-trade edge from the section above may still be real — this result says "buy literally everything" isn't how to capture it, not that the mechanism is fake.** Turning it into something tradeable would need a real selection layer (which purchases to actually act on, out of far more than any account can fund) designed and pre-registered *before* testing, not fit to this result after the fact.
