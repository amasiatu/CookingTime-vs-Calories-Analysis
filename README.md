# Quick Bites or Slow Feasts? Analyzing Calories vs. Cooking Time
#### Author: Chinyere Amasiatu
## Report for the Analyzing Calories vs. Cooking Time
### Introduction
##### Cooking has a been necessity for a long time, from home cooks to chefs, recipes are the most remebered with our taste buds rather than with our brain. So let's begin to explore the intricacies of recipes using the **Recipe and Ratings** dataset! The Recipe and Ratings dataset contains many recipes which will help us source the relationship between different recipe characteristics that we may have never thought of before
#### Central Question
#### The longer a recipe takes to make, increases the calories count?
##### This question lies under the understanding of the relationship between cooking time and calories. Cooking time is a vital attribute to recipes because it reveals the complexity of an individual recipe. By investigating the relationship between cooking time and calories we can uncover what information users would appreciate to balance their time and calorie intake.
##### In addition to understanding cooking time, this dataset provides us with nutritional information that can further reveal the complexity of a recipe such as the number of steps and the number ingredients involved in a recipe. This can also help us answer a bigger question: what key nutritional information make a recipes calorie count higher? So whether users are a home cook or a professional chef, the findings of this analysis will offer data driven guidance about the choice of recipe one can choose rather than analyzing with just our tastebuds.
#### Introduction of Columns
##### The dataset in the RAW_recipes.csv has **83782** rows containing information relevent to our analysis such as:
| Column         | Description                                                                                      |
|----------------|--------------------------------------------------------------------------------------------------|
| `name`         | Title of the recipe                                                                              |
| `minutes`      | Total time required to prepare and cook the recipe                                               |
| `contributor_id` | Unique identifier for the user who contributed the recipe                                      |
| `submitted`    | Date when the recipe was submitted                                                               |
| `tags`         | List of tags associated with the recipe, such as `['desserts', 'chocolate', 'easy']`            |
| `nutrition`    | A string containing a list of nutritional values: `['calories', 'total fat', 'sugar', 'sodium', 'protein', 'saturated fat', 'carbs']` |
| `n_steps`      | Number of steps in the recipe instructions                                                       |
| `steps`        | List of instructions for preparing the recipe                                                    |
| `n_ingredients`| Total number of distinct ingredients used                                                        |

### Data Cleaning and Exploratory Data Analysis
##### Cleaning the data was a necessary step to make sure the data was consistent and reliable enough to be analyzed our process included:
##### 1. Parsing the Nutrition Column
* ##### The nutrition column in our dataset stored all the nutrition information in lists so in order to extract the calorie information we extracted the calorie data from the list and stored it in a new column
##### 2. Identifying and Removing Outliers
* ##### There were many absurd and unrealistic cooking time values that were above 2 years. We calculated the z-scores and filtered out the z-scores that were above 3 (outliers in the distribution). We also limited cooking time to 200 minutes to get a clearer view of our data.
#### Cleaned Data Sample
| Index  | Minutes | Calories | N_Steps | N_Ingredients |
|--------|---------|----------|---------|----------------|
| 0      | 40      | 138.4    | 10      | 9              |
| 1      | 45      | 595.1    | 12      | 11             |
| 2      | 40      | 194.8    | 6       | 9              |
| 83779  | 40      | 59.2     | 7       | 8              |
| 83780  | 29      | 188.0    | 9       | 10             |
| 83781  | 20      | 174.9    | 5       | 7              |

##### 1.1 Univariate Analysis (Calories)
* ##### Below is the distribution of calories in the dataset limited to 5000 for clarity. It has a strong right skew which means most people's recipes typically fall between 150-500 calories per recipe.
 <iframe
 src="assets/cal_dist.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

##### 1.2 Univariate Analysis (Cooking Time)
* ##### Below is the distribution of cooking times (minutes) in the dataset limited to 5000 for clarity. It has a strong right skew just like our calorie distribution which by observation means most people don't cook for more than an 1 hour.
 <iframe
 src="assets/min_dist.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

