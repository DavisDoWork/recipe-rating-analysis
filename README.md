# Cooking Time and Ratings Statistical Analysis

Cooking Time and ratings Statistical Analysis is a comprehensive data science project conducted at UCSD. The project takes place a sequence of analysis on Recipe and Rating dataset to explore the relationship between recipe cooking time and average user ratings. It first goes through the exploratory data analysis stage to missingness and performs hypothessi testing. Ultimately, The project builds regression models to predict average recipe ratings and evaluate the fairness of the model across recipes with various cooking times.

Author: Thinh Duy Do.

---

## Introduction

We are living in a world that substantial amount of information is easy to access in the Internet. Leading to the fact that more and more home cooks seek for recipes on common review flatforms. As a result, they find it challenging to select the appropriate one to cook. One of the criteria persuade them to select a particular recipe is its rating. What truly the element decide if a recipe is highly rated? The project investigate whether recipe's cooking time relate to the ratings of recipes. The study will work on subset of the raw data which was announed in 2008 on Food.com. The original dataset is used for the recommender system research paper by Majumder et al.

### Relevant Columns

The first dataset, RAW_recipes.csv, consistent of 83782 distinct recipes prepresenting for 83782 rows, including the following columns:

| Column           | Description                                        |
| :--------------- | :------------------------------------------------- |
| `name`           | Recipe name                                        |
| `id`             | Recipe ID                                          |
| `minutes`        | Cooking time, in minutes                           |
| `contributor_id` | User ID of the recipe's submitter                  |
| `submitted`      | Date the recipe was submitted                      |
| `tags`           | Food.com tags associated with the recipe           |
| `nutrition`      | Nutrition information (calories, fat, sugar, etc.) |
| `n_steps`        | Number of steps in the recipe                      |
| `steps`          | Text describing each step, in order                |
| `description`    | User-provided description of the recipe            |
| `ingredients`    | List of ingredients used                           |
| `n_ingredients`  | Number of ingredients                              |

The second dataset named RAW_interactions.csv, consists of 731927 distinct review from the useer on particular recipe. It contains the following columns:

| Column      | Description                                        |
| :---------- | :------------------------------------------------- |
| `user_id`   | User ID of the reviewer                            |
| `recipe_id` | Recipe ID being reviewed                           |
| `date`      | Date of the interaction                            |
| `rating`    | Rating given (1–5, or 0 if no rating was selected) |
| `review`    | Text of the written review                         |

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

| name                               |     id | minutes | submitted           | n_steps | avg_rating | day_of_month |
| :--------------------------------- | -----: | ------: | :------------------ | ------: | ---------: | -----------: |
| 1 brownies in the world best ever  | 333281 |      40 | 2008-10-27 00:00:00 |      10 |          4 |           27 |
| 1 in canada chocolate chip cookies | 453467 |      45 | 2011-04-11 00:00:00 |      12 |          5 |           11 |
| 412 broccoli casserole             | 306168 |      40 | 2008-05-30 00:00:00 |       6 |          5 |           30 |
| millionaire pound cake             | 286009 |     120 | 2008-02-12 00:00:00 |       7 |          5 |           12 |
| 2000 meatloaf                      | 475785 |      90 | 2012-03-06 00:00:00 |      17 |          5 |            6 |

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

The scatter plot shows the relationship between every recipe's average rating against its cooking time (only contains below the 99th percentile of minutes). The figure shows vertical banding rather than a specific shape cloud of points since most recipes only receive limited number of reviews leading to the common fractions. There is no linear relationship and most of the points gather at the right lower of the figure proving that most of the high rating recipes take under 200 minutes for cooking.

### Interesting Aggregates

| time_group | count |  mean | median |   std |
| :--------- | ----: | ----: | -----: | ----: |
| 0-30 min   | 36419 | 4.645 |      5 | 0.617 |
| 31-60 min  | 24570 | 4.607 |      5 | 0.655 |
| 61-120 min | 11840 | 4.627 |      5 | 0.653 |
| 120+ min   |  7566 | 4.588 |      5 | 0.676 |

