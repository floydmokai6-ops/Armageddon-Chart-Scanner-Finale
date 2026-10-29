# Armageddon Chart Scanner 2.8 — deployment bundle

## Deploy to GitHub Pages
1. Extract this ZIP.
2. Upload **the contents** of this folder to the root of your GitHub Pages repository (not the containing folder itself).
3. Make sure `index.html`, `manifest.webmanifest`, `sw.js`, and the `icons/` folder are all at the repository root/relative paths shown here.
4. In GitHub, open **Settings → Pages** and select the correct branch and root folder.
5. Open the deployed URL in Chrome on Android. Test **Demo (simulated)** first, then try live data.
6. To install, open the deployed site in Chrome → menu ⋮ → Install app / Add to Home screen.

## What is included
- Preserved self-contained mobile HTML app.
- 714 Method strategy for SAST windows 07:00–08:00, 13:00–14:00 and 16:00–17:00, using M5 pullback/continuation confirmation and a fixed target distance based on a configured pip convention.
- Voice alerts and 1% default risk calculator.
- PWA manifest, icons and a basic app-shell service worker.
- Licence-key system intentionally omitted.

## Changes in 2.8 (stop-loss fix)
Problem: backtests of 2.7 showed almost every signal hitting stop loss.
- Minimum stop distance: every stop is now at least 1.3 ATR from entry (was often 0.3-0.6 ATR, i.e. inside normal candle noise). Stops are never tightened, only widened; trades needing more than 3.5 ATR are skipped. TP1-3 are recalculated from the new risk.
- Hard higher-timeframe filter: a signal against the higher-timeframe trend (EMA20 vs EMA50 on M1->M15, M5->H1, M15->H4 ...) is dropped instead of just losing 8 confidence. 714 Method is exempt.
- Default minimum confidence 60 -> 70 (an existing phone keeps its saved value; change it in Settings).
- Cooldown: after a stop-out, no new signal on the same symbol and direction for 8 candles; only one live signal per symbol and direction.
- Spread check: signals are skipped when the spread is more than 12% of the planned risk.
- Tested on synthetic replay only (same engine, old vs new settings): no errors, about 57% fewer signals. Synthetic random data cannot show profitability. Re-run your own backtest on real prices before trusting it.
- If a strategy still loses on real data, switch it off per strategy rather than using ALL.

## Changes in 2.7
- Aggressive Scalping (M5 default, use M1-M5): looser and faster than Scalping. EMA 5/13/34 stack + micro-breakout of the last 3 candles, very tight SL, quick TPs (0.8R / 1.2R / 1.8R), 3-candle expiry. Expect more signals; it will not chase a candle that has already run more than 1.2 ATR from the EMA5.
- Zone to Zone (M15): buys a demand zone / sells a supply zone and targets the NEXT opposite zone. TP1 halfway, TP2 at the front edge of the next zone, TP3 at its far side. Skipped unless the trip pays at least 1.5R to the zone.
- Mean Reversion (M15, use M5-M15): fades a close outside Bollinger(20, 2) once a bullish/bearish candle closes back inside, only in ranging markets (efficiency ratio below 0.35) with RSI below 35 (above 65 for sells). TP1 = the mean, TP2 = mean + 1 deviation, TP3 = mean + 1.5. Needs at least 0.8R to the mean.
- Fixed a wording bug found while testing (a reason line said "undefined").
- Tested on constructed patterns (BUY and SELL) and 300 random-walk series: no errors, TP levels always ascending. Not backtested on real prices.

## Changes in 2.6
- Midpoint Strategy added (M15 default). Two modes: (a) the 50% midpoint of a strong impulse candle that price returns to and holds; (b) the 50% midpoint of the last swing leg on its first touch. A rejection candle triggers the entry. Works for BUY and SELL, TP1-3, reasons on the card, brain + AI analyst apply. These rules are an interpretation; tell the developer if your Midpoint rules differ.
- Trendlines was already added in 2.5.

## Changes in 2.5 (new strategies)
- Support & Resistance: zones from clustered swing points (3 touches, or 2 with a perfect rejection), entry on a shadow/engulfing rejection at the zone.
- Breakout: tight 20-candle range tested at least twice, then a strong expansion candle closes outside it (not chased).
- Trendlines: auto-drawn 3-touch lines through swing points; bounces off a rising/falling line that held, and breakouts through a line that capped price.
- SMC: break of structure / change of character with displacement, entry at the unmitigated order block in discount (buy) or premium (sell), with FVG, killzone and higher-timeframe bonuses.
- All four work for BUY and SELL, give TP1-3, show their reasons on the card, go through the brain and AI analyst, and are included in "All". They default to M15.
- Tested on constructed patterns (both directions) and 300 random-walk series without errors. Not backtested on real prices.

## Changes in 2.4 (AI upgrade)
- Brain (no key needed): reads each market's regime (trending / ranging / volatile / dead), skips dead markets, boosts or cuts confidence depending on whether a strategy fits the regime, and learns from the scanner's own closed signals (win/loss record per strategy and symbol).
- AI Analyst (optional): Claude reviews each new signal (candles, regime, news) and returns TAKE / WAIT / SKIP, risks and trade management. Needs your own Anthropic API key (Settings -> AI Analyst); sent only to api.anthropic.com; capped at 8 automatic reviews per 10 minutes. Each card also has an "Ask AI analyst" button.
- Voice: choose voice and speed, speaks new signals (with regime), AI analyst verdicts and trade updates (triggered / TP / SL), plus a spoken market briefing.
- None of this guarantees profit; the scanner is still rule-based and unproven by backtest.

## Changes in 2.3 (fixes)
- 714 Method: the 50-pip target used the display precision for non-forex markets, so Silver got a 5.00 target (~17% of price) and Bitcoin/indices/synthetics got ~0.5. It now uses real pip sizes (forex 0.0001 / JPY 0.01, Gold 0.1, Silver 0.01, Platinum/Palladium 0.1, oil 0.01). Markets with no standard pip are skipped until you set one in Settings -> 714 Method pip sizes.
- 714 Method: it fired a new signal on every continuation candle in the hour; now one signal per symbol per session.
- Notifications pointed at icon files that do not exist (assets/...); now icons/icon-192.png.
- Duplicate-news guard: the stored news flags were pruned alphabetically instead of oldest-first, which could re-fire a news straddle after a few weeks of use.
- Loading-failure message no longer mentions css/js/assets folders (the app is one file).
- Service-worker cache renamed so installed phones pick up this version.

## Important verified limitations
- The scanner does not place trades.
- The 714 Method has logic-level tests for bullish and bearish setups and out-of-session rejection, but is not backtested for profitability.
- The public Deriv WebSocket feed is not assumed to provide WTI oil. Crude Oil WTI is configured for the MetaApi broker bridge; it requires a connected broker offering oil and the exact broker symbol. A common name such as `USOIL` or `XTIUSD` is broker-specific; do not assume one is correct without checking your MT5 symbol list.
- The risk calculator estimates lots only from values you enter; it does not fetch account balance or contract specifications automatically.
- News calendar events are not the same as verified live fundamental analysis.
- The service worker caches the app shell only. Live market prices and external data still require an internet connection.

## Tests performed on this build
- Main JavaScript module syntax check with Node.js.
- Unit-style tests for bullish 714 detection, bearish 714 detection, fixed target distance, and no signal outside configured session windows.
- Manifest JSON parse and ZIP integrity checks.
- Live broker connectivity and a full browser session could not be verified in this environment; complete those checks on your hosted GitHub Pages URL and broker account.
