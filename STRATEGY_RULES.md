# STRATEGY_RULES.md

## Strategy name

```text
9ema-v1
```

## Strategy purpose

Identify high-quality, long-only momentum continuation setups among active Binance Spot USDT markets.

The strategy focuses on tokens that are already receiving meaningful market attention and are interacting constructively with a rising 9-period exponential moving average.

This is a screening and paper-tracking strategy.

It is not an automatic trading system.

---

# Market scope

Use:

* Exchange: Binance Spot
* Quote currency: USDT
* Direction: Long only
* Primary setup timeframe: 15 minutes
* Trend confirmation timeframe: 1 hour
* Broader context timeframe: 4 hours
* Market context: BTC/USDT
* Optional secondary context: ETH/USDT

Use only closed candles for confirmed signals.

An open candle may only produce a forming signal.

---

# Market universe

Start with active Binance Spot pairs quoted in USDT.

Exclude:

* Inactive markets
* Non-spot markets
* Stablecoin base assets
* Fiat base assets
* Leveraged tokens
* Tokens with insufficient candle history
* Tokens below minimum quote-volume requirements
* Tokens with excessive bid-ask spread
* Markets with missing required data

Default stablecoin and fiat exclusions should include, but not be limited to:

```text
USDC
FDUSD
TUSD
DAI
USDP
BUSD
EUR
GBP
TRY
BRL
AUD
```

Default leveraged-token suffix exclusions should include:

```text
UP
DOWN
BULL
BEAR
```

All exclusions must be configurable.

---

# Required candle history

Fetch at least 200 candles for:

* 15-minute timeframe
* 1-hour timeframe
* 4-hour timeframe

Do not evaluate a setup when there is insufficient data to calculate the required indicators and warm-up periods.

---

# Indicators

Calculate:

* EMA 9
* EMA 20
* EMA 50
* ATR 14
* Average volume over 20 completed candles
* Relative volume
* EMA 9 slope
* EMA 20 slope
* Distance from EMA 9 as a percentage
* Distance from EMA 9 in ATR units
* Candle range
* Candle body size
* Candle body percentage
* Upper wick
* Lower wick
* Rolling recent high
* Rolling recent low
* Recent swing high
* Recent swing low
* Distance to 24-hour high
* Relative strength versus BTC
* Optional rolling or session VWAP

Do not use future candles.

---

# Indicator formulas

## EMA distance percentage

```text
ema9_distance_percent =
abs(close - ema9) / ema9
```

## EMA distance in ATR units

```text
ema9_distance_atr =
abs(close - ema9) / atr14
```

## Relative volume

Use only completed candles.

```text
relative_volume =
latest_closed_candle_volume
/
average_volume_of_previous_20_closed_candles
```

Do not include the latest candle in the 20-candle average denominator.

## Distance from 24-hour high

```text
distance_from_24h_high =
(24h_high - current_price) / 24h_high
```

## EMA slope

Use a configurable lookback.

Initial default:

```text
ema9_slope =
(ema9_current - ema9_3_candles_ago)
/
ema9_3_candles_ago
```

A positive value means the EMA is rising.

## Candle body percentage

```text
body_percent =
abs(close - open)
/
(high - low)
```

Handle zero-range candles safely.

---

# Default screening thresholds

Initial defaults:

```text
minimum_24h_quote_volume: 10,000,000 USDT
minimum_positive_24h_move: 4%
minimum_15m_relative_volume: 1.5
maximum_bid_ask_spread: 0.20%
maximum_ema9_distance_percent: 0.50%
maximum_ema9_distance_atr: 0.25 ATR
minimum_alert_score: 80
minimum_reward_to_risk: 2.0
```

These are starting values, not permanent truths.

All thresholds must be configurable.

---

# Token “in play” definition

A token is considered in play when it demonstrates sufficient liquidity, movement, and current participation.

Evaluate:

* 24-hour percentage move
* 24-hour quote volume
* Relative volume
* Recent volume acceleration
* Distance from 24-hour high
* Trade activity when available
* Spread
* Relative strength versus BTC
* CoinGecko trend context when available

An asset should normally satisfy:

* Active Binance Spot USDT market
* Minimum 24-hour quote volume
* Positive 24-hour movement above the configured threshold
* Acceptable spread
* Sufficient candle history

