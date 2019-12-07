---
title:  "Predicting Customer Churn"
date: 2019-11-29
tags: [Classification, Machine Learning, Data Science]
excerpt:  #"Predicting Customer Churn"
---

Customer retention is critical for all businesses.  Customers are the primary source of company revenue and
have a direct impact on the bottom line. Understanding engagement - and better yet being able to predict
engagement - can be particularly helpful to intercede in issues **before** they become problems.

In this post we examine three months of pre-paid phone card data and develop a model to predict customers who will drop the service in the fourth month.

All work is done in R.

## The Data

The data is contained in a single file with multiple sheets.  One sheet contains features. These include but are not limited to customer spending, phone use, SMS use, and data use. A second sheet contains a binary churn indicator showing which customers dropped service in September 2013. A third page has variable definitions in English.

### Load Data and Munge data

Loading and munging is fairly straightforward.

In short we do the following:

* Load data from csv files.

* Identify categorical variables.  This is done by manual review of data.  We find some continuous features coded as integers.

* Convert dataframe to datatable for efficiency purposes.

The script snippet below shows the code for the process.

<script src="https://gist.github.com/mksamelson/8367db04f5a0aa6ade6d374ec35e73b5.js"></script>

Next we run some basic pre-processing on the data.

I do the following:

* Check the structure
* Check the datatable dimensions
* Check for missingness
* Run a Summary of each Feature

<script src="https://gist.github.com/mksamelson/63c43236f841d7a27857effadb945143.js"></script>

The structure command provides the following overview of data set shape and features:

<script src="https://gist.github.com/mksamelson/37b811bbb3e5f73e9eabfff23ab9c2ed.js"></script>

The data set has 64 features and 182,343 observations. However, this is slightly misleading.

The dataset consists of three months data in *stacked* format.  Customers that had service for three months have three observations in the set.  Customers who acquired/terminated service during that period have one or two observations.

Since we are using 3 months of data to predict termination in the fourth month and the data is stacked, we actually have *3 times* the number of features.  This is because we have the value for each feature at different points in time:  Termination minus one month; Termination minus two months; and Termination minus three months.

Accordingly, we restructure the data in *wide* format.  We then have one observation for each client with features for each of the three preceding months..

We restructure the data as follows:

* Split the stacked data into individual datatables by month
* Add a month suffix to each feature name: 6 for June, 7 for July, and 8 for August
* Drop month and data columns.  Also customer "user_lifetime" feature from June and
July as August will have the most accurate value
* Merge the monthly datatables together on the "client_id" field.
* Eliminate customers that were not present all three months.

<script src="https://gist.github.com/mksamelson/a6d7a87a42dd8f7b409d93f76e1dbfa1.js"></script>

After revisions, we see the shape of more reflective dataset as 182 features and
66,469 observations.  165 of the features are continuous with the balance binary
categorical.  The number of observations reflects customers who had service for all 3 months, the balance were eliminated during the merge process shown above.

## Exploratory Data Analysis (EDA)

EDA is essential to understand the data set.  However, presentation of summary statistics, distributions, correlations, and other interpretations of data of approximately 180 features does not conveniently lend itself to presentation in the context of a web page.  

I looked at missingness, outliers, distributions, and feature correlations.

There was no missing data in the initial data set.

I elected to use a boosted tree classifier since required data preprocessing and not perform feature selection given my initial objective was to get the best performing model.  I did not remove correlated features as these models handle them in training.  While feature standardization is known not to impact performance significantly, I still
elected to perform it.

Most features exhibit an exponential distribution.  I show on histogram as an example.

## Outliers

I examined feature distributions individually to identify outliers and find
erroneous data.  Only one feature, user_spendings, had any suspicious data a deminimis 27 negative values.  For good form, I eliminated these values as erroneous, substituting an NA for the negative values.

The figures below demonstrate the process through which I generated histograms, reviewed, and modified the data.

<img src="{{site.url}}{{ site.baseurl }}/images/user_spendings_pre.jpeg" alt="this is a placeholder image">  

<script src="https://gist.github.com/mksamelson/ccca5078c2a4f455bfda1ad9e721f681.js"></script>

<img src="{{site.url}}{{ site.baseurl }}/images/user_spendings_post.jpeg" alt="this is a placeholder image">  

## Model

I chose a boosted tree model to predict churn due to it's powerful predictive capability, it's ability to work well with non-normally distributed inputs,
and its ability to easily and powerfully assess feature importance.

The xgbtree method in the R's Caret package makes use of XgBoost - a variation
of a boosted tree ensemble. I used Caret instead of the XgBoost package because of conveniences in data pre-processing, cross validation, and grid search not present in
the XgBoost package itself.

<script src="https://gist.github.com/mksamelson/9073eac9e4ab9f0320d3cb8b44c68876.js"></script>

The best model had an AUC of approximately 0.92 and an error of 0.11 using accuracy.

Accuracy is often not the best metric in classification.  In the case
of an imbalanced data set such as this one in which negative observations (Customers that kept the service in the 4th month) far outnumber positive observations (customers that drop the service in the 4th month), the model with the objective of maximizing accuracy will attempt to minimize *overall* classification error.  

Our objective is slightly different.  We want to maximize true positives. We do this by adjusting the classification threshold value.  This is done by balancing senitivity and specificity.  We do this by selecting the threshold value at the point on ROC Curve where the trade off between obtaining an additional true positive equals that of obtaining an additional false positive.  In our case, that threshold value is 0.04.  This, of course, results in a slightly lower overall accuracy.

<img src="{{site.url}}{{ site.baseurl }}/images/ROC_curve.jpeg" alt="ROC curve">

<img src="{{site.url}}{{ site.baseurl }}/images/cm_optimal.jpeg" alt="">

Training the model is time and resource intensive. The model was trained in an AWS instance with 8G RAM and 30G memory. Run time in that environment was approximately one hour.

Full code can be found in my Github repository here.
