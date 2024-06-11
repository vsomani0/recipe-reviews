# Recipe Reviews Data Analysis
This is an in-progress prediction and an open-ended investigation into the recipes dataset that I'm currently working on for the DSC 80 Final Project at UCSD.

## Introduction

This report is based on food.com, a website where home cooks share recipes that they came up with. Reviews are becoming a more and more important part of today's world, where items from movies to restaurants to almost any product are getting reviewed. In this project, I will analyze these reviews in more depth to see whether later reviews are significantly different (i.e more positive or negative) than earlier reviews, in order to help us get a better feel for reviewer behavior. Then, I will see how well we can use this better information about reviews to see if I can predict a recipe's average rating, with only using knowledge beforehand (such as the creator of a recipe, its ingredients etc.)

This dataset is split up into two separate pieces of data -- recipes and reviews. First of all, we have the recipes dataframe.

This dataframe has one element for every rating since 2008.  In total, it has 83782 rows (representing 83782 recipes) by 5 columns (shown above).

| Column Name | Type |
| ----------- | ----------- |
| 'name'| Recipe name |
| 'id' |	Recipe ID |
| 'minutes' |	Minutes to prepare recipe |
| 'contributor_id' |	User ID who submitted this recipe |
| 'submitted' |	Date recipe was submitted |
| 'tags' |	Food.com tags for recipe |
| 'nutrition' | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| 'n_steps' |	Number of steps in recipe |
| 'steps'	| Text for recipe steps, in order |
| 'description' |	User-provided description |


The reviews dataframe has one element for every review since 2008.  In total, it has 731929 rows (representing 731929 reviews) by 5 columns shown below.

| Column |	Description |
| ------- | -------- |
| 'user_id'	| User ID |
'recipe_id'	| Recipe ID |
'date' |	Date of interaction |
'rating' |	Rating given |
'review' |	Review text |




## Data Cleaning and Exploratory Data Analysis

The first step to almost any data science project is to clean the data and get a feel for the numbers. My first step in doing so was going through every column individually, and ensuring that the values seemed right. 

1) Fill 0 ratings with NA Values 

In the rating column, I saw that our reviews without any ratings were stored as zeroes. Ignoring this might lead to several issues, for instance in us underestimating the average review score of a recipe. 

2) Convert Strings to Lists 

Several columns, including nutrition, ingredients, and steps are stored as strings. Converting them to a list makes accessing each individual element much easier. 

3) Convert submitted from a string to a datetime 

This is really helpful for more convenient timestamp comparisons.

4) Left Merge recipes and reviews by recipe_id 

This allows us to see a review by its corresponding recipe, and helps us convert the two datasets into one. A left merge keeps all the recipes, even if they don't have a corresponding review.

5) Add Column for review time after recipe submitted 

The review time after recipe submitted is a crucial element for the hypothesis test, so it's really helpful. 

Note: In extremely rare cases, a recipe was submitted after a review of it according to this dataset -- which should not be possible. I checked food.com, and the same issues held. For fairness, I let these values remain as is.

6) Created a column for average rating by recipe \
Helpful piece of information for the prediction later.

Next, I explored the data, and found these 3 interesting images.

<iframe
  src="assets/number_of_reviews_by_year.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This shows the popularity of food.com reviews -- but it's important to note that this is not all of them, just the ones that were scraped. Still, it's a good indicator to show that food.com is no longer as popular as it once was. 

<iframe
  src="assets/review_average_by_time_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This is the review average by time. It stays extremely stable, but later reviews are pronouncedly more negative. This effect becomes more apparent when zooming into the plot (for instance by setting its axes from 4 to 5).

![heaven-aggregate](heaven-agg-screenshot.jpg)



## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis





