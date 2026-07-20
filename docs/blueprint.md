# CryptoPing Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A personal Telegram bot for tracking crypto price alerts and watchlists with customizable thresholds, percent-change notifications, and optional morning summaries while respecting user privacy and quiet hours.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Regular crypto traders
- Crypto hobbyists
- Telegram users seeking lightweight price alerts

## Success criteria

- Users can create and manage private watchlists with active alerts
- Alerts are delivered according to configured thresholds and suppression rules
- Daily morning summaries are sent at user-specified times
- Owner receives aggregated privacy-preserving metrics

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding and open main menu
- **/price** (command, actor: user, command: /price) — Check current price of specified coin or full watchlist
- **Add Coin** (button, actor: user, callback: watchlist:add) — Add a cryptocurrency to watchlist with quick buttons for popular coins
- **Manage Alerts** (button, actor: user, callback: alerts:manage) — View and modify active alerts for selected coin
- **/admin_stats** (command, actor: owner, command: /admin_stats) — View aggregated user metrics (owner-only)

## Flows

### Onboarding
_Trigger:_ /start

1. Display welcome message
2. Request timezone selection
3. Quick-add popular coins (BTC, ETH, TON)
4. Explain alert types
5. Set quiet hours
6. Optional morning summary setup

_Data touched:_ user_profile

### Alert Creation
_Trigger:_ button:alerts:create

1. Select coin from watchlist
2. Choose alert type (price threshold or percent change)
3. Configure parameters (price level, percent, window)
4. Confirm and save alert

_Data touched:_ alert, watchlist_item

### Morning Summary
_Trigger:_ scheduled:user_time

1. Check user's watchlist
2. Compile current prices and recent changes
3. Format summary message respecting quiet hours
4. Send to user

_Data touched:_ price_sample, user_profile

### Price Check
_Trigger:_ /price

1. Parse coin identifier
2. Fetch latest price from feed
3. Calculate percent change
4. Display formatted price info

_Data touched:_ price_sample, user_profile

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User preferences and settings
  - fields: telegram_id, display_name, timezone, quiet_hours, summary_time, alert_suppression
- **watchlist_item** _(retention: persistent)_ — Tracked cryptocurrency with optional nickname
  - fields: coin_symbol, nickname, alerts
- **alert** _(retention: persistent)_ — Active price monitoring rule
  - fields: type, direction, target_value, window, last_fired, cooldown
- **price_sample** _(retention: session)_ — Timestamped price data for calculations
  - fields: coin_symbol, timestamp, price
- **owner_metrics** _(retention: persistent)_ — Aggregated usage statistics
  - fields: active_users, alert_counts, top_coins

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
- **Crypto Price Feed** (required) — External price data source
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /admin_stats command for viewing aggregated metrics
- Access to raw data for future monetization features

## Notifications

- Price threshold alerts with price comparison
- Percent change alerts with time window
- Morning summary with watchlist prices
- Quiet hours alert queuing

## Permissions & privacy

- All user data is private and not shared
- Owner metrics are aggregated counts only
- Price samples are short-term cached

## Edge cases

- Unknown/ambiguous coin tickers
- Price feed failures and retries
- Alert suppression during cooldown
- Quiet hours overlapping with alert triggers

## Required tests

- End-to-end alert suppression workflow
- Morning summary delivery during non-quiet hours
- Price sample caching for percent calculations
- Admin metrics aggregation without PII

## Assumptions

- Users set timezone during onboarding (default UTC)
- USD is default price currency
- 1-hour window for percent changes
- 6-hour cooldown for threshold alerts
