---
title: "Projects"
permalink: /projects/
toc: true
toc_label: "Projects"
toc_sticky: true
---

These are the projects I've built end to end, from raw data to tested code and outputs someone could check. Each one has its code on GitHub, and most have a longer write-up on the blog.

## Forest carbon MRV prototype: Kakum National Park, Ghana

![Classified forest map of Kakum National Park](https://raw.githubusercontent.com/joenewton10/kakum-carbon-mrv/master/docs/images/classified_forest_map.png)
*Forest and non-forest inside the park boundary, classified from Sentinel-2.*

A prototype of the measurement side of a Verra REDD+ project (VM0048). A Random Forest classifier in Google Earth Engine maps forest from 2023 Sentinel-2 imagery with 95.5% accuracy on held-out points. NDVI alone couldn't separate forest from farmland here, so the classifier relies on shortwave-infrared bands. A tested Python package then converts the map to forest area (18,817 ha of the park's 21,168 ha), applies IPCC Tier 1 biomass defaults to estimate carbon stock, and propagates the uncertainty.

| Metric | Value |
|---|---|
| Carbon stock, including roots | ≈ 12.47 million tCO₂e |
| Combined uncertainty | ≈ 61.5% |
| Conservative (lower-bound) estimate | ≈ 4.80 million tCO₂e |

The uncertainty analysis showed that the map contributes only about 4.5% of the error. Almost all the rest comes from the Tier 1 biomass default, which ranges from 130 to 510 t/ha. So the next step is site-specific biomass from NASA's GEDI lidar, not more classifier tuning.

This is a carbon stock at a single date, not an emissions estimate, and not a registered project.

**Tools:** Google Earth Engine, Sentinel-2, Random Forest, Python, rasterio, pytest

[Code on GitHub](https://github.com/joenewton10/kakum-carbon-mrv) · [Full report (PDF)](https://github.com/joenewton10/kakum-carbon-mrv/blob/master/Kakum_MRV_Report.pdf) · [Read the write-up]({% post_url 2026-10-17-kakum-forest-carbon-uncertainty %})

---

## GHG inventory tool: Scope 1 and 2 for a precision cutting site

![First page of the GHG inventory report](https://raw.githubusercontent.com/joenewton10/ghg-inventory/main/docs/report_page1.png)
*First page of the generated PDF report.*

A GHG Protocol Scope 1 and 2 inventory for a contract cutting business, modelled on HTWest GmbH in Bonn. It reads bills and service records from an Excel workbook, uses versioned emission factors with cited sources (UBA, AIB, DESNZ, IPCC AR6), and produces a PDF report, an auditor workpaper, Power BI tables and a reproducible run manifest.

| | t CO₂e |
|---|---|
| Scope 1 | 40.5 |
| Scope 2, location-based | 68.8 |
| Scope 2, market-based | 140.3 |
| Total, location / market-based | 109.3 / 180.8 |

Market-based Scope 2 is about twice the location-based figure, because standard-tariff electricity in Germany is counted at the residual mix (701 g/kWh) rather than the grid average (344 g/kWh). The tool calculates each gas separately, includes refrigerant leaks, runs a 10,000-draw Monte Carlo for uncertainty, and maps each figure to VSME, ESRS E1 and ISO 14064-1. It also ranks seven reduction options by cost per tonne. A green tariff is the cheapest way to lower the market-based number, while a heat pump on the current tariff would actually raise it.

The activity data is representative rather than real bills, and Scope 3 is outside the boundary, though a rough screen of purchased materials is included to show its size. There are 62 tests.

**Tools:** Python, pandas, Streamlit, Excel, Power BI tables, pytest

[Live demo](https://ghg-inventory-io.streamlit.app/) · [Code on GitHub](https://github.com/joenewton10/ghg-inventory) · [Read the write-up]({% post_url 2026-10-16-ghg-inventory-scope-1-2 %})

---

## EUDR plot-level deforestation screening (cocoa, Ghana)

![Sample EUDR screening report](https://raw.githubusercontent.com/joenewton10/eudr-plot-screening/master/docs/report_sample.png)
*Sample report from the demo dataset: 3 of 6 plots flagged RED.*

Under the EU Deforestation Regulation, cocoa sold in the EU must not come from land deforested after 31 December 2020. This tool checks farm plots against that cutoff.

Google Earth Engine counts Hansen Global Forest Change loss pixels from 2021 onwards inside each operator-supplied plot polygon and flags the plot RED (loss above 1% of the area), AMBER (some loss, 1% or less) or GREEN (none). A tested Python module then recalculates every flag from the config file, warns if its answer disagrees with the Earth Engine export, and writes a PDF report in the style of a due diligence statement.

The commodity, thresholds and cutoff are all set in config, so the same code can screen rubber, coffee or palm plots. I also documented why shaded cocoa looks like forest in satellite imagery, which is the reason the EUDR requires operators to provide plot polygons instead of relying on remote detection.

It's a screening tool for deforestation risk only. It doesn't check legality or traceability, and at 30 m resolution it can miss very small clearings.

**Tools:** Google Earth Engine, Hansen GFC, Python, GeoJSON, pytest

[Code on GitHub](https://github.com/joenewton10/eudr-plot-screening) · [Read the write-up]({% post_url 2026-10-15-eudr-screening-polygons-not-farms %})

---

## Green roof climate data pipeline and dashboard

![Green roof thermal signal distribution](https://raw.githubusercontent.com/joenewton10/bingen-greenroof-ETL-pipeline/master/docs/screenshots/01_overall_signal.png)
*Temperature difference between the green roof and the parking lot.*

The data pipeline behind my MSc thesis at TH Bingen, which compares air temperature above a green roof and a paved parking lot. It takes raw exports from three logger formats (Empower, Kissel/MXmini, Campbell Scientific TOA5), filters impossible readings, maps everything onto one schema, and aligns both sites minute by minute. The research dataset came to about 2.18 million synchronised minutes from 2020 to 2026.

Ingestion is incremental, so only changed files are re-read. Updates build into a staging database and replace the live one only after passing checks, so a failed run never breaks the dashboard. Physical metrics such as radiation balance, albedo and specific humidity are calculated in SQL, so every part of the project uses the same numbers.

The 11-tab Streamlit dashboard tests the cooling effect year by year with 95% confidence intervals, splits it by season and by day and night, and includes an energy-balance view. The public repo runs on synthetic data, because the original sensor data belongs to the research lab.

**Tools:** Python, DuckDB, SQL, pandas, Streamlit, Plotly

[Live dashboard](https://bingen-greenroof-etl-pipeline-4ads4zvvdgfzzc36labs6p.streamlit.app/) · [Code on GitHub](https://github.com/joenewton10/bingen-greenroof-ETL-pipeline) · [Read the write-up]({% post_url 2026-10-14-green-roof-etl-pipeline %})
