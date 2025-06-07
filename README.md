# Analyzing Power Outages

by Charles Zhang

---

## Introduction

In this project, I leveraged the Data on Major Power Outage Events in the Continental U.S. dataset—which evaluates risk predictors associated with sustained outages across the continental United States—to both analyze and forecast significant blackout incidents. This dataset was obtained from Purdue University’s Laboratory for Advancing Sustainable Critical Infrastructure at
https://engineering.purdue.edu/LASCI/research-data/outages.

For my analysis, I began by performing thorough data cleaning and exploratory data analysis, including an examination of the dataset’s missing-value mechanisms. I then concentrated on building a model to predict whether a given outage would qualify as a major event—one that disrupts service for a large number of customers—in order to help utilities proactively allocate repair and restoration resources.

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