Relative volume and proximity to the 24-hour high affect quality and scoring.

Do not require every asset to meet every preferred condition before it can appear on a watchlist.

---

# Market context

Evaluate BTC/USDT on:

* 15 minutes
* 1 hour
* 4 hours

Record:

* Close relative to EMA 9
* Close relative to EMA 20
* EMA 9 slope
* EMA 20 slope
* ATR and abnormal volatility
* Current short-term trend

BTC weakness should reduce setup quality but should not automatically reject every setup.

An individual token showing strong relative strength may still qualify.

---

# Relative strength versus BTC

Measure token performance against BTC over the same lookback.

Initial method:

```text
token_return = token_close_current / token_close_lookback - 1
btc_return = btc_close_current / btc_close_lookback - 1

relative_strength = token_return - btc_return
```

Calculate on at least:

* 15-minute context
* 1-hour context

Use configurable lookbacks.

A positive result means the token is outperforming BTC.

---

# Higher-timeframe bullish trend

## One-hour requirements

For full trend qualification:

* Latest closed candle closes above EMA 9.
* EMA 9 is rising.
* EMA 9 is above EMA 20.
* EMA 20 is flat or rising.

Preferred additional conditions:

* Close above EMA 20.
* EMA 20 above EMA 50.
* Four-hour trend aligned.
* No major bearish displacement candle immediately before the setup.

A setup may remain on the watchlist when one preferred condition is absent, but the score should be reduced.

## Four-hour alignment

Preferred bullish four-hour context:

* Close above EMA 9.
* EMA 9 above EMA 20.
* EMA 9 rising.
* EMA 20 flat or rising.

Four-hour misalignment does not always invalidate a setup, but it reduces the trend score.

---

# Setup 1: 9 EMA pullback

## Purpose

Identify an orderly pullback toward a rising 9 EMA after a bullish impulse.

## Required context

* Asset is in play.
* One-hour bullish trend remains valid.
* Fifteen-minute EMA 9 is rising or not materially falling.
* Price was recently extended above EMA 9.
* The pullback does not destroy bullish structure.

## Recent extension

Before the pullback, price must have traded meaningfully above EMA 9.

Initial definition:

```text
maximum close above ema9 within previous 3 to 8 candles
>= 0.5 ATR
```

Make the lookback and extension threshold configurable.

## EMA proximity

At least one closed pullback candle must satisfy either:

```text
abs(close - ema9) / ema9 <= 0.005
```

or:

```text
abs(close - ema9) <= 0.25 * ATR(14)
```

The detector may use either threshold or require both, based on configuration.

## Orderly pullback characteristics

Preferred characteristics:

* Pullback candles are smaller than the preceding impulse candles.
* Pullback volume contracts.
* Price does not close materially below EMA 9.
* No large bearish candle breaks the recent swing low.
* The pullback forms a higher low.
* Pullback depth is reasonable relative to ATR.

## Material EMA loss

Initial definition:

```text
close < ema9 - 0.35 * ATR
```

A close below this level should invalidate or significantly penalize the setup.

Make the threshold configurable.

## Pullback-volume contraction

Compare average pullback volume against average impulse volume.

Initial definition:

```text
average_pullback_volume
<
0.85 * average_impulse_volume
```

The impulse and pullback windows must be deterministic and configurable.

## Confirmation trigger

A pullback becomes confirmed when a closed candle does at least one of the following:

* Closes above EMA 9 after testing it.
* Breaks the high of the prior pullback candle.
* Breaks the defined pullback trigger level.
* Forms a bullish reclaim with acceptable candle structure.

Preferred confirmation:

* Trigger candle volume exceeds pullback average volume.
* Trigger candle closes in the upper portion of its range.
* Trigger occurs without immediate resistance overhead.

## Invalidation

Use one of:

* Below the pullback swing low.
* Below the trigger candle low.
* ATR-adjusted level below the pullback structure.

The selected rule must be documented in the setup result.

---

# Setup 2: 9 EMA reclaim

## Purpose

Identify price temporarily trading below or around EMA 9 and then reclaiming the EMA while the higher-timeframe trend remains bullish.

## Required context

* Asset is in play.
* One-hour bullish trend remains valid.
* The pullback does not invalidate the recent bullish structure.
* Price trades below or around EMA 9.
* A later candle closes back above EMA 9.

