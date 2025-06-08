# Analyzing Power Outages
Project for Dsc 80 at UCSD
By Charles Zhang

---

## Introduction
I am using the Major Power Outage Events dataset from Purdue University’s LASCI Lab, which records every sustained blackout in the continental United States from 2000 through 2020. The dataset can be accessed through https://engineering.purdue.edu/LASCI/research-data/outages. In total there are 1,535 outage events and 57 different measurements for each event. My project centers on one question: when an outage first begins, can we tell whether it will turn into a severe incident that cuts power to ten thousand or more customers?

Early prediction of large outages is important because utilities can send additional crews and resources to the hardest-hit areas, reducing downtime and economic loss. To build a model that does this, I will only use information that is known at the moment the outage starts—such as the date, the region affected, initial cause signals, and the size of the customer base.

Below is a summary of the most important columns for my prediction task:

| Column                     | Description                                                                                  |
|----------------------------|----------------------------------------------------------------------------------------------|
| YEAR                       | The year in which the outage began                                                          |
| MONTH                      | The month in which the outage began                                                         |
| OUTAGE.START.DATE          | The calendar date when the outage event started                                             |
| OUTAGE.START.TIME          | The time of day at which the outage began                                                   |
| OUTAGE.RESTORATION.DATE    | The calendar date when power was fully restored                                             |
| OUTAGE.RESTORATION.TIME    | The time of day at which service was restored                                               |
| NERC.REGION                | The North American Electric Reliability Corporation region where the outage occurred         |
| CLIMATE.REGION             | The U.S. climate zone (one of nine regions defined by NOAA)                                  |
| ANOMALY.LEVEL              | The three-month Oceanic Niño Index value, indicating El Niño or La Niña conditions           |
| CLIMATE.CATEGORY           | A label of “Cold,” “Normal,” or “Warm” based on the Niño index threshold                    |
| CAUSE.CATEGORY             | A broad category of what caused the outage (for example, severe weather or equipment failure) |
| OUTAGE.DURATION            | The total length of the outage in minutes                                                    |
| DEMAND.LOSS.MW             | The maximum megawatts of demand that were lost during the outage                             |
| CUSTOMERS.AFFECTED         | The number of customers who lost power                                                       |
| TOTAL.CUSTOMERS            | The total number of customers served in the affected state                                   |
| TOTAL.PRICE                | The average monthly electricity price in cents per kilowatt-hour                             |

I will use these features to train a binary classifier that tells “severe” versus “non-severe” outages, and I will evaluate my model using the F1-score so that it balances the goals of catching serious events and avoiding unnecessary alarms.


# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

Before jumping into modeling, I carried out a systematic cleanup and initial exploration of the outage dataset. Below is a concise summary of each step, written in the same style as our project template:

1. **Filter out incomplete timing records**  
   We loaded the raw CSV into pandas and dropped any rows missing **month** or **time** information—since the precise timing of an outage is essential, records without these fields were removed.

2. **Combine date and time into timestamps**  
   We merged the separate start-date/start-time and restoration-date/restoration-time columns into two unified datetime fields, `OUTAGE.START` and `OUTAGE.RESTORE`, then dropped the original split-out columns.

| YEAR | MONTH | U.S._STATE | NERC.REGION | CLIMATE.REGION     | ANOMALY.LEVEL | CLIMATE.CATEGORY | OUTAGE.START         | OUTAGE.RESTORATION      | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL | OUTAGE.DURATION | DEMAND.LOSS.MW | … |
|-----:|------:|:-----------|:------------|:-------------------|--------------:|:-----------------|:---------------------|:-----------------------|:-------------------|:----------------------|----------------:|---------------:|---|
| 2011 |     7 | Minnesota  | MRO         | East North Central |          -0.3 | normal           | 2011-07-01 17:00:00  | 2011-07-03 20:00:00    | severe weather     | nan                   |            3060 |            nan | … |
| 2014 |     5 | Minnesota  | MRO         | East North Central |          -0.1 | normal           | 2014-05-11 18:38:00  | 2014-05-11 18:39:00    | intentional attack | vandalism             |               1 |            nan | … |
| 2010 |    10 | Minnesota  | MRO         | East North Central |          -1.5 | cold             | 2010-10-26 20:00:00  | 2010-10-28 22:00:00    | severe weather     | heavy wind            |            3000 |            nan | … |
| 2012 |     6 | Minnesota  | MRO         | East North Central |          -0.1 | normal           | 2012-06-19 04:30:00  | 2012-06-20 23:00:00    | severe weather     | thunderstorm          |            2550 |            nan | … |
| 2015 |     7 | Minnesota  | MRO         | East North Central |           1.2 | warm             | 2015-07-18 02:00:00  | 2015-07-19 07:00:00    | severe weather     | nan                   |            1740 |            250 | … |

---
## Exploratory Data Analysis

### Univariate Analysis

In my exploratory data analysis, I first perform univariate analysis to examine the distribution of single variables.

To begin, I looked at how the total number of outages has trended over the years. This helps us see whether blackouts are becoming more or less frequent over time.  
<iframe  
  src="assets/Monthly_Outages.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  
