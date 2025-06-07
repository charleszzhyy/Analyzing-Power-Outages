# Analyzing Power Outages

by Charles Zhang

---

## Introduction

In this project, I leveraged the Data on Major Power Outage Events in the Continental U.S. dataset—which evaluates risk predictors associated with sustained outages across the continental United States—to both analyze and forecast significant blackout incidents. This dataset was obtained from Purdue University’s Laboratory for Advancing Sustainable Critical Infrastructure at
https://engineering.purdue.edu/LASCI/research-data/outages.

For my analysis, I began by performing thorough data cleaning and exploratory data analysis, including an examination of the dataset’s missing-value mechanisms. I then concentrated on building a model to predict whether a given outage would qualify as a major event—one that disrupts service for a large number of customers—in order to help utilities proactively allocate repair and restoration resources.

In the analysis that follows, I will extract the key data from the original dataset for analysis and model building.

| Column                    | Description                                                                                       |
|---------------------------|---------------------------------------------------------------------------------------------------|
| `YEAR`                    | Year an outage event occurred                                                                      |
| `MONTH`                   | Month an outage event occurred                                                                     |
| `U.S._STATE`              | U.S. state where the outage occurred                                                               |
| `NERC.REGION`             | North American Electric Reliability Corporation region involved in the outage                      |
| `CLIMATE.REGION`          | U.S. Climate region as specified by National Centers for Environmental Information (9 regions)     |
| `ANOMALY.LEVEL`           | Oceanic El Niño/La Niña (ONI) index (cold/warm episodes by season; 3-month running mean in Niño 3.4) |
| `CLIMATE.CATEGORY`        | Climate episode category (“Warm”, “Cold” or “Normal”) based on ± 0.5 °C ONI threshold               |
| `OUTAGE.START`            | Combined start date & time of the outage event (day of year + time of day)                         |
| `OUTAGE.RESTORATION`      | Combined restoration date & time when power was fully restored (day of year + time of day)         |
| `CAUSE.CATEGORY`          | High‐level category of the event causing the outage                                                |
| `CAUSE.CATEGORY.DETAIL`   | Detailed description of the cause                                                                  |
| `OUTAGE.DURATION`         | Duration of the outage event (in minutes)                                                          |
| `DEMAND.LOSS.MW`          | Peak demand lost during the outage (in megawatts)                                                  |
| `CUSTOMERS.AFFECTED`      | Number of customers affected by the outage                                                         |
| `TOTAL.PRICE`             | Average monthly electricity price in the state (cents per kWh)                                     |
| `TOTAL.SALES`             | Total monthly electricity consumption in the state (megawatt-hours)                                 |
| `TOTAL.CUSTOMERS`         | Total annual number of customers served in the state                                               |
| `POPDEN_URBAN`            | Urban population density (persons per square mile)                                                 |

---

## Cleaning and EDA

<iframe src="assets/10-80-enrollment.html" width=800 height=600 frameBorder=0></iframe>

---

## Assessment of Missingness

Here's what a Markdown table looks like. Note that the code for this table was generated _automatically_ from a DataFrame, using

```py
print(counts[['Quarter', 'Count']].head().to_markdown(index=False))
```

| Quarter     |   Count |
|:------------|--------:|
| Fall 2020   |       3 |
| Winter 2021 |       2 |
| Spring 2021 |       6 |
| Summer 2021 |       4 |
| Fall 2021   |      55 |

---

## Hypothesis Testing


---
