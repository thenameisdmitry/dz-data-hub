# About this project

<tldr>

**Status:** demo project

**Author:** Dmitrii Zhukov, Senior technical writer

**Built with:** JetBrains Writerside, GitHub Actions, GitHub Pages

</tldr>

DZ Data Hub is a demo project created to showcase data documentation
practices. DZ Software is a fictional company; all metrics, datasets,
figures, and tools mentioned in this portal are invented for
illustration purposes.

## What this project demonstrates

- **Data documentation genres.** Metric documentation with honest
  caveats, a data catalog entry with schema and lineage, and a
  procedural guide with governance context: the core content types
  of an internal data portal.
- **Template-driven consistency.** All metric pages follow a single
  [documented template](Metric-documentation-template.md), so readers
  always know where to find the definition, the caveats, or the owner.
- **Docs-as-code.** The portal lives in a public GitHub repository,
  is written in Writerside's hybrid Markdown + semantic markup, and is
  built, tested, and deployed by a CI pipeline on every push.

## Design decisions and trade-offs

- **A domain I know deeply.** The fictional world (transaction
  processing for banks and investment firms) shows my 10+ years of
  documentation work in fintech B2B SaaS. Believable documentation
  starts with domain understanding: the metrics, the statuses, and
  the caveats here are the kind that real data operations live with.
- **A consistent fictional world.** All pages share one stack
  (Snowflake, Airflow, Metabase, YouTrack) and cross-reference each
  other: each metric points to its source dataset, the dataset points
  back to the metrics it feeds and to the access guide, and the
  status model is defined once and referenced everywhere.
- **Internal links of the fictional company are not clickable.**
  Dashboards and channels (for example, `metabase.dz.internal`) are
  formatted as code, not hyperlinks. A demo should not contain links
  that lead nowhere.
- **Caveats are a first-class section.** Every metric documents what
  can mislead a reader: pending backlogs distorting recent values,
  batch sources with a built-in latency floor, duplicate files
  deflating a source's STP. In real data work, this section builds
  more trust than any other.
- **CI tests the docs.** The pipeline runs the Writerside checker on
  every push and fails the build on broken links or malformed markup,
  a quality feedback loop borrowed from software engineering.

## About the author

I am a senior technical writer with ~10 years in B2B SaaS, most
recently leading documentation at a fintech treasury platform.
This project was built as a practical exploration of the modern data
documentation stack.

Portfolio: [thenameisdmitry.github.io/KBDZ](https://thenameisdmitry.github.io/KBDZ/)