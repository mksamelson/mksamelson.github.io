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

This article is presented in a way to make it easier for less technical individuals to understand.  I train and discuss the model in terms of accuracy with is an easier concept to understand than the Receiver Operator Curve.  While I reference the ROC curve, it is done in a tangential way that makes theory explanation largely unnecessary.

All work is done in R.

## The Data

The data is contained in a single file with multiple sheets.  One sheet contains features. These include but are not limited to customer spending, calls, SMS use, and data use. A second sheet contains a binary churn indicator indicating whether a particular customer dropped or maintained service in September 2013. A third page has variable definitions in plain English.

### Independent variables

Loading and munging is fairly straightforward.  Fortunately, this data set is very clean.

In short we do the following:

* Load data from csv files.

* Identify categorical variables.  This is done by manual review of data.  

* Identify continuous features.  Manual review of data indicates some continuous features are encoded as integers while others are encodes as floats.  

* Convert dataframe to datatable for efficiency purposes.

The script snippet below shows the code for the process.

<script src="https://gist.github.com/mksamelson/8367db04f5a0aa6ade6d374ec35e73b5.js"></script>

I then perform some basic pre-processing on the data.

* Check the structure
* Check the data dimensions
* Check for missingness
* Run a Summary of each Feature

<script src="https://gist.github.com/mksamelson/63c43236f841d7a27857effadb945143.js"></script>

The structure command provides the following overview of data set shape and features:

<script src="https://gist.github.com/mksamelson/37b811bbb3e5f73e9eabfff23ab9c2ed.js"></script>

The data set has 64 features and 182,343 observations. However, this is slightly misleading.

The dataset consists of three months data in *stacked* format.  Customers that had service for three months have three observations in the set.  Customers who acquired/terminated service during that period have one or two observations.

Since I am using 3 months of data to predict termination in the fourth month and the data is stacked, there is actually *3 times* the number of features.  This is because I have a value for each feature at different points in time:  Termination minus one month; Termination minus two months; and Termination minus three months.

Accordingly, I restructure the data in *wide* format.  I then have one observation for each client with features for each of the three preceding months.

I restructure the data as follows:

* Split the stacked data into individual datatables by month
* Add a month suffix to each feature name: 6 for June, 7 for July, and 8 for August
* Drop month and data columns.  Also customer "user_lifetime" feature from June and
July as August will have the most accurate value
* Merge the monthly datatables together on the "client_id" field.
* Eliminate customers that were not present all three months.

<script src="https://gist.github.com/mksamelson/a6d7a87a42dd8f7b409d93f76e1dbfa1.js"></script>

After revisions, we see the shape of more reflective dataset as 182 features and 57,656 observations.  165 of the features are continuous with the balance binary
categorical.  The number of observations reflects customers who had service for all 3 months, the balance were eliminated during the merge process shown above.

### Dependent Variable (Target)

The dependent variable is a binary classifier.  It has a value of 0 (service maintained) or 1 (service dropped).

The proportion of customers that dropped service vs. those who kept service is as follows:

<script src="https://gist.github.com/mksamelson/d523bd36f015484dacaec0c528ff236a.js"></script>

The results indicate about 20% of customers drop in the fourth month.  Accordingly we can consider this data set skewed.

## Exploratory Data Analysis (EDA)

EDA is essential to understand the data set.  However, presentation of summary statistics, distributions, correlations, and other interpretations of data of approximately 180 features does not conveniently lend itself to presentation in the context of a web page.  

I looked at missingness, outliers, distributions, and feature correlations.

There was no missing data in the initial data set.

I elected to use a boosted tree classifier since required data preprocessing and not perform feature selection given my initial objective was to get the best performing model.  I did not remove correlated features as these models handle them in training.  I also did not center and scale data for the same reason.

Most features exhibit an exponential distribution.  Historgrams for one data element for each of the three months for reloads_count are shown below:

<img src="{{site.url}}{{ site.baseurl }}/images/churn/reloads_count_june.png" alt="this is a placeholder image">  

<img src="{{site.url}}{{ site.baseurl }}/images/churn/reloads_count_july.png" alt="this is a placeholder image">  

<img src="{{site.url}}{{ site.baseurl }}/images/churn/reloads_count_august.png" alt="this is a placeholder image">  

## Erroneous Data

I examined feature distributions individually to identify outliers and find erroneous data.  A few features, such has user_spendings, had deminimis  negative values.  For good form, I eliminated these values as erroneous, substituting 0 for the negative values.