## Forming reclaim

A reclaim is forming when:

* The active candle is trading back above EMA 9.
* The candle has not closed.

A forming reclaim must never be labeled confirmed.

## Confirmed reclaim

A reclaim is confirmed only when a completed candle:

* Closes above EMA 9.
* Previously traded below or near EMA 9.
* Has acceptable body structure.
* Does not close with a severe upper rejection.
* Does not reclaim directly beneath nearby resistance.

Preferred additional confirmation:

* Reclaim volume is higher than average pullback volume.
* The reclaim candle breaks the prior candle high.
* The candle closes in the upper 40% of its range.
* Relative volume is above 1.0.

## Reclaim candle quality

Initial preferred body rule:

```text
body_percent >= 0.50
```

Initial preferred close-location rule:

```text
(close - low) / (high - low) >= 0.60
```

Handle zero-range candles safely.

These thresholds are configurable.

## Reclaim invalidation

Use one of:

* Reclaim candle low.
* Pullback swing low.
* Reclaim candle low minus an ATR buffer.

Penalize reclaims requiring unusually wide risk relative to ATR.

---

# Setup 3: 9 EMA continuation

## Purpose

Identify a tight consolidation above a rising EMA 9 followed by a volume-supported breakout.

## Required context

* Asset is in play.
* One-hour bullish trend is valid.
* Fifteen-minute price remains above or close to a rising EMA 9.
* Price forms a tight consolidation.
* Consolidation volume contracts.
* A closed candle breaks the consolidation high.

## Consolidation window

Initial default:

```text
3 to 6 completed 15-minute candles
```

Make the range configurable.

## Tight consolidation definition

Use a transparent quantitative definition.

Initial rule:

```text
consolidation_high - consolidation_low
<= 1.0 * ATR(14)
```

Preferred stronger condition:

```text
average candle range during consolidation
<
0.75 * average candle range of previous 10 candles
```

At least 70% of consolidation candle closes should remain above EMA 9.

The consolidation low must not break the recent meaningful swing low.

## Consolidation-volume contraction

Initial rule:

```text
average consolidation volume
<
0.80 * average volume of previous 10 candles
```

Make the threshold configurable.

## Breakout confirmation

A continuation is confirmed when a completed candle:

* Closes above the consolidation high.
* Has acceptable body structure.
* Shows volume expansion.

Initial volume rule:

```text
breakout volume
>= 1.5 * average volume of previous 20 completed candles
```

The breakout should not be confirmed solely because the candle wick traded above resistance.

## False breakout

Treat the breakout as weak or invalid when:

* The candle trades above the range but closes back inside it.
* The breakout candle has a severe upper wick.
* Breakout volume is below the configured threshold.
* Nearby higher-timeframe resistance leaves inadequate reward-to-risk.

---

# Market structure

Use deterministic swing detection.

Initial approach:

A swing high occurs when a candle high is greater than the highs of a configurable number of candles on both sides.

A swing low occurs when a candle low is lower than the lows of a configurable number of candles on both sides.

Default pivot width:

```text
2 candles on each side
```

Because right-side confirmation requires future candles, do not use an unconfirmed pivot for real-time confirmation.

For real-time scanning, use only fully confirmed historical pivots.

Calculate:

* Latest confirmed swing high
* Latest confirmed swing low
* Previous confirmed swing high
* Previous confirmed swing low
* Higher-high status
* Higher-low status
* Distance to recent swing high
* Distance to rolling resistance

---

# Nearby resistance

Use transparent resistance estimates.

Evaluate:

* Latest confirmed swing high
* Highest high over a configurable lookback
* 24-hour high
* Four-hour recent high

Select the nearest valid resistance above the trigger price.

Calculate:

```text
room_to_resistance =
resistance_price - trigger_price
```

Calculate estimated reward-to-risk:

```text
risk_per_unit =
trigger_price - invalidation_price

reward_to_risk =
room_to_resistance / risk_per_unit
```

Reject or penalize setups when:

* Invalidation is at or above trigger price.
* Risk is zero or negative.
* Resistance is below trigger price.
* Reward-to-risk is below the configured minimum.
* Risk distance is excessive relative to ATR.

---

# Trigger levels

## Pullback trigger

Use one of:

* High of the bullish reclaim candle
* High of the prior pullback candle
* Defined local pullback resistance

