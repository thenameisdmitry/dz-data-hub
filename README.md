# DZ Data Hub

**Live site:** https://thenameisdmitry.github.io/dz-data-hub/

A demo project created to showcase data documentation practices:
an internal data documentation portal for a fictional fintech
company that processes financial transactions for banks and
investment firms. Metric definitions, a dataset catalog entry, and
procedural guides — written in
[JetBrains Writerside](https://www.jetbrains.com/writerside/),
built and tested by GitHub Actions, deployed to GitHub Pages.

## What's inside

- Processing health metrics built on a single template:
  Straight-Through Processing Rate and End-to-End Processing Latency
  (p50/p95)
- A data catalog entry (`fct_transactions`): schema, processing
  status model, lineage, freshness, known issues
- A procedural guide (dataset access requests) using Writerside's
  semantic `<procedure>` markup
- CI pipeline: build → documentation checks → deploy

## Author

Dmitrii Z. — senior technical writer.
Portfolio: https://thenameisdmitry.github.io/KBDZ/