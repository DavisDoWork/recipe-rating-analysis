# Cooking Time and Ratings Statistical Analysis

Cooking Time and ratings Statistical Analysis is a comprehensive data science project conducted at UCSD. The project takes place a sequence of analysis on Recipe and Rating dataset to explore the relationship between recipe cooking time and average user ratings. It first goes through the exploratory data analysis stage to missingness and performs hypothessi testing. Ultimately, The project builds regression models to predict average recipe ratings and evaluate the fairness of the model across recipes with various cooking times.

Author: Thinh Duy Do.

---

## Introduction

We are living in a world that substantial amount of information is easy to access in the Internet. Leading to the fact that more and more home cooks seek for recipes on common review flatforms. As a result, they find it challenging to select the appropriate one to cook. One of the criteria persuade them to select a particular recipe is its rating. What truly the element decide if a recipe is highly rated? The project investigate whether recipe's cooking time relate to the ratings of recipes. The study will work on subset of the raw data which was announed in 2008 on Food.com. The original dataset is used for the recommender system research paper by Majumder et al.

### Relevant Columns

The first dataset, RAW_recipes.csv, consistent of 83782 distinct recipes prepresenting for 83782 rows, including the following columns:

- 'name': Recipe name
- 'id': Recipe ID
- `minutes`: cooking time of the recipe
- 'contributor_id': User ID who submitted the recipe
- 'submitted': Date recipe was submitted
- 'tags': Food.com tags for recipe
- 'nutrition': Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”
- `n_steps`: number of preparation steps
- 'steps': Text for recipe steps, in order
- `n_ingredients`: number of ingredients

The second dataset named RAW_interactions.csv, consists of 731927 distinct review from the useer on particular recipe. It contains the following columns:

- 'user_id': User ID
- 'recipe_id': Recipe ID
- 'date': Date of interaction
- 'rating': Rating given
- 'review': Review text

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The following steps are conducted to get the dataset ready for analysis:

1. Merged the recipes and interactions datasets: I performed a left merged of RAW_recipes.csv with inteactions.csv on recipe ID, so every single recipe is kept inspite of the missingness of reviews
2. Replaced ratings of 0 with NaN: The star rating is decided by users with values from 1-5 excluding the star of 0. It does not reflect the bad recipe rating, instead, it is recorded as 0 by default since users not select a star rating. Treating them as NaN avoids the average ratings being dragged down, so I converted them to NaN to analyze accurately.
3. Computed the average rating per recipe: a specific recipe contains numerous ratings from distinguish users, so I grouped the recipe ID and took the mean rating to get a single 'avg_rating' value per recipe. Finally, merge back onto the original dataset. As a result, I mostly had a dataset ('merged') used for the analysis.
4. Converted 'submitted' to a datetime type and extarcted 'day_of_month', serving to learn whether the missingness of 'avg_rating' relattes to the day a recipe was submitted.

After processing the data cleaning, I had a merged dataset consists of 83782 recipes including totally three columns have missing values: 'name' (1 missing), 'description (70 missing), and 'avg_rating' (2609 missing).

Here is the first 5 rows of the cleaned DataFrame:
| name | id | minutes | submitted | n_steps | avg_rating | day_of_month |
|:--------------------------------------|-------:|----------:|:--------------------|----------:|-------------:|----------------:|
| 1 brownies in the world best ever | 333281 | 40 | 2008-10-27 00:00:00 | 10 | 4 | 27 |
| 1 in canada chocolate chip cookies | 453467 | 45 | 2011-04-11 00:00:00 | 12 | 5 | 11 |
| 412 broccoli casserole | 306168 | 40 | 2008-05-30 00:00:00 | 6 | 5 | 30 |
| millionaire pound cake | 286009 | 120 | 2008-02-12 00:00:00 | 7 | 5 | 12 |
| 2000 meatloaf | 475785 | 90 | 2012-03-06 00:00:00 | 17 | 5 | 6 |

### Univariate Analysis

#### Distribution of Average Ratings

<iframe
  src="assets/avg_rating_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution is heavily left-skewed, with the majority of recipes is rated as 5 stars. The figue shows that very few ratings below 3.0, meaning the low-rated recipes are rare in the dataset. The trend suggests strong positive rating bias, once a new recipe is submitted, it is likely to rated from 4-5.

### Bivariate Analysis

<iframe
  src="assets/rating_vs_cooking_time_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This scatter plot shows every recipe's average rating against its cooking
time (recipes above the 99th percentile of `minutes` are excluded, as
discussed in Data Cleaning). The plot shows pronounced vertical banding
rather than a smooth cloud of points — this is expected, since
`avg_rating` only takes a limited set of discrete values (most recipes
have relatively few reviews, so their average lands on a small number of
common fractions like 4.0, 4.5, or 5.0). No strong linear relationship is
visually obvious, though there's a slight tendency for points with very
long cooking times to sit somewhat lower on the rating axis. This
motivates a more targeted look at the relationship using grouped
statistics below.

### Interesting Aggregates

