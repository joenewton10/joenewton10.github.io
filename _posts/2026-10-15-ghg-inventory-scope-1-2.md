---
title: Building a Scope 1 & 2 Inventory for a German Manufacturer"
excerpt: "A GHG Protocol inventory for a precision cutting site, and why its market-based Scope 2 figure came out at roughly double the location-based one."
categories:
  - GHG Accounting
tags:
  - GHG Protocol
  - Scope 2
  - VSME
  - Python
  - Streamlit
---

Small manufacturers are increasingly being asked a question they've never had to answer: *what are your emissions?* The request usually comes from larger customers who report under the EU's CSRD and need figures from their suppliers. For a small site, the hard part isn't the arithmetic; it's knowing which numbers to use and being able to defend them.

I built a tool to produce a full GHG Protocol Scope 1 and 2 inventory for a contract precision cutting site, modelled on the kind of operation I work in at HTWest GmbH in Bonn.

**[Try the live demo →](https://ghg-inventory-io.streamlit.app/)**

![First page of the sample GHG inventory report](https://raw.githubusercontent.com/joenewton10/ghg-inventory/main/docs/report_page1.png)
*First page of the generated PDF report.*

## What the tool does

It reads activity data (gas and electricity bills, fuel-card statements, refrigerant service logs) from an Excel workbook, and emission factors from versioned, cited factor files. It then produces:

- a **PDF report** a manager can actually read
- an **Excel workpaper** for an auditor, with every line traceable
- **star-schema tables** ready for Power BI or Tableau
- a **run manifest** with hashes of every input, so any result can be reproduced

A Streamlit app wraps the same engine: you can edit the input data, adjust settings, explore abatement scenarios and export everything from the browser.

## The results

On representative data for a site of this type (more on that below):

| | t CO₂e |
|---|---|
| Scope 1 (gas heating, vans, forklift, refrigerant leaks) | 40.5 |
| Scope 2, location-based | 68.8 |
| Scope 2, market-based | 140.3 |
| **Total, location-based** | **109.3** |
| **Total, market-based** | **180.8** |

Electricity makes up 63% of the location-based total. That's the expected shape for a cutting operation, where compressors, extraction and the cutting lines all run on power.

## Why market-based Scope 2 is twice as high

This was the most instructive result. The GHG Protocol requires two Scope 2 figures:

- **Location-based** uses the average carbon intensity of the German grid: **344 g CO₂/kWh**.
- **Market-based** reflects what the company actually buys. In Germany, much of the renewable power is claimed by others through guarantees of origin. Electricity bought on a standard tariff is therefore counted at the **residual mix: 701 g CO₂/kWh**, roughly double the grid average.

So the same kilowatt-hours produce twice the emissions on the market-based method. Customers who ask suppliers for market-based figures will see the larger number. The report shows both and explains what it would take to close the gap.

## Getting the details right

Most of the work in an inventory is in choices, not arithmetic:

- **Per gas, then CO₂e.** Combustion emits CO₂, CH₄ and N₂O, and refrigerants leak HFCs. The tool calculates each gas separately and then applies a declared GWP set (IPCC AR6 by default). A test confirms the results reconcile exactly with the UK DESNZ published totals.
- **Refrigerants.** Leaks from air conditioning and refrigerant air dryers are the Scope 1 source most often missed. The tool supports both the screening method and a simplified mass balance.
- **Uncertainty.** Every activity row carries a data-quality score, and every factor has its own uncertainty. A 10,000-draw Monte Carlo produces 95% ranges. Factor errors are treated as correlated across months, so they don't artificially average out.
- **Framework mapping.** Each figure is mapped to where a customer would look for it: the GHG Protocol, the EFRAG voluntary SME standard (VSME), ESRS E1 and ISO 14064-1.

The engine is covered by 62 tests, including the market-based hierarchy, unit conversions and Excel round trips.

## From inventory to decisions

An inventory is only useful if it leads somewhere. The tool ranks seven abatement options by cost per tonne. A few results surprised me:

- **Fixing compressed-air leaks and recovering compressor heat pay for themselves.** Compressed air is typically one of the most wasteful uses of electricity in a plant like this.
- **A green tariff is the cheapest way to cut the market-based number** (about €14 per tonne), but it changes nothing physically. The report says so plainly.
- **A heat pump can *raise* market-based emissions** on the current tariff, because the extra electricity is counted at the residual mix. It only pays off in carbon terms alongside a cleaner electricity contract.

## Honest limitations

- **The data is representative, not real meter readings.** The method, factors and code are real; the activity numbers are realistic placeholders. Real bills can be dropped into the input workbook with no code changes.
- **Scope 3 is outside the boundary.** A rough screen of the incoming filter media alone comes to about 1,200 t CO₂e, around seven times Scope 1 and 2 combined. The exclusion is deliberate (Scope 1 and 2 are what the company controls and can measure from its own bills), but it's sized and shown, not hidden.
- **Some factors are UK averages** (DESNZ fuel factors), which are close to but not identical to German values.

## Takeaway

For a small manufacturer, a credible inventory isn't about sophisticated modelling. It's about transparent factor choices, both Scope 2 methods side by side, a visible boundary and an audit trail. Building this made me appreciate how much the *presentation* of a number, and its honest caveats, matters to the people who have to act on it.

[Try the live demo](https://ghg-inventory-io.streamlit.app/){: .btn .btn--primary}
[View the code on GitHub](https://github.com/joenewton10/ghg-inventory){: .btn .btn--inverse}
