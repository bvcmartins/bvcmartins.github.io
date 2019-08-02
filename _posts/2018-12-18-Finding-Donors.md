---
layout: post
title "DSND project 1: Finding Donors"
date: 2018-12-18
categories: jekyll update
---


## Project 1 for Udacity's Data Scientist Nanodegree

[Link to Notebook](https://github.com/bvcmartins/dsndProject1/blob/master/finding_donors.ipynb)

### Overview

Income determination from publicly available data 
is a powerful tool 
for customer base identification. In this project we 
employed several supervised learning algorithms to infer 
individual income using the 1994 census data sample 
contained in the Adults dataset 
[UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Census+Income) 

The objectives of this project were:

1. Build a model capable of predicting whether an 
individual makes more or less than $50,000 annually
2. Clean and preprocess the dataset and analyze it using 
a number of supervised learning models to choose 
which one led to higher prediction scores
3. Optimize the hyperparameters of the model
4. Perform feature importance analysis to determine 
which features are the most relevant for income 
prediction.  

The notebook contains many more features than required
by Udacity's project. The original file submitted and
aproved by Udacity
is in notebook finding_donors_udacity.ipynb.

### Methods

We used supervised learning methods from Scikit-learn
for the analysis. AdaBoost was the best performing model.
The optimization phase used the following techinques:

- Random Search
- Grid Search
- Model calibration

### Conclusions

- AdaBoost was the best model. We achieved F1 = 0.78
with testing set
- Logistic regression also performed well with a fraction
of training time. It performed well with smaller training
sets
- We determined that age, hours-per-week, education years,
capital-loss and capital-gain were the most important
features for income determination


