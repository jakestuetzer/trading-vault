# PEAD Options Overlay Research Plan (architected 2026-09-09)

Same pre-registered-design discipline as [[Options Premium Selling Research Plan]] and the Congress/Insider plans. Written before any code, not after.

## The real idea

PEAD (`strategies/pead.py`, 32 live tickers) already identifies a real, validated signal: a company just posted an earnings surprise large enough (7.8%+) to predict continued drift over the next ~60 trading days. Today it expresses that signal by buying the stock outright. This overlay expresses the *same signal* with options instead -- calls on a positive surprise, puts on a negative one -- for more leverage on a conviction that's already been through this platform's full validation chain (pooled FDR test, risk-adjusted placebo, clustered-by-ticker check, real walk-forward/DSR backtest, cross-family correlation gate).

**Honest framing, stated up front**: this is leverage on an already-real signal, not a new signal. If PEAD's edge isn't large enough to survive an option's real cost of time decay, this makes things worse, not better -- the same test every leveraged idea in this project has had to clear.

## Real data, checked directly before designing around it (2026-09-09)

Confirmed via `data/options_chain.py` (already built, already proven live for covered calls): real option chains with 80-160 real days to expiration exist for PEAD tickers -- 121 real contracts across 2 expirations checked live for IBM. Same honest limitation as the covered-call build: Yahoo's free chain is live-only, no historical archive, so **this can only ever be forward-tested, never backtested**. No new data-cost decision needed -- reuses exactly what's already paid for (nothing) and already working.

## Real design decisions (fixed, not searched -- same discipline as every other strategy here)

- **Structure: calls on a positive-surprise entry, puts on a negative-surprise entry** -- mirrors `PEADLongShortStrategy`'s existing long/short symmetry exactly, not a new directional rule.
- **Strike: higher-delta than the covered-call build's ~30-delta, targeting ~0.65-0.70 (slightly in-the-money), not a classic far-OTM "lottery ticket."** Deliberate, real reasoning: PEAD's edge is real but modest (~1.5% average excess drift), and a long, 60-trading-day hold gives time decay a lot of runway to work against a cheap, deep-OTM contract. A higher-delta contract behaves closer to leveraged stock ownership -- it still requires much less capital than owning the shares outright, but isn't purely betting on decay outrunning a big move. This is the mathematically appropriate structure for "leverage a modest, long-duration edge," not the structure for a speculative short-term swing.
- **Expiration: real contracts expiring materially past the 60-trading-day hold (~84 calendar days) -- targeting ~90-150 real calendar days to expiration at entry**, confirmed available. Real buffer so the position is never forced to exit into a fast-decaying, near-expiration contract at the natural 60-day PEAD exit point.
- **Position size: a small, fixed fraction of account equity per trade (not the full $10k-per-position convention `check_and_update()` uses for stock), since an option's real loss can reach 100% of what was paid within the hold window in a way stock rarely does.** Real number not yet fixed -- needs a deliberate choice before building, same "the human decides on real risk parameters" precedent as everywhere else.
- **Exit: the same fixed 60-trading-day hold PEAD's own stock version already uses and was actually validated at** -- not a new exit rule invented for the options version.
- **Put delta**: Black-Scholes put delta = call delta − 1, a standard, direct relationship -- `data/options_chain.py` needs this added (currently only has the call-side formula, since covered calls never needed puts).

## Real infrastructure needed

- Extend `data/options_chain.py` with `black_scholes_put_delta()` and a wider default DTE window (currently tuned to covered calls' 25-50 day range; PEAD's overlay needs ~90-150).
- A new state machine, `options_income.py`/covered-calls-shaped, not stock-`check_and_update()`-shaped: on a real PEAD entry signal, buy the selected call/put instead of stock; hold to the 60-trading-day mark or let it ride to expiration if that comes first (shouldn't happen given the DTE buffer, but a real edge case to handle, not ignore); record real premium paid, real strike, real expiration.
- Reuses `strategies/pead.py`'s existing entry/exit signal logic unchanged -- this overlay only changes what gets bought when a signal fires, not when or why it fires.

## Validation plan -- forward only, with a real, natural comparison built in

No backtest is possible (same real data gap as covered calls). But there's a genuinely useful natural control already running: plain stock-based PEAD is already live on all 32 tickers. When a real signal fires, both versions can act on the identical real event -- meaning every real trade this overlay makes has a direct, apples-to-apples comparison already available (what did the stock version of the exact same signal actually do?) without waiting for a separate backtest. That's a real, if slow, way to learn whether the leverage is worth its cost.

## Real open decisions, not made here

- Exact position-size fraction per trade.
- Whether to run this on all 32 PEAD tickers immediately or a smaller pilot subset first, given it's forward-validated-only and adds real new operational complexity (selecting a real contract each time a signal fires, not just buying shares).

**Status: architected, not built.** No code written yet -- this is the complete design so building it is a mechanical next step once the two open decisions above are made.
