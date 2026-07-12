# AGENTS.md

## Project

Build a production-quality MVP for a cryptocurrency market screener that identifies high-quality long setups based primarily on the 9-period exponential moving average.

The application is a research, alerting, and paper-tracking tool.

It must not place real trades or require exchange trading credentials in the MVP.

Read these files before writing code:

* `PROJECT_BRIEF.md`
* `STRATEGY_RULES.md`
* `reference-links.md`
* `.env.example`

Create and maintain `PROJECT_PLAN.md` throughout implementation.

---

# Product objective

Build a crypto market screener that:

1. Finds Binance Spot USDT tokens that are currently in play.
2. Evaluates their trend across multiple timeframes.
3. Detects 9 EMA pullback, reclaim, and continuation setups.
4. Scores each setup from 0 to 100.
5. Displays ranked results in a usable web dashboard.
6. Saves historical scans and setup alerts.
7. Tracks what happened after each alert.
8. Supports configurable thresholds without requiring code changes.
9. Uses closed candles for confirmed signals.
10. Clearly labels incomplete signals as forming.
11. Does not execute trades.

---

# Phase 1 scope

Build a Binance Spot, USDT-quoted, long-only screener.

Do not include:

* Futures
* Margin
* Short setups
* Automatic order execution
* Portfolio management
* Exchange trading API keys
* Real-money position sizing
* Machine learning
* Social sentiment
* On-chain analytics

Design the application so additional exchanges and short setups can be added later.

---

# Technology stack

Use:

* Python 3.12 or newer
* CCXT for normalized exchange REST access
* Binance public REST endpoints when CCXT does not expose required information efficiently
* Binance public WebSocket streams only after the polling MVP works
* CoinGecko API for optional enrichment
* FastAPI
* SQLAlchemy 2.x
* Alembic
* PostgreSQL
* SQLite fallback for simple local development if useful
* Pandas or Polars
* Pydantic Settings
* Streamlit for the initial dashboard
* Pytest
* Docker
* Docker Compose
* Ruff
* Mypy or Pyright

Prefer maintainable, understandable code over unnecessary infrastructure.

Do not add Redis, Celery, Kafka, TimescaleDB, or separate microservices unless there is a demonstrated need.

---

# Data-source responsibilities

## Binance and CCXT

Use Binance as the primary market-data source.

Use CCXT for:

* Loading markets
* Retrieving tickers
* Fetching OHLCV candles
* Retrieving bid and ask data when available
* Normalizing exchange symbols and fields

Use public endpoints.

The MVP must not require a Binance API key.

Use native Binance public REST endpoints only when:

* CCXT does not provide the required field
* The native endpoint materially reduces API calls
* The native endpoint provides more reliable bulk market data

Document any direct Binance endpoint used.

## CoinGecko

CoinGecko is optional enrichment.

Use it for:

* Trending assets
* Market capitalization
* Token metadata
* Categories
* Market-wide discovery
* General market context

CoinGecko must not be the primary source for entry signals.

The main Binance scan must continue functioning if CoinGecko is disabled, unavailable, or rate limited.

CoinGecko enrichment must not block or fail the main scan.

---

# Application architecture

Use a modular structure similar to:

```text
crypto-9ema-screener/
  app/
    api/
    core/
    data_sources/
    indicators/
    scanners/
    strategies/
    scoring/
    tracking/
    models/
    schemas/
    services/
    dashboard/
  config/
    strategy.example.yaml
  migrations/
  scripts/
  tests/
    fixtures/
  .env.example
  .gitignore
  AGENTS.md
  PROJECT_BRIEF.md
  PROJECT_PLAN.md
  STRATEGY_RULES.md
  reference-links.md
  Dockerfile
  docker-compose.yml
  Makefile
  pyproject.toml
  README.md
```

Adjust this structure only when there is a clear architectural reason.

Keep exchange integrations separate from strategy logic.

Keep indicator calculations separate from setup detection.

Keep setup detection separate from scoring.

Keep scanning orchestration separate from API and dashboard code.

---

# Configuration

Create a human-editable configuration file:

```text
config/strategy.yaml
```

Also provide:

```text
config/strategy.example.yaml
```

Configuration must include:

* Exchange name
* Quote currency
* Excluded base assets
* Excluded symbol patterns
* Required timeframes
* Candle history limits
* Minimum 24-hour quote volume
* Minimum price movement
* Relative-volume thresholds
* Maximum spread
* EMA proximity thresholds
* ATR thresholds
* Resistance lookback
* Swing-detection parameters
* Setup expiration
* Alert score threshold
* Scan interval
* CoinGecko enabled or disabled
* Database settings through environment variables

Validate configuration at startup.

Fail with a useful error when configuration is invalid.

Do not scatter strategy thresholds throughout the code.

---

# Time handling

Use UTC internally.

Store all timestamps in UTC.

Include timezone-aware datetime objects.

