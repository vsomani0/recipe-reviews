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

### Exploratory Data Analysis

I found several interesting visuals, including these ones:

<iframe
  src="assets/number_of_reviews_by_year.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This shows the popularity of food.com reviews since 2008, and these peaked at around 2009. It's still a popular site to write reviews on, but nowhere near as popular as it once was.

<iframe
  src="assets/review_average_by_time_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This is the review average by time. It stays extremely stable, but later reviews are pronouncedly more negative. This effect becomes more apparent when zooming into the plot (for instance by setting its axes from 4 to 5). 

| review                         |   ('rating', 'mean') |   ('rating', 'count') |
|:-------------------------------|---------------------:|----------------------:|
| Review does not contain heaven |              4.67888 |                218521 |
| Review contains "heaven"       |              4.9411  |                   815 |

Here's an example of some of the review searches that we can do. Clearly and unsurprisingly, reviews with 'heaven' in them are far more positive on average than reviews without heaven.

## Assessment of Missingness

### NMAR Analysis
Not Missing at Random (NMAR) or Missing Not at Random (MNAR) means that a column's missing values are not random and depending not on any column in the dataframe, but on the value itself. These are the trickiest values to impute. In this dataset, people who are busy are less likely to write a review. Busier people would likely prefer some kinds of recipes, such as ones with low effort preparation or ones where they can package a lot of recipes together. We can somewhat identify this by using the minutes column, but not fully -- so this is NMAR. If we had a column for minutes of active cooking (rather than simply passive cooking), then it would be MAR since we could predict it based on a column.

### Permutation Testing for MAR

We use permutation testing to determine if a column is MAR (missing at random), which counter to its name, implies that a column's missingness depends on other columns. Typically, we use permutation tests to determine whether a column is MAR or not.

First, let's see if number of steps helps determines whether the column is missing. In order to do so, we run a permutation test. Let's see a histogram.

<iframe
  src="assets/review_average_by_time_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

These distributions look a little different -- specifically rating missing seems to have slightly higher values for minutes, especially in the extreme. The K-S test statistic, which computes the largest difference within the CDFs of the two distributions is my test statistic to see if the distributions came from the same population. Its worth noting that the K-S statistic does not tell us if one is greater than others -- just whether the distribution comes from the same population.

Let's run a permutation test with these settings
Alpha: 0.01 

Null Hypothesis: Distribution of number of steps for missing ratings is the same as distribution of number of steps for non-missing ratings. 

Alternate Hypothesis: Distribution of number of steps for missing ratings is different than distribution of number of steps for non-missing ratings.

After running a ks statistic, we get a p-value of 10^(-49), which is extremely low. We reject the null hypothesis for the alternate hypothesis. 

Let's also see if the missingness of ratings depends on the minutes column. The minutes dataframe is self-reported, so there are lots of outlier values. I filtered out everything above 50 minutes to make it easier to visualize

<iframe
  src="assets/review_missing_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Seems like columns with rating missing have higher minutes on average. Let's try another permutation test, with absolute difference in means as our test statistic this time.

Alpha: 0.01 

Null Hypothesis: Distribution of number of minutes for missing ratings is the same as distribution of number of steps for non-missing ratings. 

Alternate Hypothesis: Mean of number of minutes for missing ratings is different than distribution of number of steps for non-missing ratings.

We get a p-value of 0.12, which is lower than the threshold. Thus, we fail to reject the null hypothesis, and conclude that minutes may not affect the missingness of ratings. Even though we did not conclude that rating is missing based on chance, it still is considered MAR, because it is dependent on the number of steps.

## Hypothesis Testing

Building off the plot earlier when exploring the data, lets test whether the days after a recipe is submitted affects the actual score of the review.
<iframe
  src="assets/review_average_by_time_plot.html"
  width="400"
  height="300"
  frameborder="0"
></iframe>

Let's test whether the reviews after a certain point of the recipe being submitted have significantly lower mean ratings. Here, let's use a cutoff of 3 years after the recipe submission to see if there's a difference, based off the graph made previously. I have 'sliced up' this data by using the data to generate the hypothesis. To counteract this, I will require more evidence in order to reject the null hypothesis. 

Alpha: 0.001 

Null Hypothesis: Reviews submitted over 3 years after the recipe submitted have the same mean as all reviews 

Alternate Hypothesis: Reviews submitted over 3 years after a recipe submitted do not have the same mean as all reviews

