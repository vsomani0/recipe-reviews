# Recipe Reviews Data Analysis
This is an investigation into understanding and predicting reviews of food recipes. It is a project for DSC 80 (Applications of Data Science). 

## Introduction

 Reviews are becoming a more and more important part of today's world, where items from movies to restaurants to almost any product are getting reviewed. I really wanted to do something from analyzing reviews, to examine how the behavior changes by time. A research paper has scraped hundreds of thousands of reviews on food.com since 2008, and that is perfect for this. 

Overall, I analyze reviews in depth whether later reviews are significantly more positive or negative than earlier reviews, in order to get a better feel for reviewer behavior.  Then, I will see how well we can use this better information about reviews to see if I can predict a recipe's average rating, with only using knowledge beforehand (such as the creator of a recipe, its ingredients etc).

This dataset is split up into two separate pieces of data -- recipes and reviews. First of all, we have the recipes dataframe, which has one element for every rating since 2008.  In total, it has 83782 rows, representing 83782 recipes, by 5 columns (shown below).

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


The reviews dataframe has one element for every review since 2008.  In total, it has 731929 rows, representing 731929 reviews, by 5 columns shown below.

| Column |	Description |
| ------- | -------- |
| 'user_id'	| User ID |
| 'recipe_id'	| Recipe ID |
| 'date' |	Date of interaction |
| 'rating' |	Rating given |
| 'review' |	Review text |

## Data Cleaning and Exploratory Data Analysis

The first step to almost any data science project is to clean the data and get a feel for the numbers. My first step in doing so was going through every column individually, and ensuring that the values seemed right. 

1) **Fill 0 ratings with NA Values** 

In the rating column, I saw that our reviews without any ratings were stored as zeroes. Ignoring this might lead to several issues, for instance in us underestimating the average review score of a recipe. 

2) **Convert Strings to Lists**

Several columns, including nutrition, ingredients, and steps are stored as strings. Converting them to a list makes accessing each individual element much easier. 

3) **Convert submitted from a string to a datetime**

This is really helpful for more convenient timestamp comparisons.

4) **Left Merge recipes and reviews by recipe id**

This allows us to see a review by its corresponding recipe, and helps us convert the two datasets into one. A left merge keeps all the recipes, even if they don't have a corresponding review.

5) **Add Column for review time after recipe submitted** 

The review time after recipe submitted is a crucial element for the hypothesis test, so it's really helpful. 

Note: In extremely rare cases, a recipe was submitted after a review of it according to this dataset -- which should not be possible. I checked food.com, and the same issues held. For fairness, I let these values remain as is.

6) **Created a column for average rating by recipe** 
Extremely helpful piece of information for the prediction later.

This is how the cleaned dataframe looks:
<iframe
  src="assets/df-head.png"
  width="550"
  height="108"
  frameborder="0"
></iframe>

### Exploratory Data Analysis

I found several interesting visuals, including these ones:

<iframe
  src="assets/number_of_reviews_by_year.html"
  width="550"
  height="400"
  frameborder="0"
></iframe>

This shows the popularity of food.com reviews since 2008, and these peaked at around 2009. It's still a popular site to write reviews on, but nowhere near as popular as it once was.

<iframe
  src="assets/review_avg_by_time_plot.html"
  width="550"
  height="400"
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
  src="assets/n_steps_missing.html"
  width="550"
  height="400"
  frameborder="0"
></iframe>

These distributions look a little different -- specifically rating missing seems to have slightly higher values for minutes, especially in the extreme. The K-S test statistic, which computes the largest difference within the CDFs of the two distributions is my test statistic to see if the distributions came from the same population. Its worth noting that the K-S statistic does not tell us if one is greater than others -- just whether the distribution comes from the same population.

Let's run a permutation test with these settings
**Alpha**: 0.01 

**Null Hypothesis**: Distribution of number of steps for missing ratings is the same as distribution of number of steps for non-missing ratings. 

**Alternate Hypothesis**: Distribution of number of steps for missing ratings is different than distribution of number of steps for non-missing ratings.

After running a ks statistic, we get a p-value of 10^(-49), which is extremely low. We reject the null hypothesis for the alternate hypothesis. 

Let's also see if the missingness of ratings depends on the minutes column. The minutes dataframe is self-reported, so there are lots of outlier values. I filtered out everything above 50 minutes to make it easier to visualize

<iframe
  src="assets/review_missing_rating.html"
  width="550"
  height="400"
  frameborder="0"
></iframe>

Seems like columns with rating missing have higher minutes on average. Let's try another permutation test, with absolute difference in means as our test statistic this time.