Use exchange candle open times consistently.

Do not treat an active candle as closed.

A signal must not be marked confirmed until the relevant candle has closed.

---

# Scan modes

Implement two scan modes.

## Scheduled scan

The scheduled scan should:

1. Run every five minutes.
2. Refresh the eligible Binance market universe.
3. Fetch required market data and candles.
4. Apply liquidity and in-play filters.
5. Detect setups.
6. Score candidates.
7. Save scan results.
8. Create alerts for newly confirmed setups meeting the configured score threshold.
9. Update the lifecycle of previously confirmed setups.

## On-demand scan

Allow scans to be triggered through:

* CLI
* API
* Dashboard

Support filtering by:

* Symbol
* Minimum score
* Setup type
* Setup status

---

# Alert lifecycle

Use the following lifecycle states:

* `forming`
* `confirmed`
* `invalidated`
* `expired`
* `target_1_hit`
* `target_2_hit`
* `target_3_hit`

Prevent duplicate alerts.

Do not emit the same confirmed alert repeatedly for the same:

* Exchange
* Symbol
* Setup type
* Trigger candle
* Strategy version

Create database-level or application-level safeguards for idempotency.

---

# Historical tracking

For every confirmed setup, store:

* Exchange
* Symbol
* Setup type
* Setup status
* Detection timestamp
* Trigger candle timestamp
* Setup score
* Grade
* Full score breakdown
* Indicator snapshot
* Market-data snapshot
* Trigger price
* Invalidation price
* Resistance level
* 1R level
* 2R level
* 3R level
* Estimated reward-to-risk
* BTC market context
* CoinGecko context when available
* Maximum favorable excursion
* Maximum adverse excursion
* Whether 1R, 2R, or 3R was reached
* Whether invalidation was reached first
* Time or candle count to outcome
* Final status
* Strategy version

Do not store every raw candle indefinitely unless required.

Store enough information to reproduce and audit why a setup qualified.

---

# API endpoints

Implement at least:

```text
GET  /health
GET  /api/config
GET  /api/markets
GET  /api/setups
GET  /api/setups/{setup_id}
POST /api/scans/run
GET  /api/scans
GET  /api/stats/summary
GET  /api/stats/by-setup-type
GET  /api/stats/by-score-band
```

Support query parameters for:

* Minimum score
* Setup type
* Symbol
* Status
* Date range
* Limit
* Sort order

Use documented Pydantic request and response schemas.

Expose FastAPI-generated API documentation.

---

# Dashboard requirements

Create a clean functional dashboard.

## Overview

Display:

* Last successful scan
* Number of markets scanned
* Number of in-play tokens
* Number of forming setups
* Number of confirmed A and A+ setups
* BTC market context
* Binance data-source status
* CoinGecko data-source status

## Ranked screener table

Display:

* Symbol
* Score
* Grade
* Setup type
* Status
* Current price
* 24-hour percentage move
* 24-hour quote volume
* Relative volume
* Distance to EMA 9
* One-hour trend
* Four-hour trend
* BTC relative strength
* Bid-ask spread
* Trigger price
* Invalidation price
* Reward-to-risk before resistance
* Detection time

Provide:

* Sorting
* Minimum score filter
* Setup-type filter
* Status filter
* Symbol search

## Setup detail

Display:

* Score breakdown
* Setup-rule explanation
* Indicator values
* Trigger price
* Invalidation price
* Nearby resistance
* Reward-to-risk levels
* BTC market context
* CoinGecko context when available
* Lifecycle history
* Candlestick chart
* EMA 9
* EMA 20
* Volume

Clearly distinguish forming signals from confirmed signals.

## Performance section

Display:

* Confirmed setup count
* 1R hit rate
* 2R hit rate
* 3R hit rate
* Invalidation-first rate
* Average maximum favorable excursion
* Average maximum adverse excursion
* Results by setup type
* Results by score band
* Results by market regime

Do not label these results as profitability unless fees, slippage, and an execution model are included.

---

# Reliability requirements

Implement:

* Request timeouts
* Retry logic with exponential backoff
* Rate-limit awareness
* Graceful HTTP 429 handling
* Structured logging
* Partial scan completion when one symbol fails
* Missing-data handling
* Exchange connection health
* CoinGecko failure isolation
* Database migrations
* Idempotent scans
* Duplicate-alert prevention
* Clear error reporting

Do not silently swallow exceptions.

Do not aggressively retry rate-limit responses.

One failed symbol must not fail the entire scan.

---

# Security requirements

* Do not commit secrets.
* Include `.env.example`.
* Use environment variables for credentials.
* The MVP must work without Binance trading credentials.
* Support an optional CoinGecko API key.
* Never print API keys in logs.
* Validate API inputs.
* Do not expose production stack traces.
* Do not implement withdrawal or order permissions.
* Do not add real order-placement code.

---

# Testing requirements

