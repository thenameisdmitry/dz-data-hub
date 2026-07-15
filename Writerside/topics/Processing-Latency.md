# Monthly Active Users (MAU)

<tldr>

**Owner:** Product Analytics (`product-analytics@dz.software`)

**Status:** Approved

**Domain:** Product

**Primary source:** `fct_usage_sessions`

</tldr>

MAU is the number of distinct users who performed at least one
qualifying action in a DZ Software product during a calendar month.
It is the default measure of product reach.

## Definition

A user is **active** in a month if telemetry recorded at least one
qualifying in-product action for them in that calendar month.
A **qualifying action** is any tracked user-initiated event; background
events (telemetry heartbeats, automatic updates, license checks) do
not count.

## Calculation

- **Formula:** `COUNT(DISTINCT user_id)` per calendar month.
- **Grain:** user-month, reported per product and as a company total.
- **Excluded:** internal users, service accounts, users who opted out
  of telemetry.

> The company total is **not** the sum of per-product MAU: a user
> active in two products counts once in the total. Summing the
> per-product numbers double-counts multi-product users.
> {style="warning"}

```sql
-- Reference query: MAU per product
SELECT
    DATE_TRUNC('month', session_start_ts) AS activity_month,
    product_code,
    COUNT(DISTINCT user_id)               AS mau
FROM analytics.marts.fct_usage_sessions
WHERE is_qualifying = TRUE
GROUP BY 1, 2
ORDER BY 1 DESC, 2;
```

## Dimensions

| Dimension | Description | Example values |
|-----------|-------------|----------------|
| `product_code` | Product the activity belongs to | `DZ_TRACK`, `DZ_DOCS` |
| `platform` | Operating system of the client | `windows`, `macos`, `linux` |
| `plan_type` | License state at the time of activity | `trial`, `paid`, `community` |
| `geo_country` | Country derived from IP geolocation | `DE`, `RS`, `US` |

## Data sources and lineage

Product telemetry events land in Snowflake hourly, are sessionized by
dbt into `fct_usage_sessions`, and aggregated into MAU by the daily
dbt run at 04:00 UTC. The metric is available in Metabase
(`metabase.dz.internal/dashboard/17` — Product usage overview).

## Freshness and availability

- **Update schedule:** daily at 04:00 UTC.
- **Month closing:** a month is considered final on the 3rd day of the
  following month — clients that were offline upload buffered events
  with a delay of up to 72 hours.

## Caveats and known limitations

- **Telemetry opt-out.** Roughly 12% of users disable telemetry, so
  MAU systematically undercounts real usage. Treat MAU as a
  directional metric, not a census.
- **Calendar-month bias.** Months differ in length (28–31 days).
  For trend analysis use the rolling 28-day active users metric
  (`au_28d`), which removes this bias.
- **Geography.** `geo_country` is IP-based and does not match
  `billing_country` from license data. Never join MAU with revenue
  by country — the two attributes disagree for a noticeable share
  of accounts.

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2025-09-30 | Excluded license-check events from qualifying actions | Product Analytics |
| 2025-05-20 | Initial version | Product Analytics |