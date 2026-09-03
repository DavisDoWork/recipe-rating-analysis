# Cooking Time and Recipe Ratings

This project investigates the relationship between recipe cooking time and average user ratings. It first explores patterns in recipe ratings and cooking times, then examines missingness and performs hypothesis testing. Finally, it builds regression models to predict average recipe ratings and evaluates whether model performance is fair across recipes with different cooking times.

---

## Introduction

The dataset contains recipes and user interactions from Food.com.

The main question for the exploratory portion of this project is:

**What is the relationship between the cooking time and average rating of recipes?**

The prediction portion of the project focuses on predicting a recipe's `avg_rating` using characteristics that are available before users provide ratings.

### Relevant Columns

- `minutes`: cooking time of the recipe
- `avg_rating`: average rating received by the recipe
- `n_steps`: number of preparation steps
- `submitted`: date the recipe was submitted
- `tags`: tags associated with the recipe
- `n_ingredients`: number of ingredients

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

Describe:

- how `recipes` and `interactions` were merged
- why ratings of `0` were replaced with missing values
- how `avg_rating` was calculated
- the final number of rows and columns
- which important columns contain missing values

### Univariate Analysis

#### Distribution of Average Ratings

[INSERT AVG_RATING HISTOGRAM HERE]

Explain what the distribution shows.

#### Distribution of Cooking Time

[INSERT MINUTES HISTOGRAM HERE]

Explain the strong right skew and why the visualization may be restricted to the 99th percentile.

### Bivariate Analysis

[INSERT COOKING TIME VS AVG RATING PLOT HERE]

Explain the apparent relationship between cooking time and average rating.

### Interesting Aggregates

[INSERT TIME-GROUP AGGREGATE TABLE HERE]

Discuss mean and median ratings for:

- 0–30 minutes
- 31–60 minutes
- 61–120 minutes
- 120+ minutes

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
