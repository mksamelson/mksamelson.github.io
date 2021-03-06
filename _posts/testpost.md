---
title:  "Predicting Customer Churn"
date: 2019-11-29
tags: [Classification, Machine Learning, Data Science]
excerpt:  #"Predicting Customer Churn"
---

Customer retention is critical for all businesses.  Customers are the primary source of company revenue and
have a direct impact on the bottom line. Understanding engagement - and better yet being able to predict
engagement - can be particularly helpful to intercede in issues **before** they become problems.

In this post we examine three months of real pre-paid phone card data and develop a model to predict
customers who will drop the service in the fourth month.

All work is done in R.

## The Data

The data is contained in a single file with multiple sheets.  One sheet contains features. These include but are not limited to customer spending, phone use, SMS use, and data use. A second sheet contains a binary churn indicator showing which customers dropped service in September 2013. A third page has variable definitions in English.

Data cleaning is required prior to exploratory data analysis.  As we are using three months of data to predict churn we actually have 3 times 65 or 195 features.  Each of the 65 features becomes a "feature_month_minus_1", "feature_month_minus_2" and "feature_month_minus_3"  Reflecting the activity at each of these different time points relative to service drop.

### Load Data and Munge data

Loading and munging is fairly straightforward.

In short I do the following:

* Load data from csv files.

* Specify categorical variables (obtained through manual review of data since there are some continuous features coded as integers).

* Convert dataframe to datatable for efficiency purposes.

The script snippet below shows the code for the process.

<script src="https://gist.github.com/mksamelson/8367db04f5a0aa6ade6d374ec35e73b5.js"></script>

Next I run some basic pre-processing on the data.

I do the following:

* Check the structure
* Check the datatable dimensions
* Check for missingness
* Run a Summary of each Feature

<script src="https://gist.github.com/mksamelson/63c43236f841d7a27857effadb945143.js"></script>

The structure command provides the following overview of data set shape and features:

<script src="https://gist.github.com/mksamelson/37b811bbb3e5f73e9eabfff23ab9c2ed.js"></script>

The data set has 65 features and 182,343 observations. However, this is slightly misleading.

The dataset consists of three months data in *stacked* format.  Customers that had service for three months have three observations in the set.  Customers who acquired/terminated service during that period have one or two observations.

Since we are using 3 months of data to predict termination in the fourth month and the data is stacked, we actually have *3 times* the number of features.  This is because we have the value for each feature at different points in time:  Termination minus one month; Termination minus two months; and Termination minus three months.

Accordingly, we must restructure the data in preparation for making a solution.

We restructure the data as follows:

* Split the stacked data into individual datatables by month
* Add a month suffix to each feature name: 6 for June, 7 for July, and 8 for August
* Drop month and data columns.  Also customer "user_lifetime" feature from June and
July as August will have the most accurate value
* Merge the monthly datatables together on the "client_id" field.
* Eliminate customers that were not present all three months.

<script src="https://gist.github.com/mksamelson/a6d7a87a42dd8f7b409d93f76e1dbfa1.js"></script>

After revisions, we see the shape of more reflective dataset as 185 features and
66,469 observations.  165 of the features are continuous with the balance binary
categorical.  The number of observations reflects customers who had service for all 3 months, the balance were eliminated during the merge process shown above.

## Exploratory Data Analysis (EDA)

EDA is essential to understand the data set.  However, presentation of summary statistics, distributions, correlations, and other interpretations of data of approximately 180 features does not conveniently lend itself to presentation in the context of a web page.  

I looked at missingness, outliers, distributions, and feature correlations.

There was no missing data in the initial data set.

I created a feature matrix to examine the one-to-one relationship between features.
Unfortunately, a 150+ by 150+ matrix does not lend itself well to visualization in
the terminal space.  Understanding that I would be applying low-bias boosted tree classifier and my objective was to get the best performing model possible, I did not perform any type of feature selection.  Furthermore, I did not remove correlated variables as boosted tree models are generally insensitive to feature standardization and will pick relevant features in the context of model training.

I examined feature distributions individually to identify outliers and find
erroneous data.  Only one feature, user_spendings, had any suspicious data a deminimis 27 negative values.  For good form, I eliminated these values as erroneous, substituting an NA for the negative values.

The figures below demonstrate the process through which I generated histograms, reviewed, and modified the data.

<img src="{{site.url}}{{ site.baseurl }}/images/user_spendings_pre.jpeg" alt="this is a placeholder image">  

<script src="https://gist.github.com/mksamelson/ccca5078c2a4f455bfda1ad9e721f681.js"></script>

<img src="{{site.url}}{{ site.baseurl }}/images/user_spendings_post.jpeg" alt="this is a placeholder image">  

## Model

I chose a boosted tree model to predict churn due to it's powerful predictive capability, it's ability to work well with non-normally distributed inputs,
and its ability to easily and powerfully assess feature importance.

I used the xgbtree method in the Caret package which makes use of XgBoost a variation
of a boosted tree ensemble. Boosted trees are exceptionally powerful and the XgBoost package is known to be particularly powerful. I used the  R Caret package to access XgBoost because I found data pre-processing, use  of cross validation in model training, and performing a grid search to obtain optimal model parameters to be easier to use in Caret than in XgBoost proper.

<script src="https://gist.github.com/mksamelson/9073eac9e4ab9f0320d3cb8b44c68876.js"></script>

The best model had an AUC of approximately .92 and an error of approximately .11 using the accuracy metric.




To address the business problem (i.e., identify and contact customers who will leave in a subsequent month based) I found it best to use an ROC cutoff that balanced specificity and sensitivity and resulted in slightly lower accuracy but yielded a preferable mix of true positives and false positives beneficial to addressing the business problem.

Training the model is time and resource intensive. The model was trained in an AWS instance with 8G RAM and 30G memory. Run time in that environment was approximately one hour.

Since training the model may not be practical in terms of time and resources I have provided the analytics output as comments in the code file. Plots are in their own separate files in the repository. Unfortunately the trained model file was too large to upload to the Github repository.

<img src="{{site.url}}{{ site.baseurl }}/images/ROC_curve.jpeg" alt="ROC curve">

<img src="{{site.url}}{{ site.baseurl }}/images/cm_optimal.jpeg" alt="">