The aggregate table about grouping recipes by cooking time reveal interesting patterns of its distribution: as the cooking time increase, the mean rating decreases slightly, from 4.645 for recipes within 30 minutes down to 4.588 for recipes required more than 120 minutes. It reflects the string left-skw seen throughout the plot when the gap is 0.06 with the median is 5.0 in every group. Moreover, the difference in number of recipe between 0-30 and 120+ minutes group is significant as 5 times. The relationship formally tested when move to the Hypothessi Testing section below

---

## Assessment of Missingness

### NMAR Analysis

There is numerous missing data from 'avg_rating' column from invalid users (they leave it as blank or literally no one review it, recorded as 0). This considers as **NMAR**

The rating behavior is likely to the recipe's own quality. Users are more likely to select recipes that they are interesting and have a higher chance to leave a rating. With that being said, the mean of probability of avg_rating missingnesses depends on its value, which is a characteristic of NMAR. In spites of that idea, the missingness of 'avg_ratings' could turn to MAR when it is associated with 'n_steps' (recipes recorded wil more steps are less likely get the missingnes). We will test this formally in the Missingness Dependency section below.

### Missingness Dependency

I investigated whether the missingness of `avg_rating` depends on other
recipe characteristics by running a permutation test for 'n_steps' and 'day_of_month' columns. In both tests, I compared the group of recipes with a missing
`avg_rating` against the group with a non-missing `avg_rating`, as shown below.

#### 1. Missingness vs. Number of Steps

**Null Hypothesis:** The mean number of steps (`n_steps`) is the same for
recipes with a missing average rating and recipes with a non-missing
average rating.

**Alternative Hypothesis:** The mean number of steps differs between
recipes with a missing average rating and recipes with a non-missing
average rating.

**Test Statistic:** Absolute difference in group means.

**Significance Level:** 0.05

**Observed Statistic:** 1.493

**p-value:** 0.0

<iframe
  src="assets/missingness_n_steps.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The two distributions above appear visually similar in shape, but with a
large sample size (~80,000 recipes), even a small shift in the mean
number of steps can be detected as statistically significant.

With the p-value below the significance level of 0.05, we
**reject the null hypothesis**. There is strong evidence that missingness
in `avg_rating` depends on `n_steps` — recipes with a missing rating tend
to have a different average number of steps than recipes with a rating.
It is fit to the **MAR**: a missingness type which occur when a column is explainable by an observed column.

#### 2. Missingness vs. Day of Month

**Null Hypothesis:** The mean day of the month a recipe was submitted
(`day_of_month`) is the same for recipes with a missing average rating
and recipes with a non-missing average rating.

**Alternative Hypothesis:** The mean day of the month differs between
recipes with a missing average rating and recipes with a non-missing
average rating.

**Test Statistic:** Absolute difference in group means.

**Significance Level:** 0.05

**Observed Statistic:** 0.065

**p-value:** 0.714

<iframe
  src="assets/missingness_day_of_month.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The p-value (0.722) is much bigger than the significance level of 0.05, we
**fail to reject the null hypothesis**. There is insufficient evidence
that missingness in `avg_rating` depends on `day_of_month` — the day a
recipe was submitted does not appear to be related to whether its rating
is missing.

Together, these two tests give one column that missingness clearly
depends on (`n_steps`) and one column it does not depend on
(`day_of_month`), as required for this analysis.

---

## Hypothesis Testing

**Question:** Do recipes with short cooking times (0–30 min) have a
different average rating than recipes with long cooking times (120+
min)?

**Null Hypothesis:** The mean average rating is the same for recipes in
the 0–30 minute group and recipes in the 120+ minute group.

**Alternative Hypothesis:** The mean average rating is different for
recipes in the 0–30 minute group and recipes in the 120+ minute group.

