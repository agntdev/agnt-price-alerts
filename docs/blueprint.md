# Crypto Price Tracker & Alerts — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for tracking crypto prices, setting threshold/percent-change alerts, managing watchlists, and receiving daily summaries. Users control quiet hours, alert cooldowns, and get deduplicated notifications. Owner receives anonymized usage metrics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual crypto traders
- crypto holders
- non-technical Telegram users

## Success criteria

- users can manage watchlists with inline buttons
- price alerts trigger without spam
- morning summaries sent at configured times
- owner sees anonymized metrics via /stats

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Onboarding menu with timezone setup and basic instructions
- **/price** (command, actor: user, command: /price) — Show current price of a specific ticker or full watchlist summary
- **Manage Watchlist** (button, actor: user, callback: watchlist:open) — Inline menu to add/remove coins, configure alerts
- **Set Price Alert** (button, actor: user, callback: alert:price) — Configure threshold-based price alerts
- **Set % Alert** (button, actor: user, callback: alert:percent) — Configure percent-change alerts with window
- **/stats** (command, actor: owner, command: /stats) — Show aggregated usage metrics (active users, top alerts)

## Flows

### Onboarding
_Trigger:_ /start

1. Ask for timezone preference
2. Set default quiet hours (23:00-07:00)
3. Explain core features

_Data touched:_ user profile

### Watchlist Management
_Trigger:_ watchlist:open

1. Show popular coin buttons
2. Custom ticker input
3. Inline actions for each coin (remove, set alerts)

_Data touched:_ watchlist entry

### Price Alert Setup
_Trigger:_ alert:price

1. Select coin
2. Enter threshold value
3. Choose above/below direction
4. Confirm rule

_Data touched:_ price alert

### Percent Alert Setup
_Trigger:_ alert:percent

1. Select coin
2. Enter percent change
3. Choose window (1h, 6h, etc)
4. Select direction (up/down/both)
5. Confirm rule

_Data touched:_ price alert

### Morning Summary
_Trigger:_ scheduled:summary

1. Check quiet hours status
2. Generate price changes summary
3. Send compact message with key movements

_Data touched:_ user profile, notification event

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User preferences and settings
  - fields: telegram_id, timezone, quiet_hours, summary_time, alert_cooldowns
- **watchlist_entry** _(retention: persistent)_ — Tracked cryptocurrency with alert rules
  - fields: ticker, display_name, price_alerts, percent_alerts, last_notified
- **price_alert** _(retention: persistent)_ — Active alert configuration
  - fields: type, threshold_value, percent_window, direction, enabled
- **notification_event** _(retention: session)_ — Sent alert record
  - fields: coin, old_price, new_price, percent_change, timestamp
- **owner_metrics** _(retention: persistent)_ — Anonymized usage statistics
  - fields: active_users, alert_triggers_by_ticker, alert_type_counts

## Integrations

- **Telegram** (required) — Messaging and inline buttons
- **Crypto Price API** (required) — Price data polling with retries
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /stats command to view aggregated metrics
- Ability to configure API price source
- Set default cooldown periods and alert windows

## Notifications

- Price threshold crossed alerts
- Percent change alerts
- Morning summary digest
- Error notifications for invalid tickers

## Permissions & privacy

- All user data is private and not shared
- Owner only sees anonymized metrics
- No cross-user visibility of watchlists

## Edge cases

- Unknown ticker normalization and suggestions
- API failure retries with silent alert suppression
- Quiet hours alert queuing
- Cooldown period enforcement

## Required tests

- Verify alert deduplication during price oscillations
- Test morning summary formatting with multiple coins
- Validate timezone-based quiet hours suppression

## Assumptions

- Default 1-hour window for percent alerts
- 6-hour cooldown for threshold alerts
- Telegram locale used for default timezone
