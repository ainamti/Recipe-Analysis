# Recipe Analysis Final Project
Name: Ankita Inamti 

## Introduction
In this project I will be analyzing two dataframes: recipes and interactions. Recipes contains 83782 rows and a few relevant columns include the recipe name and id, nutrition information, the number of steps in the recipe, the text for recipe steps, and the user provided description. Interactions contains 731927 rows and the columns include the user and recipe id, date of interaction, and the rating and review of the recipe. 
  
After looking at the information given in the two datasets, I will be performing analyzing the healthiness of the recipes given based on the nutrition information given in the recipes dataframe. In order to evaluate the healthiness of a recipe, I will look for recipes with a low amount of carbohydrates and high amounts of protein. My analysis will include a implementing a hypothesis test to see if recipes with meat are healthier than recipes without meat and a regression model to predict the number of calories in a recipe. 

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
One of the columns in recipe_interact that i analyzed was the protein column. I created a scatter plot to see the overall distribution of protein content of all the recipes given and saw that there were a number of recipes with a very high protein content. 

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
NMAR (Not Missing At Random) means the missing values are related to the missing data itself. The avg_rating column may be NMAR if there is no rating given for the actual recipe. The avg_rating column also may have some missing values due to the difficulty or popularity of the recipe, which would make the column MAR. I would like to look at the number of steps or preparation time and the popularity metrics (social media shares and views) to evaluate the difficulty and popularity of the recipe and see if there is a pattern with the missing data. 

### Missingness Dependency

I used the minutes column and the date column to analyze the dependency of missingness on the avg_recipe column. If a recipe takes a long time to complete or is published a long time ago that may explain why a recipe is missing a rating. I conducted permutation tests for both columns and recieved a p-value of 0.047 for the minutes column and a p-value of 0.146 for the date column. Since the p-value for the minutes permutation test is less than 0.05, there is a significance and longer/shorter recipes are linked to the missingness of the avg_rating column. The p-value for the date permutation text is greater than 0.05, so there is no significance and older/newer recipes are not linked to the missingness of the avg_rating column. 

<iframe
  src="assets/dist_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe> /

<iframe
  src="assets/empirical_dist_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


## Hypothesis Testing


## Framing a Prediction Problem


## Baseline Model


## Final Model

## Fairness Analysis