## Reclaim trigger

Default:

```text
high of the confirmed reclaim candle
```

## Continuation trigger

Default:

```text
consolidation high
```

Record which trigger rule was used.

---

# Invalidation levels

## Pullback

Default:

```text
recent pullback swing low
```

Optional ATR buffer:

```text
swing_low - 0.10 * ATR
```

## Reclaim

Default:

```text
reclaim candle low
```

Alternative:

```text
pullback swing low
```

## Continuation

Default:

```text
consolidation low
```

Alternative:

```text
breakout candle low
```

Select the configured rule and include it in the setup explanation.

---

# R levels

For a long setup:

```text
risk_per_unit = trigger_price - invalidation_price
```

Then calculate:

```text
1R = trigger_price + risk_per_unit
2R = trigger_price + 2 * risk_per_unit
3R = trigger_price + 3 * risk_per_unit
```

Do not calculate R levels when the invalidation is not below the trigger.

---

# Setup scoring

Score each candidate from 0 to 100.

Use proportional scoring where practical instead of abrupt binary cutoffs.

Every score must include:

* Category score
* Individual rule contributions
* Deductions
* Explanation codes
* Raw metric values

---

# Category 1: In-play quality

Maximum: 25 points

## 24-hour move

Maximum: 5 points

Initial scale:

```text
below 2%: 0
2% to 4%: proportional up to 2
4% to 8%: proportional from 2 to 4
above 8%: 5
```

Avoid rewarding extreme moves indefinitely.

Consider adding a penalty for excessively extended assets.

## Quote volume

Maximum: 5 points

Initial scale:

```text
below minimum threshold: 0
10M to 25M USDT: 2
25M to 75M USDT: 3
75M to 200M USDT: 4
above 200M USDT: 5
```

Make bands configurable.

## Relative volume

Maximum: 5 points

Initial scale:

```text
below 1.0: 0
1.0 to 1.5: proportional up to 2
1.5 to 2.5: proportional from 2 to 4
above 2.5: 5
```

## Proximity to 24-hour high

Maximum: 5 points

Higher points when price is close to the 24-hour high without being excessively extended.

Initial concept:

```text
within 2%: 5
within 5%: 4
within 10%: 2
more than 10% away: 0
```

## CoinGecko enrichment

Maximum: 5 points

Possible factors:

* Trending asset
* Strong category
* Strong market-cap-adjusted volume
* Positive market context

When CoinGecko is disabled or unavailable:

* Mark the category unavailable.
* Normalize the final score to the available maximum, or redistribute according to configuration.
* Do not automatically cap every setup below A quality.

---

# Category 2: Trend quality

Maximum: 25 points

* One-hour close above EMA 9: 5
* One-hour EMA 9 rising: 5
* One-hour EMA 9 above EMA 20: 5
* Four-hour trend alignment: 5
* Relative strength versus BTC: 5

Use proportional scoring for slope and relative strength where appropriate.

---

# Category 3: Pattern quality

Maximum: 25 points

* Healthy EMA 9 interaction: 5
* Contracting pullback or consolidation volume: 5
* Higher-low structure: 5
* Acceptable pullback depth in ATR units: 5
* No immediate major resistance: 5

Apply setup-specific interpretations.

For continuation setups, replace pullback-specific logic with consolidation-quality logic.

---

# Category 4: Trigger quality

Maximum: 15 points

* Confirmed reclaim or breakout candle: 5
* Trigger volume expansion: 5
* Break of the defined trigger level: 5

A forming signal cannot receive full confirmation points.

---

# Category 5: Liquidity and risk context

Maximum: 10 points

* Tight spread: 3
* Adequate liquidity: 3
* At least 2R room before nearby resistance: 4

Penalize:

* Excessive spread
* Thin liquidity
* Wide invalidation
* Immediate resistance
* Poor reward-to-risk

---

# Grades

```text
A+: 90 to 100
A: 80 to 89
B: 70 to 79
Watch: 60 to 69
Ignore: below 60
```

Only confirmed setups with a score at or above the configured alert threshold should create immediate alerts.

Default alert threshold:

```text
80
```

Forming setups may appear on the dashboard but must not generate confirmed alerts.

---

# Extension penalties

A token may be strong but too extended to offer a quality entry.

Apply penalties when:

* Price is too far above the 15-minute EMA 9.
* Price is too far above the 1-hour EMA 9.
* The recent move is multiple ATR units without consolidation.
* Trigger-to-invalidation distance is excessive.
* Entry would occur directly below resistance.

Initial excessive-extension condition:

```text
close - ema9 > 1.5 * ATR
```

Make this configurable.

---

# Liquidity rules

Calculate spread:

```text
spread_percent =
(ask - bid) / mid_price
```

where:

```text
mid_price = (ask + bid) / 2
```

Default maximum spread:

```text
0.20%
```

When order-book depth is available, calculate approximate quote liquidity within configurable price bands.

Do not reject a market solely because detailed depth is temporarily unavailable if volume and spread remain acceptable.

Mark depth as unavailable.

---

# Setup status rules

## Forming

Use when:

* Conditions are developing.
* The required confirmation candle remains open.
* A breakout or reclaim has not closed.

## Confirmed

Use when:

* All mandatory conditions are satisfied.
* The trigger candle is closed.
* The setup score meets the configured confirmation minimum.

## Invalidated

Use when:

* Price trades through the invalidation level before reaching the tracked objective.
* Higher-timeframe structure is materially broken before activation.
* The setup explicitly violates a mandatory strategy rule.

## Expired

Use when:

* The setup does not trigger within the configured number of candles.
* The setup becomes stale.
* Market conditions materially change without a direct invalidation.

Default expiration should be configurable by setup type.

---

# Paper outcome tracking

After confirmation, track:

* Maximum favorable excursion
* Maximum adverse excursion
* 1R reached
* 2R reached
* 3R reached
* Invalidation reached
* Which occurred first
* Time to each event
* Number of candles to each event
* Setup expiration

Use candle high and low conservatively.

When both a target and invalidation occur within the same candle and intrabar order is unknown:

* Mark the outcome ambiguous, or
* Apply a documented conservative assumption.

Do not silently assume the favorable event occurred first.

---

# Explainability

Every setup must include human-readable reasons.

Example:

```text
Qualified because:
- 24h quote volume is 84.2M USDT.
- 15m relative volume is 2.1.
- 1h close is above a rising EMA 9.
- 1h EMA 9 is above EMA 20.
- Pullback volume contracted by 28%.
- Price reclaimed the 15m EMA 9 on a closed candle.
- Estimated room to resistance is 2.6R.

Deductions:
- 4h EMA 9 slope is nearly flat.
- BTC 1h context is weak.
```

Also store machine-readable reason codes.

Example codes:

```text
HTF_CLOSE_ABOVE_EMA9
HTF_EMA9_RISING
HTF_EMA9_ABOVE_EMA20
PULLBACK_VOLUME_CONTRACTING
EMA9_RECLAIM_CONFIRMED
BREAKOUT_VOLUME_EXPANDING
NEARBY_RESISTANCE
BTC_CONTEXT_WEAK
SPREAD_TOO_WIDE
SETUP_TOO_EXTENDED
```

---

# No-lookahead requirements

* Never use future candles in indicator calculations.
* Never use unconfirmed swing points.
* Never classify an open candle as a confirmed trigger.
* Never include the current candle in a prior-volume average when comparing that candle to history.
* Backtests and fixtures must process candles sequentially.
* A detector must produce the same result in live and historical sequential processing.

---

# Initial alert format

An alert should contain:

```text
A Setup: ABC/USDT
Score: 87
Setup: 15m 9 EMA Reclaim
Status: Confirmed
24h Move: +9.8%
24h Quote Volume: 84.2M USDT
Relative Volume: 2.3
1h Trend: Bullish
4h Trend: Bullish
BTC Relative Strength: Positive
Trigger: 1.245
Invalidation: 1.217
Nearest Resistance: 1.318
Estimated Reward-to-Risk: 2.6R
Strategy Version: 9ema-v1
```

Include a short explanation of the strongest qualifying factors and material risks.

---

# Strategy limitations

This strategy does not account for:

* News catalysts
* Token unlocks
* Exchange maintenance
* Manipulative order-book behavior
* Slippage
* Trading fees
* Tax consequences
* Position sizing
* Portfolio correlation
* Real execution quality
* Fundamental token quality

Performance statistics must be described as paper-tracking results unless a complete execution model is later added.
