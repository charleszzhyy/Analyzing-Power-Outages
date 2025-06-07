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

Before jumping into modeling, we carried out a systematic cleanup and initial exploration of the outage dataset. Below is a concise summary of each step, written in the same style as our project template:

1. **Filter out incomplete timing records**  
   We loaded the raw CSV into pandas and dropped any rows missing **month** or **time** information—since the precise timing of an outage is essential, records without these fields were removed.

2. **Combine date and time into timestamps**  
   We merged the separate start-date/start-time and restoration-date/restoration-time columns into two unified datetime fields, `OUTAGE.START` and `OUTAGE.RESTORE`, then dropped the original split-out columns.


---
## Exploratory Data Analysis

### Univariate Analysis
In my exploratory data analysis, I first perform univariate analysis to examine the distribution of single variables.

First, I wanted to see how the number of outages has changed over time.  
<iframe  
  src="assets/monthly_outages.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

I also wanted to see the distribution of major causes of power outages.  
<iframe  
  src="assets/Distribution%20of%20CAUSE.CATEGORY.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

Then, I wanted to see the distribution of the number of outages by each U.S. state.  
<iframe  
  src="assets/map1.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

### Bivariate Analysis
I conducted several bivariate analyses; the most significant results are shown below.

I examined the relationship between Outage Duration and Customers Affected—two metrics of outage severity.  
<iframe  
  src="assets/duration_cust.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

The plot below shows the relation between outage duration and cause category.  
<iframe  
  src="assets/duration_cause.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

### Grouping and Aggregates
I grouped the data by NERC Region and calculated the mean of key severity metrics:

| NERC.REGION | OUTAGE.DURATION | CUSTOMERS.AFFECTED | DEMAND.LOSS.MW |
|-------------|----------------:|-------------------:|---------------:|
| ASCC        |            nan  |               14273|              35|
| ECAR        |        5603.31  |             256354|          1314.48|
| FRCC        |        4271.12  |             375007|          1072.60|
| FRCC, SERC  |           372   |                nan|             nan|
| HECO        |        895.333  |             126729|           466.67|

I also created a pivot table showing counts of outage causes by Climate Region:

| CLIMATE.REGION     | equipment failure | fuel supply emergency | intentional attack | islanding | public appeal | severe weather | system operability disruption |
|--------------------|------------------:|----------------------:|-------------------:|----------:|--------------:|---------------:|-------------------------------:|
| Central            |                 7|                      4|                 38|          3|              2|             135|                              11|
| East North Central |                 3|                      5|                 20|          1|              2|             104|                               3|
| Northeast          |                 5|                     14|                135|          1|              4|             176|                              15|
| Northwest          |                 2|                      1|                 89|          5|              2|              29|                               4|
| South              |                10|                      7|                 28|          2|             42|             113|                              27|


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