*The plot shows a general rise in outage counts from 2000 through 2011, followed by a gradual decline—suggesting increased grid resilience after 2011.*

Next, I examined the breakdown of outage causes to understand which types of events occur most often.  
<iframe  
  src="assets/Distribution_of_CAUSE.CATEGORY.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  
*Severe weather clearly dominates as the leading cause, while equipment failures and intentional attacks appear much less frequently.*

Finally, I checked how outage durations are distributed across states to identify any regional hotspots of long outages.  
<iframe  
  src="assets/Histogram_of_Outage_Duration.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  
*Most outages last under 1,000 minutes, but there is a long tail of rare events that exceed 5,000 minutes—highlighting the importance of modeling extreme durations separately.*

---

### Bivariate Analysis

To see how two severity metrics relate, I plotted outage duration against the number of customers affected.  
<iframe  
  src="assets/Demand_Loss_vs_Customers_Affected.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  
*There is a loose positive trend: longer outages often impact more customers, but some high-impact events end quickly, indicating other factors at play.*

I then explored how outage duration varies by cause category to see which causes tend to produce the longest disruptions.  
<iframe  
  src="assets/Outage_Duration_vs_Demand_Loss.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  
*The scatter shows only a weak upward trend—longer outages often mean more demand loss, but the points are widely scattered. This dispersion suggests that factors like cause category or customer density also play a big role in how much power is lost.*  

Finally, I grouped outages by climate region and month to uncover seasonal and geographic patterns.  
<iframe  
  src="assets/Outage_Frequency_and_Average_Duration_by_Climate_Region_&_Month.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe>  
*This heatmap reveals that the South experiences more frequent outages in hurricane season (August–October), whereas the Northwest sees its peak in winter months due to storms.*


### Grouping and Aggregates

I created a pivot table showing counts of outage causes by Climate Region:

| CLIMATE.REGION        | equipment failure | fuel supply emergency | intentional attack | islanding | public appeal | severe weather | system operability disruption |
|-----------------------|------------------:|----------------------:|-------------------:|----------:|--------------:|---------------:|-------------------------------:|
| Central               |                 5 |                     4 |                34 |         3 |             2 |            133 |                             10 |
| East North Central    |                 3 |                     4 |                20 |         1 |             2 |            104 |                              3 |
| Northeast             |                 5 |                    14 |               131 |         1 |             4 |            175 |                             14 |
| Northwest             |                 2 |                     1 |                85 |         3 |             2 |             25 |                              4 |
| South                 |                 9 |                     4 |                28 |         2 |            42 |            106 |                             27 |
| Southeast             |                 4 |                     0 |                 9 |         0 |             5 |            116 |                             16 |
| Southwest             |                 5 |                     1 |                61 |         1 |             1 |             10 |                              9 |
| West                  |                21 |                    10 |                31 |        28 |             9 |             67 |                             39 |
| West North Central    |                 1 |                     0 |                 4 |         5 |             2 |              4 |                              0 |

*Severe weather remains the most common cause everywhere. The West reports the highest equipment failures (21) and islanding events (28), while the South leads in public appeals (42). The Northeast sees the most intentional attacks (131).*


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
  src="assets/Permutation_Test.html"  
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
  src="assets/Permutation_Distribution_of_the_K–S_Statistic.html"  
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


Features in my model
- 4 nominal features (`NERC.REGION`, `CLIMATE.REGION`, `CAUSE.CATEGORY`, `ANOMALY.LEVEL`)  
- 3 quantitative features (`YEAR`, `MONTH`, `log_TOTAL_CUSTOMERS`)  

I chose these because:  
- The categorical features describe the grid area and climate context, which affect outage patterns.  
- The numeric features (year/month) capture seasonality and trends.  
- The log of total customers scales down large counts for stable learning.


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

## Fairness Analysis

For the fairness analysis, I compared model performance on two groups defined by CLIMATE.REGION:

- Group X: Northwest  
- Group Y: South

We chose to compare the Northwest and South regions because they experience very different weather drivers (e.g. winter storms vs. hurricanes). Our goal was to check whether the model’s F1-score stays consistent across these two climates. 

Evaluation metric: F1-score on the same 20% hold-out test set.

Null Hypothesis (H₀):  
The model’s F1-score for Northwest outages equals its F1-score for South outages; any difference is due to random chance.

Alternative Hypothesis (H₁):  
The model’s F1-score for Northwest outages differs from its F1-score for South outages.

Test statistic:  
Δ = F1(Northwest) − F1(South)

- Observed F1(Northwest) = 0.889  
- Observed F1(South)     = 0.923  
- Observed Δ             = –0.034

<iframe  
  src="assets/Permutation_Distribution_of_F1_Difference.html"  
  width="800"  
  height="600"  
  frameborder="0">  
</iframe> 

I performed a permutation test with 1 000 shuffles of the group labels to build a null distribution of Δ.  
p-value = 0.59 (two-sided)

Because p = 0.59 > 0.05, we fail to reject H₀. There is no significant evidence of performance disparity; the model appears fair across these two climate regions.

