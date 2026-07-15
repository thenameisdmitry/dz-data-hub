# End-to-End Processing Latency

<tldr>

**Owner:** Data Operations (`data-ops@dz.software`)

**Status:** Approved

**Domain:** Data Operations

**Primary source:** [fct_transactions](fct-transactions.md)

</tldr>

Processing Latency measures how long a transaction takes to travel
through the DZ pipeline — from the moment it arrives from a source
system to the moment it is available in the warehouse. Together with
[STP Rate](STP-Rate.md), it answers the two questions clients ask
about our processing: *how much* goes through automatically, and
*how fast*.

## Definition

The time elapsed between `ingested_at` (the record arrives from the
source system) and `loaded_at` (the record becomes queryable in
`fct_transactions`), reported in minutes.

The metric is reported as two numbers, never one:

- **Median (p50)** — the typical experience: half of all transactions
  are processed faster than this.
- **95th percentile (p95)** — the slow tail: only 1 in 20 transactions
  takes longer. This is the number our processing SLA is written
  against.

> Never report latency as an average. A handful of stuck transactions
> can drag the average far above what any typical record experiences,
> while the median stays honest. Percentiles are the industry
> standard for latency for exactly this reason.
> {style="warning"}

## Calculation

- **Formula:** `loaded_at − ingested_at`, in minutes, aggregated as
  p50 and p95 per period.
- **Grain:** one transaction, aggregated monthly by default; the
  operations dashboard also shows a daily view.
- **Included:** transactions with final status `auto` or `manual`.
- **Excluded:** `rejected` transactions (they never reach the
  warehouse, so they have no end-to-end latency) and test feeds.

```sql
-- Reference query: monthly processing latency, p50 and p95
SELECT
    DATE_TRUNC('month', ingested_at)     AS month,
    ROUND(MEDIAN(
        DATEDIFF('minute', ingested_at, loaded_at)
    ), 1)                                AS latency_p50_min,
    ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (
        ORDER BY DATEDIFF('minute', ingested_at, loaded_at)
    ), 1)                                AS latency_p95_min
FROM marts.fct_transactions
WHERE processing_status IN ('auto', 'manual')
GROUP BY 1
ORDER BY 1 DESC;
```

## Dimensions

| Dimension | Description | Example values |
|-----------|-------------|----------------|
| `source_system` | System the transaction arrived from | `core_banking`, `payment_gateway`, `custodian` |
| `processing_status` | How the transaction was resolved | `auto`, `manual` |
| `transaction_type` | Business type of the transaction | `payment`, `trade`, `fee` |

The `processing_status` split is the most revealing one: `manual`
transactions are typically 20–40× slower than `auto`, since they wait
for an operator. When overall latency degrades, check the STP Rate
first — a latency problem is often an automation problem in disguise.

## Data sources and lineage

Both timestamps live on the transaction record itself: `ingested_at`
is set by the ingestion layer on arrival, `loaded_at` by the warehouse
load job. The metric is calculated directly from
[fct_transactions](fct-transactions.md) and visualized in Metabase
(`metabase.dz.internal/dashboard/12` — Data Operations overview).

## Freshness and availability

- **Update schedule:** daily warehouse load at 03:00 UTC
  (Airflow DAG `warehouse_daily`).
- **SLA context:** the client-facing commitment is p95 ≤ 4 hours for
  payment-gateway transactions. The SLA dashboard tracks this
  separately; this metric is the internal, all-sources view.

## Caveats and known limitations

- **Batch sources have a built-in floor.** Custodian feeds arrive as
  end-of-day files, so their latency can never drop below the batch
  cycle — comparing custodian latency to real-time payment-gateway
  latency is meaningless. Always segment by `source_system`.
- **Manual latency measures queues, not effort.** For `manual`
  transactions, most of the elapsed time is waiting for an operator,
  not processing. During backlogs, p95 reflects queue depth.
- **Clock consistency.** Both timestamps are set by our own systems
  in UTC. Latency is not affected by source-system clock skew — but
  it also does not include delays *before* arrival (a custodian
  sending its file late looks fast in this metric).

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-01-15 | Added p50 alongside p95 on the operations dashboard | Data Operations |
| 2025-10-01 | Initial version | Data Operations |