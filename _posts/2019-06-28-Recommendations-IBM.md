---
layout: post
title: "Recommendations with IBM"
date: 2019-06-28
categories: jekyll update
---

## Recommendations with IBM

### Project 6 of Udacity Data Scientist Nanodegree

[Notebook link](https://github.com/bvcmartins/recommendations_IBM/blob/master/recommendations.ipynb)

#### Overview

In this project we developed a Recommendation System using
a article readership dataset kindly provided by IBM. The
dataset was compiled from Watson Studio user base 
and it contains information about Data Science articles to
which readers these articles were suggested. 

The objectives of this project were: 

1. Clean and preprocess dataset using NLP and Matrix
Decomposition techiniques
2. Calculate different similarity measures between users 
and assess their relevance to the problem
3. Implement a Rank-based recommendation system
4. Implement a User-User based collaborative filtering system
5. Develop a Content-based recommendation system for article suggestion automation

#### Methods

This project heavily relied on Pandas tools for dataframe
processing. Many of the solutions were based on the
Functional Programming paradigm. We also implemented 
plain SVD and Funk SVD decomposition algorithms. We tested
both using train and test datasets for recommendation
accuracy assessment.

#### Conclusions

- We showed that a mixed approach using Rank-based,
User-User based and Content-based recommendation system
can be very effective to cover cases of both experienced
and new users
- We showed that for cases without missing entries
plain SVD and Funk SVD have similar performances. We
also found that at least 300 latent features are necessary
to account for class imbalance effects.