| time_group | count |  mean | median |   std |
| :--------- | ----: | ----: | -----: | ----: |
| 0-30 min   | 36419 | 4.645 |      5 | 0.617 |
| 31-60 min  | 24570 | 4.607 |      5 | 0.655 |
| 61-120 min | 11840 | 4.627 |      5 | 0.653 |
| 120+ min   |  7566 | 4.588 |      5 | 0.676 |

Grouping recipes by cooking time and looking at their average rating
statistics reveals a small but consistent pattern: mean rating decreases
slightly as cooking time increases, from 4.645 for recipes under 30
minutes down to 4.588 for recipes over 120 minutes — a gap of about 0.06
points. The median is 5.0 in every group, reflecting the strong
left-skew seen throughout this dataset, so the differences show up in the
mean rather than the median. Recipe counts also drop off sharply for
longer cooking times (36,419 recipes take 30 minutes or less, versus only
7,566 taking over 2 hours), meaning the higher-time groups are estimated
from noticeably less data. This is the relationship formally tested in
the Hypothesis Testing section below.

---

## Assessment of Missingness

### NMAR Analysis

Discuss whether `avg_rating` could be NMAR and what additional information would help explain its missingness.

### Missingness Dependency

I investigated whether the missingness of `avg_rating` depends on other recipe characteristics.

#### Missingness vs. Number of Steps

**Null Hypothesis:** The mean number of steps is the same for recipes with missing and non-missing average ratings.

**Alternative Hypothesis:** The mean number of steps differs between recipes with missing and non-missing average ratings.

**Test Statistic:** Absolute difference in group means.

**Significance Level:** 0.05

[INSERT RESULT / P-VALUE HERE]

State conclusion.

#### Missingness vs. Day of Month

**Null Hypothesis:** The mean day of the month is the same for recipes with missing and non-missing average ratings.

**Alternative Hypothesis:** The mean day of the month differs between recipes with missing and non-missing average ratings.

**Test Statistic:** Absolute difference in group means.

**Significance Level:** 0.05

[INSERT RESULT / P-VALUE HERE]

State conclusion.

---

## Hypothesis Testing

I tested whether short-cooking recipes and long-cooking recipes have different average ratings.

**Null Hypothesis:** Recipes taking 0–30 minutes and recipes taking more than 120 minutes have the same mean `avg_rating`.

**Alternative Hypothesis:** The two groups have different mean `avg_rating`.

**Test Statistic:** Absolute difference in mean average rating.

**Significance Level:** 0.05

[INSERT OBSERVED STATISTIC HERE]

[INSERT P-VALUE HERE]

[INSERT PERMUTATION DISTRIBUTION PLOT HERE]

### Conclusion

Explain whether you reject or fail to reject the null hypothesis and interpret the result in the context of cooking time and recipe ratings.

---

## Framing a Prediction Problem

The prediction task is a **regression problem**.

The response variable is:

`avg_rating`

The goal is to predict the average rating of a recipe using information that would be available before users rate the recipe.

I evaluate the models using **Root Mean Squared Error (RMSE)** because the target variable is quantitative and RMSE measures the typical magnitude of prediction error while penalizing larger errors more heavily.

---

## Baseline Model

The baseline model is a `LinearRegression` model using:

- `minutes`
- `n_steps`

Both features are quantitative.

[INSERT BASELINE TRAIN RMSE]

[INSERT BASELINE TEST RMSE]

Discuss what these values indicate.

---

## Final Model

The final model is a `RandomForestRegressor`.

It retains the baseline features:

- `minutes`
- `n_steps`

and adds two engineered features:

- `years_since_submission`
- `is_dessert`

### Feature Engineering

**`years_since_submission`**

Explain why recipe age may be associated with rating behavior.

**`is_dessert`**

Explain why different recipe categories may receive different ratings.

### Hyperparameter Tuning

I tuned the Random Forest's `max_depth` hyperparameter using 5-fold cross-validation.

Candidate values:

`2, 3, 4, 5, 7, 10, 15, 20`

The best value was:

**`max_depth = 5`**

### Model Performance

| Model                      | Test RMSE |
| -------------------------- | --------: |
| Baseline Linear Regression |   [VALUE] |
| Final Random Forest        |   [VALUE] |

Explain that the final model achieved a lower RMSE than the baseline, even if the improvement is small.

---

## Fairness Analysis

I evaluate whether the final model performs differently for short-cooking and long-cooking recipes.

The groups are defined using the median cooking time.

- **Group X:** shorter-cooking recipes
- **Group Y:** longer-cooking recipes

**Evaluation Metric:** RMSE

**Null Hypothesis:** The model performs equally well for short- and long-cooking recipes, and any observed difference in RMSE is due to random chance.

**Alternative Hypothesis:** The model has a higher RMSE for long-cooking recipes.

**Test Statistic:**

`RMSE_long - RMSE_short`

[INSERT OBSERVED DIFFERENCE]

[INSERT P-VALUE]

[INSERT PERMUTATION DISTRIBUTION]

### Conclusion

State whether there is sufficient evidence that the model performs worse for long-cooking recipes.
