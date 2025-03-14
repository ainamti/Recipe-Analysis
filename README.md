# Assesing the Healthiness of Recipes and Predicting its Calories
Name: Ankita Inamti 

## Introduction
In this project I will be analyzing two dataframes: recipes and interactions. Recipes contains 83782 rows and a few relevant columns include the recipe name and id, nutrition information, the number of steps in the recipe, the text for recipe steps, and the user provided description. Interactions contains 731927 rows and the columns include the user and recipe id, date of interaction, and the rating and review of the recipe. 
  
After looking at the information given in the two datasets, I will be analyzing the healthiness of the recipes given based on the nutrition information given in the recipes dataframe. In order to evaluate the healthiness of a recipe, I will look for recipes with a low amount of carbohydrates and high amounts of protein. My analysis will include a implementing a hypothesis test to see if recipes with meat are healthier than recipes without meat and a regression model to predict the number of calories in a recipe. 

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning
I left merged the recipe and interactions dataframes and filled the rating column with NaN becuase there are multiple ratings for the same recipe in the interactions dataset. In order to get one rating value for each recipe in the merged dataframe, I found the average rating for each recipe and created a new column named average_rating in the recipe_interact dataframe to store all the values. 

In order to analyze the nutritional information, I first created a helper function which converts a string into a list, since the nutrition column was formatted as a list but was of a string type. After changing the nutrition column's type, I created separate columns for each nutritional value (calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), and carbohydrates (PDV)). I then dropped any duplicate values that may have existed in the current dataframe. 

```py
print(recipe_interact[['name', 'id', 'protein', 'carbohydrates']].head().to_markdown())
```

|    | name                                 |     id |   protein |   carbohydrates |
|---:|:-------------------------------------|-------:|----------:|----------------:|
|  0 | 1 brownies in the world    best ever | 333281 |         3 |               6 |
|  1 | 1 in canada chocolate chip cookies   | 453467 |        13 |              26 |
|  2 | 412 broccoli casserole               | 306168 |        22 |               3 |
|  6 | millionaire pound cake               | 286009 |        20 |              39 |
|  7 | 2000 meatloaf                        | 475785 |        29 |               2 |

### Visualizations
One of the columns in recipe_interact that I analyzed was the protein column. I created a scatter plot to see the overall distribution of protein content of all the recipes given and saw that there were a number of recipes with a very high protein content. 

<iframe
  src="assets/protein_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
I then created a new data frame only containing recipes with high protein and low carbohydrates by filtering recipes with the top 25% protein amount and the bottom 25% of carbohydrates. 

```py
high_protein_threshold = recipe_interact['protein'].quantile(0.75)  # Top 25% protein
low_carb_threshold = recipe_interact['carbohydrates'].quantile(0.25)  # Bottom 25% carbs

# Filter dataset for high-protein, low-carb recipes
high_protein_low_carb = recipe_interact[(recipe_interact['protein'] >= high_protein_threshold) & (recipe_interact['carbohydrates'] <= low_carb_threshold)]

high_protein_low_carb.head()
```

With this new dataframe, I generated a bar chart that compared the protein and carbohydrate amount between healthy (high protein, low carb) recipes and all recipes. I found that there was a noticeable difference in amount, especially with the protein column. 

<iframe
  src="assets/protein_carb_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

I created a pivot table to visualize the difference in nutrition means between healthy recipes and all recipes. It is clear that there is a big difference in protein means and and carbohydrate means, and an even bigger difference in the sugar amounts between the two types of recipes. This confirms that the recipes stored in the high protein low carb dataset are generally considered healthy even with the factors that I did not consider while filtering. 


|               |   All Recipes (Mean) |   High-Protein, Low-Carb (Mean) |
|:--------------|---------------------:|--------------------------------:|
| protein       |              33.1334 |                         82.0507 |
| carbohydrates |              13.7868 |                          2.0019 |
| calories      |             429.927  |                        479.133  |
| total_fat     |              32.6254 |                         46.3607 |
| sugar         |              68.6644 |                         10.8307 |


## Assessment of Missingness
NMAR (Not Missing At Random) means that the missing values are related to the missing data itself. The avg_rating column may be NMAR if there is no rating given for the actual recipe. The avg_rating column also may have some missing values due to the difficulty or popularity of the recipe, which would make the column MAR (Missing at Random). I would like to look at the number of steps or preparation time and the popularity metrics (social media shares and views) to evaluate the difficulty and popularity of the recipe and see if there is a pattern with the missing data. 

### Missingness Dependency
I used the minutes column and the date column to analyze the dependency of missingness on the avg_recipe column. If a recipe takes a long time to complete or is published a long time ago that may explain why a recipe is missing a rating. I conducted permutation tests for both columns and recieved a p-value of 0.047 for the minutes column and a p-value of 0.146 for the date column. 

Since the p-value for the minutes permutation test is less than 0.05, there is a significance and longer/shorter recipes are linked to the missingness of the avg_rating column. The p-value for the date permutation text is greater than 0.05, so there is no significance and older/newer recipes are not linked to the missingness of the avg_rating column. 

This plot shows the distribution of the avg_rating column when the minutes column is missing and when it is not missing. 

<iframe
  src="assets/dist_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe> 
This plot shows the empirical distribution of the test statistic for minutes.

<iframe
  src="assets/empirical_dist_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>## Hypothesis Testing
I combined all the recipe names in the high protein low carb dataset and extracted the most common words. I found that a majority of the words were related to dishes containing meat. 

