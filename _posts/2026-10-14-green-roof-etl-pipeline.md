---
title: "Cleaning up years of sensor data for a green roof thesis"
excerpt: "How I built the pipeline behind my MSc thesis at TH Bingen, which compares air temperature above a green roof and a parking lot, minute by minute."
categories:
  - Climate Data
tags:
  - ETL
  - DuckDB
  - Streamlit
  - Urban Climate
  - Green Roofs
---

My MSc thesis at TH Bingen asks whether a green roof cools the air above it compared with a paved surface, and at what times of day and year. The two sites are a vegetated green roof and a parking lot nearby, both logging readings every minute for several years.

Before I could do any statistics, I had to get the data into one place.

![Overall thermal signal distribution](https://raw.githubusercontent.com/joenewton10/bingen-greenroof-ETL-pipeline/master/docs/screenshots/01_overall_signal.png)
*Distribution of the temperature difference between the two sites.*

## The data

Each site had loggers from two different vendors, and there was an optional network of black-globe thermometers on top. The raw exports came in three formats (Empower, Kissel/MXmini and Campbell Scientific TOA5). The same measurement had different column names depending on the vendor, some files used German decimal commas, and new files kept arriving on top of years of history.

## The pipeline

It runs in five stages.

First it parses each vendor's files into its own table. It identifies the format from the file contents, because the filenames weren't reliable. Next it filters out physically impossible readings using bounds for each sensor type (temperature, humidity, radiation, pressure). Then it maps every vendor's columns onto one shared schema so readings from different loggers can be compared directly.

The fourth stage lines both sites up on the same minute-by-minute timeline and calculates the values the analysis needs: the temperature difference, radiation balance, albedo, and specific humidity using the Magnus formula, as a rough proxy for evapotranspiration. Finally, the table is split by year into a DuckDB database so queries stay fast.

The full research dataset came to about 2.18 million synchronised minutes, from 2020 to 2026.

## Decisions I'd make again

The join between the two sites is an inner join, so a minute only counts if both sites have data. A comparison with one side missing doesn't tell you anything. The black-globe network is joined with a left join, because it's extra context, and losing it for a few hours shouldn't throw away good readings from the main sensors.

Ingestion is incremental. The pipeline keeps a manifest of each file's size and modification time and only re-reads files that changed, so a routine update doesn't mean reprocessing thousands of files.

Updates build into a separate staging database first. Only after it passes checks does it replace the live one. If a run crashes halfway, the dashboard keeps showing the last good version. I was relying on this data for months of thesis work, so I cared about this more than about speed.

I also calculate the physical metrics in the SQL step rather than in the dashboard, so every part of the project uses the same numbers.

## The dashboard

The Streamlit dashboard has 11 tabs. It shows whether the cooling effect is statistically significant year by year (with 95% confidence intervals), breaks results down by season and by day and night, includes an energy-balance view, and lets you zoom into single days at minute resolution.

![Year-by-year significance](https://raw.githubusercontent.com/joenewton10/bingen-greenroof-ETL-pipeline/master/docs/screenshots/02_year_significance.png)
*Cooling significance by year, with 95% confidence intervals.*

![Day/night and seasonal breakdown](https://raw.githubusercontent.com/joenewton10/bingen-greenroof-ETL-pipeline/master/docs/screenshots/03_day_night_seasonal.png)
*Day and night by season. The cooling effect changes a lot through the day and the year.*

<!-- ADD YOUR HEADLINE THESIS FINDING HERE, e.g. "On summer afternoons the air above the green roof was on average X °C cooler than above the parking lot, while at night the difference was Y." Then delete this comment. -->

## About the data in the repo

The public repository uses synthetic data: daily and seasonal curves with added noise, written in the same raw formats the real loggers produce. The original pipeline ran on a research lab's sensor data, which isn't public. The code, structure and dashboard are the same as the version I used for my thesis.

Most of the time on this project went into parsing files and deciding what counts as a valid minute. None of that shows up in the final charts, but the charts wouldn't mean much without it.

[Open the live dashboard](https://bingen-greenroof-etl-pipeline-4ads4zvvdgfzzc36labs6p.streamlit.app/){: .btn .btn--primary}
[View the code on GitHub](https://github.com/joenewton10/bingen-greenroof-ETL-pipeline){: .btn .btn--inverse}