Create automated tests for:

* Configuration validation
* EMA calculations
* ATR calculations
* Relative volume
* EMA slope
* Distance-to-EMA calculations
* Pullback detection
* Reclaim detection
* Continuation detection
* Score calculation
* Score-breakdown totals
* Candle-closure safeguards
* No-lookahead behavior
* Duplicate-alert prevention
* Stablecoin exclusions
* Leveraged-token exclusions
* Missing-data handling
* Reward-to-risk calculations
* Setup expiration
* Lifecycle updates

Create fixed OHLCV fixtures for:

* Valid pullback
* Invalid pullback
* Valid reclaim
* Forming but unconfirmed reclaim
* Valid continuation
* False breakout
* Illiquid candidate
* Candidate with excessive spread
* Candidate blocked by nearby resistance
* Candidate with insufficient candle history

Mocks are acceptable in tests.

The final scanner must also retrieve real public Binance market data.

---

# Developer experience

The project must run with:

```bash
docker compose up --build
```

Also provide non-Docker setup instructions.

Include:

* `README.md`
* `.env.example`
* `config/strategy.example.yaml`
* `Makefile`
* `Dockerfile`
* `docker-compose.yml`
* Alembic migrations
* Seed or fixture command
* Test command
* Lint command
* Type-check command
* Manual scan command

Provide Makefile commands similar to:

```text
make setup
make up
make down
make scan
make test
make lint
make typecheck
make migrate
make logs
```

---

# Implementation order

Follow this order:

1. Inspect the repository.
2. Report what already exists.
3. Create or update `PROJECT_PLAN.md`.
4. Create the project skeleton.
5. Add configuration and settings validation.
6. Add database models and migrations.
7. Implement Binance and CCXT data retrieval.
8. Implement indicator calculations.
9. Add unit tests for indicators.
10. Implement market-universe filtering.
11. Implement the in-play filter.
12. Implement setup detectors from `STRATEGY_RULES.md`.
13. Add detector tests.
14. Implement scoring and score explanations.
15. Add scoring tests.
16. Implement scan orchestration.
17. Implement persistence.
18. Implement alert deduplication.
19. Implement lifecycle tracking.
20. Implement API endpoints.
21. Implement dashboard.
22. Add CoinGecko enrichment behind a feature flag.
23. Add Docker support.
24. Complete documentation.
25. Run tests.
26. Run linting.
27. Run type checking.
28. Run database migrations.
29. Run a real public-data scan.
30. Fix all discovered errors.
31. Provide a final build report.

---

# Working behavior

Do not build the entire project in one unverified pass.

At each major stage:

1. Implement the smallest coherent unit.
2. Run relevant tests.
3. Inspect the result.
4. Fix failures before continuing.
5. Update `PROJECT_PLAN.md`.

Do not replace existing files without inspecting them.

Do not leave core functionality as pseudocode, placeholders, or TODO comments.

Do not mock production market-data retrieval.

Do not implement order execution.

If a requirement is ambiguous, choose the safest and simplest deterministic implementation, document the assumption, and continue.

---

# Coding standards

* Use type hints.
* Use clear domain-specific names.
* Keep functions focused.
* Avoid deeply nested logic.
* Prefer pure functions for indicators and setup detection.
* Use dependency injection for data sources where practical.
* Separate API schemas from database models.
* Add docstrings for non-obvious strategy logic.
* Include structured reason codes for score additions and deductions.
* Version the strategy logic.

Use a strategy version such as:

```text
9ema-v1
```

Store the version with every confirmed setup.

---

# Definition of done

The MVP is complete when:

1. The application starts with Docker Compose.
2. The database migrations run successfully.
3. A user can trigger a Binance Spot scan.
4. The scanner identifies active USDT markets.
5. Stablecoins, leveraged products, and illiquid markets are filtered.
6. Required indicators are calculated correctly.
7. Pullback, reclaim, and continuation setups are detected.
8. Closed-candle safeguards are enforced.
9. Results are scored and ranked.
10. Every score has an understandable breakdown.
11. Results are persisted.
12. The dashboard displays current ranked setups.
13. Confirmed setups include trigger, invalidation, and resistance context.
14. Duplicate alerts are prevented.
15. Historical outcomes can be tracked.
16. CoinGecko enrichment can be enabled or disabled.
17. Automated tests pass.
18. Linting passes.
19. Type checking passes.
20. The README contains exact setup and operating instructions.
21. No exchange trading keys are required.
22. No real trades can be placed.

---

# Final response required

When implementation is complete, provide:

* Summary of what was built
* Final project structure
* Files created or changed
* Exact startup commands
* Exact scan command
* Dashboard URL
* API documentation URL
* Database migration status
* Test results
* Lint results
* Type-check results
* Example scan output
* Known limitations
* Recommended next development milestone

Begin by inspecting the repository and creating `PROJECT_PLAN.md`.
