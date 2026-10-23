---
title: "Estimating carbon in Kakum National Park"
excerpt: "I expected the forest map to be the weak point in my carbon estimate. It turned out to be one number from an IPCC table."
categories:
  - Carbon MRV
tags:
  - REDD+
  - Verra VM0048
  - Google Earth Engine
  - Sentinel-2
  - Uncertainty
---

Kakum National Park, in Ghana's Central Region, is one of the larger blocks of tropical forest left in the south of the country. I grew up not far from it in Cape Coast. For this project I wanted to estimate how much carbon the forest holds, and how confident that estimate can be.

![Classified forest map of Kakum National Park](https://raw.githubusercontent.com/joenewton10/kakum-carbon-mrv/master/docs/images/classified_forest_map.png)
*Forest (dark green) and non-forest (tan) inside the official WDPA boundary (red). The pins are training points.*

## The pipeline

The prototype follows the measurement steps of a Verra REDD+ project under the VM0048 methodology.

A Random Forest classifier in Google Earth Engine, trained on a 2023 Sentinel-2 composite, splits the park into forest and non-forest. It scored 95.5% on held-out points. A Python package then reads the exported 10 m GeoTIFF and counts pixels: 18,817 ha of forest out of the park's 21,168 ha. IPCC Tier 1 biomass defaults turn that area into biomass, carbon and CO₂e. Finally, classification error and the biomass range are combined into an overall uncertainty and a conservative lower estimate.

All the science parameters live in a config file, and the calculation functions have unit tests, so each number can be traced back to where it came from.

## NDVI didn't work

My first idea was to map forest with NDVI, the usual greenness index. In Kakum it's close to useless. Forest and the farmland around it differ by only about 0.04, because everything there is green. Only towns and roads stand out.

![NDVI over the study area](https://raw.githubusercontent.com/joenewton10/kakum-carbon-mrv/master/docs/images/ndvi_comparison.png)
*NDVI over the study area. Forest and farmland are almost the same shade.*

The classifier leans on Sentinel-2's shortwave-infrared bands instead, which pick up differences in canopy structure and moisture that NDVI misses.

## Results

| Metric | Value |
|---|---|
| Carbon stock, above-ground | ≈ 10.0 million tCO₂e |
| Carbon stock, including roots | ≈ 12.47 million tCO₂e |
| Combined uncertainty | ≈ 61.5% |
| Conservative (lower-bound) estimate | ≈ 4.80 million tCO₂e |

The headline figure is 12.47 million. The figure I'd defend in front of a verifier is 4.80 million. The gap between them is why I spent most of my time on the uncertainty.

## Where the error comes from

I assumed the map would be the main source of error. It contributes about 4.5%.

Most of the remaining ~61% comes from the IPCC Tier 1 biomass default for tropical forest. It's an average across a whole continent, with a range of 130 to 510 tonnes per hectare. Apply that to one particular forest and the range overwhelms everything else in the calculation.

![Biomass sensitivity chart](https://raw.githubusercontent.com/joenewton10/kakum-carbon-mrv/master/outputs/sensitivity_plot.png)
*Total carbon stock across the biomass default range. The mean of 310 t/ha is marked.*

That changed what I'm working on next. More classifier tuning would hardly move the final uncertainty. Replacing the default with biomass measured for this site would. The plan is a Tier 2 upgrade using biomass estimates from NASA's GEDI lidar.

This matters for carbon credits, because the conservative estimate is closer to what actually gets credited. A narrower range means a larger, more believable number.

## Limitations

This is a carbon stock at one date. To estimate emissions or avoided deforestation you need at least a second date to measure change.

It's a portfolio prototype, not a registered project. It covers the measurement side of VM0048 (map, area, carbon stock). A real project also needs a jurisdictional baseline that comes from outside the project developer.

And because it uses Tier 1 defaults, the 12.47 million figure should always be quoted with its uncertainty.

The main thing I'd tell someone starting an MRV project is to run the uncertainty analysis early. I would have spent weeks improving the wrong part of the pipeline otherwise.

[View the code on GitHub](https://github.com/joenewton10/kakum-carbon-mrv){: .btn .btn--primary}
[Read the full report (PDF)](https://github.com/joenewton10/kakum-carbon-mrv/blob/master/Kakum_MRV_Report.pdf){: .btn .btn--inverse}

