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

**p-value:** 0.722

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
`avg_rating`, a continuous value ranging from 1 to 5, rather than a
discrete class label.

**Response Variable:** `avg_rating` — the average rating a recipe
receives. I chose this because it directly extends the question explored
in Steps 1–4 (the relationship between cooking time and rating), keeping
the project's theme coherent from the exploratory analysis through to
prediction.

**Evaluation Metric:** I use **RMSE (Root Mean Squared Error)**. I chose
RMSE over an alternative like MAE (Mean Absolute Error) because RMSE
penalizes large errors more heavily than small ones (by squaring
residuals before averaging), which is appropriate here since a model that
is wildly wrong about a recipe's rating (e.g., predicting 5.0 for a
recipe that's actually rated 1.0) is a more serious failure than several
small, spread-out errors. Since `avg_rating` is continuous, RMSE is also
directly interpretable in the original units of the response variable
(rating points), which makes the error easy to reason about.

**Features and Time-of-Prediction:** All features used in this model are
known **before any ratings exist** for a recipe — that is, at the moment
a recipe is submitted, before any user has cooked or reviewed it. This
includes `minutes`, `n_steps`, `n_ingredients`, `tags`, `nutrition`, and
`submitted`. I explicitly exclude any information derived from
`RAW_interactions.csv` (review counts, review text, or the ratings
themselves), since those only exist _after_ a recipe has already
accumulated ratings — using them would leak information from the future
relative to the prediction task and defeat the purpose of predicting a
rating before it exists.

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
