---
title: "A 95.5% Accurate Map and 61.5% Uncertainty: Where Forest Carbon Error Really Comes From"
excerpt: "Estimating the carbon stock of Kakum National Park, Ghana, taught me that a good classifier is not the same as a good carbon estimate."
categories:
  - Carbon MRV
tags:
  - REDD+
  - Verra VM0048
  - Google Earth Engine
  - Sentinel-2
  - Uncertainty
---

Kakum National Park in Ghana's Central Region is one of the last large blocks of tropical forest in the country's south. I wanted to answer a question that sits at the heart of every forest carbon project: **how much carbon does this forest hold, and how sure can we be?**

The second half of that question turned out to be the more interesting one.

![Classified forest map of Kakum National Park](https://raw.githubusercontent.com/joenewton10/kakum-carbon-mrv/master/docs/images/classified_forest_map.png)
*Forest (dark green) and non-forest (tan) inside the official WDPA boundary (red). Pins show training points.*

## The pipeline

I built a prototype that mirrors the measurement logic of a Verra REDD+ project under the VM0048 methodology:

1. **Classification in Google Earth Engine.** A Random Forest classifier trained on a 2023 Sentinel-2 composite separates forest from non-forest across the park, reaching **95.5% accuracy** on held-out data.
2. **Area verification.** A Python package reads the exported 10 m GeoTIFF and converts the pixel count to hectares: **18,817 ha of forest** within the 21,168 ha park.
3. **Carbon accounting.** IPCC Tier 1 biomass defaults convert forest area to above-ground biomass, then to carbon, then to CO₂e.
4. **Uncertainty.** Classification error and the biomass-default range are propagated and combined, producing a conservative lower-bound estimate.

The whole calculation is driven by a config file and covered by unit tests, so every number can be traced back to a parameter.

## A detail worth noticing: NDVI doesn't work here

The usual shortcut for mapping forest is a greenness index like NDVI. In Kakum's humid landscape, it fails. Forest and the surrounding farmland differ by only about **0.04 in NDVI**: everything is green. Only towns and roads stand out.

![NDVI over the study area](https://raw.githubusercontent.com/joenewton10/kakum-carbon-mrv/master/docs/images/ndvi_comparison.png)
*Forest and farmland are almost indistinguishable in NDVI.*

The classifier gets around this by using Sentinel-2's shortwave-infrared bands, which respond to canopy structure and moisture rather than just greenness.

## The results

| Metric | Value |
|---|---|
| Carbon stock, above-ground | ≈ 10.0 million tCO₂e |
| Carbon stock, including roots | ≈ 12.47 million tCO₂e |
| Combined uncertainty | ≈ 61.5% |
| Conservative (lower-bound) estimate | ≈ 4.80 million tCO₂e |

That gap between 12.47 million and 4.80 million is the real story of this project.

## Where the uncertainty actually lives

My first instinct was that the map would be the weak point. It wasn't. The 95.5%-accurate classifier contributes only about **4.5%** of the total error.

The other roughly 61% comes from a single number: the **IPCC Tier 1 biomass default** for tropical forest. It's a continent-wide average with a range of 130 to 510 tonnes per hectare. Applied to one specific forest, that range swamps everything else.

![Biomass sensitivity chart](https://raw.githubusercontent.com/joenewton10/kakum-carbon-mrv/master/outputs/sensitivity_plot.png)
*Total carbon stock across the full biomass-default range, with the mean of 310 t/ha marked.*

This diagnosis changed my priorities. Spending more time tuning the classifier would barely move the final uncertainty. The improvement that matters is replacing the default with **site-specific biomass**, and that's the planned next step: a Tier 2 upgrade using biomass derived from NASA's GEDI spaceborne lidar.

In carbon markets, this isn't academic. Conservative estimates are what get credited, so a narrower uncertainty range translates directly into more credible, and more valuable, outcomes.

## Honest limitations

- **This is a carbon stock, not an emissions estimate.** Measuring emissions or avoided deforestation needs at least a second image date to measure change over time.
- **It's a prototype, not a registered project.** It reproduces the measurement side of VM0048: the forest map, area and carbon stock. A real VM0048 project also relies on an externally provided jurisdictional baseline, which a project developer doesn't produce.
- **Tier 1 defaults** are the main reason the uncertainty is so wide. That's the point of the analysis, but it also means the headline number should always be read alongside the conservative estimate.

## Takeaway

A highly accurate map can still produce a highly uncertain carbon number. Running the uncertainty analysis end to end, rather than stopping at classification accuracy, is what showed me where the real work is. For anyone building MRV systems, my main lesson is to **find your dominant error source before you optimise anything.**

[View the code on GitHub](https://github.com/joenewton10/kakum-carbon-mrv){: .btn .btn--primary}
[Read the full report (PDF)](https://github.com/joenewton10/kakum-carbon-mrv/blob/master/Kakum_MRV_Report.pdf){: .btn .btn--inverse}
