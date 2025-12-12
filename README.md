# The Implications of Protein Content on Caloric Density in Recipes 

## Introduction

Oftentimes one of the most common challenges as an adult is figuring out what to make for dinner. Combine that with trying to maintain health and fitness goals and you end up realizing nobody prepared you for the amount of times you would ask "How much protein does it have though?" in your adult life. It ends up becomes discouraging to keep up with your protein goals, but **what if there was correlation behind highly rated recipes that prioritized protein being less calorie dense?** 

If there is a consistent relationship between protein content and calorie density, it could make it easier to choose healthier recipes without having to analyze nutrition labels every time. To explore this we will be analyzing the [food.com](food.com). Recipes and Interactions dataset, which contains over 200,000 recipes with nutritional information (including calories, fat, and protein) dating back to 2008. 

### Research Question
**Do recipes with higher protein content tend to have lower calorie density?** 

### Dataset Overview 
The `RAW_recipes.csv` file includes 231,637 rows and 17 columns. For this specific question, the columns that are the most relevant are:

- `nutrition`: A list containing seven nutritional values for each recipe, following the format `[calories, fat, sugar, sodium, protein, saturated_fat, carbohydrates]`.
- `calories`: One of the main values that will be extracted to evaluate the calorie content per serving. 
- `protein`: The other main value that will be extracted to evaluate the total grams of protein per serving.
- `minutes`: The reported cook time for the recipes to understand the time difference in nutritional recipes.
- `n_ingredients`: Number of ingredients used in the recipe to contextualize calorie density. (The simplier the recipe the more likely it's not as calorie dense)

These features help contextualize nutritional patterns and evaluate calorie density across recipes.

## Data Cleaning and Exploratory Data Analysis (EDA)

To analyze protein and calorie density, I first cleaned the nutrition column, which stores seven nutritional values inside a single list. I extracted these values into seperate columns(`caloires`, `protein`, `fat`, `sugar`, `sodium`, `saturated_fat`, and `carbohydrates`) and then created `calorie_density`, defined as the calories per serving of each recipe. This gave me clean, numeric variables that I can directly use for hypothesis testing and modeling. 

For the univariate EDA, I looked at the distribution of `protein` and `calorie_density`. Both variables are heavily right-skewed, with most recipes concentrated at lower values and a small number of extreme outliers. Because these outliers squeeze the axes and make plots unreadable, I created a filtered EDA sample by removing the top 1% of values before visualizing, although the **full dataset is preserved for modeling**. 

For my bivariate analysis, I created a scatterplit comparing protein and calorie density. Most recipes cluster in a low-protein, moderate-calorie range, but there's considerable spread, suggestings that the relationship between protein and calories density isn't perfectly linear. This motivates using statistical testing in Step 4 to determine whether higher protein actually corresponds to lower calorie density. 

<iframe
  src="assets/protein_vs_calorie_density.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Identifying Outliers
Because there are a few very distinct outliers, I first wanted to identify which recipes they were and how extreme they are. A small number of recipes contain very high calorie counts (30,000+ calories per serving), which compress the scale of the plots and make it difficult to see the rest of the data.

### Extreme Outlier Recipes

| Name                    | Calories | Nutrition List (Abbreviated)                   |
|------------------------|----------|------------------------------------------------|
| moonshine easy         | 36188.8  | [36188.8, 106.0, 30260.0, 80.0, 329.0, ...]    |
| powdered hot cocoa mix | 45609.0  | [45609.0, 3379.0, 16901.0, 1478.0, 4356.0, ...]|


Before exploring distribution, I examined extreme values in the calories and protein columns. A small number of recipes contained extreme high calorie counts (30,000 + calories per serving), which compress the scale of the plots and make it difficult to visualize the rest of the data. These outliers are valied entries, but for the sake of interpretable visualizations, I filtered them out only for EDA plots. The full dataset is still preserved for hypothesis testing and modeling.

**`Left off on graph for Distribution of Protein per Serving`**

## Missingness 
To understand whether missing descriptions occur at random or follow a pattern, I examined the `description` column, which had 70 missing values. All nutritional columns (`protein`, `calories`, `fat`, etc.) contain no missing data, so `description` was the most appropriate variable for this analysis. 

I compared the average number of ingredients between recipes **with** and **without** descriptions. The observed difference in means was approximately **1.47 ingredients**, with recipes lacking descriptions tending to use fewer ingredients. To test whether this difference could be due to chance, I conducted a permutation test with 1,000 shuffles of missgness labels. 

The resulting p-value was effectively 0, meaning it is extremely unlikely to observe a difference this large if missingness were completely random. Therefore, missingness in `description` is **not MCAR**. It is most likely **MAR**, meaning it depends on another observed variable, in this case, recipe complexity (measured by the number of ingredients). Simpler recipes appear less likely to include a written description. 

Since `description` is not central to my modeling or hypothesis testing, this missingness pattern does not affect downstream analysis, but it provides useful context for understanding how recipe data is recorded on Food.com




