---
layout: post
title:  "Airbnb: Boston and Seattle"
date:   2019-04-23 
categories: jekyll update
---

![](/assets/images/airbnb_project_files/boston_seattle.png){: .center-image }

<!-- <img src="/assets/images/airbnb_project_files/boston_seattle.png" alt="drawing" width="3000"/> -->

## Introduction

This is my first (last?) blog post which is actually an assignment for the second term of Udacity's Data Scientist nanodegree. The notebook with the code and all results can be found [here](https://github.com/bvcmartins/dsndProject4/blob/master/airbnb_project.ipynb)


In this post we will explore a dataset, kindly provided to us by Airbnb through Kaggle, which covers all renting activity for one year in Boston and Seattle. The files consist of:

- property listings
- a calendar for each property
- guest reviews

Property listings has quite many features and it is by far the most important data resource. It contains all the information we need to know about the properties with details about ratings and some bookkeeping information. Calendar shows day-by-day if each property was occupied or not. It also shows the price when the property was vacant. These are the only two files that we will use for our analysis. 

The last file of the dataset has the reviews. It basically contains the text of the review and who wrote it. Unfortunately we will not use this file. Reviews are hard to parse because of the very colloquial writing. Another issue is that many of them are written in languages other than English. Moreover, by far and wide, the greatest majority of the reviews are good. That's at least what I figured out after doing some prospective tests. It seems that most of the people only write reviews when they like the place. However, that takes away some of the explanatory power that the reviews might contain.

The post will follow the so-called CRISP-DM process. I actually wasn't really sure if this was really a thing but it looks like it has been around for some time as we can see [here](https://en.wikipedia.org/wiki/Cross_Industry_Standard_Process_for_Data_Mining)

The CRISP-DM process consists of the following steps:

1. Business understanding
2. Data understanding
3. Data preparation
4. Data modelling
5. Results evaluation
6. Deploy

We will not deploy our solution but we still need to take care of all the previous steps. 

Let's start with the Business questions and develop data understanding as we go. The data will also be prepared according to the demands of each question.

## Business Questions

The dataset is quite rich and it was easy to come up with some interesting business questions. Here are the ones that caught my attention:

1. How is property occupation along the year? (easy one for warming up)
2. For how long properties are occupied and how much owners earn?
2. What are the main predictors for property price?
3. What are the characteristics of the busiest properties?
4. What are the most popular neighbourhoods? Can we understand why they are popular?

That's it. Let's solve these questions.

## 1. How occupation changes with time? 

The first question that came to my mind was to check how occupation and earnings change as a function of time. Despite being a pretty easy one, this question can give us some insights about the dataset. 

According to the calendar if a unit is occupied the price is shown as NaN. So, to measure occupation we just need to count the number of NaNs per month for all units. Here is how the counts are distributed over time:

![png](/assets/images/airbnb_project_files/airbnb_project_24_1.png)

For Boston, the occupation is quite steady during spring and summer and lower during winter. That looks like a trend but we can see that the supposed baseline is lower in 2017 compare to 2016. The data timeframe is limited to only one year so we can't be quite sure about this trend. For Seattle, January is popular, probably due to winter birds escaping from the snow further north. We also see some high demand during the peak of summer. The overall occupation in Boston is higher than in Seattle for all months. 

## 2. For how long properties are occupied? 

### By the way, how much owners earn?

Instead of considering an aggregated metric like overall occupation per month, let's check for how long individual units are occupied. To extract this information from Calendar we only need to count for how long each unit has not been available and plot the histogram:

![png](/assets/images/airbnb_project_files/output_28_0.png)

Wow, we found something interesting here. Property occupation in Boston has a peak around 350 days. Interestingly, about 24% of all Boston units are long-term rents. Are these properties rented out to students? Let's check if that is the case.

Here is my strategy to test the hypothesis: let's create a dataframe containing just the units that were rented out for more than 350 days and check the neighbourhoods with more units. Here are the results: 

<div>

<table align="center">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Neighbourhood</th>
      <th>Number of Properties</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Allston</td>
      <td>108</td>
    </tr>
    <tr>
      <th>1</th>
      <td>South End</td>
      <td>80</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jamaica Plain</td>
      <td>79</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Fenway</td>
      <td>69</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Back Bay</td>
      <td>66</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Brighton</td>
      <td>58</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Beacon Hill</td>
      <td>55</td>
    </tr>
    <tr>
      <th>7</th>
      <td>North End</td>
      <td>49</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Mission Hill</td>
      <td>47</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Downtown</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