##### 2.1 Bivariate Analysis (Calories vs Cooking Time)
* ##### Below shows the relationship between calories and cooking time in the dataset limited to 5000 for clarity. It seems many recipes have similar calorie counts and it seems as though the longer a recipe takes it doesn't necessarily translate to more calories according to the trend since the variance of the data seems high. However we see a bit of cluster of points between 0-50 minutes being under 500 calories.
 <iframe
 src="assets/min_cook_dist.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

#### Interesting Aggregates
| Cooking Time (Minutes) | Average Calories |
|------------------------|------------------|
| 0–10                   | 269.74           |
| 11–20                  | 322.87           |
| 21–30                  | 358.55           |
| 31–45                  | 381.42           |
| 46–60                  | 423.89           |
| 61–120                 | 456.93           |
| 120+                   | 473.11           |

##### Acoording to the table we group the cooking time in minutes into 10 minute bins and calculate the average calories per bin. According to the table we see an upward trend in the data which supports the idea the longe recipes seem to be more calorie dense.
#### Imputation
##### Most of the features we planned on analyzing were numerical so to prevent skewing in our analysis we wanted to mean impute missing values with the average mean of that column in all the columns necessary in the dataset. However, none of the numerical columns we were going to use had any missing values so we skipped this step.

### Framing a Prediction Problem
#### Can we predict the number of calories in a recipe using features like cooking time, number of ingredients, and steps involved?
* ##### Type of Problem: Regression Problem
    * ##### Calories is a continuous and numeric variable so regression is ideal for this problem
* ##### Evaluation Metric: Mean Squared Error (MSE)
    * ##### MSE penalizes larger errors so it's ideal for this problem
* ##### Potential Features: 
    * ##### Cooking Time (`minutes`) 
    * ##### Number of steps (`n_steps`) 
    * ##### Nutrition (`calories`) 
    * ##### Number of Ingredients (`n_ingredients`) 

### Baseline Model
* ##### To establish a starting point for our prediction we trained our Baseline Model using **Linear Regression**. The model used a pipeline that included feature scaling via StandardScaler so we can directly and accurately compare our numeric values. We didn't use any categorical variables so no encoding was needed and the resulting **Baseline Model MSE** was `~88642.90`.
* ##### Our **Root Mean Squared Error (RMSE)** was `~297.7` which means our predictions were off by 297 calories on average, which is pretty large. These results definitely leave room for improvement, so I wouldn't necessarily call our model "good" just yet. Our results of a high MSE show signs of underfitting so we may have to change our model.

### Final Model
#### Feature Engineering
* ##### We created 2 new features:
    * ##### `calories_per_minute`
        * ##### Defined by dividing the calories and minutes column, this feature captures the caloric density per unit of time, which is important because longer recipes may not be as calorie dense as you would typically think, some time is just taken passively cooking or preparing.
    * ##### `log_minutes`
        * ##### Defined by doing a log transformation of the `minutes` column, this helps us reduce right skew we observed in the analysis of cooking time, which is important because the log scale flattens the distribution making it a more useful linear model.
#### A New Model
* ##### Ridge Regression:
    * ##### We used a Ridge Regression model which is still a Linear Regression model just with L2 regularization, that included a pipeline that contained:
        * ##### `PolynomialFeatures` with degree=2 to capture nonlinear interactions
        * ##### `StandardScaler` to normalize all numeric inputs
    * ##### We chose a Ridge Regression Model because it balances interpretability =, sped, and the ability to model complex relationship (with polynomial terms) while regularizing to prevent overfitting.
#### Hyperparameter Tuning
* #####  We tuned the regularization strength alpha using GridSearchCV with 5 fold cross-validation and the following parameter grid: 
        'ridge__alpha': [0.01, 0.1, 1, 10, 100] 
* ##### Our best performing model was when:
    * ##### PolynomialFeatures(degree=2)
    * ##### ridge_alpha = 10
#### Performance Comparison
| Model                                      | MSE       | RMSE   |
|--------------------------------------------|-----------|--------|
| **Baseline Model** (Linear Regression)     | 88642.90  | ≈ 297.7|
| **Final Model** (Ridge + Poly Features + Feature Engineering) | **26,573.7** | **≈ 163.0** |

#### Our final model significant improves:
* ##### By capturing the nonlinear relationships
* ##### By adding domain-informed features
#### This is a >60% reduction in MSE which means our predications are on average much closer to the actual calorie values, I would say this model is not that bad!