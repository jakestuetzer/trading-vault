# Congress Trading Research Plan (pre-registered 2026-07-28)

A **pre-registered** list of hypotheses about *what makes a congressional trade informed*, written down BEFORE any of it is run against the data. Follow-on to [[Platform Status]] item 84, which found a real DSR/FDR-corrected historical edge for 5 senators but no way to act on it forward.

## Why this document exists (the whole point)

The natural instinct is "go through every trade and look for patterns in the winners." That instinct is exactly what items 75 and 84 exist to guard against. With ~6,000 trades and enough ways to slice them (sector, size, timing, day of week, owner, committee, holding period...), **something will look predictive purely by chance** — that's the look-elsewhere problem, and it does not care that the dataset is new and interesting.

The fix is not "be careful." The fix is mechanical:

1. **Write the full hypothesis list down first** (this document).
2. **Run every hypothesis on the list**, including the ones that turn out boring.
3. **Apply the FDR correction using n = the full pre-registered list**, not n = however many survived.
4. Anything found *after* this list was written is a new, separate, honestly-labeled search — not a bonus finding from this one.

Skipping step 3 by quietly dropping the failures is the single easiest way to fool ourselves here, and it would invalidate the whole exercise.

## Test design: pooled first, per-member only on a survivor

Testing 7 hypotheses × 17 members = 119 tests, which needs a brutal correction and would likely pass nothing.

