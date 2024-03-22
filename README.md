In this project we investigate the question:
# How do the characteristics of a recipe, such as the number of ingredients and nutritious content, influence the complexity of a recipe measured by the number of steps required to make it?

**Name(s)**: Laura Sofia Diaz Rodriguez, Nida Firdaws

**Welcome to our exploration of the interplay between recipe characteristics and complexity!** In this project, we delve into the fascinating world of culinary arts to understand how various factors influence the complexity of a recipe.

***Why should you care?***

Cooking is not just about following instructions; it's an intricate dance of flavors, techniques, and timing. Understanding what makes a recipe complex can greatly enhance both home cooking and professional culinary endeavors. By examining how recipe characteristics contribute to complexity, we not only gain a deeper appreciation for the art of cooking but also unlock practical knowledge that can streamline meal preparation and elevate dining experiences.

## Introduction

Our dataframe `recipes` is the result of merging two key sources: `RAW_interactions.csv` and `RAW_recipes.csv`. `RAW_recipes` contains recipes and several descriptors for them, such as such as the recipe name, `name`, number of ingredients, `n_ingredients`, preparation time in minutes, `minutes`, and and a breakdown of nutritional information, `nutrition`. `RAW_interactions` has interaction descriptors such as user written reviews, `review`, user IDs, `user_id`, and star count ratings, `rating`. After merging,  `recipes` has a every review in `RAW_interactions` plus its recipe descriptors. This resulting dataframe has 25 columns and 234,428 rows.

The columns relevant to our question are `nutrition` (corresponding breakdown of 
`calories (#)`, `total fat (PDV)`, `sugar (PDV)`, `protein (PDV)`, etc.!), `n_ingredients`, `minutes`, and `tags`. Every element originally from `nutrition` is of the datatype `float64`. `minutes` and `n_ingredients` are the `int64` datatype. Lastly and mainly, `n_steps` is of type `int64`. These columns are relevant to our question because they indicate an important aspect of how many steps a submitted recipe may have based on the time it takes to cook it and how nutritionally complex it is with regard to ingredients and specific nutritional values. 

## Data Cleaning and Exploratory Data Analysis

In our preliminary data cleaning of `data`, we converted the `nutrition` column into individual columns correspondng to each value in `nutrition`: `calories (#)`, `total fat (PDV)`, `sugar (PDV)`, `sodium (PDV)`, `protein (PDV)`, `saturated fat (PDV)`, and `carbohydrates (PDV)`. Keeping in line with our question, we did this so that we can investigate the specific relationships of numerical nutritional values with other variables in the dataset. In addition, we dropped the duplicate column storing recipe ids after the merging of `recipes` and `interactions`. We did this because we did not want our dataset to have a column storing repetitive information. Finally, we replaced 0 ratings with NaN. We did this after investigating whether food.com hosted any reviews with 0 ratings. Upon closer inspection, we found that ratings of 0 are not possible, and simply mean the user never gave a numerical rating for that recipe. We replace these values with `NaN` because they do not actually represent a low rating, simply ratings that are missing. We also added a column `average_rating`, which indicated the average rating for the recipe reviewed. We did this so we could better understand characteristics of recipes that rated highly. 

The `ingredients` and `tags`  columns are originally list-like strings. Although we acknowledge this, we keep the string version for easier use when playing with the distributions of descriptors for recipes with specific ingredients, like chocolate. However, we convert `date` to a `Timestamp` object for the option to use dates reviews were uploaded to understand its relationship with `n_steps` and `rating` when formulating our baseline model. 

Here are the first five rows of relevant columns in our cleaned dataframe: 