**Test Statistic:** Absolute difference in mean `avg_rating` between the
two groups.

**Significance Level:** 0.05

I chose a permutation test using the absolute difference in group means
because the question is fundamentally about comparing the central
tendency of `avg_rating` between two independent groups defined by
cooking time, with no assumption of normality required — a permutation
test is nonparametric and only relies on the two groups being
exchangeable under the null hypothesis, which is appropriate since we care whether short vs. long cooking time affects rating. I used the _absolute_ difference rather than the signed difference because the alternative hypothesis is two-sided — an absolute-value
statistic correctly captures a difference in either direction. Both
groups were drawn from the outlier-filtered dataset (`minutes` at or
below the 99th percentile), consistent with the cleaning applied
throughout this analysis.

**Observed Statistic:** 0.0564

**p-value:** 0.0

<iframe
  src="assets/hypothesis_test_distribution.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The histogram above shows the empirical distribution of the test statistic
under 1,000 permutations of the null hypothesis, with the observed
statistic (0.0564) marked in red. Since the observed statistic falls far
outside the bulk of the simulated distribution, this visually confirms
the p-value of 0.0.

Since the p-value (0.0) is far below the significance level of 0.05, we
reject the null hypothesis. There is strong evidence that average rating
differs between recipes with short (0–30 min) and long (120+ min)
cooking times, with short-cooking recipes tending to have a slightly
higher average rating. Because this is a statistical hypothesis test
rather than a randomized controlled trial, this result does not prove
that cooking time causes a difference in rating, nor does it establish
the alternative hypothesis as true with certainty — it indicates that the
observed difference is unlikely to have arisen by chance alone under the
assumption that the null hypothesis holds.

---

## Framing a Prediction Problem

**Prediction Problem:** This is a **regression problem**. I am predicting
`avg_rating`, a continuous value ranging from 1 to 5.

**Response Variable:** `avg_rating` — the average rating a recipe
receives. I chose this because it is an extension of the question explored
in Steps 1–4 (the relationship between cooking time and rating), the theme of the project keeps smoothly and it is also a great features representing ratings.

**Evaluation Metric:** I use **RMSE (Root Mean Squared Error)**. I chose
RMSE over an alternative like MAE (Mean Absolute Error) since we need RMSE's characteristic in large error penalty (by squaring
residuals before averaging), which is appropriate here since a model that
is wildly wrong about a recipe's rating (e.g., predicting 5.0 for a
recipe that's actually rated 1.0) is a more serious failure than several
small. Since `avg_rating` is continuous, RMSE is also help to interpret easily in the original units of the response variable, which makes the error easy to reason about.

**Features and Time-of-Prediction:** All features include here is all known before the prediction — that is, at the moment
a recipe is submitted, before any user follow the instruction and cook the recipe. This includes `minutes`, `n_steps`, `n_ingredients`, `tags`, `nutrition`, and `submitted`. I explicitly exclude any information derived from `RAW_interactions.csv` (review counts, review text, or the ratings themselves), as these information is collected after a recipe is rated. The project avoids to use it, otherwise, the information will be leaked from the future relative to the prediction task and defeat the purpose of predicting a rating before it exists.

---

## Baseline Model

The baseline model is a `LinearRegression` model using the two following features:

- `minutes`
- `n_steps`

Both features are quantitative, so no categorical encoding was needed —
each was passed through a `StandardScaler` inside a single `sklearn`
`Pipeline` (via a `ColumnTransformer`), as the result the training and prediction tasks go through one consistent, reproducible object. Recipes
with `minutes` above the 99th percentile were excluded prior to
train/test splitting, consistent with the outlier handling as describe above from Data Cleaning phase.

**Baseline Train RMSE:** 0.6396

**Baseline Test RMSE:** 0.6411