Instead, each hypothesis is first tested **pooled across all members** — "does this feature predict forward returns across congressional trading in general?" That is:
- **7 tests, not 119** — a far smaller correction.
- A **stronger, more useful claim** — a general rule about informed trading beats "this one senator was good," which is also the thing most likely to keep working going forward (see item 84's core problem: the surviving members are largely out of office or out of data).

Only if a pooled hypothesis survives does it earn a per-member drill-down, counted as its own new search.

## Data verified on hand (2026-07-28, `congress_trades`, 6,024 rows)

| Field | Status |
|---|---|
| `owner` | Populated — Joint 2,633 / Spouse 2,308 / Self 908 / Child 130 / blank 45 |
| `amount_low` / `amount_high` | Fully populated (6,024/6,024), $1,001 → $50M |
| disclosure lag | median 20 days; **267 filings exceed the 45-day legal limit** |
| `transaction_type` | Purchase 2,973 / Sale 2,985 / Exchange 66 — near-symmetric |

**Data-quality flag found during this check:** 7 rows (0.12%) have `disclosure_date` *earlier* than `transaction_date`, which is impossible and is a genuine (if tiny) lookahead risk, since `congress_copy.py` correctly dates all execution from `disclosure_date`. Two of the 7 touch item 84's surviving members (Hoeven 1, Roberts 1). Almost certainly immaterial to that result at 2 rows out of thousands, but a cheap guard clamping `disclosure_date >= transaction_date` should go in before any of the work below.

---

## Testable now (no new data needed)

### H1 — Purchases only, ignoring sales
**Claim:** Copying only disclosed *Purchases* outperforms copying purchases and sales together.
**Mechanism:** A purchase is a deliberate, costly commitment. A sale is routine far more often — diversification, liquidity, taxes, blind-trust rebalancing. Academic insider-trading work treats the two as different signals for this reason.
**Why it's not settled already:** `build_purchase_hold_portfolio()` was built for SEC insiders precisely because they almost never round-trip. Congress is verified near-symmetric here (2,973 vs 2,985), so the same reasoning does *not* automatically transfer — it needs its own test.
**Code:** `build_purchase_hold_portfolio()` already exists. Cheapest test on the list.

### H2 — Slow disclosers are more informed
**Claim:** Trades disclosed close to (or past) the 45-day legal limit outperform trades disclosed promptly.
**Mechanism:** Prompt filing suggests routine, compliance-driven trading. Sitting on a filing suggests either deliberate delay or a trade the member wasn't eager to publicize.
**Critically: this is actionable, not lookahead.** At the moment of disclosure we know *both* dates, so lag is a filter available at trade time.
**Data support:** median lag 20 days with 267 filings over the limit — real spread to test against, not a degenerate variable.

### H3 — Unusually large trades (for that member) are more informed
**Claim:** Trades in the top tier of a member's *own* trade-size distribution outperform their typical trade.
**Mechanism:** Conviction. Size relative to their own baseline.
**Design note:** Must normalize per-member (against that member's median trade size), **not** in absolute dollars — absolute size would just select for personally wealthy members, which is a different and far less interesting claim.

### H4 — Self-owned trades differ from spouse/child trades
**Claim:** Trades held in the member's own name perform differently from those in a spouse's/dependent's name.
**Mechanism:** Genuinely ambiguous in *direction*, which makes it a clean test rather than a motivated one — either the member's own account is where real conviction lands, or informed trades get routed to a spouse's account for optics.
**Data support:** Self 908 vs Spouse 2,308 vs Joint 2,633 — enough of each.

### H5 — Multi-member clustering
**Claim:** A ticker bought by *several* members within a short window outperforms one bought by a single member.
**Mechanism:** Independent corroboration. If several offices reach the same conclusion near-simultaneously, that's more likely a shared real catalyst than one person's hunch.
**Reuse:** This is structurally the same "several independent signals landing close together" shape as the platform's existing `confluence.py`.

### H6 — Repeat accumulation
**Claim:** Repeated purchases of the same ticker by the same member over time outperform one-off purchases.
**Mechanism:** Sustained conviction across multiple decision points, vs a single trade that might be noise or advisor-driven.

---

## Needs one new (free) data source

### H7 — Committee–sector overlap ★ the marquee hypothesis
**Claim:** A member's trades in sectors overseen by their *own committee assignments* outperform their trades in sectors outside it.
**Mechanism:** This is the only hypothesis on the list with a clear, direct causal story for genuinely non-public information — committee members see briefings, draft legislation, and hear testimony about the industries they regulate before the public does. It is also where the actual academic literature concentrates.
**Needs:** committee rosters (free — congress.gov API or ProPublica Congress API) + ticker→sector mapping (`yfinance` already exposes sector; the platform already fetches from it).
**Priority:** Highest theoretical value, highest build cost. Worth doing properly rather than first.

---

## Explicitly NOT hypotheses (parameter searches in disguise)

These belong in the DSR-corrected parameter grid, where `congress_copy_promotion.py` already handles them. Reporting any of them as a "discovery" would be precisely the error this document exists to prevent:

- Optimal holding period (`holding_days`, currently 126)
- Optimal execution lag (`lag_days`, currently 1/2/3)
- Optimal position size (`position_fraction`, currently 3%/5%)

DSR already corrects the *selected* result for how many of these were searched. They are knobs, not findings.

---

## Sequencing

1. **Now, free:** the `disclosure_date >= transaction_date` guard (data-integrity fix, independent of everything else).
2. **Now, free:** H1–H6 pooled, all six, FDR-corrected together at n=7 (reserving H7's slot in the correction, since it's on the pre-registered list even if it runs later).
3. **Only if something survives:** per-member drill-down on the survivor, labeled as its own new search.
4. **Separately, needs Quiver ($30/mo) to matter forward:** none of the above requires the subscription to *test* — the existing 6,024-row archive is enough to test every hypothesis here. Quiver is what makes a surviving result **tradeable going forward**, which is a different question from whether it's real. Worth knowing: **the research can be done first, for free, before spending anything.**

That last point is the practical headline — testing this list costs nothing, and its outcome should probably inform whether the $30/month is worth spending at all.

---

# RESULTS (run 2026-07-28, same day as pre-registration)

**Honest headline: nothing survives.** Two hypotheses passed the first stage and both collapsed under the dependence correction that was flagged as a limitation *before* the tests ran.

Code: `congress_hypotheses.py` + `scripts/congress_hypotheses_run.py` (fully reproducible, one command).
Panel: 4,592 usable trades of 5,951 raw (1,179 dropped for no price history — mostly delisted; 180 for lacking a full 126-day forward window).

## Stage 1 — pooled per-trade test, FDR-corrected at n=7

| # | Hypothesis | Signal mean | Control mean | Diff | p | FDR |
|---|---|---|---|---|---|---|
| H1 | Purchases vs sales | +0.83% | −0.85% | **+1.68%** | 0.0069 | **passed** |
| H2 | Slow vs fast disclosure | +1.38% | +0.24% | +1.14% | 0.0988 | no |
| H3 | Large-for-member vs typical | +0.97% | +0.80% | +0.16% | 0.4436 | no |
| H4 | Self vs spouse/child/joint | −0.19% | +1.03% | −1.21% | 0.3205 | no |
| H5 | 2+ members clustered vs 1 | −0.74% | +1.19% | −1.92% | 0.9704 | no |
| H6 | Repeat buy vs first buy | +1.58% | −0.20% | **+1.78%** | 0.0240 | **passed** |

Notable: **H5 came out backwards** — tickers bought by several members within 30 days *underperformed* single-member buys by 1.9%. Not significant, so not a finding in reverse either, but worth recording that the "corroboration" intuition had no support at all.

## Stage 2 — the decisive test: cluster by member

The pooled test treats thousands of trades with overlapping 126-day windows as independent observations, which inflates the effective sample size and therefore the significance. `KNOWN_LIMITATIONS` item 1 flagged this in advance. The correction: give each **member** one number (their own signal-minus-control difference) and test whether that is reliably positive across members.

| # | n members | Mean per-member diff | Median | Members positive | p | Verdict |
|---|---|---|---|---|---|---|
| H1 | 18 | **−0.11%** | +1.04% | 11/18 | 0.5384 | **collapses** |
| H6 | 15 | **−0.11%** | −0.32% | 7/15 | 0.5318 | **collapses** |

Both evaporate. The pooled "significance" was an artifact of counting dependent observations as independent — exactly the failure mode pre-flagged as limitation 1.

## Robustness check that did *not* change anything

H6 was also re-run excluding the single heaviest trader (Perdue, 568 of 1,351 repeat buys) on the suspicion it was one member's artifact. It got *stronger* pooled (+2.24%, p=0.0151) — so H6 was never a Perdue effect. It still collapses under clustering. Worth recording: the obvious confound was checked and was not the explanation; the dependence was.

## What this means

- **No evidence** that any of these six features distinguishes an informed congressional trade from a routine one, at this sample size, once the statistics are done honestly.
- H1 (purchases beat sales) retains a *hint* — median per-member difference +1.04%, 11 of 18 members positive, and it is the best-established claim in the academic literature. 61% of members isn't much better than a coin flip at n=18. Call it unproven, not disproven.
- Even taking Stage 1 at face value, the absolute edge was thin: purchases at **+0.83% excess over six months, before costs**. That is not a tradeable margin, even if it were real.
- **This changes the Quiver recommendation.** Item 84 found per-senator edges; this finds no general mechanism behind them. Those are technically compatible (specific individuals could have edges while no general feature does), but the simpler joint explanation is that congressional trade-copying has no robust exploitable edge in this data. Paying $30/month for fresher data to feed a strategy with no demonstrated general mechanism is much weaker than it looked before these tests ran.
- **The pre-registration did its job.** Had the list not been fixed in advance, the natural move would have been to report H1 and H6 as discoveries, quietly drop H2–H5, and never run Stage 2. That version of this document would have been wrong and would have justified spending money.

## H7 — committee–sector overlap (run 2026-07-28, same day)

The marquee hypothesis, and the only one with a direct causal story. **Result: fails, and fails in the direction opposite to the prediction.**

Code: `congress_committee.py`.

### Three data problems solved honestly first

1. **Era-correct assignments.** The free `congress-legislators` dataset has committee membership **current only** — using 2026 rosters to explain 2014–2021 trades would be serious measurement error (assignments change each Congress; party control flipped twice in this window) *and* would silently drop every member who has since left office. That's not missing-at-random: the departed set includes the single heaviest trader in the data (Perdue, 408 purchases in-era) and item 84's own surviving members. Solved with **Stewart & Woon `senate_assignments_103-115`** (MIT, free), which gives real per-Congress assignments including for members who later left.
2. **Coverage bound, accepted rather than papered over.** Stewart & Woon ends at the 115th Congress (Jan 2019), so this tests the **1,421 purchases disclosed through 2018 (61% of the panel)** with era-correct data and deliberately does *not* extend to the 902 later ones by substituting the wrong era's rosters. A smaller honest test beats a larger contaminated one.
3. **The crosswalk is a researcher degree of freedom**, so it was fixed from statutory jurisdiction *before* running: Agriculture→Consumer Defensive/Basic Materials; Armed Services→Industrials; Banking/Housing→Financial Services/Real Estate; Commerce→Industrials/Communication Services/Technology; Energy→Energy/Utilities; Environment→Utilities/Basic Materials; Finance→Financial Services/Healthcare; HELP→Healthcare. **Appropriations and Budget deliberately map to nothing** despite being the most powerful committees — their jurisdiction is universal, so mapping them would make nearly every trade "overlapping" and collapse the split into noise. That choice *costs* the test power rather than gaining it.

### Member matching verified by hand before trusting any number

19 of 20 members matched, covering 1,419 of 1,421 purchases. The matcher correctly resolved the genuinely hard nickname/legal-name cases — `John F Reed`→"Reed, Jack", `Rafael E Cruz`→"Cruz, Ted", `A. Mitchell McConnell, Jr.`→"McConnell, Mitch", `William Cassidy`→"Cassidy, Bill". Surname **and** first initial are both required, because two Udalls and two Nelsons served in this era and a silent wrong match would corrupt the test invisibly. Only Tina Smith missed (took office Jan 2018, 2 purchases).

### Result

Testable: 1,324 purchases (95 dropped for no sector metadata, 2 for no member match). 321 in-jurisdiction vs 1,003 outside.

| Stage | In-committee sector | Outside | Diff | p | Verdict |
|---|---|---|---|---|---|
| Pooled per-trade | +0.90% (n=321) | +2.92% (n=1,003) | **−2.02%** | 0.9766 | no evidence |
| Member-clustered | — | — | **−0.49%** mean, 6/13 positive | 0.5868 | no evidence |

**It goes the wrong way.** Senators' purchases in sectors their own committees oversee *underperformed* their purchases elsewhere by 2 percentage points.

**And no, that reverse cannot be claimed as a finding.** The direction tested was pre-registered as "greater"; flipping it after seeing the data is exactly the move this document forbids. For completeness the reverse doesn't hold up either — the clustered test gives p≈0.41 against it (7 of 13 members negative is a coin flip). This is a clean null, not a hidden inverse edge.

---

# FINAL VERDICT: all 7 pre-registered hypotheses fail

| # | Hypothesis | Outcome |
|---|---|---|
| H1 | Purchases beat sales | passed pooled, **collapsed** clustered |
| H2 | Slow disclosers | no evidence |
| H3 | Large-for-member trades | no evidence |
| H4 | Self vs spouse/child | no evidence |
| H5 | Multi-member clustering | no evidence (came out backwards) |
| H6 | Repeat accumulation | passed pooled, **collapsed** clustered |
| H7 | Committee–sector overlap | no evidence (came out backwards) |

**No general mechanism distinguishes an informed congressional trade from a routine one in this data.** Two of seven looked real until dependence was corrected for; the marquee causal hypothesis failed outright and in reverse.

**Practical conclusion: do not buy the Quiver subscription on the strength of the congress-copy idea.** Item 84's per-senator edges now look considerably more like survivorship among 17 candidates than a repeatable mechanism — there is nothing underneath them. Fresher data would buy a faster feed into a strategy with no demonstrated basis.

This thread is closed with a clean negative result, which is a real outcome and cost $0.
