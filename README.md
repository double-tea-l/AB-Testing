# A/B Testing Case Study with Python Example

- [A/B Testing Case Study with Python Example](#ab-testing-case-study-with-python-example)
  - [1. Background](#1-background)
    - [Create a dummy dataset for the case](#create-a-dummy-dataset-for-the-case)
  - [2. Creat A/B Groups](#2-creat-ab-groups)
    - [2.1 Selection Bias & Randomization Concept](#21-selection-bias--randomization-concept)
    - [2.2 Calculate Mahalanobis Distance & Remove Outliers](#22-calculate-mahalanobis-distance--remove-outliers)
    - [2.3 Two-stage Cluster Sampling](#23-two-stage-cluster-sampling)
    - [2.4 Check Balance](#24-check-balance)

## 1. Background

XYZ company has a customer service department to offer assistances for addressing any issues customers report after they purchase the products. At the end of the service, customers are requested to fill a survey for rating their satisfication level of the service: 1 for satisfied and 0 for disatisfied. The company analyzes this data to evaluate the performence of customer service agents as well as monitor overall customer satisification level. Two metrics are being collected for each customer service agent: number of calls they received and customer disatification rate of those calls. 

After reviewing the result, the company plans to roll out a new SOP training to all customer service agents, hoping this could help improve the performance and better customer ratings. However, before investing training resources at full scope, the company wanted to design a A/B testing, at a smaller scale, for experiementing whether the new SOP training can actually make any difference. 

As a result, a total of 1,000 customer service agents were selected randomly for the experiement. The goal is to distribute them into two groups, treatment and control, run the experiement for X months, and evaluate the efficacy of the training program. 

### Create a dummy dataset for the case

Use faker to create a dataframe that includes:

* Agent ID 
* Agent Name
* Number of calls
* Disatisfication Rate 

Check the dataframe df_agent

|    |   Agent ID | Agent Name    |   Number of calls |   Disatisfication Rate |
|---:|-----------:|:--------------|------------------:|-----------------------:|
|  0 |       1000 | Amber Hill    |               791 |             0.0515969  |
|  1 |       1001 | Raymond Boyd  |               478 |             0.402953   |
|  2 |       1002 | Karen Moore   |              1853 |             0.00966783 |
|  3 |       1003 | Julia Buckley |              1568 |             0.0289132  |
|  4 |       1004 | Renee Smith   |               119 |             0.106509   |


<img src="img/dummy.png"/>

## 2. Creat A/B Groups

### 2.1 Selection Bias & Randomization Concept

The principle challenge in assigning A/B groups is elimination of "selection bias" - typically introduced by improper randomization. To allow apples-to-apples comparison in an *ceteris paribus* experiement, the two groups need to meet a few criteria:

* each is representative of the entire population
* are independent from each other
* remain covariate balance[^1] between groups

In other words, these two groups need to be representative of the population and similar in every way. Therefore, we need to make sure samples in two groups start off from the same baseline before the experiement starts. That means, in this example, agents in each group should have similar disatisfication rate and number of customer calls. 

### 2.2 Calculate Mahalanobis Distance & Remove Outliers

Mahalanobis distance[^1] can be useful to evaluate data variances where there're multiple dimensions. Both metrics, disatisfication rate and number of customer calls are being evaluated of how far a data point diverge from the population center. The `p-value` is set at 0.05, less than which is deemed as an outlier. 

<img src="img/remove_outlier.png"/>

### 2.3 Two-stage Cluster Sampling

In this analysis, a two-stage cluster sampling method was applied. First, the population was splitted into four different clusters based on a K-Mean algorithm. After that, equal amount of data points were randomly drew from each cluster and formed a control and treatment group. 

<img src="img/cluster.png"/> <img src="img/ab_groups.png"/>

### 2.4 Check Balance

Checking balance is an essential step in sampling. A few things were checked in this step:

* Sampling distribution
* Homogeneity of variances
* Sample mean

A `Two-sample Kolmogorov-Smirnov Test` was performed with the null hypothesis as "the two distributions of A/B groups are identical". The result is that `p-value > 0.05`, which means we cannot reject the null hypothesis. 

```
stats.ks_2samp(df_A['Disatisfication Rate'], df_B['Disatisfication Rate'])
```

A `Levenie Test` was peformed to test the homogeneity of variances of two groups. The `p-value is > 0.05`, indicating that the variances between A/B groups are not statistically significant. 

```
stats.levene(df_A['Disatisfication Rate'], df_B['Disatisfication Rate']) # less sensitive to normality
```

And lastly, a `T-test` was performed to test the differences bewteen two sample means. 

|    | Variable   |   N |     Mean |       SD |         SE |   95% Conf. |   Interval |
|---:|:-----------|----:|---------:|---------:|-----------:|------------:|-----------:|
|  0 | Control    | 481 | 0.159433 | 0.127517 | 0.00581427 |    0.148009 |   0.170858 |
|  1 | Treatment  | 481 | 0.158891 | 0.128788 | 0.00587221 |    0.147353 |   0.170429 |
|  2 | combined   | 962 | 0.159162 | 0.128087 | 0.0041297  |    0.151058 |   0.167267 |

The `p-value` of t-test is also > 0.05 and hence we cannot reject the null hypothesis that the difference in A/B groups means are zero. 

|    | Independent t-test                 |   results |
|---:|:-----------------------------------|----------:|
|  0 | Difference (Control - Treatment) = |    0.0005 |
|  1 | Degrees of freedom =               |  960      |
|  2 | t =                                |    0.0656 |
|  3 | Two side test p value =            |    0.9477 |
|  4 | Difference < 0 p value =           |    0.5262 |
|  5 | Difference > 0 p value =           |    0.4738 |
|  6 | Cohen's d =                        |    0.0042 |
|  7 | Hedge's g =                        |    0.0042 |
|  8 | Glass's delta1 =                   |    0.0043 |
|  9 | Point-Biserial r =                 |    0.0021 |

As of now, we have created two fairly similar experiement sample groups: control (A) and treatment (B). In the next section, we'll be creating dummy data for experiement results, and analyze the program efficancy. 




[^1] Covariate balance: [Assessing Balance](https://cran.r-project.org/web/packages/MatchIt/vignettes/assessing-balance.html#:~:text=Covariate%20balance%20is%20the%20degree,across%20levels%20of%20the%20treatment.) 

[^2] Learn more about [Mahalanobis Distance method](http://mccormickml.com/2014/07/22/mahalanobis-distance/) and the [implementation in python](https://www.geeksforgeeks.org/how-to-calculate-mahalanobis-distance-in-python/). 