In this project we investigate the question:
# How do the characteristics of a recipe, such as the number of ingredients and nutritious content, influence the complexity of a recipe measured by the number of steps required to make it?

**Name(s)**: Laura Sofia Diaz Rodriguez, Nida Firdaws

**Welcome to our exploration of the interplay between recipe characteristics and complexity!** In this project, we delve into the fascinating world of culinary arts to understand how various factors influence the complexity of a recipe.

***Why should you care?***

Cooking is not just about following instructions; it's an intricate dance of flavors, techniques, and timing. Understanding what makes a recipe complex can greatly enhance both home cooking and professional culinary endeavors. By examining how recipe characteristics contribute to complexity, we not only gain a deeper appreciation for the art of cooking but also unlock practical knowledge that can streamline meal preparation and elevate dining experiences.

## Introduction

Our dataframe `recipes` is the result of merging two key sources: `RAW_interactions.csv` and `RAW_recipes.csv`. `RAW_recipes` contains recipes and several descriptors for them, such as such as the recipe name, `name`, number of ingredients, `n_ingredients`, preparation time in minutes, `minutes`, and and a breakdown of nutritional information, `nutrition`. `RAW_interactions` has interaction descriptors such as user written reviews, `review`, user IDs, `user_id`, and star count ratings, `rating`. After merging,  `recipes` has a every review in `RAW_interactions` plus its recipe descriptors. This resulting dataframe has 25 columns and 234,428 rows.

## Data Cleaning and Exploratory Data Analysis

In our preliminary data cleaning of `data`, we converted the `nutrition` column into individual columns correspondng to each value in `nutrition`: `calories (#)`, `total fat (PDV)`, `sugar (PDV)`, `sodium (PDV)`, `protein (PDV)`, `saturated fat (PDV)`, and `carbohydrates (PDV)`. We did this with the intention to investigate the relationships of nutritional values with other variables in the dataset. We dropped the duplicate column storing recipe ids after the merging of `recipes` and `interactions`. In addition, we replaced 0 ratings with NaN. We did this after investigating whether food.com hosted any reviews with 0 ratings and confirmed they are invalid. We replace these values with `NaN` because they do not represennt a low rating, but missing data. Lastly, we added a column `average_rating`, which indicated the average rating for the recipe reviewed.

The `ingredients` and `tags` columns are originally list-looking strings. Although we acknowledge this, we keep the string version for easier use when playing with the distributions of descriptors for recipes with specific ingredients, like chocolate. However, we convert `date` to a `Timestamp` object for the option to use dates reviews were uploaded to understand its relationship with `n_steps` and `rating` when formulating our baseline model. 

Here are the first five rows of relevant columns in our cleaned dataframe: 

