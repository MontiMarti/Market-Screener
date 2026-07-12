# Market-Screener

## Goal

Build a crypto market screener that finds high-quality, long-only 9 EMA setups among Binance Spot USDT pairs.

The system should identify tokens that are in play, rank setup quality, issue research alerts, and track what happened after each signal.

## Initial user

The application is initially for one private user.

## Primary workflow

1. Open the dashboard.
2. See which Binance Spot tokens are currently in play.
3. Review A and A+ 9 EMA setups.
4. Inspect why each setup received its score.
5. Review its trigger, invalidation, and nearby resistance.
6. Track the result after the alert.
7. Use historical statistics to improve the strategy.

## Initial strategy

* Long-only
* Binance Spot
* USDT pairs
* 15-minute setup timeframe
* 1-hour trend confirmation
* 4-hour broader context
* EMA 9 pullback
* EMA 9 reclaim
* EMA 9 continuation
* Volume and liquidity confirmation
* BTC-relative strength
* No automated trading

## Main product principles

* Simple and transparent
* No lookahead bias
* Closed candles for confirmed signals
* Every score must be explainable
* All thresholds must be configurable
* Public market data should be enough for the MVP
* The system must save signals for later validation

## First milestone

A local Dockerized dashboard that retrieves live public Binance data and produces ranked, explainable setup results.