<iframe
  src="assets/most_common_words.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>So, my hypothesis test aims to explore whether recipes containing meat are healthier than vegetarian recipes. 

**Null Hypothesis:** Recipes containing meat are as healthy(high protein and low carbohydrates) as recipes without meat.

**Alternative Hypothesis:** Recipes containing meat are healthier(high protein and low carbohydrates) than recipes without meat.

**Test Statistic:** Difference in group means (mean of healthiness metric of meat recipes and mean of healthiness metric of vegetarian recipes)

I added a column named contains_meat filled with boolean values which determines whether a recipe contains meat or not. 

```py
meat_keywords = ["chicken", "beef", "pork", "fish", "shrimp", "bacon", "lamb", "turkey", "ham", "sausage", "steak", "duck"]

# Convert ingredients column to lowercase for better matching
recipe_interact['ingredients'] = recipe_interact['ingredients'].astype(str).str.lower()

def contains_meat(ingredients):
    return any(meat in ingredients for meat in meat_keywords)

# Create 'contains_meat' column
recipe_interact['contains_meat'] = recipe_interact['ingredients'].apply(contains_meat)

recipe_interact[['name', 'contains_meat']].head()
```

| name                                 | contains_meat   |
|:-------------------------------------|:----------------|
| 1 brownies in the world    best ever | False           |
| 1 in canada chocolate chip cookies   | False           |
| 412 broccoli casserole               | True            |
| millionaire pound cake               | False           |
| 2000 meatloaf                        | True            |

To set up the permutation test to calculate the p-value, I first calculated my test statistic which included the following setup:

```py
from scipy import stats
# Separate recipes into meat and non-meat groups
meat_recipes = recipe_interact[recipe_interact['contains_meat'] == True]
non_meat_recipes = recipe_interact[recipe_interact['contains_meat'] == False]

# Calculate healthiness metric
meat_healthiness = meat_recipes['protein'] - meat_recipes['carbohydrates']
non_meat_healthiness = non_meat_recipes['protein'] - non_meat_recipes['carbohydrates']

# Calculate the difference in means (test statistic)
difference_in_means = meat_healthiness.mean() - non_meat_healthiness.mean()

```
I used this test statistic in my permutation test which combines the healthiness scores from both datasets, which gets repeatedly shuffled. The difference in means is calculated for each permuted group. The p-value uses the absolute value of the differences to perform a two tailed test and in this specific test I got a p-value of 0. Since the value is less than 0.05, there is a statistical signficance and we reject the null hypothesis. This means that there is evidence suggesting that recipes containing meat are healthier(high protein and low carbohydrates) than recipes without meat.

## Framing a Prediction Problem
I will be predicting the number of calories in recipe which is a regression problem, since the value being predicted is a continuous numerical value. 
At the time of prediction, the relevant features that are known are the recipe name, ingredients, minutes, number of steps, number of ingredients, and all nutritional information. I will be using MSE (mean squared error) and RMSE (root mean squared error) as the evaluation metric because they represent the model's prediction error in units of calories, which will help in determining the accuracy of my models. 

## Baseline Model
In my baseline model, I am using the two features total_fat and sugar to predict the calories column. Total fat is quantitative and represents the amount of total fat in the recipe. Sugar is also a quantitative variable and represents the amount of sugar in the recipe. I used a linear regression model for my predictions and calculated the MSE and RMSE to evaluate the performance of my model. After implementing my baseline model I got an RMSE value of 196.17170158545017 and a MSE value of 38483.33650293091. Since these values are pretty big I would say that this model's performance isn't the best considering I have only used two features to predict the number of calories in a recipe, when there are more features that can be considered/engineered to buid an accurate model.

## Final Model
In my final model I have added two new features: fat_sugar_interaction and protein_carb_ratio. Fat_sugar_interaction was created by multiplying the total_fat and sugar columns and I implemented this feature because the fat and sugar amounts significantly affect the calorie density of a recipe. It is also reasonable to assume that recipes high in both fat and sugar tend to have a higher calorie count than recipes high in only one of these components. Protein_carb_ratio was created by by dividing the protein column by the carbohydrates column (with a small value added to the denominator to prevent division by zero). This feature is relevant to the calorie predicting model because a recipe with a high protein and low carb ratio can have a significant influence on the calorie amount of a recipe. 

I used a RandomForestRegressor algorithm for my model, which is great for regression and builds decision trees and computes an an average of its predictions. I also used GridSearchCV, which finds hyperparameters that minimizes the neg_mean_squared_error. After implementing my final model I got an RMSE value of 89.02679605025783 and a MSE value of 7925.770414974202. The MSE and RMSE values were significantly lower in the final model compared to the baseline model which suggests that was a great improvement in the prediction accuracy of the recipe calories. 

## Fairness Analysis
In my fairness analysis I have chosen Group X to be recipes with high-fat (total fat above the median) and Group Y to be recipes with low-fat (total fat below the median) with the RMSE (root mean squared error) being the evaluation metric. 

**Null Hyopthesis:** The model performs equally well for high-fat and low-fat recipes.

**Alternative Hypothesis:** The model performs significantly worse for one of the groups.

**Test Statistric:** Observed difference in RMSE

**Significance Level:** 0.05

After implementing the permutation test, I recieved a p-value of 1, which is greater than 0.05. This means that we fail to reject the null hypothesis and there is evidence that the model performs equally as well for high-fat and low-fat recipes. 



