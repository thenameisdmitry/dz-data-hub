# fct_transactions

<tldr>

**Owner:** Data Platform (`data-platform@dz.software`)

**Status:** Production

**Location:** `marts.fct_transactions` (Snowflake)

**Update schedule:** daily at 03:00 UTC (Airflow DAG `warehouse_daily`)

</tldr>

`fct_transactions` is the single source of truth for processed
financial transactions at DZ Software. One row represents one
transaction that finished its journey through the ingestion pipeline —
successfully or not. All processing metrics, including
[STP Rate](STP-Rate.md) and
[Processing Latency](Processing-Latency.md), are built on this table.

## Schema

| Column | Type | Description |
|--------|------|-------------|
| `transaction_id` | `VARCHAR` | Unique transaction identifier assigned at ingestion (primary key) |
| `source_system` | `VARCHAR` | Where the record came from: `core_banking`, `payment_gateway`, `custodian`, `test` |
| `source_ref` | `VARCHAR` | Original identifier in the source system; unique only within one source |
| `account_id` | `VARCHAR` | Account the transaction belongs to; joins to `dim_accounts` |
| `payee_id` | `VARCHAR` | Counterparty; joins to `dim_payees`. `NULL` where no counterparty applies (e.g. fees) |
| `transaction_type` | `VARCHAR` | Business type: `payment`, `trade`, `fee`, `transfer` |
| `amount` | `NUMBER(18,2)` | Transaction amount in the original currency |
| `currency` | `VARCHAR(3)` | ISO 4217 currency code: `EUR`, `USD`, `RSD` |
| `booking_date` | `DATE` | Business date the transaction belongs to |
| `processing_status` | `VARCHAR` | Pipeline outcome: `auto`, `manual`, `rejected`, `pending` — see [Processing statuses](#statuses) |
| `ingested_at` | `TIMESTAMP_NTZ` | When the record arrived from the source system, UTC |
| `loaded_at` | `TIMESTAMP_NTZ` | When the record became queryable in the warehouse, UTC |

## Processing statuses {id="statuses"}

| Status | Meaning | In STP Rate | In Latency |
|--------|---------|-------------|------------|
| `auto` | Processed straight through, no human involvement | Yes — numerator | Yes |
| `manual` | Resolved by an operator (unmatched account, failed enrichment) | Yes — denominator only | Yes |
| `rejected` | Failed validation terminally; kept for audit | Yes — denominator only | No |
| `pending` | Still in the review queue | No | No |

> `pending` is a transient status: every pending transaction
> eventually becomes `auto`, `manual`, or `rejected`. Recent
> aggregates shift as the queue drains — both metric pages carry
> a caveat about this.
> {style="note"}

## Lineage

- **Upstream.** Transactions arrive from three production source
  systems (core banking, payment gateway, custodian feeds). The
  validation engine checks format, enriches records, deduplicates
  re-sent files, matches accounts, and assigns `processing_status`.
  The daily warehouse job then loads resolved and pending records
  into this table.
- **Downstream.** Metrics [STP Rate](STP-Rate.md) and
  [Processing Latency](Processing-Latency.md); Metabase dashboards
  Data Operations overview and Client SLA report; monthly regulatory
  extracts.
- **Related dimensions.** `dim_accounts` (account attributes,
  client tier), `dim_payees` (counterparty reference data). Account
  and payee attributes are **not** stored on this table — join the
  dimensions instead.

## Freshness

The daily load completes by ~03:40 UTC and includes transactions
resolved up to the previous midnight UTC. Late status changes are
possible: a `pending` transaction resolved today updates its row in
tomorrow's load, so recent daily aggregates are revised as the
review queue drains.

## Access

Read access is granted through the `DATA_READER_OPS` Snowflake role.
To request it, follow
[Request access to a dataset](Request-access-to-a-dataset.md).

## Sample queries

```sql
-- Daily processing volume by source, last 30 days
SELECT
  booking_date,
  source_system,
  COUNT(*) AS transactions
FROM marts.fct_transactions
WHERE booking_date >= CURRENT_DATE - 30   -- rolling 30-day window
GROUP BY 1, 2                             -- per day, per source
ORDER BY 1 DESC, 2;
```

```sql
-- Current review queue by source system
SELECT
  source_system,
  COUNT(*)         AS pending_transactions,
  -- Arrival time of the oldest unresolved transaction:
  -- shows how long the queue has been building up
  MIN(ingested_at) AS oldest_pending_since
FROM marts.fct_transactions
WHERE processing_status = 'pending'
GROUP BY 1
ORDER BY 2 DESC;  -- biggest queue first
```

## Known issues

- **Duplicate custodian files.** Custodians occasionally re-send an
  entire end-of-day file. The deduplication step catches these by
  `source_system` + `source_ref` + `booking_date`; the duplicates
  land as `rejected`. See the STP Rate
  [caveats](STP-Rate.md) for the metric impact.
- **`payee_id` completeness.** Around 3% of historical transactions
  (before the 2025-06 payee-matching upgrade) have `NULL` payee_id
  where a counterparty existed. Treat payee-level analysis before
  July 2025 as approximate.
- **Test feed hygiene.** Records with `source_system = 'test'` are
  synthetic traffic from release checks. All metric queries must
  exclude them; the metrics layer does this automatically.

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2025-06-30 | Payee-matching upgrade: `payee_id` now populated for custodian trades | Data Platform |
| 2025-03-12 | Added `source_ref` for source-system reconciliation | Data Platform |
| 2024-09-01 | Initial version | Data Platform |