---
title: " A Scope 1 and 2 inventory for a German manufacturer"
excerpt: "Building a GHG Protocol inventory for a precision cutting site, and why its market-based Scope 2 came out at roughly double the location-based figure."
categories:
  - GHG Accounting
tags:
  - GHG Protocol
  - Scope 2
  - VSME
  - Python
  - Streamlit
---

Small manufacturers are now getting emissions questionnaires from their customers, usually larger companies that report under the EU's CSRD and need supplier data. The arithmetic behind an answer is easy. Knowing which factors to use, and being able to explain them to an auditor, is harder.

I built a tool that produces a GHG Protocol Scope 1 and 2 inventory for a contract precision cutting site. It's modelled on HTWest GmbH in Bonn, where I work.

[Try the live demo](https://ghg-inventory-io.streamlit.app/)

![First page of the sample GHG inventory report](https://raw.githubusercontent.com/joenewton10/ghg-inventory/main/docs/report_page1.png)
*First page of the generated PDF report.*

## What it does

You give it activity data in an Excel workbook (gas and electricity bills, fuel-card statements, refrigerant service records) and it pulls emission factors from versioned files that cite their sources. It produces a PDF report, an Excel workpaper an auditor can trace line by line, tables for Power BI, and a manifest with file hashes so any run can be reproduced.

There's also a Streamlit app on top of the same code, where you can edit the data, change settings, try abatement scenarios and download everything.

## Results

On representative data for a site like this:

| | t CO₂e |
|---|---|
| Scope 1 (gas heating, vans, forklift, refrigerant leaks) | 40.5 |
| Scope 2, location-based | 68.8 |
| Scope 2, market-based | 140.3 |
| **Total, location-based** | **109.3** |
| **Total, market-based** | **180.8** |

Electricity is 63% of the location-based total, which is what you'd expect at a cutting plant. Compressors, extraction and the cutting lines all run on power.

## Why market-based is double

The GHG Protocol asks for two Scope 2 numbers. The location-based one uses the average intensity of the German grid, 344 g CO₂/kWh. The market-based one reflects the contract the company actually has.

In Germany, a lot of renewable electricity is claimed by other buyers through guarantees of origin. If you're on a standard tariff, your power is counted at the residual mix, which was 701 g CO₂/kWh in 2025. Same kilowatt-hours, twice the emissions.

Customers who ask for market-based figures will see 140 tonnes, not 69. The report shows both and explains what would close the gap.

## The fiddly parts

Combustion produces CO₂, CH₄ and N₂O, and refrigerants leak HFCs, so the tool works out each gas separately before converting to CO₂e with IPCC AR6 values. I wrote a test that checks the results match the UK DESNZ published totals exactly when run with AR5.

Refrigerant leaks from air conditioning and air dryers are the Scope 1 source people most often forget. The tool handles them with either the GHG Protocol screening method or a simple mass balance.

For uncertainty, each activity row has a data-quality score and each factor has its own uncertainty range. A 10,000-run Monte Carlo gives 95% ranges. I treated factor errors as the same across all months, since a wrong factor is wrong every month, which stops the uncertainty from shrinking unrealistically.

Each figure is also mapped to where a customer would look for it: GHG Protocol, the EFRAG voluntary SME standard (VSME), ESRS E1 and ISO 14064-1. There are 62 tests in total.

## Turning it into decisions

The tool ranks seven reduction options by cost per tonne. Some results were not what I expected.

Fixing compressed-air leaks and recovering compressor heat both pay for themselves. Compressed air wastes a lot of electricity in plants like this.

A green tariff is the cheapest way to lower the market-based number, at about €14 per tonne. It doesn't change anything physical, and the report says so.

A heat pump can actually raise market-based emissions on the current tariff, because the extra electricity is counted at the residual mix. It only makes sense in carbon terms together with a cleaner electricity contract.

## Limitations

The activity data is representative, not HTWest's real bills. The method, factors and code are real, and real bills can go into the input workbook without any code changes.

Scope 3 is outside the boundary. A rough screen of the incoming filter media comes to about 1,200 t CO₂e, roughly seven times Scope 1 and 2 together. I left it out because Scope 1 and 2 are what the company controls and can measure from its own bills, but the report shows the estimate so the exclusion is visible.

The fuel factors are UK averages from DESNZ. German fuels are close, not identical.

The part of this project I'd most like to see in real use is the report itself: both Scope 2 numbers side by side, every factor cited, and the gaps written down.

[Try the live demo](https://ghg-inventory-io.streamlit.app/){: .btn .btn--primary}
[View the code on GitHub](https://github.com/joenewton10/ghg-inventory){: .btn .btn--inverse}