**Alpha**: 0.01 

**Null Hypothesis**: Distribution of number of minutes for missing ratings is the same as distribution of number of steps for non-missing ratings. 

**Alternate Hypothesis**: Mean of number of minutes for missing ratings is different than distribution of number of steps for non-missing ratings.

We get a p-value of 0.12, which is lower than the threshold. Thus, we fail to reject the null hypothesis, and conclude that minutes may not affect the missingness of ratings. Even though we did not conclude that rating is missing based on chance, it still is considered MAR, because it is dependent on the number of steps.

## Hypothesis Testing

Building off the plot earlier when exploring the data, lets test whether the days after a recipe is submitted affects the actual score of the review.
<iframe
  src="assets/review_avg_by_time_plot.html"
  width="550"
  height="400"
  frameborder="0"
></iframe>

Let's test whether the reviews after a certain point of the recipe being submitted have significantly lower mean ratings. Here, let's use a cutoff of 3 years after the recipe submission to see if there's a difference, based off the graph made previously. I have 'sliced up' this data by using the data to generate the hypothesis. To counteract this, I will require more evidence in order to reject the null hypothesis. 

**Alpha**: 0.001 

**Null Hypothesis**: Reviews submitted over 3 years after the recipe submitted have the same mean as all reviews 

**Alternate Hypothesis**: Reviews submitted over 3 years after a recipe submitted do not have the same mean as all reviews

After simulating 10000 samples, we see these values

<iframe
  src="assets/hyp_test.html"
  width="550"
  height="400"
  frameborder="0"
></iframe>

Clearly, the value we observed is far greater than what we see by chance alone. The simulated statistics are all on the left and the observed statistic is much larger. It should no surprise that we get a p-value of 0 when running the test, indicating that none of the 10000 simulated statistics was as or more extreme than the observed value. Therefore, we reject the null hypothesis in favor of the alternate. Thus, we conclude that later reviews likely have a different true mean than the alternate hypothesis. Again, it is theoretically possible that this value did arise by chance alone.

## Framing a Prediction Problem
Let's dive deeper into analyzing review behavior, this time trying to see if we can predict the average review rating based on the recipe with at least 8 reviews via regression. The average review rating is potentially misleading, since review ratings are ordinal variables -- and I treat them as quantitative. However, it is the most reasonable way to aggregate review ratings, so I will use it.

The average rating is invaluable to estimate. It can, for instance, greatly help an algorithm choose which products will be better liked and recommend those products more. However, it will likely pose a big challenge since predicting the quality of a recipe based largely on the text of its review and its title seems very difficult.

This dataset is littered with products with very few reviews. If there are only one or two reviews on a product, it becames less valuable to estimate the average rating since it conveys only one consumers' estimate. It also becomes harder to estimate -- even an excellent model will struggle to estimate the average rating of a product with only one sample size. For instance, with a low sample size a food item may be really good, but the one reviewer happened to dislike it. Therefore, to make the predictions both more informative and reward a good model more, let me set the cutoff to 8 reviews. This still leaves us with a nice sample size of about 3500 recipes.

I ensure that only variables known at the time of a recipe's submission can be used. Variables like the text of a review or later recipes by the creator of the current recipe are not known when a recipe is submitted, so a potential algorithm could not have access to these until after the recipe is submitted. In order to measure score, I will use use primarily R^2. I use this instead of something like mean absolute error in order to punish getting a prediction badly wrong substantially. I also edit the dataframe (changing its granularity) so that there's one datapoint per recipe (not per review as before), and only recipes with 8 or more reviews are included. 

## Baseline Model

For the baseline model, I rely heavily on intuition rather than testing a number of different combinations out in order to give a general gist of how good the model can be. More complicated feature creation, hyperparameter tuning, etc. will be done in the final model. Our biggest asset in prediction is the text that we have for each recipe -- in its ingredients information, recipe title, and recipe description. We will use the bag of words encoding (which can be effective for small strings like ingredients). It can give really large amounts of features with a lot of unique words, which can potentially cause overfitting. In order to reduce the number of features, I won't use the description or title columns in the model yet.

In sklearn, the CountVectorizer class makes word comparison a lot easier for us. It automatically ensures all data is lowercase, and splits by punctuation. I will store these all as 1-grams (to just store the words) for the baseline. These are my features:

| Feature Name | Type
|:-------------------------------|---------------------:|
| Words in Ingredients | Categorical Nominal
| Days After 2008 | Quantitative Discrete |
| Minutes of Cooking Time| Quantitative Discrete |

