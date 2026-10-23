---
title: "Screening Polygons, Not Finding Farms: An EUDR Deforestation Check for Ghanaian Cocoa"
excerpt: "What building a plot-level EUDR screening tool taught me about the 2020 cutoff, 30 m data, and why cocoa agroforestry looks like forest from space."
categories:
  - Remote Sensing
tags:
  - EUDR
  - Google Earth Engine
  - Cocoa
  - Deforestation
  - Python
---

Ghana is one of the world's largest cocoa producers, and much of that cocoa ends up in Europe. Under the EU Deforestation Regulation (EUDR), every company placing cocoa on the EU market has to show that it was not grown on land deforested after **31 December 2020**. That single date turns a sustainability ambition into a data problem: for every farm plot in a supply chain, someone has to check what happened to the forest there after the cutoff.

I built a tool to do that first check.

![Sample DDS-style screening report](https://raw.githubusercontent.com/joenewton10/eudr-plot-screening/master/docs/report_sample.png)
*A sample report generated from the demo dataset: 3 of 6 plots flagged RED for post-2020 forest loss.*

## How the screening works

The tool runs in two stages.

**1. Google Earth Engine does the geospatial analysis.** For each plot polygon supplied by the operator, the script counts the Hansen Global Forest Change pixels with a loss year of 2021 or later inside the boundary, and classifies the plot:

- **RED:** post-2020 loss above 1% of the plot area
- **AMBER:** some loss detected, at or below 1%
- **GREEN:** no post-2020 loss detected

The results are exported as a CSV (one row per plot) and a GeoJSON (the same data plus geometry, for mapping).

**2. Python turns the results into a report.** A tested, config-driven module reads both exports, renders an overview map, and writes a PDF in the style of an EUDR due diligence statement: operator reference, commodity, geolocation, area, post-2020 loss, risk flag and, for every RED plot, an explicit "mitigation or exclusion required" notice.

## A design choice I care about: never trust the input blindly

The Python stage doesn't simply copy the risk flag from the Earth Engine export. It **re-derives every flag** from the thresholds in `config.yaml` and surfaces any disagreement as a data-integrity warning.

That sounds like a small detail, but in compliance work it's the whole point. If a threshold is changed in one place and not the other, or an export is edited by hand, the report should say so loudly rather than quietly pass on a wrong answer. A due diligence output is only as defensible as its weakest unchecked step.

The thresholds, cutoff date, commodity and file paths all live in configuration, not in the code. The same pipeline can screen rubber, coffee or palm oil plots just by swapping the input.

## Why cocoa farms look like forest from space

The most interesting thing I learned wasn't technical. Much of Ghana's cocoa is grown under shade trees: **agroforestry**. From a satellite, a mature shaded cocoa farm and a patch of secondary forest can look almost identical. Both show a closed, green canopy.

This has a direct consequence for how the regulation works. You can't reliably find cocoa farms from imagery alone. That's why the EUDR requires **operators to supply the GPS polygons of their plots**, rather than leaving regulators to detect farms remotely. My tool reflects that: it screens the polygon it's given; it doesn't try to discover farms on its own.

It also means a GREEN flag has to be read carefully. A cleared forest that was converted to shaded cocoa may lose canopy only partially, and the change can be subtle at 30 m resolution.

## Honest limitations

I want to be clear about what this tool is and isn't.

- **It answers one question only:** was there post-2020 forest loss inside this polygon? It is not a legality or traceability check, and its output is not a legal Due Diligence Statement.
- **Resolution matters.** Hansen GFC is a 30 m global dataset. It can miss sub-hectare clearance and mis-measure loss along plot edges, which is exactly where smallholder plots are most vulnerable to error.
- **The demo plots are illustrative.** The repository ships with six plots in Ghana's Western North cocoa belt so the pipeline runs out of the box; they aren't a real supplier's farms.

## What I'd add next

The natural next step is a **forest baseline as of 2020**. Loss data tells you what was cleared, but EUDR is really asking whether the land *was forest* at the cutoff. Combining loss data with a dedicated 2020 forest-cover map would reduce false flags on land that was never forest, such as old cocoa farms, and tighten the screening considerably.

## Takeaway

Deforestation-free supply chains depend less on fancy models than on clear rules, honest limits and outputs a compliance team can trust. Building this tool made me appreciate how much of that trust comes from small engineering decisions, like refusing to pass on a number you haven't checked.

[View the code on GitHub](https://github.com/joenewton10/eudr-plot-screening){: .btn .btn--primary}