|    |   recipe_id | name                                 |   rating |   calories (#) |   total fat (PDV) |   sugar (PDV) |   protein (PDV) |   carbohydrates (PDV) |   n_ingredients |
|---:|------------:|:-------------------------------------|---------:|---------------:|------------------:|--------------:|----------------:|----------------------:|----------------:|
|  0 |      333281 | 1 brownies in the world    best ever |        4 |          138.4 |                10 |            50 |               3 |                     6 |               9 |
|  1 |      453467 | 1 in canada chocolate chip cookies   |        5 |          595.1 |                46 |           211 |              13 |                    26 |              11 |
|  2 |      306168 | 412 broccoli casserole               |        5 |          194.8 |                20 |             6 |              22 |                     3 |               9 |
|  3 |      306168 | 412 broccoli casserole               |        5 |          194.8 |                20 |             6 |              22 |                     3 |               9 |
|  4 |      306168 | 412 broccoli casserole               |        5 |          194.8 |                20 |             6 |              22 |                     3 |               9 |


We noticed that recipes taking more than five hours are less than 5% of the data and those taking more than a day are around 0.07% of our data. So, we chose to examine the distributions of recipes under five hours and under day. In the below graph, we observe that the recipes taking under five hours have a strong right skew, with most recipes under five horus taking 30-40 minutes on average. 

<iframe
  src="assets/recipes-hours.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

We plotted the distribution of `n_ingredients` to investigate how many ingredients recipes posted on food.com tend to have compared to different cooking times. This distribution seems fairly normal with a slight right skew, and we observe that on average recipes have 5-10 ingredients. Recipes that take longer also seem to have a larger third quartile for number of steps, in ascending order. 

<iframe
  src="assets/steps-distribution.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

We also investigate the relationship between categories of `rating` and the time it takes to cook a recipe (`minutes`). We observed that the Interquartile Range for each rating category is roughly the same (considering the number of datapoints in each group), and noticed that 4 and 5 star rating recipes have the most number of `minutes` outliers.

<iframe
  src="assets/cooking-time-by-rating.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

We  wanted to gain a better understanding of the distribution of ratings per recipe, represented by their `name`, so we aggregated by `rating` and created a pivot table. We noticed while sampling randomly many times that few recipes have at least one rating in each star count yet tend to be concentrated around 4 and 5 stars. Knowing the distribution of ratings for `recipes`, this made us think more about what causes people to leave a review and a rating. Is it the recipe ingredients? Is it the time it takes to make them? Is is the nutritious value?

| name                                 |   1.0 |   2.0 |   3.0 |   4.0 |   5.0 |
|:-------------------------------------|------:|------:|------:|------:|------:|
| 0 carb   0 cal gummy worms           |     0 |     0 |     0 |     1 |     3 |
| 0 point ice cream  only 1 ingredient |     0 |     0 |     0 |     0 |     1 |
| 0 point soup   ww                    |     0 |     0 |     0 |     2 |     7 |
| 0 point soup  crock pot              |     0 |     0 |     0 |     0 |     5 |
| 007  martini                         |     0 |     0 |     0 |     0 |     1 |

## Assessment of Missingness

Upon examination of the missingness in the `review` column of our recipe dataset, it becomes evident that the missing data are not randomly distributed. We posit that the missingness is best characterized as NMAR (Not Missing at Random), where the probability of a `review` being absent depends on unobserved factors, likely associated with user behavior and preferences.

An inherent asymmetry exists between leaving a numerical rating and composing a detailed review. Users may opt for the former due to its convenience and time efficiency, while reserving the latter for instances where they feel strongly about the recipe or wish to provide detailed feedback. This behavior introduces a non-random mechanism behind the missingness, where the decision to engage with a review is contingent upon the user's willingness to allocate additional time and effort, and the review to be left itself. In addition, public interaction with a recipe does not necessitate a written review. Users may visit for various reasons, such as browsing, planning future meals, or simply seeking inspiration. Consequently, the absence of a review cannot be attributed solely to the engagement with the recipe content but rather to the selective nature of user interactions. Lastly, user demographics, including age and location, likely influence review behavior. Certain demographic groups may be more predisposed to leaving reviews, either due to cultural norms, technological proficiency, or personal preferences. Understanding these nuances can illuminate patterns in missingness, as specific user segments may exhibit differential propensities to engage with recipe reviews.


We believe that the following supplementary data can help us further investigate the missingness of `review`:

**Visitor Tracking:** Capturing metrics on the number of visits and reviews to recipe pages, can elucidate general engagement patterns. Analyzing visit frequency, interactions and duration may uncover insights into user behavior.

**Demographic Profiling:** Demographic information enables group analysis to discern whether certain user groups exhibit distinct review behaviors.

We conduct a permutation test to check whether the missingness of `rating` is dependent on `minutes`. First, we created a column `is_missing` to indicate whether or not the value is missing in `rating`, and then we shuffle this column repeatedly 1000 times, each time calculating the difference in means between missing and non-missing values for in `rating` for minutes as our test statistic. We then take the average number of times this test statistic is greater than our observed statistic to calculate our p-value below. 

***Permutation test missingness of `rating` vs `minutes`***

**Null Hypothesis (H<sub>0</sub>)**  There is no relationship between the missingness of `rating` and the `minutes` column. The difference in means of `minutes` between recipes that have missing `rating` and recipes that do not is purely due to chance. 

**Alternative Hypothesis (H<sub>a</sub>)**  There is a relationship between the missingness of `rating` and the `minutes` column. The difference in means of `minutes` between recipes that have missing `rating` and recipes that do not is not due to random chance alone.

Our resulting p-value after performing a permutation test using difference in means is 0.115. At an alpha level of 0.05, we fail to reject the null. The columns `rating` and `minutes` are independent of each other. 

<iframe
  src="assets/minutes_missing.html"
  width="800"
  height="1200"
  frameborder="0"
></iframe>

We see in the above graph the distributions of `minutes` when rating is missing vs. when `rating` is not missing. Both distribution are quite similar, which falls in line with our conclusion that whether or not `rating` is missing does not have a significantly clear relationship with `minutes`.

Upon our results for this first experiment, we perform the same experiment using `rating` and all remaining columns in `recipe`, like the following.

***Permutation test for missingness of `rating` vs `n_steps`***

We then investigate the relationship between the missingness of `rating` and `n_steps`. Our hypotheses are as follows:

**H<sub>0</sub>** : There is no relationship between the missingness of `rating` and the `n_steps` column. The difference in means of `n_steps` between recipes that have missing `rating` and recipes that do not is purely due to chance. 

**H<sub>a</sub>** : There is a relationship between the missingness of `rating` and the `n_steps` column. The difference in means of `n_steps` between recipes that have missing `rating` and recipes that do not is not due to random chance alone. 

Our resulting p-value after performing a permutation test using difference in means is 0.0. Therefore, at an alpha of 0.05, we reject the null hypothesis and conclude that the missingness of `rating` is dependent on `n_steps`.

<iframe
  src="assets/n-steps-missing.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

In the graph above, we observe that the distribution of `n_steps` for `rating` when it is missing is quite similar to the distribution for when it is not missing. However, when `rating` is missing, we can see that the blue line (non-missing) differs from the orange line (missing), meaning there is still some differences between both distributions. This difference, as shown above, has a relationship to missingness of `rating`.

## Hypothesis Testing

**H<sub>0</sub>** : There is no difference in mean ratings between recipes with the word 'chocolate' in ingredients and recipes without the word 'chocolate' in ingredients.

**H<sub>a</sub>** : There is a difference in mean ratings between recipes with the 'chocolate' ingredients and recipes without the 'chocolate' in ingredients.

The difference of means is the appropriate statistic to use for this hypothesis test because we are comparing the means of two different groups: recipes with chocolate in the ingredients and recipes without chocolate in the ingredients. We’re interested in determining if there is a statistically significant difference between the rating distributon of these groups. 

Assumption: We realize that the word 'chocolate' as a standalone word is not sufficient to check for in our `ingredients` column as there are several unique ingredients containing this word. However, we make the assumption that if the word is included is because the recipe either contains chocolate, an imitation, or an alternative to it.

With a p-value of 0.03 and alpha of 0.05, we reject the null hypothesis and conclude there is a statistically significant difference in mean ratings between recipes with the word chocolate in ingredients and recipes without the word chocolate in ingredients. We infer that that the inclusion of chocolate as an ingredient tends to be in recipes that are desserts, which tend to evoke positive responses from individuals.

## Framing a Prediction Problem

We aim to forecast the number of steps required to prepare recipes based on various characteristics, encompassing factors such as the quantity of ingredients and their nutritional composition.

This prediction task falls under the category of regression. The response variable, or the target variable, is the number of steps required to prepare a recipe. We chose this variable as it represents a comprehensive metric for recipe complexity. By predicting the number of steps, we can capture the intricacies involved in executing the recipe, including preparation, cooking, and assembly, which collectively reflect the overall complexity.

We opted to use the coefficient of determination R-squared (R²) and the root mean squared error (RMSE) as the evaluation metrics for our model. R-squared provides insight into the proportion of variance in the target variable (number of steps) that is explained by the model. A higher R² value indicates a better fit of the model to the data. Additionally, RMSE measures the average deviation of the predicted values from the actual values. We chose these metrics over others like accuracy or F1-score because they offer comprehensive understanding of model performance in terms of both predictive accuracy (RMSE) and explanatory power (R²) while being easy to interpret.

## Baseline Model

Our baseline model aims to predict the number of steps required to prepare a recipe based on several features. 

The features used in our model:

Quantitative Features

`minutes`: The time in minutes required to prepare the recipe.

`n_ingredients`: The number of ingredients needed to make the recipe.

`rating`: The overall rating of the recipe (can also be considered ordinal).

`calories (#)`: The calorie content of the recipe.

In the design of our baseline model features, we observed the pairwise correlation coefficients between all quantitative columns of our dataframe `data`. We naively observe that our target variable, `n_steps` has the highest correlation coefficients with `n_ingredients`, `calories (#)`, and `total fat (PDV)`. We also presume that there is some relationship between `n_steps` and `minutes` despite the correlation coefficient of 0.01, because we infer that recipes that take longer (larger # of `minutes`) have more steps, a higher value of `n_steps`. In fact, we can see this relationship in our bivariate analysis above, in the figure `Distribution of Number of Steps Across Minute Groups`, where the third quartile of each minute group increases as we enter a bigger minute category.

<iframe
  src="assets/correlation_matrix_heatmap.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

In this graph we can see the numerical values of coefficients of determination between all quantitative columns. We also notice there are a few high relatively significant correlations between nutritional values and `n_steps`, our selected response variable.

We also observed the variances across different quantitative columns that we are interested in using, seeing that `calories (#)`, `sugar (PDV)`, and so on are highly varied. The variances for these values is more than 10000 units each. We want to use `calories (#)` and `total fat (PDV)` as features for their correlations with `n_steps`, so we choose to use `QuantileTransformer` to handle the largly varying scales of these features. This ensures that our features are uniformly distributed, which helps mitigate the impact of outliers and enhances the performance of our model. We also selected `n_ingredients`, as a feature because we observed a fairly strong linear relationship with `n_steps`.

Thus we have chosen our features to be `n_ingredients`, quantile transformed `calories (#)`, and quantile transformed `total_fat (PDV)`. We chose a `LinearRegression` model to be fit on our features because we so far we only know there is a significant coefficient of determination between these variables and the response variable `n_steps`. Since we recognize that rows with missing ratings constitute only 6% of the dataset, we opt to drop these rows to preserve data integrity for linear regression.

We split our dataset into training and testing sets using a 75-25 split ratio to ensure robust evaluation of our model's performance. The model pipeline consists of two main components: the preprocessing stage, encapsulated within a ColumnTransformer, and the linear regression model for prediction.


The performance of our updated baseline model is evaluated using two metrics:

**R-squared (R²)** : The R-squared value of approximately 0.19 suggests that our model accounts for approximately 19% of the variance in the number of steps.

**Root Mean Squared Error (RMSE)** : With an RMSE of approximately 5.6, our model exhibits a moderate level of accuracy in predicting the number of steps required for recipe preparation.

Overall, our baseline model provides a foundation for understanding the relationship between features and the number of steps in recipe preparation. However, there are still be opportunities for further refinement and feature engineering to enhance predictive accuracy. Continued exploration of advanced modeling techniques could yield better insights and further improve model performance. Therefore, while our baseline model serves as a valuable starting point, there remains room for enhancement to achieve a more robust and accurate prediction of recipe complexity.

## Final Model

In our final model, we introduced the feature `protein (PDV)` based on its correlation with the variable `n_ingredients`, the most correlated variable to `n_steps`, and its correlation to `n_steps` itself. This addition is grounded in the understanding that protein-rich ingredients like meat may necessitate specific preparation techniques or additional steps, thus influencing recipe complexity. By incorporating `protein (PDV)`, we aimed to capture this nuanced relationship and enhance the predictive capability of our model regarding recipe complexity.

We added two new transformed features and changed our model selection. We decided to add a binary feature which indicates whether tags for a recipe contain the tag `easy` because, as we observe below, recipes without it tend to have a higher number of steps. We also added `sugar (PDV)` as a feature because we knew from our prior analysis that `sugar (PDV)` and `n_steps` have a higher correlation compared to other nutritional values. `sugar (PDV)` is also highly correlated with `calories (#)`. However, `sugar (PDV)` also has very high variance. We wanted to preserve the relative distances between each datapoint, so we also transformed `sugar (PDV)` using `QuantileTransformer`.

<iframe
  src="assets/kde_plots.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

As seen in this graph, the distributions of `n_steps` for recipes without the `easy` tag and recipes with the `easy` tag has a significant difference. We can see that the recipes without the `easy` tag have more steps on average.

Moreover, we changed our model selection to `DecisionTreeRegressor` because we believed most relationships in our features have with `n_steps` are nonlinear. `DecisionTreeRegressor` is robust against the outliers in our quantitative features and will more effectively handle the non-linear relationships within our data. `DecisionTreeRegressor` also makes decisions based on feature importance and the interactions between features themselves, which allows us to beenift form the linear relationships within features, like those of nutritional values. 

We tuned the hyperparameter `max_depth` because we wanted to prevent overfitting. We ran a simulation to check for optimal values of `max_depth` with respect to our RMSE and r^2 value. We utilized a simple loop to achieve this and then plotted the changes of RMSE and r^2 as the max_depth increased.

<iframe
  src="assets/max_depth_vs_R2_plot.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>


<iframe
  src="assets/max_depth_vs_RMSE_plot.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

As seen above, `max_depth` is optimally 30, since the RMSE and r^2 values plateau at this point and do not increase by a significant amount. We therefore take max_depth = 30 as our bound for the `DecisionTreeRegressor` depth of our model.

Our final model achieved notable performance improvements over the baseline model, as evidenced by the following evaluation metrics:

**R-squared (R²)** : 0.64

**Root Mean Squared Error (RMSE)**: 3.8

Compared to the baseline model's R-squared of 19% and RMSE of 5.6, our final model demonstrates a substantial enhancement in predictive accuracy and precision. The significantly higher R-squared value indicates that our model explains approximately 64% of the variance in the number of steps required for recipe preparation, signifying a more comprehensive understanding of recipe complexity.

Moreover, the substantially lower RMSE underscores the model's improved ability to predict the number of steps accurately, with deviations from the actual values reduced to an average of approximately 3.8 steps. Considering that the standard deviation of 'n_steps' in the dataset is 6.44, an RMSE of 3.8 suggests that our model's predictions are, on average, within one standard deviation of the true values and provide reasonably accurate predictions of recipe complexity.

While there is still room for improvement, particularly in reducing the RMSE further, achieving an RMSE that is significantly smaller than the standard deviation of the target variable suggests that our model is indeed performing well in capturing the underlying patterns and variability in the data related to recipe complexity.

Overall, our final model represents a significant advancement, incorporating relevant features and leveraging an appropriate algorithm with tuned hyperparameters to provide more accurate predictions of recipe complexity. The inclusion of 'protein (PDV)' and fine-tuning of model parameters contribute to this enhanced performance, offering valuable insights into the intricate dynamics of recipe preparation.

## Fairness Analysis

**Group X**: Recipes with fewer than 10 ingredients (few_ingredients = 1).
**Group Y**: Recipes with 10 or more ingredients (few_ingredients = 0).

**Evaluation Metric** : Root Mean Squared Error (RMSE) of the prediction for the number of steps.

**Null Hypothesis (H<sub>0</sub>)** : There is no significant difference in the RMSE between recipes with fewer than 10 ingredients and recipes with 10 or more ingredients. The model is fair for these groups.

**Alternative Hypothesis (H<sub>a</sub>)** : There is a significant difference in the RMSE between recipes with fewer than 10 ingredients and recipes with 10 or more ingredients. The model is not fair for these groups.

**Choice of Test Statistic** : We are computing the difference in RMSE between the two groups.

**Significance Level** : We will use a significance level of 0.05.

**Resulting p-value** : The resulting p-value from the permutation test is 0.0.

By performing a permutation test and calculating the proportion of instances where the difference in RMSE between the two groups is greater than or equal to the observed difference, we get a **p-value of 1.0**.

This means that the observed difference in RMSE between recipes with fewer than 10 ingredients and recipes with 10 or more ingredients is not significant at the chosen significance level of 0.05. In other words, **there is no evidence to reject the null hypothesis, so we fail to reject**.

It implies that the model's predictions do not exhibit a significant difference in accuracy between recipes with fewer than 10 ingredients and those with 10 or more ingredients. Therefore, based on this analysis, there is no indication of fairness issues concerning prediction accuracy with respect to the number of ingredients in the recipes.