</div>

That's interesting. Allston is quite close to Harvard and more affordable than Cambridge. South End is close to the Boston University School of Medicine. Despite not conclusive, these results lends some evidence to the hypothesis that these year-round rented units are occupied by students.

Coming back to our original question, property occupation in Seattle is what I would consider as typical: most of the properties are rented out for just about a month over an year. Differently from Boston, just a few properties are in high demand. For both cities most of the properties are occupied for less than 50 days an year. That's the answer to the first part of the question.

However, are these long-term rents a good measure of success? Of course the unit is occupied and making money but that might be by just one tenant for the whole year. It would be interesting to measure how many times the listing was converted in a new renting. 

Let's create a new column containing the average number of times the unit was rented out. To calculate it we just divide the days the unit was occupied by the average time it was available, which is given by (maximum_nights + minimum_nights) / 2.

To finalize, we calculate earning by multiplying occupation by price. The results are shown below:
                 
![png](/assets/images/airbnb_project_files/output_44_0.png)

That's very good. Measuring with number of rents (n_rent) instead of occupation eliminated most of the long-term renting effect. That's important if we want to measure how effective our listing is and how well our property stacks with others in the same category. 

Here are the conclusions for this question:

1. Units are rented out in Seattle for periods shorter than in Boston. In Boston, 24% of the units are rented out for more than 350 days an year. We found some evidence that these units are occupied by students.
2. If we consider the number of times the units are rented, most of them are rented out less than 20 times an year for both cities. In Boston 9 units are rented more than 300 times an year.
3. In Boston most of the units make less than 100k a year while in Seattle most of them make less than 50k a year. Maximum earnings in Boston are around 500k and there is an outlier making more than 1 million (that doesn't seem right). In Seattle, maximum earnings are around 175k and there are two outliers around 300k an year. 

## 3. What are the main predictors for property price?

This one might be a common question for property owners: 

        What are most important property features that I need to take care of if I want to raise the price?

My idea to solve this question is to train a supervised learning method that returns a feature importance list and then take the five first elements. AdaBoost is a good choice for this problem.

Starting with dataframes containing the listings and the occupation, we must (very subjectively) choose the columns that might be the most important for this prediction. This step requires some research and a bit of pondering about which columns were worthy keeping. Here is my list:

```python
# I arbitrarily chose these columns 
list_columns = ['id',
                'host_since',
                'host_response_rate',
                'host_acceptance_rate',
                'host_is_superhost',
                'host_listings_count',
                'neighbourhood_cleansed',
                'property_type',
                'room_type',
                'accommodates',
                'bathrooms',
                'bedrooms',
                'beds',
                'bed_type',
                'amenities',
                'price',
                'minimum_nights',
                'maximum_nights',
                'n_rent',
                'occupied',
                'number_of_reviews',
                'review_scores_rating',
                'review_scores_accuracy',
                'review_scores_cleanliness',
                'review_scores_checkin',
                'review_scores_communication',
                'review_scores_location',
                'review_scores_value',
                'instant_bookable',
                'cancellation_policy',
                'require_guest_profile_picture',
                'require_guest_phone_verification' ]
```

Some of the columns are obvious choices but, for example, 'require_guest_profile_picture' is certainly not a clear one. I chose it because if I am a privacy-oriented guest, I would rather not have my picture available on Airbnb (Maybe not. That's just a guess. Maybe I would not even use Airbnb...).

After cleaning and preprocessing the dataframe, it was straightforwatd to train a regressor. However, after much work, I could only get an R2 score of 0.097 for Boston. Not very promising.

My guess was that the dataset was too sparse, too high-dimensional and with too few instances for any regressor to perform well (don't forget that data comprises only one year). Moreover, if we inspect pairs of variables using scatter plots, we can see that there is no clear trend in the data (and that there are one or two outliers).

![png](/assets/images/airbnb_project_files/airbnb_project_54_0.png)

One solution for this issue is to turn a regression problem into a classification problem. We can define four price categories based on the price quartiles and make predictions about these classes.

Using Ada Boost as the classifier and running 5-fold cross-validation cycles we ended up getting an acceptable result:

    Boston accuracy: 0.55
    Seattle accuracy: 0.57

Here is the feature importance list, which is the answer to our question:

![png](/assets/images/airbnb_project_files/airbnb_project_58_1.png)

So, back to our question, can we determine what are the main predictors for property price? Yes, of course. Here are the most important predictors for prices in Boston:

1. Room is a private room
2. Number of people that can be accommodated in the property
3. Number of amenities
4. If the property is in South End
5. Nummber of bedrooms

For Seattle the result is almost the same:

1. How many people the room can accomodate
2. Room is a private room
3. Number of bedrooms
4. How many days unit is occupied during the year
5. Number of reviews

As we can see the trends are about the same for both cities. The most valuable properties have private rooms, they can accomodate a family, they have more than one bedroom and they are located in central areas. This result makes sense to me.

## 4. What makes a property desirable?

Now, we would like to know what are the characteristics of properties that are in high demand in terms of days occupied. To solve this question we can use the same trick as we used above: define a four-star classification metric based this time on occupation instead of price.

The idea here is to use PCA to reduce dimensionality of the dataset and to aggregate original features into new variables, which are more meaningful for this problem. Then, using k-means clustering, we can find out how these features are aggregated in different classes. Last but not the least, we will determine which classes have more 4-star properties. That solves our question.

Fear not! This is not as complicated as it sounds.

Our PCA model has 20 dimensions, which corresponds to about 80% of explained variance. Keep in mind that the original dataframe had 71 dimensions. With the new PCA features in place, we can use k-means to generate clusters in this new space. We choose 10 as the pre-defined number of clusters.

Now, for the great moment, we check which clusters have the highest population of properties with the maximum score (4). Let's take a look at how the 10 clusters are populated by all properties compared to the percentage of top-score properties: 

![png](/assets/images/airbnb_project_files/airbnb_project_84_0.png)


Wow. The analysis worked very well for Boston and just well for Seattle. In Boston, cluster 9 is clearly overpopulated by top units while, in Seattle, clusters 0 and 1 have an excess of top units. Let's check what are the attributes of these units.

![png](/assets/images/airbnb_project_files/airbnb_project_86_0.png)

#### For Boston
* For cluster 9 the most important attributes are 0, 2 (positive) and 1 (negative)
    * attribute 0 describes units in Jamaica Plain or Dorchester with moderate cancellation policy. Property is a house with private rooms. 
    * attribute 2 describes units that are occupied for most of the time. Property is also a house with private rooms and several amenities. Cancellation policy is strict.
    * attribute 1 describes units in Black Bay or Fenway. They are heavily occupied and the host has several properties. Cancellation policy is moderate.
    
#### For Seattle
* For cluster 0 the most important attributes are 1 (negative) and 2 (positive)
    * attribute 1 describes properties in Belltown and Broadway with moderate cancellation policy and long-time hosts.
    * attribute 2 describes houses with private rooms, many amenities and cancellation policy strict.
    
* For cluster 2 the most important attributes are 0, 1, 3 (positive) and 4 (negative)
    * attribtue 0 describes units with cancellation policy strict, that require guest phone and profile verification. Host acceptance rate and response rate are high. (this feature is a bit too broad).
    * attribute 1 describes houses that have private rooms.
    * attribute 3 describes houses that are constantly occupied. They have several bedrooms and can accomodate many people. They have cancellation policy moderate.
    * attribute 4 describes houses that are instantly_bookable with long-time hosts. They have high response rate and high acceptance rate.
    
Yes, we made it. These are the characteristics of the properties that have been in hghest demand in both cities. They share several features in common like being a house, with many bedrooms and private rooms. Properties in central locations are also popular. 

I was actually not absolutely sure if this analysis was correct. The combination PCA - K-means should be applied to a problem with no labeling or with labels covering only a fraction of the total dataset. That is how unsupervised or semi-supervised learning work. In our case all instances had a label (4-star rate) so we could have just ran AdaBoost and collected the most important features. On the other hand, the labels were not actually used for training, so this is still unsupervised learning. The labels were just used later to identify the best clusters. Moreover, PCA - K-means gives a very fine-grained information about different facets of the problem. In our case it detected the importance of some neighbourhoods.  


## 5. What are the most popular neighbourhoods? 
### Can we understand why they are popular?

To solve this last question we can use the neighborhood overview field from the listings dataframe. The interesting thing about overviews is that they tend to be more well-behaved than the wild west of the reviews. The host will take more time to tailor the language which turns text easier to be processed. Moreover, from what I have checked, all overviews are in English.

The idea here is to aggregate reviews per neighborhood, turn them into a bag of words and process them using [Latent Dirichlet Allocation](http://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf), a Bayesian tool for analysis of trends in texts. 

Because the results of this analysis are quite lenghty I will do it only for Boston.

Before we start, a fun fact about the listings dataframe is that almost 40% of the overviews are missing (come on, how can someone post a property and not write an overview?). 

The analysis is quite straightforward. We group the entries by neighborhood, process the text using vectorization and perform the LDA analysis. Finally we sort the neighborhoods by average rent score which is just one more incarnation of our trick of converting the number of rents per unit into a 1-4 score based on quartiles. This score higher for properties that are rented out more times. Here are the top 5 neighbourhoods:

<div>
<table align='center'>
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>neighbourhood</th>
      <th>message</th>
      <th>rent score</th>
     </tr>
  </thead>
  <tbody>
    <tr>
      <th>16</th>
      <td>Mission Hill</td>
      <td>fine arts longwood medical museum fine minute walk mission hill medical area northeastern university</td>
      <td>2.919355</td>
     </tr>
    <tr>
      <th>0</th>
      <td>Allston</td>
      <td>harvard square min walk charles river students young walk away young professionals restaurants bars</td>
      <td>2.819231</td>
     </tr>
    <tr>
      <th>21</th>
      <td>South Boston Waterfront</td>
      <td>walking distance seaport district section city district hottest fort point harpoon brewery lots businesses</td>
      <td>2.734940</td>
     </tr>
    <tr>
      <th>22</th>
      <td>South End</td>
      <td>south end copley square walking distance newbury street end bay minute walk neighborhoods south</td>
      <td>2.708589</td>
     </tr>
    <tr>
      <th>3</th>
      <td>Beacon Hill</td>
      <td>beacon hill charles street charles river state house hill historic antique shops brick sidewalks</td>
      <td>2.675258</td>
     </tr>
    
  </tbody>
</table>
</div>

And here is how we can interpret the results:

1. Mission Hill: It's at walking distance from the fine arts museum, it has the Longwood medical center and it is close to Harvard university
2. Allston - close to Harvard, within minutes walk from Charles river. Many young professionals leave there and it has several restaurants and bars
3. South Boston Waterfront - it is at walking distance from the seaport district. It is one of the hottest districts in the city and it is the location of the Harpoon brewery. There are lots of businesses.
4. South End - Copley square is at walking distance. It is crossed by Newbury street which has many restaurants and retails. 
5. Beacon Hill - Close to Charles street and Charles river. It is an historic neighborhood with brick sidewalls and antique shops.

That was interesting. It looks like we managed to get some meaningful information out of plain text. We used the rent score to determine the most popular neighbourhoods and the text to understand why these places are popular.

## Conclusions

Using data manipulation and machine learning we managed to generate quite interesting insights from a particularly challenging dataset. Here is what we learned:

* demand is high during spring and summer. For all months Boston is in higher demand than Seattle
* 25% of the properties in Boston are rented all year round. These properties are probably mostly rented to students
* price was predicted with around 60% accuracy. The most important features for price determination that are common to both cities are:
    * rooms are private
    * number of people that can be accommodated
    * number of bedrooms
* the most sought-after properties can be grouped by common fetures. Some of the most common features were:
    * rooms are private
    * number of bedrooms
    * cancellation policy is moderate
    * hosts have high reputation
* for Boston, the hottest neighbourhoods were Mission Hill, Allston, South Boston waterfron, South End, Beacon Hill. These places were in general close to universities, historic neighbourhoods or street shopping.

We covered a lot of ground but we could have done much more with this dataset. One question that I would like to tackle is if the reviews are consistent with the ratings for each property. It also would be interesting to extend this analysis to a longer timeframe so we could get better accuracy for the price predictions. Maybe some other time.