|    |   recipe_id | name                                 |   rating |   calories (#) |   total fat (PDV) |   sugar (PDV) |   protein (PDV) |   carbohydrates (PDV) |   n_ingredients |
|---:|------------:|:-------------------------------------|---------:|---------------:|------------------:|--------------:|----------------:|----------------------:|----------------:|
|  0 |      333281 | 1 brownies in the world    best ever |        4 |          138.4 |                10 |            50 |               3 |                     6 |               9 |
|  1 |      453467 | 1 in canada chocolate chip cookies   |        5 |          595.1 |                46 |           211 |              13 |                    26 |              11 |
|  2 |      306168 | 412 broccoli casserole               |        5 |          194.8 |                20 |             6 |              22 |                     3 |               9 |
|  3 |      306168 | 412 broccoli casserole               |        5 |          194.8 |                20 |             6 |              22 |                     3 |               9 |
|  4 |      306168 | 412 broccoli casserole               |        5 |          194.8 |                20 |             6 |              22 |                     3 |               9 |


We noticed that recipes taking more than five hours are less than 5% of the data and those taking more than a day are around 0.07% of our data. So, we chose to examine the distributions of recipes under five hours and under day. In the below graph, we observe that the recipes taking under five hours have a strong right skew, with most recipes under five hours taking 30-40 minutes on average. 

<iframe
  src="assets/recipes-hours.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>
In addition, we plotted the distribution of `n_steps` to investigate how many steps recipes posted on food.com tend to have. We observed that this graph has a right skew, with most recipes having between 10-15 steps on average.
<iframe
  src="assets/steps-distribution.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>
We also investigate the relationship between categories of `rating` and the time it takes to cook a recipe (`minutes`). We observed that the Interquartile Range for each rating category is roughly the same. We can also observe that 4 and 5 star rating recipes have the most number of `minutes` outliers.
<iframe
  src="assets/cooking-time-by-rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


We plotted the distribution of the number of steps across different groups of minutes, observing that the IQR for quick, medium, and long recipes tends to increase as the number of steps increases. This was particularly interesting to us as we thought it implied that recipes that take longer intuitively have more steps to follow. This realization is what barred us from using `minutes` as a feature in our model's prediction problem, since we would not necessarily have access to a variable that can directly be associated with more steps in a recipe. 
<iframe
  src="assets/distribution_of_steps_across_minute_groups.html"
  width="800"
  height="700"
  frameborder="0"
></iframe>
We aggregated by `rating` for each recipe and created a table showing the counts of each 
rating category for each recipe. In randomly sampling from this table, we observe that most recipes tend to be concentrated in either 4 or 5 stars, with few having at least 1 rating in every category. We infer this may be due to the fact that users feeling strongly about a recipe tend to feel reviews, while fewer users will take the time to leave a rating if they feel the recipe is subpar.

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

### Permutation test for missingness of `rating` vs `minutes`

**H<sub>0</sub>** : There is no relationship between the missingness of `rating` and the `minutes` column. The difference in means of `minutes` between recipes that have missing `rating` and recipes that do not is purely due to chance. 

**H<sub>a</sub>** : There is a relationship between the missingness of `rating` and the `minutes` column. The difference in means of `minutes` between recipes that have missing `rating` and recipes that do not is not due to random chance alone. 

The p-value of the missigness of rating by minutes is 0.115. At an alpha level of 0.05, we fail to reject the null. The columns `rating` and `minutes` are independent of eaach other. 
<iframe
  src="assets/minutes_missing.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
We see in the above graph the distributions of `minutes` when rating is missing vs. when `rating` is not missing. Both distribution are quite similar, which falls in line with our conclusion that whether or not `rating` is missing does not have a significantly clear relationship with `minutes`. This makes sense since these two variables seem quite unrelated with respect to a user leaving a `rating` or not. We can see that the distributions have a similar right skew and visibly overlapping center.

### Permutation test for missingness of `rating` vs `n_steps`

We then investigate the relationship between the missingness of `rating` and `n_steps`. Our hypotheses are as follows:

**H<sub>0</sub>** : There is no relationship between the missingness of `rating` and the `n_steps` column. The difference in means of `n_steps` between recipes that have missing `rating` and recipes that do not is purely due to chance. 

**H<sub>a</sub>** : There is a relationship between the missingness of `rating` and the `n_steps` column. The difference in means of `n_steps` between recipes that have missing `rating` and recipes that do not is not due to random chance alone. 

Our resulting p-value after performing a permutation test is 0.0. Therefore, at an alpha of 0.05, we reject the null hypothesis and conclude that the missingness of `rating` is dependent on `n_steps`.
<iframe
  src="assets/n-steps-missing.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
In the graph above, we observe that the distribution of `n_steps` for `rating` when it is missing is quite similar to the distribution for when it is not missing. However, when `rating` is missing, we can see that the blue line (non-missing) differs from the orange line (missing), meaning there is still some differences between both distributions. This difference, as shown above, has a relationship to missingness of `rating`. We infer this is due to the fact that recipes with missing ratings might have had too many steps to follow, leading users to review them differently than for a different number of steps.

## Hypothesis Testing

**H<sub>0</sub>** : There is no difference in mean ratings between recipes with the word 'chocolate' in ingredients and recipes without the word 'chocolate' in ingredients.

**H<sub>a</sub>** : There is a difference in mean ratings between recipes with the 'chocolate' ingredients and recipes without the 'chocolate' in ingredients.

We justify our choice of difference in mean ratings as a test statistic because we want to see if there is a systematic difference in distributions of ratings for recipes with and without chocolate, and rating is a variable that we can look at the average of in between 1-5. The difference of means is the appropriate statistic to use for this hypothesis test because we are comparing the means of two different groups: recipes with chocolate in the ingredients and recipes without chocolate in the ingredients. We’re interested in determining if there is a statistically significant difference between the means of those groups. By calculating the difference of means and then conducting a hypothesis test, we are essentially investigating whether the observed difference in mean ratings between recipes with chocolate and recipes without chocolate is statistically significant, or if it could have occurred by random chance alone under the null hypothesis (that there is no difference in mean ratings between the two groups).

Assumption: We realize that the word 'chocolate' as a standalone word is not sufficient to check for in our `ingredients` column as there are several unique ingredients containing this word. However, we make the assumption that if the word is included is because the recipe either contains chocolate, an imitation, or an alternative to it.

With a p-value of 0.03 and alpha of 0.05, we reject the null hypothesis and conclude there is a statistically significant difference in mean ratings between recipes with the word chocolate in ingredients and recipes without the word chocolate in ingredients. We infer that that the inclusion of chocolate as an ingredient tends to be in recipes that are desserts, which in turn tend to have more favorable ratings.


## Framing a Prediction Problem

We aim to forecast the number of steps required to prepare recipes based on various characteristics, encompassing factors such as the quantity of ingredients and their nutritional composition.

This prediction task falls under the category of regression. The response variable, or the target variable, is the number of steps required to prepare a recipe. We chose this variable as it represents a comprehensive metric for recipe complexity. By predicting the number of steps, we can capture the intricacies involved in executing the recipe, including preparation, cooking, and assembly, which collectively reflect the overall complexity.

We opted to use the coefficient of determination (R-squared) and the root mean squared error (RMSE) as the evaluation metrics for our model. R-squared provides insight into the proportion of variance in the target variable (number of steps) that is explained by the model. A higher R^2 value indicates a better fit of the model to the data. Additionally, RMSE measures the average deviation of the predicted values from the actual values. We chose these metrics over others like accuracy or F1-score because they offer comprehensive understanding of model performance in terms of both predictive accuracy (RMSE) and explanatory power (R^2) while being easy to interpret.

## Baseline Model
To begin, since we recognize that rows with missing ratings constitute only 6% of the dataset, we opt to drop these rows to preserve data integrity.

In the design of our baseline model features, we observed the pairwise correlation coefficients between all quantitative columns of our dataframe `data`. We naively observe that our target variable, `n_steps` has the highest correlation coefficients with `n_ingredients`, `calories (#)`, and `total fat (PDV)`. We also presume that there is some relationship between `n_steps` and `minutes` despite the correlation coefficient of 0.01, because we can infer that recipes that take longer (larger # of `minutes`) must take longer due to a combination of a large `n_steps` and other factors. In fact, we can see this relationship in our bivariate analysis above, in the figure `Distribution of Number of Steps Across Minute Groups`, where the third quartile of each minute group increases as we enter a bigger minute category.
<iframe
  src="assets/correlation_matrix_heatmap.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
In this graph we can see the numerical values of coefficients of determination between all quantitative column. There are quite a few high correlations between nutritional values, and `n_steps` has our selected features as its highest correlations as well.

We also observed the variances across different quantitative columns that we want to use, seeing that `calories (#)`, `sugar (PDV)`, and so on are highly varied. The variances for these values is more than 10000 units each. We want to use `calories (#)` and `total fat (PDV)` as features because they have higher correlations with `n_steps`, but their high variances and very large convinced us to utilize `QuantileTransformer` on them both. This would ensure that our model remained robust to outliers and still preserve the distances between datapoints. We also selected `n_ingredients` as a feature because we saw that it had a fairly strong linear relationship with `n_steps` and also a low variance. 

Thus we have chosen our features to be `n_ingredients`,quantile transformed `calories (#)`, and quantile transformed `total_fat (PDV)`. We chose a `LinearRegression` model to be fit on our features because we are simply using features that we observe a high coefficient of determination for with the response variable `n_steps`. 

We don't believe this current model is very good because on average our R**2 value for the test set is 0.18 and our RMSE is 5.8135. From our r-squared value we can see the proportion of variance for `n_steps` that is explained by the features in our model is quite low, meaning 18% of the variability in `n_steps` can be explained by our features. This shows that the model explains only a small portion of the variance in the data, which likely means there are underlying relationships that it is not capturing. We suspect this is because there are other nuances in the recipes, such as additional nutritional values, that explain an associated increase or decrease in `n_steps`. 

Further, the RMSE (Root Mean Squared Error) as a measure of the differences between predicted values and the observed values is 5.8135. This suggests our model's predictions are off by approximately 5.8135 units units from the actual values. We can postulate that this is a moderate level of accuracy given the context of our assumption that the relationship is strongly linear. 

Overall, our baseline model provides a foundation for understanding the relationship between features and the number of steps in recipe preparation. We think it is a valuable starting point, but since we know there are features that have a high correlation with other features within our feature set, we want to take advantage of multicollinearity in our final model to improve the baseline accuracy and RMSE. 

## Final Model

In our final model, we introduced the feature `protein (PDV)` based on its correlation with the variable `n_ingredients`, the most correlated variable to `n_steps`, and its correlation to `n_steps` itself. This addition is grounded in the understanding that protein-rich ingredients like meat may necessitate specific preparation techniques or additional steps, thus influencing recipe complexity. By incorporating `protein (PDV)`, we aimed to capture this nuanced relationship and enhance the predictive capability of our model regarding recipe complexity.

We added two new transformed features and changed our model selection. We decided to add a boolean feature which shows whether the tags for a recipe contain the tag `easy` because we observed below that those recipes tend to have a higher number of steps on average. We also added `sugar (PDV` as a feature because we knew from our prior analysis that `sugar (PDV)` and `n_steps` have a higher correlation compared to other nutritional values. `sugar (PDV)` is also highly correlated with `calories (#)` at a correlation coefficient of   `sugar (PDV)` has an extremely high variance with quite a few outliers (we observed this after plotting the distributions of `sugar (PDV`. We wanted to preserve the relative distances between each datapoint and also make it robust to the outliers without having to outright remove them, so we used `QuantileTransformer` to transform `sugar (PDV)`.
<iframe
  src="assets/kde_plots.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
As seen in this graph, the distributions of `n_steps` for recipes without the `easy` tag and recipes with the `easy` tag has a significant difference. We can see that the recipes without the `easy` tag tend to have more steps on average. We can infer this is because users are less likely to tag recipes with a high number of steps as `easy`, since those recipes should require less effort.

Secondly, we changed our model selection to `DecisionTreeRegressor` because we believed that there are a lot of nonlinear relationships between `n_steps` and the multicollinearity of other features in the dataset. `DecisionTreeRegressor` works well with data that contains many outliers, and we believe that the specificity of data on recipes published on a food website can only increase as more and more recipes are added, so we need a model that can appropriately make decisions on where the data is similar and where it is not. `DecisionTreeRegressor` also makes decisions based on feature importance and the interactions between features themselves, which is something we wanted because we know that many of these features have linear relationships with each other, like the nutritional values. 

We aimed to tune the hyperparameter `max_depth` because we wanted to prevent overfitting with a `DecisionTreeRegressor` that creates too many splits on very specific sections of our features. The `max_depth` parameter limits the depth of the `DecisionTreeRegressor` during training, so that it can decide how much to grow the tree while splitting into sections of features that are related to each other. We run a simple loop checking to see what values of `max_depth` optimally plateaus with respect to our RMSE and r^2 value. We utilized a simple loop to achieve this and then plotted the changes of RMSE and r^2 as the max_depth increased, as seen below: 
<iframe
  src="assets/max_depth_vs_R2_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="assets/max_depth_vs_RMSE_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
As seen above, `max_depth` is optimally 30, since the RMSE and r^2 values plateau at this point and do not increase by a significant amount. We therefore take max_depth = 30 as our lower bound for the `DecisionTreeRegressor` in our model. 

After running our model with this new hyperparameter and transformed features, we observed our r^2 value for the test set jumped to 63% and our RMSE value decreased to 3.84 units. We believe this is a big improvement from our baseline model, which was relying on simple linear relationships to make predictions on `n_steps`. We wanted to use `max_features` as a hyperparameter as well, but ultimately decided against it because we observed that this was overfitting for our model, and it did not make sense to limit our `DecisionTreeRegressor` to a certain number of features when we were already operating on a small subset of the available features in our dataset. The lower RMSE certainly means that our final model performed better because the distance between the actual target variables and response variables was minimized.

The high variances in quantitative values made us less confident about using very specifically transformed features, because in investigating the relationships between each of these variables we were unable to find specific relationships such as linear, root, or exponential trends. Ultimately, we stuck to simpler transformations to try to predict our `n_steps` response variable so that we could attempt to generalize as best as possible. 


## Fairness Analysis
For our fairness, analysis, we chose to investigate recipes with fewer than 10 ingredients and recipes with more than 

Group X: Recipes with fewer than 10 ingredients (few_ingredients = 1).
Group Y: Recipes with 10 or more ingredients (few_ingredients = 0).

Evaluation Metric : Root Mean Squared Error (RMSE) of the prediction for the number of steps.

Null Hypothesis (H<sub>0</sub>) : There is no significant difference in the RMSE between recipes with fewer than 10 ingredients and recipes with 10 or more ingredients. The model is fair for these groups.

Alternative Hypothesis (H<sub>a</sub>) : There is a significant difference in the RMSE between recipes with fewer than 10 ingredients and recipes with 10 or more ingredients. The model is not fair for these groups.

Choice of Test Statistic : We are computing the difference in RMSE between the two groups.

Significance Level : We will use a significance level of 0.05.

Resulting p-value : The resulting p-value from the permutation test is 0.0.
This means that the observed difference in RMSE between recipes with fewer than 10 ingredients and recipes with 10 or more ingredients is not significant at the chosen significance level of 0.05. In other words, there is little statistically significant evidence to reject the null hypothesis, so we fail to reject. 