After simulating 10000 samples, we see these values

<iframe
  src="assets/hyp_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Clearly, the value we observed is far greater than what we see by chance alone. Clearly, the p-value gives us a p-value of 0, indicating that none of the 10000 simulated statistics was as or more extreme than the observed value. Therefore, we reject the null hypothesis in favor of the alternate. Thus, we conclude that later reviews likely have a different true mean than the alternate hypothesis. Again, it is theoretically possible that this value did arise by chance alone.

## Framing a Prediction Problem
Let's dive deeper into analyzing review behavior, this time trying to see if we can predict the average review rating based on the recipe with at least 8 reviews via regression.

The average rating can be valuable to estimate. It can, for instance, help an algorithm choose which products will be better liked and recommend those products more.

This dataset is littered with products with very few reviews. If there are only one or two reviews on a product, it becames less valuable to estimate the average rating since it conveys only one consumers' estimate. It also becomes harder to estimate -- even an excellent model will struggle to estimate the average rating of a product with only one sample size. For instance, with a low sample size a food item may be really good, but the one reviewer happened to dislike it. Therefore, to make the predictions both more informative and reward a good model more, let me set the cutoff to 8 reviews. This still leaves us with a nice sample size of about 3500 recipes.

I ensure that only variables known at the time of a recipe's submission can be used. Variables like the text of a review or later recipes by the creator of the current recipe are not known when a recipe is submitted, so a potential algorithm could not have access to these until after the recipe is submitted. In order to measure score, I will use use primarily R^2. I use this instead of something like mean absolute error in order to punish getting a prediction badly wrong substantially. I also edit the dataframe (changing its granularity) so that there's one datapoint per recipe (not per review as before), and only recipes with 8 or more reviews are included. 

## Baseline Model

For the baseline model, I rely heavily on intuition rather than testing a number of different combinations out in order to give a general gist of how good the model can be. More complicated feature creation, hyperparameter tuning, etc. will be done in the final model. Our biggest asset in prediction is the text that we have for each recipe -- in its ingredients information, recipe title, and recipe description. We will use the bag of words encoding (which can be effective for small strings like ingredients). It can give really large amounts of features with a lot of unique words, which can potentially cause overfitting. There are other ways to reduce the number of features, I won't use the description column yet.

In sklearn, the CountVectorizer class makes word comparison a lot easier for us. It automatically ensures all data is lowercase, and splits by punctuation. I will store these all as 1-grams (to just store the words) for the baseline. These are my features:

| Feature Name | Type
|:-------------------------------|---------------------:|
| Words in Title | Categorical Nominal
| Words in Ingredients | Categorical Nominal
| Days After 2008 | Quantitative Discrete |
| Minutes of Cooking Time| Quantitative Discrete |

Note that times are typically continuous -- but these are stored as discrete variables in this case. The scraper only finds days, so Days After 2008 is technically discrete. Additionally, food.com only asks users to specify minutes without decimals, so it is also discrete.

In this case, a tree-based model would be nice since the heavy amount of text data makes having interaction terms (or a combination of words) more important. Tree-based models are non-parametric, and can therefore, fit this really well. I will use the random forest regressor to counteract the overfitting of having too many columns.

I divide the model into a test and a training set (with 80% of the data in the training set). After fitting the model, I have the opportunity to do some simple hyperparameter tuning of the model. For the baseline, I will only tune the max_depth, and compute the cross-validation error. Here are the R^2 values:

|   Max Depth |   Training R^2 |   Validation R^2 |
|------------:|---------------:|-----------------:|
|           3 |         0.1156 |           0.006  |
|           5 |         0.1649 |           0.0094 |
|           7 |         0.2059 |           0.013  |
|          10 |         0.2604 |           0.0189 |
|          13 |         0.3093 |           0.0238 |
|          15 |         0.3385 |           0.0266 |
|          20 |         0.4001 |           0.031  |

<iframe
  src="assets/validation_r2_baseline.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Clearly, a depth of 20 gives the best validation error, so I end up using it. The model is overfitting substantially. However, interestingly, it has a test R^2 of 0.106, which is actually substantially better than the cross-validation error. This is likely because we have more data available to us (since all of the training data is used on every iteration) -- which is particularly useful in high-variance cases like this one. In the final model, I explore how to reduce this overfitting

## Final Model

## Fairness Analysis





