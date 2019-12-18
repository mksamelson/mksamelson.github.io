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

## Model

I chose a boosted tree model to predict churn due to it's powerful predictive capability, it's ability to work well with non-normally distributed inputs,
and its ability to easily and powerfully assess feature importance.

The xgbtree method in the R's Caret package makes use of XgBoost - a variation of a boosted tree ensemble. I used Caret instead of the XgBoost package because of conveniences in data pre-processing, cross validation, and grid search not present in the XgBoost package itself.

I created training and validation sets maintaining class split proportions identfied above.

<script src="https://gist.github.com/mksamelson/9073eac9e4ab9f0320d3cb8b44c68876.js"></script>

I then trained the model.  By default, the latest version of R's Caret package performs a grid search using a range of hyperparameter values and picks the settings.  As indicated above, for simplicity of explanation I use the accuracy metric and attempt to maximize it.  

Using the training data I employ 5 fold cross validation to test performance.

<script src="https://gist.github.com/mksamelson/31042cc7818595589753baa6e9f30640.js"></script>

The best model had an accuracy of 0.9078 and the following hyperparameters:

* min_child_weight = 1
* nrounds = 150
* max_depth = 1
* eta = 0.3
* gamma = 0
* colsample_bytree = 0.8
* min_child_weight = 1
* subsample = 0.75

Meaningful performance is determined by making predictions on the unseen validation set and assessing performance.  

Accuracy on the validation set 0.9065 - surprisingly nearly the same as on the training data.

The confusion matrix displaying the distribution of true positive, true negative, false positive, and false negative predicted values on the validation set is as follows:

<img src="{{site.url}}{{ site.baseurl }}/images/churn/cm_validation_acc.jpeg" alt="this is a placeholder image">  

This may look great but it's not exactly what we want. Maximizing model accuracy unfortunately not satisfactorily address our business problem.  

Maximum accuracy will produce a model that attempts to maximize *combined* accuracy for true positives and true negatives.  

We are more interested in *maximizing true positives and accepting a larger number of false positives*.  Why?  To take corrective action, the business must reach out to customers *before* they drop the service.  Accordingly, the objective is to identify the most properly identified customers (true positives) recognizing that to boost this number we will also falsely identify some customers as intending to drop that won't (false positives).  And that's ok.  Reaching out to the "wrong" customers can also have favorable benefits.  But we don't want the imbalance between true positives and false positives to be extreme.  This requires a judgment call.

So how do we find the "right" combination of true positives and false positives?

The ROC Curve represents the universe of confusion matrices.  Each point on the curve represents the trade off between true positives and false positives for a specific threshold.  The threshold, or "cutoff", sets the probability above which a model classifies as positive and below which the model classifies as negative.  Normally, with a balance data set (approximately equal positives and negatives) the threshold is set to 0.5.  

The area under the ROC Curve (AUC) is also a model performance metric.  As an aside it is worth noting the AUC of our final model is approximately 0.92.  It's a good model.

The ROC Curve is show below:

<img src="{{site.url}}{{ site.baseurl }}/images/churn/ROC_curve.jpeg" alt="ROC curve">

So how is the "best" model determined?  The ROC curve provides the best "universe" of solutions.  We must determine the appropriate "cutoff" - or decision threshold - that gives us the optimal mix of true positives, false positives, and false negatives.  Again, in other words, the "optimal" precision and recall for our purposes.

We find the optimal threshold / point on the ROC curve in this case by balancing sensitivity and specificity.  Stated a bit differently, we want to find the point on the curve where rate-of-change in the sensitivity (true positive rate) equals the rate-of-change in 1 - specificity (false positive rate).  This gives us a "balance" between true positives and false positives.  This point occurs where the threshold value = 0.1204.




However, AUC alone does not help address our optimal solution for the business problem at hand.  The ROC curve summarizes the universe of confusion matrices associated with this model.  A confusion matrix summarizes model performance at a given *theshold* (point on the ROC curve) in terms of true and false positives and negatives.  

I need to find the confusion matrix associated with a point the ROC curve that gives us a "good balance" of true positives and false positives.  


The maximum accuracy the model achieves is approximately 91% at a threshold of .481.

<img src="{{site.url}}{{ site.baseurl }}/images/churn/AccuracyPlot.jpeg" alt="Accuracy Plot">

The confusion matrix at this point shows the following.

<img src="{{site.url}}{{ site.baseurl }}/images/churn/cmmaxaccuracy.jpeg" alt="Accuracy Plot">






<img src="{{site.url}}{{ site.baseurl }}/images/churn/cm_optimal.jpeg" alt="">

## Other Considerations

I mention at the outset that I did not center, scale, or transform the data.  This is because boosted tree models are not particularly sensitive to these transformations and performance was very strong without them.  Furthermore, omitting these steps enabled me to focus more succinctly on explaining the derivation of the optimal model using an accuracy metric.

If I were using a model more sensitive to transformations I would have performed the following:

* Center and Scale.  Centering and scaling data helps eliminate variance caused by data on different scales.  

* Log Transformation.  Nearly all continuous data features show an exponential distribution.  High bias / low variance models such as regression perform better when input data is in the form of a Gaussian (normal) distribution.  Exponential distributions can be changed to normal distributions but apply a log() function.

Full code can be found in my Github repository here.