The figures below demonstrate the process through which I generated histograms, reviewed, and modified the data.

<img src="{{site.url}}{{ site.baseurl }}/images/user_spendings_pre.jpeg" alt="this is a placeholder image">  

<script src="https://gist.github.com/mksamelson/ccca5078c2a4f455bfda1ad9e721f681.js"></script>

<img src="{{site.url}}{{ site.baseurl }}/images/user_spendings_post.jpeg" alt="this is a placeholder image">  

## Model

I chose a boosted tree model to predict churn due to it's powerful predictive capability, it's ability to work well with non-normally distributed inputs,
and its ability to easily and powerfully assess feature importance.

The xgbtree method in the R's Caret package makes use of XgBoost - a variation of a boosted tree ensemble. I used Caret instead of the XgBoost package because of conveniences in data pre-processing, cross validation, and grid search not present in the XgBoost package itself.

<script src="https://gist.github.com/mksamelson/9073eac9e4ab9f0320d3cb8b44c68876.js"></script>

The best model had an AUC of approximately 0.92 for the training data and 0.91 against validation data.

<img src="{{site.url}}{{ site.baseurl }}/images/churn/ROC_curve.jpeg" alt="ROC curve">

However, AUC alone does not help address our optimal solution for the business problem at hand.  The ROC curve summarizes the universe of confusion matrices associated with this model.  A confusion matrix summarizes model performance at a given *theshold* (point on the ROC curve) in terms of true and false positives and negatives.  

I need to find the confusion matrix associated with a point the ROC curve that gives us a "good balance" of true positives and false positives.  

Maximizing *overall* model accuracy is unfortunately not the answer.  A threshold value that provides overall classification accuracy will attempt to maximize *combined* true positives and true negatives.  The data is highly skewed in that the number of customers that drop the service is small in comparison to those that don't.  In attempting to maximize accuracy the model tends to correctly classify negatives in which we are not interested.    

The maximum accuracy the model achieves is approximately 91% at a threshold of .481.

<img src="{{site.url}}{{ site.baseurl }}/images/churn/AccuracyPlot.jpeg" alt="Accuracy Plot">

The confusion matrix at this point shows the following.

<img src="{{site.url}}{{ site.baseurl }}/images/churn/cmmaxaccuracy.jpeg" alt="Accuracy Plot">

Accuracy is often not the best metric for classification.  This is because the model will attempt to maximize *overall* accuracy.

I am not interested in overall accuracy (maximizing true positives + true negatives).  The business objective is to identify customers predicted to drop the service (true positives).  We want to maximize true positives *subject to* an "acceptable" number of false positives and false negatives.  In other words, we're ok with a model that has some predictive error if it gets us incrementally more true positives than we get using overall accuracy so long as the error (false positives and false negatives) are not "overwhelming".  This is a subjective decision supported by analytics.

So how do I find the "best" model?  We have already turned the model's hyperparameters.  This gives us the best "universe" of solutions.  We must determine the appropriate "cutoff" - or decision threshold - that gives us the optimal mix of true positives, false positives, and false negatives.  Again, in other words, the "optimal" precision and recall for our purposes.

We find the optimal threshold / point on the ROC curve in this case by balancing sensitivity and specificity.  Stated a bit differently, we want to find the point on the curve where rate-of-change in the sensitivity (true positive rate) equals the rate-of-change in 1 - specificity (false positive rate).  This gives us a "balance" between true positives and false positives.  This point occurs where the threshold value = 0.04.

<img src="{{site.url}}{{ site.baseurl }}/images/churn/cm_optimal.jpeg" alt="">

## Other Considerations

I mention at the outset that I did not center, scale, or transform the data.  This is because boosted tree models are not particularly sensitive to these transformations and performance was very strong without them.  Furthermore, omitting these steps enabled me to focus more succinctly on explaining the derivation of the optimal model using an accuracy metric.

If I were using a model more sensitive to transformations I would have performed the following:

* Center and Scale.  Centering and scaling data helps eliminate variance caused by data on different scales.  

* Log Transformation.  Nearly all continuous data features show an exponential distribution.  High bias / low variance models such as regression perform better when input data is in the form of a Gaussian (normal) distribution.  Exponential distributions can be changed to normal distributions but apply a log() function.

Full code can be found in my Github repository here.