I consider the baseline is good enough but it could perform better: the Train and Test RMSE are close to each other, proving that the model isn't overfitting,
but an RMSE of roughly 0.64 rating points (on a 1–5 scale) means the
model's predictions are, on average, slightly off the true value — largely because `minutes` and `n_steps` alone shows weak relationshop to `avg_rating`. To improve the model, I would like to engineer two new features in Final Model stage.

---

## Final Model

For my final model, I engineered two new features beyond the baseline:

- **`years_since_submission`**: the number of years between a recipe's
  `submitted` date and the most recent submission date in the dataset,
  `FunctionTransformer` is used for computation. I included this because recipes
  submitted longer ago provide more information for users to decide if they want to rely on the recipes leading to change the behavior and ratings later on.
- **`is_dessert`**: a binary indicator derived from whether "dessert"
  appears in a recipe's `tags`. I included this because dessert recipes
  may systematically differ in how they would rate the recipes compared to the other like savory recipes, given the strong left-skew toward high ratings we observed throughout this dataset.

Both new features, along with the baseline's `minutes` and `n_steps`,
were combined into a single `sklearn` `Pipeline` using a
`ColumnTransformer`, so that all preprocessing and modeling happen
through one reproducible object.

I decided to use **`RandomForestRegressor`** as my final model, tuned via
**`GridSearchCV`**. Before tuning, I decided to search over
**`max_depth`**, since it directly controls the trade-off between
underfitting (a shallow tree that can't capture the relationship between
features and rating) and overfitting (a very deep tree that memorizes
noise in the training data) — this is a solid hyperparameter for the selected model with modest number of engineered features above.

**Best Hyperparameter (max_depth):** 5

**Best Cross-Validation RMSE:** 0.6403

**Final Model Train RMSE:** 0.6378

**Final Model Test RMSE:** 0.6342

**Baseline Test RMSE (for comparison):** 0.6411

**RMSE Improvement over Baseline:** ~0.0069

The final model outperforms the baseline on the test set. The improvement is recorded as increase 0.0069, which suggests that `minutes`,
`n_steps`, `years_since_submission`, and `is_dessert` capture only a
limited amount of the variation in `avg_rating` — consistent with what
the EDA and hypothesis test in earlier steps showed: cooking time and
related recipe attributes have a real but small relationship with
rating, rather than a dominant one. The result shows that he training and test RMSE remain close to each other, it agains show off that the overfitting does not occur despite the added complexity of a `RandomForestRegressor` over a linear model.

---

## Fairness Analysis

**Question:** Does my final model perform worse for recipes with long
cooking times than for recipes with short cooking times?

I split the test set into two groups using the **median cooking time
(35.0 minutes)** as the cutoff: the "short" label is considered as below the median, in the opposite, the "long" label contain recipes above the median.

**Evaluation Metric:** RMSE, computed separately within each group.

**Null Hypothesis:** The model's RMSE is the same for the "long" cooking
time group and the "short" cooking time group; any difference observed
is due to random chance.

**Alternative Hypothesis:** The model's RMSE is higher for the "long"
cooking time group than for the "short" cooking time group.

**Test Statistic:** RMSE(long) − RMSE(short).

**Significance Level:** 0.05

**RMSE (long-cooking group):** 0.6601

**RMSE (short-cooking group):** 0.6106

**Observed Statistic:** 0.0495

**p-value:** 0.001

A permutation test is conducted with the repetition numbers at 1000, the test randomly shuffling the long and short group labels each time, then continue to recompute the difference in group RMSE, building an empirical null distribution.

With the p-value (0.0001) below the significance level of 0.05, we
reject the null hypothesis. There is evidence that the model performs
worse — has a higher RMSE — for recipes with long cooking times than for
recipes with short cooking times. In spite of that conclusion, it does not prove that cooking time causes the model to perform worse, instead, it indicates there is a gap in RMSE between these two groups is not have arisen by chance. It is plausible to explain that recipes with long cooking time are less common in the dataset. It is a statistical test, it needs to be seen based on observational split of the data, not randomly trial.
