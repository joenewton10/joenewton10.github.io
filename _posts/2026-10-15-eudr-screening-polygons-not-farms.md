---
title: "Screening polygons, not finding farms: an EUDR deforestation check for Ghanaian cocoa"
excerpt: "Notes from building a plot-level EUDR screening tool: the 2020 cutoff, the limits of 30 m data, and why shaded cocoa looks like forest from space."
categories:
  - Remote Sensing
tags:
  - EUDR
  - Google Earth Engine
  - Cocoa
  - Deforestation
  - Python
---

A lot of Ghana's cocoa ends up in Europe. Under the EU Deforestation Regulation (EUDR), a company selling that cocoa on the EU market has to show it wasn't grown on land deforested after 31 December 2020. In practice that means someone has to look at every farm plot in the supply chain and check what happened to the forest there after the cutoff.

I wanted to build the first step of that check.

![Sample DDS-style screening report](https://raw.githubusercontent.com/joenewton10/eudr-plot-screening/master/docs/report_sample.png)
*Sample report from the demo dataset. Three of the six plots are flagged RED for post-2020 forest loss.*

## How it works

There are two stages.

In Google Earth Engine, the script takes each plot polygon the operator supplies and counts the Hansen Global Forest Change pixels inside it with a loss year of 2021 or later. A plot is RED if that loss is more than 1% of its area, AMBER if there's some loss at or below 1%, and GREEN if there's none. The results go out as a CSV with one row per plot, plus a GeoJSON with the geometry for mapping.

Then a Python module reads both files, draws an overview map and writes a PDF laid out like an EUDR due diligence statement: operator reference, commodity, location, area, post-2020 loss and the risk flag. Every RED plot gets a "mitigation or exclusion required" notice.

## Not trusting my own input

One decision I'd make again: the Python side doesn't copy the risk flag from the Earth Engine export. It recalculates every flag from the thresholds in `config.yaml`, and if its answer disagrees with the export, it prints a data-integrity warning.

I added this after imagining a fairly boring failure. Someone changes the threshold in one place and forgets the other, or edits a CSV by hand, and the report quietly passes on the wrong flag. For a compliance document I'd rather it complained.

Thresholds, cutoff date, commodity and file paths all sit in the config file. To screen rubber or coffee plots instead, you swap the input. The code doesn't change.

## Why cocoa looks like forest

This was the part I found most interesting, and it isn't really about code. Much of Ghana's cocoa grows under shade trees. Seen from a satellite, an older shaded cocoa farm and a patch of secondary forest can look very similar: both are a closed green canopy.

So you can't reliably find cocoa farms from imagery alone, and the regulation doesn't ask you to. Operators have to supply the GPS polygons of their plots. My tool follows the same logic. It screens whatever polygon it's given and doesn't try to discover farms on its own.

It also means I'd read a GREEN result with some care. Forest that was thinned and planted with shaded cocoa may only lose part of its canopy, and at 30 m that change can be hard to see.

## What it doesn't do

It answers one question: was there forest loss inside this polygon after 2020? It doesn't check legality or traceability, and the PDF is not a legal due diligence statement.

Hansen GFC is 30 m data. It can miss clearings smaller than a hectare and get the numbers wrong along plot edges, which is a real problem for smallholder plots that are often only a few hectares.

The six demo plots in the Western North cocoa belt are there so the pipeline runs out of the box. They aren't a real supplier's farms.

## What I'd add next

A forest map for 2020. Loss data tells you what was cleared, but the EUDR question is really whether the land was forest at the cutoff in the first place. Adding a 2020 forest baseline would cut false flags on land that was never forest, like long-established cocoa farms.

Building this made it clear to me that most of the work in a tool like this is in the rules, the caveats and the checks, more than in the satellite analysis itself.

[View the code on GitHub](https://github.com/joenewton10/eudr-plot-screening){: .btn .btn--primary}
