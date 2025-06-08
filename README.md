# Analyzing Power Outages
Project for Dsc 80 at UCSD
By Charles Zhang

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

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

Before jumping into modeling, I carried out a systematic cleanup and initial exploration of the outage dataset. Below is a concise summary of each step, written in the same style as our project template:

1. **Filter out incomplete timing records**  
   We loaded the raw CSV into pandas and dropped any rows missing **month** or **time** information—since the precise timing of an outage is essential, records without these fields were removed.

2. **Combine date and time into timestamps**  
   We merged the separate start-date/start-time and restoration-date/restoration-time columns into two unified datetime fields, `OUTAGE.START` and `OUTAGE.RESTORE`, then dropped the original split-out columns.

|   YEAR |   MONTH | U.S._STATE | NERC.REGION | CLIMATE.REGION       |   ANOMALY.LEVEL | CLIMATE.CATEGORY | OUTAGE.START        | OUTAGE.RESTORATION     | CAUSE.CATEGORY   | CAUSE.CATEGORY.DETAIL |   OUTAGE.DURATION |   DEMAND.LOSS.MW |
|-------:|--------:|:-----------|:------------|:---------------------|----------------:|:-----------------|:---------------------|:----------------------|:-----------------|:----------------------|------------------:|-----------------:|
|   2011 |       7 | Minnesota  | MRO         | East North Central   |            -0.3 | normal           | 2011-07-01 17:00:00 | 2011-07-03 20:00:00   | severe weather   | nan                  |              3060 |              nan |
|   2014 |       5 | Minnesota  | MRO         | East North Central   |            -0.1 | normal           | 2014-05-11 18:38:00 | 2014-05-11 18:39:00   | intentional attack | vandalism            |                 1 |              nan |
|   2010 |      10 | Minnesota  | MRO         | East North Central   |            -1.5 | cold             | 2010-10-26 20:00:00 | 2010-10-28 22:00:00   | severe weather   | heavy wind           |              3000 |              nan |
|   2012 |       6 | Minnesota  | MRO         | East North Central   |            -0.1 | normal           | 2012-06-19 04:30:00 | 2012-06-20 23:00:00   | severe weather   | thunderstorm         |              2550 |              nan |
|   2015 |       7 | Minnesota  | MRO         | East North Central   |             1.2 | warm             | 2015-07-18 02:00:00 | 2015-07-19 07:00:00   | severe weather   | nan                  |              1740 |              250 |

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
  src="assets/Histogram_of_Outage_Duration.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

### Bivariate Analysis
I conducted several bivariate analyses; the most significant results are shown below.

I examined the relationship between Outage Duration and Customers Affected—two metrics of outage severity.  
<iframe  
  src="assets/Demand_Loss_vs_Customers_Affected.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

The plot below shows the relation between outage duration and cause category.  
<iframe  
  src="assets/Outage_Duration_vs_Demand_Loss.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

### Grouping and Aggregates

I created a pivot table showing counts of outage causes by Climate Region:

| CLIMATE.REGION     | equipment failure | fuel supply emergency | intentional attack | islanding | public appeal | severe weather | system operability disruption |
|--------------------|------------------:|----------------------:|-------------------:|----------:|--------------:|---------------:|-------------------------------:|
| Central            |                 7|                      4|                 38|          3|              2|             135|                              11|
| East North Central |                 3|                      5|                 20|          1|              2|             104|                               3|
| Northeast          |                 5|                     14|                135|          1|              4|             176|                              15|
| Northwest          |                 2|                      1|                 89|          5|              2|              29|                               4|
| South              |                10|                      7|                 28|          2|             42|             113|                              27|


## Assessment of Missingness
### NMAR Analysis
I think the missing values in DEMAND.LOSS.MW might be NMAR. Calculating how much electricity demand is lost is complicated, and the bigger a blackout is, the harder it is to fully count the lost power. So, more severe blackouts are more likely to have no DEMAND.LOSS.MW data. 
If we could also get information on the economic losses caused by each blackout, it might explain why DEMAND.LOSS.MW is missing and let us treat it as MAR.

### Missingness Dependency
I focused on whether the missing values in CAUSE.CATEGORY.DETAIL depend on other factors. I ran permutation tests separately to check its relationship with CAUSE.CATEGORY and with ANOMALY.LEVEL.

#### CAUSE.CATEGORY
Null hypothesis: The distribution of CAUSE.CATEGORY is the same whether CAUSE.CATEGORY.DETAIL is missing or not.
Alternate hypothesis: The distribution of CAUSE.CATEGORY is different when CAUSE.CATEGORY.DETAIL is missing compared to when it’s not missing.
<iframe  
  src="assets/Distribution_of_Cause_Category_by_Detail_Missingness.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  
The empirical distribution from the permutation tests is shown below. Let's use α = 0.01 here. The p-value for the observed TVD is 0.001, which is much smaller than α = 0.01, so we reject the null hypothesis. This means the missingness of CAUSE.CATEGORY.DETAIL is related to the distribution of CAUSE.CATEGORY.
<iframe  
  src="assets/Permutation Test.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

#### ANOMALY.LEVEL
Null hypothesis: When CAUSE.CATEGORY.DETAIL is missing, the distribution of ANOMALY.LEVEL is the same as when CAUSE.CATEGORY.DETAIL is not missing.
Alternative hypothesis: When CAUSE.CATEGORY.DETAIL is missing, the distribution of ANOMALY.LEVEL is different from when CAUSE.CATEGORY.DETAIL is not missing.
Because ANOMALY.LEVEL is a numeric variable, I used an ECDF to compare and show the data distributions.
<iframe  
  src="assets/ECDF_of_ANOMALY.LEVEL_by_Detail_Missingness.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  
