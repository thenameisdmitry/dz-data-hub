# Metric documentation template

<tldr>

**Owner:** Team or role responsible for the metric

**Status:** Draft / Approved / Deprecated

**Domain:** Product / Marketing / Sales / Finance

**Primary source:** Table or model the metric is calculated from

</tldr>

Use this template to document a business or product metric.
Copy the structure below into a new topic and replace the guidance
in each section with real content.

> Keep the one-line definition self-contained. It is reused in
> search results, AI-generated answers, and dashboard tooltips,
> so it must make sense without the rest of the page.
> {style="note"}

## Definition

One or two sentences in plain language: what the metric measures
and what business question it answers. Avoid formulas here — this
section is for readers who may never look at SQL.

## Calculation

The exact, unambiguous calculation logic:

- **Formula** — expressed in business terms first, then in SQL.
- **Grain** — the level at which the metric is calculated
  (per user, per account, per day).
- **Filters** — what is included and excluded (test accounts,
  internal users, refunded orders).

```sql
-- Reference query. Keep it runnable against the warehouse.
SELECT ...
```

## Dimensions

List the dimensions by which the metric can be segmented.

| Dimension | Description | Example values |
|-----------|-------------|----------------|
| `country` | Billing country of the account | `DE`, `RS`, `US` |

## Data sources and lineage

Where the numbers come from, step by step: source systems →
warehouse tables → transformation models → dashboards. Link each
dataset to its catalog entry.

## Freshness and availability

- **Update schedule** — how often and when the metric refreshes.
- **Latency** — how far behind real time the data is.
- **Where to find it** — dashboards and tools that expose the metric.

## Caveats and known limitations

An honest list of everything that can mislead a reader: definition
changes over time, known data quality issues, edge cases. This
section builds trust — never skip it.

## Changelog

| Date | Change | Author |
|------|--------|--------|
| YYYY-MM-DD | Initial version | Name |