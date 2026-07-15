# Straight-Through Processing Rate (STP Rate)

<tldr>

**Owner:** Data Operations (`data-ops@dz.software`)

**Status:** Approved

**Domain:** Data Operations

**Primary source:** [fct_transactions](fct-transactions.md)

</tldr>

STP Rate measures the share of incoming transactions that flow through
the DZ data pipeline fully automatically — ingested, validated, and
loaded into the warehouse without manual intervention. It is the
primary health indicator of our data processing: a falling STP Rate
means more records are failing validation or piling up in manual
review queues.

## Definition

The percentage of transactions received in a period that reached the
warehouse in `auto` status — meaning every pipeline stage (format
validation, enrichment, deduplication, account matching) completed
without human involvement.

A transaction ends its pipeline journey in one of three final states:

- **`auto`** — processed straight through. Counts toward STP.
- **`manual`** — required an operator (unmatched account, failed
  enrichment) and was resolved by hand. Does not count.
- **`rejected`** — failed validation terminally (malformed record,
  duplicate feed file). Does not count.

> Transactions still sitting in the review queue have a `pending`
> status and are **excluded from the calculation entirely** until
> resolved. This means the STP Rate for recent days can shift as
> pending items get resolved into `auto` or `manual`.
> {style="warning"}

## Calculation

In business terms: of all transactions that finished processing in the
period, what share finished automatically.

- **Formula:** `auto_transactions / resolved_transactions × 100`
- **Grain:** one transaction (`transaction_id`), aggregated monthly
  by default.
- **Included:** all transactions with a final status
  (`auto`, `manual`, `rejected`).
- **Excluded:** `pending` transactions; records from test feeds
  (`source_system = 'test'`).

```sql
-- Reference query: monthly STP Rate
SELECT
    -- Group transactions by the month they arrived
    DATE_TRUNC('month', ingested_at)  AS month,

    -- Everything that finished processing (any final status)
    COUNT(*)                          AS resolved_transactions,

    -- Count only straight-through rows: CASE returns 1 for 'auto',
    -- 0 otherwise, so the SUM is the number of auto transactions
    SUM(CASE WHEN processing_status = 'auto'
             THEN 1 ELSE 0 END)       AS auto_transactions,

    -- Share of automatic processing, one decimal place.
    -- 100.0 (not 100) forces decimal division
    ROUND(100.0 * SUM(CASE WHEN processing_status = 'auto'
                           THEN 1 ELSE 0 END)
               / COUNT(*), 1)         AS stp_rate

FROM marts.fct_transactions

-- Pending transactions have no outcome yet — excluded until resolved
WHERE processing_status <> 'pending'

GROUP BY 1       -- group by the first column (month)
ORDER BY 1 DESC; -- newest month first
```

## Dimensions

| Dimension | Description | Example values |
|-----------|-------------|----------------|
| `source_system` | System the transaction arrived from | `core_banking`, `payment_gateway`, `custodian` |
| `transaction_type` | Business type of the transaction | `payment`, `trade`, `fee`, `transfer` |
| `currency` | Transaction currency | `EUR`, `USD`, `RSD` |
| `client_tier` | Client size segment from `dim_accounts` | `enterprise`, `mid_market` |

Segmenting by `source_system` is the most common use: custodian feeds
historically show the lowest STP due to inconsistent file formats.

## Data sources and lineage

1. Transactions arrive from three source systems (core banking,
   payment gateway, custodian feeds) into the ingestion layer.
2. The validation engine assigns each record its
   `processing_status` and writes the outcome to the staging area.
3. The daily warehouse job loads resolved records into
   [fct_transactions](fct-transactions.md), the single source of
   truth for all processing metrics.
4. The metric is visualized in Metabase
   (`metabase.dz.internal/dashboard/12` — Data Operations overview).

## Freshness and availability

- **Update schedule:** the warehouse load runs daily at 03:00 UTC
  (Airflow DAG `warehouse_daily`).
- **Latency:** the dashboard reflects transactions resolved up to the
  previous midnight UTC.
- **Where to find it:** Metabase Data Operations overview; monthly
  values are also included in the operations review deck.

## Caveats and known limitations

- **Pending backlog distorts recent values.** During review-queue
  backlogs, recent days show artificially high STP: the problematic
  records are all sitting in `pending` and are not counted yet.
  Always check the queue size before reading a recent spike as
  improvement.
- **Rejected duplicates deflate custodian STP.** Custodians
  occasionally re-send entire files; every record in the duplicate
  file is `rejected`. A single re-sent file can drop the daily
  custodian STP by several points without any real processing issue.
- **Definition note.** STP measures *processing* automation, not data
  quality: a transaction can pass straight through with a payee name
  that later needs cleanup. Data quality is tracked separately.

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-02-10 | Excluded test feeds from the calculation | Data Operations |
| 2025-10-01 | Initial version | Data Operations |