The empirical distribution of K-S_Statistic from the permutation tests is shown below. Let’s set α = 0.01. We can see the observed KS p-value is 0.011, which is higher than α = 0.01. So, we don’t have enough evidence to reject the null hypothesis. It’s very likely that missing CAUSE.CATEGORY.DETAIL values are not related to the distribution of ANOMALY.LEVEL.
<iframe  
  src="assets/Empirical_Distribution_of_the_K-S_Statistic.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  



## Hypothesis Testing
I will use a permutation version of the paired t-test to check if the average yearly number of blackouts in the Northeast and Northwest regions is significantly different.
Null hypothesis: The average number of blackouts per year is the same in the Northeast and the Northwest.
Alternative hypothesis: The average number of blackouts per year in the Northeast is greater than in the Northwest.

<iframe  
  src="assets/Paired_t_Permutation.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  


## Framing a Prediction Problem

For this background, I want to predict when a power outage begins, whether it will be severe (impacting 10 000 or more customers) or non-severe (fewer than 10 000) power outage. To solve this, we frame the task as a binary classification problem and train a model that sees only the information available at outage start.

Features available at prediction time:

Calendar data: YEAR, MONTH

Grid location: NERC.REGION, CLIMATE.REGION

Initial cause tags: CAUSE.CATEGORY, ANOMALY.LEVEL

System scale: TOTAL.CUSTOMERS (logged to log_TOTAL_CUSTOMERS)

Evaluation metric: We use the F1-score, which balances precision (avoiding false alarms) and recall (catching true severe outages). This is critical because our classes are slightly imbalanced (≈ 53 % severe, 47 % non-severe). By relying solely on these instantaneously known features and optimizing F1, the model can provide timely guidance on resource deployment when an outage first occurs.


## Baseline Model

My baseline model is a binary classifier that predicts whether a power outage will be severe (≥ 10 000 customers affected) or non-severe. I built it using an `sklearn` Pipeline with two main steps:

1. Feature encoding 
   - Nominal (categorical): `NERC.REGION`, `CLIMATE.REGION`, `CAUSE.CATEGORY`, `ANOMALY.LEVEL` → One-Hot Encoding  
   - Quantitative (numeric): `YEAR`, `MONTH`, `log_TOTAL_CUSTOMERS` (the log of `TOTAL.CUSTOMERS`) → StandardScaler  

2. Model
   - LogisticRegression with `max_iter=5000`  

I split the data into 80 % training / 20 % testing sets, stratified by the target label.

---

Features in my model
- 4 nominal features (`NERC.REGION`, `CLIMATE.REGION`, `CAUSE.CATEGORY`, `ANOMALY.LEVEL`)  
- 3 quantitative features (`YEAR`, `MONTH`, `log_TOTAL_CUSTOMERS`)  

I chose these because:  
- The categorical features describe the grid area and climate context, which affect outage patterns.  
- The numeric features (year/month) capture seasonality and trends.  
- The log of total customers scales down large counts for stable learning.

---

Performance on the test set

| Metric    | Value  |
|-----------|--------|
| Accuracy  | 0.885  |
| Precision | 0.852  |
| Recall    | 0.949  |
| F1    | 0.898 |

<iframe  
  src="assets/Baseline_Confusion_Matrix.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  


- High recall (≈ 95 %) means the model catches almost all truly severe outages.  
- Precision (≈ 85 %) shows there are some false alarms, but overall balance is good.

- 

I believe this baseline model is good enough to provide early warnings, but there is room to reduce false alarms. In the next step, I will engineer additional features and try a more powerful model to improve precision while keeping recall high.

## Final Model

For the final model, I improved upon the baseline by engineering two new features and using a more powerful algorithm.

New features added  
1. Seasonality:  
   - `MONTH_sin = sin(2π · MONTH / 12)`  
   - `MONTH_cos = cos(2π · MONTH / 12)`  
   These capture cyclic weather patterns (e.g. winter storms vs. summer heat) without a break between December and January.  
2. Price signal:  
   - `log_TOTAL.PRICE` (the log of `TOTAL.PRICE`)  
   Electricity price often correlates with grid size and demand stress, so taking its log stabilises extreme values and reveals economic pressure on the network.

Model and hyperparameters  
- Algorithm: `RandomForestClassifier` inside a single `sklearn` Pipeline (One-Hot for categorical, StandardScaler for numeric).  
- Hyperparameter search: Used `GridSearchCV` with 3-fold cross-validation, optimizing F1-score.  
  - `n_estimators`: [200, 400]  
  - `max_depth`: [None, 10, 20]  
  - `min_samples_leaf`: [1, 3]  
- Best parameters found:  
  - `n_estimators = 200`  
  - `max_depth = 10`  
  - `min_samples_leaf = 1`

---

Performance comparison  

| Metric | Baseline | Final  | Change |
|--------|----------|--------|--------|
| F1     | 0.898    | 0.922  | +0.024 |

<iframe  
  src="assets/Confusion_Matrix.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  

- The final model increased F1 by about 0.024 over the baseline.  
- Precision for “Severe” rose (fewer false alarms) while recall stayed above 95% (still catches almost all true severe outages).
