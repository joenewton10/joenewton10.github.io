---
title: "From Messy Sensor Exports to Research-Ready Data: A Green Roof Climate Pipeline"
excerpt: "How I turned years of multi-vendor sensor files into a synchronised, minute-level dataset to test whether a green roof actually cools the air above it."
categories:
  - Climate Data
tags:
  - ETL
  - DuckDB
  - Streamlit
  - Urban Climate
  - Green Roofs
---

My MSc thesis at TH Bingen asked a simple question: **does a green roof actually cool the air above it, compared with a paved surface, and when?** Answering it meant comparing two field sites: a vegetated green roof and a paved reference parking lot, each logging readings every minute for years.

The science question was simple. The data was not.

![Overall thermal signal distribution](https://raw.githubusercontent.com/joenewton10/bingen-greenroof-ETL-pipeline/master/docs/screenshots/01_overall_signal.png)
*Distribution of the temperature-difference signal between the two sites.*

## The problem: same physics, different files

Each site reported data from two independent logger vendors, plus an optional network of black-globe temperature loggers. That meant:

- **three different file formats**: Empower, Kissel/MXmini and Campbell Scientific TOA5 exports
- **inconsistent schemas**: the same physical measurement under different column names
- **German decimal commas**, positional columns and vendor-specific headers
- **routine incremental drops**: new files arriving regularly, on top of years of history

Before any statistics could happen, all of this had to become one clean, comparable time series.

## The pipeline

I built a five-stage ETL pipeline:

1. **Ingest.** Each vendor's format is parsed into its own table. Formats are detected by content, not filename, because filenames weren't reliable.
2. **QC filter.** Sensor-specific physical bounds remove impossible readings: temperature, humidity, radiation and pressure outside plausible ranges.
3. **Harmonise.** Each vendor's columns are mapped onto one canonical schema, so readings from different loggers become directly comparable.
4. **Synchronise.** Both sites are joined to a common minute-level timeline, and the physical metrics are derived: the temperature-difference signal, radiation balance, albedo and specific humidity (via the Magnus formula) as a proxy for evapotranspiration.
5. **Partition.** The final table is split by year into a DuckDB warehouse for fast queries.

The real research dataset reached **about 2.18 million synchronised records covering 2020 to 2026**.

## Three engineering decisions I'm proud of

**Joins that encode the science.** The two sites are joined with an `INNER JOIN`: a minute only counts if *both* sites have data, because a comparison with one side missing means nothing. The black-globe network uses a `LEFT JOIN`, because it's optional enrichment and its absence should never silently drop an otherwise good minute. It's a small choice, but it means the dataset's rules match the research design.

**Incremental, idempotent ingestion.** A file manifest tracks every file's size and modification time. Re-running the pipeline against thousands of files only touches the ones that changed, so routine updates take seconds rather than a full rebuild.

**Updates that can't break the dashboard.** Each update builds into a staging database, verifies it, and only then atomically swaps it in as the live one. If a run fails halfway, the dashboard keeps serving the last good version. For a thesis that depended on this data for months, that reliability mattered more than speed.

I also calculate the physical metrics **inside the SQL synchronisation step**, rather than in the dashboard. Every consumer of the warehouse sees exactly the same numbers.

## The dashboard

On top of the warehouse sits an 11-tab Streamlit dashboard: year-by-year cooling significance with 95% confidence intervals, seasonal and day/night breakdowns, an energy-balance diagnostic, and a minute-resolution drill-down for case-study days.

![Year-by-year significance](https://raw.githubusercontent.com/joenewton10/bingen-greenroof-ETL-pipeline/master/docs/screenshots/02_year_significance.png)
*Year-by-year cooling significance with 95% confidence intervals.*

![Day/night and seasonal breakdown](https://raw.githubusercontent.com/joenewton10/bingen-greenroof-ETL-pipeline/master/docs/screenshots/03_day_night_seasonal.png)
*Day/night by season: the cooling effect is not constant through the day or year.*

<!-- ADD YOUR HEADLINE THESIS FINDING HERE, e.g. "The green roof was on average X °C cooler than the parking lot during summer afternoons, while at night the difference was Y." Then delete this comment. -->

## A note on the data

The public repository runs on **synthetic data**: diurnal and seasonal curves plus noise, written in the exact raw formats the pipeline expects. The original pipeline was built on a research lab's real sensor data, which isn't published. The code, the architecture and the dashboard are the real thing; only the inputs are stand-ins.

## Takeaway

Climate research lives or dies on data quality. Most of my time on this project went into the unglamorous parts: parsing formats, defining what counts as a valid minute, and making updates safe. That's also where I learned the most. **Good environmental analysis starts with a pipeline you can trust.**

[Open the live dashboard](https://bingen-greenroof-etl-pipeline-4ads4zvvdgfzzc36labs6p.streamlit.app/){: .btn .btn--primary}
[View the code on GitHub](https://github.com/joenewton10/bingen-greenroof-ETL-pipeline){: .btn .btn--inverse}