Note that times are typically continuous, but in this case they are discrete. The scraper only finds days, so Days After 2008 is technically discrete. Additionally, food.com only asks users to specify minutes without decimals, so it is also discrete.

In this case, a tree-based model would be nice since the heavy amount of text data makes having interaction terms (or a combination of words) more important. Tree-based models are non-parametric, and can therefore, fit this really well. I will use the random forest regressor to counteract the overfitting of having too many columns.

I divide the model into a test and a training set (with 80% of the data in the training set). After fitting the model, I have the opportunity to do some simple hyperparameter tuning of the model. For the baseline, I will only tune the max_depth, and compute the cross-validation error. Here are the R^2 values:

|   Max Depth |   Training R^2 |   Validation R^2 |
|------------:|---------------:|-----------------:|
|           3 |         0.1009 |           0.0067 |
|           5 |         0.1481 |           0.0107 |
|           7 |         0.1881 |           0.017  |
|          10 |         0.2376 |           0.0194 |
|          13 |         0.2754 |           0.0209 |
|          15 |         0.2971 |           0.0217 |
|          20 |         0.3423 |           0.0203 |

<iframe
  src="assets/validation_r2_baseline.html"
  width="550"
  height="400"
  frameborder="0"
></iframe>

A depth of 15 gives the best validation error, so I end up using it. The model is overfitting substantially. However, interestingly, it has a test R^2 of 0.0587, which is actually substantially better than the cross-validation error. This is likely because we have more data available to us (since all of the training data is used on every iteration) -- which is particularly useful in high-variance cases like this one. In the final model, I explore how to reduce this overfitting

## Final Model

Let's start by engineering some features. I will, first, go back to the reviews dataset and create a feature for the previous reviews of the recipe creator. It is likely that recipe creators who are more experienced and have been viewed more positively in the past are viewed more positively in the future.

Let's use the countvectorizer to encode the title in this, too. In order to not get overloaded with having too many features, let's only consider words which occur a certain amount of times -- which we can tune. Additionally, in the title, I remove common words like "the", which don't convey additional information.

Let's also encode ingredients in a different, likely better way, by encoding the full ingredient -- rather than just a part of it. This allows more direct comparisons (for instance, black beans only gets compared to other black beans, and not other black items as well as other beans). In order to not get too many features, I again set a minimum occurences to not have the model consider extremely uncommon or mispelled ingredients.

Including these features along with time and minutes, and continuing with random forests led to the best validation results. Therefore, I ended up using them. I got a  higher test R^2 (0.0636) than the baseline model (0.0587), which indicates that the method generalizes better to unseen data. This improvement is likely because I included the reviewer's previous mean and counts, which are useful pieces of information, along with the title -- something that reviewers definitely check. Since I added these useful pieces of information and kept the number of features about the same by setting min_df, I was likely able to reduce bias, but keep variance about the same. This improved the model. In fact, the training R-Squared increased -- which indicates that the new model did, indeed, reduce the bias.

However, an R-Squared of 0.0636 is not seemingly impressive. It means that this model did only 6.3% better than a model that predicted the mean review every time. A lot of this is unsurprising: attempting to understand how good a recipe is based largely on the words inputted is not easy, and our model should struggle. In the future, more sophisticated word-processing like NLP may improve prediction accuracies.

## Fairness Analysis

Let's test whether my model is fair towards new users. Is it worse at predicting new users scores than old users. Here, I will be comparing the Root Mean Squared Errors of recipe creators who have already had 20+ reviews (above median) vs those who have 19 or fewer (below median). 

**Null Hypothesis**: Model is fair. Root Mean Squared Error values are equal for recipe creators who submitted more than 20 recipes in the past and people who submitted less than 20 recipes.

**Alternate Hypothesis**: Model is unfair. Root Mean Squared Error values are lower for recipe creators who submitted less than 20 reviews than for those who submitted more than 20.

**Alpha**: 0.01

This is a graph of the data:

<iframe
  src="assets/fairness.html"
  width="550"
  height="400"
  frameborder="0"
></iframe>

Note that this graph is condensed to exclude rare cases of an error greater than 1 in order to make it easier to visualize. Seems like recipes without many previous ratings have a higher error, and are thus, worse off. This is in line with the alternate hypothesis that the model is unfair. Let's test for significance.

After doing a permutation test with 10000 simulations, we get a p-value of 0.0007, which is below alpha. Therefore, we reject the null hypothesis for the alternate hypothesis and conclude that this model is significantly worse for new recipe creators than old ones. If predicting 





