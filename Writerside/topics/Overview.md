# DZ Data Hub

<tldr>

**What:** internal documentation portal for the DZ Software data platform

**For whom:** anyone who works with our data — operations, engineering, product, client services

**Maintained by:** Data Office

</tldr>

DZ Software processes financial transactions for banks and investment
firms: records arrive from client source systems, pass through
validation and enrichment, and land in the warehouse for reporting
and analytics. DZ Data Hub is the single place to find out how that
pipeline works, what our processing metrics mean, and how to use the
data behind them.

If you have ever asked "is this number final?", "why is this
transaction still pending?", or "which table should I query?" —
this portal is the answer.

## What you will find here

- **[Metrics](STP-Rate.md)** — processing health metrics with exact
  definitions, calculation logic, and known caveats:
  [STP Rate](STP-Rate.md) (how much goes through automatically) and
  [Processing Latency](Processing-Latency.md) (how fast). Every
  metric page follows the same
  [template](Metric-documentation-template.md), so you always know
  where to look.
- **[Datasets](fct-transactions.md)** — catalog entries for warehouse
  tables: schema, lineage, freshness, access, and sample queries.
- **[Guides](Request-access-to-a-dataset.md)** — step-by-step
  instructions for common tasks in the data ecosystem.

## How to use the portal

Start with search — every page is structured so that definitions and
key facts surface first. If a metric page and a dashboard disagree,
the metric page wins: dashboards visualize, this portal defines.

Found a gap or an error? Every page has an owner listed at the top —
or open an issue in the `DATA` project in YouTrack.