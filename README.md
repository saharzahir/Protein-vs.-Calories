<div style="display: flex; align-items: center; gap: 12px; margin-top: 10px;">
  <img src="{{ '/assets/sahar.jpg' | relative_url }}" 
       alt="Sahar Zahir"
       width="70"
       style="border-radius: 50%; object-fit: cover;">
  <span style="font-size: 1.1rem;"><strong>Author:</strong> Sahar Zahir</span>
</div>


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

To analyze protein and calorie density, I first cleaned the nutrition column, which stores seven nutritional values inside a single list. I extracted these values into seperate columns(`calories`, `protein`, `fat`, `sugar`, `sodium`, `saturated_fat`, and `carbohydrates`) and then created `calorie_density`, defined as the calories per serving of each recipe. This gave me clean, numeric variables that I can directly use for hypothesis testing and modeling. 

### Identifying Outliers
Because there are a few very distinct outliers, I first wanted to identify which recipes they were and how extreme they are. A small number of recipes contain very high calorie counts (30,000+ calories per serving), which compress the scale of the plots and make it difficult to see the rest of the data.

### Extreme Outlier Recipes

| Name                    | Calories | Nutrition List (Abbreviated)                   |
|------------------------|----------|------------------------------------------------|
| moonshine easy         | 36188.8  | [36188.8, 106.0, 30260.0, 80.0, 329.0, ...]    |
| powdered hot cocoa mix | 45609.0  | [45609.0, 3379.0, 16901.0, 1478.0, 4356.0, ...]|


Before exploring distribution, I examined extreme values in the calories and protein columns. A small number of recipes contained extreme high calorie counts (30,000 + calories per serving), which compress the scale of the plots and make it difficult to visualize the rest of the data. These outliers are valied entries, but for the sake of interpretable visualizations, I filtered them out only for EDA plots. The full dataset is still preserved for hypothesis testing and modeling. 

For the univariate EDA, I looked at the distribution of `protein` and `calorie_density`. Both variables are heavily right-skewed, with most recipes concentrated at lower values and a small number of extreme outliers. Because these outliers squeeze the axes and make plots unreadable, I created a filtered EDA sample by removing the top 1% of values before visualizing, although the **full dataset is preserved for modeling**. 

<iframe
src = "assets/protein_distribution.html"
width = "800"
height = "600"
frameborder = "0"
></iframe>

For my bivariate analysis, I created a scatterplot comparing protein and calorie density. Most recipes cluster in a low-protein, moderate-calorie range, but there's considerable spread, suggestings that the relationship between protein and calories density isn't perfectly linear. This motivates using statistical testing in Step 4 to determine whether higher protein actually corresponds to lower calorie density. 

<iframe
  src="assets/protein_vs_calorie_density.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

To see how protein level relates to other recipe characteristics, I grouped recipes into low, medium, and high protein levels (based on quartiles) and computed average calorie density, calories, cook time, and number of ingredients:

| protein_level | mean_calorie_density | mean_calories | mean_minutes | mean_n_ingredients |
|---------------|----------------------|---------------|--------------|--------------------|
| low           | 169.3                | 169.3         | 141.2        | 7.4                |
| mid           | 365.6                | 365.6         | 105.4        | 9.4                |
| high          | 812.7                | 812.7         | 108.0        | 10.6               |


Recipes in the high-protein group tend to have higher average calories and calorie density, as well as slightly longer cook times and more ingredients. This suggests that high-protein recipes are often more complex and energy-dense, which connects back to the hypothesis testing and modeling in later steps.


## Missingness 
To understand whether missing descriptions occur at random or follow a pattern, I examined the `description` column, which had 70 missing values. All nutritional columns (`protein`, `calories`, `fat`, etc.) contain no missing data, so `description` was the most appropriate variable for this analysis. 

I compared the average number of ingredients between recipes **with** and **without** descriptions. The observed difference in means was approximately **1.47 ingredients**, with recipes lacking descriptions tending to use fewer ingredients. To test whether this difference could be due to chance, I conducted a permutation test with 1,000 shuffles of missingness labels. 

The resulting p-value was effectively 0, meaning it is extremely unlikely to observe a difference this large if missingness were completely random. Therefore, missingness in `description` is **not MCAR**. It is most likely **MAR**, meaning it depends on another observed variable, in this case, recipe complexity (measured by the number of ingredients). Simpler recipes appear less likely to include a written description. 

Since `description` is not central to my modeling or hypothesis testing, this missingness pattern does not affect downstream analysis, but it provides useful context for understanding how recipe data is recorded on Food.com

## Hypothesis Testing
**Question:** Do high-protein recipes tend to have *fewer* calories per serving than low-protein recipes? 

- **Null Hypothesis(H₀):** High-protein and low-protein recipes have the same average calories per serving. 

- **Alternative Hypothesis(H₁):** High-protein recipes have **lower** average calories per serving compared to low-protein recipes. 

I compared the mean calories per serving of high-protein recipes (top 25% of protein) and low-protein recipes (bottom 25%), using the difference in means:

### Test Statistic:
T = average calories (low-protein group) − average calories (high-protein group)

Positive values support that alternative hypothesis. 

### Observed Statistic:
T(obs) = -643.40

A **negative** lets us know that **low-protein recipes actually have lower calories on average**, the oppposite of the hypothesis. 

### Permutation Test
To simulate a distribution under the null hypothesis 1,000 random protein labels were taken, shuffled, and recalculated as such. 

<iframe 
src = "assets/protein_perm_test.html"
width = "800"
height = "600"
frameborder = "0"
></iframe>

### P-Value
p = 1.0

We **fail to reject** the null hypothesis 

- There is **no evidence** that high-protein recipes have fewer calories per serving. The observed data instead suggests that high-protein recipes tend to be **higher** in calories.

## Framing a Prediction 
To extend the analysis, I frame a predicition task where the goal is to predict the **calorie density** of a recipe using its nutritional information and basic recipe characteristics. This is a **regression** problem since calorie density is a continuous numeric variable. Calorie density was chosen as the response variable because it directly reflects hour energy-dense a recipe is and aligns with the project's focus on nutritional tradeoffs. Atthe time of prediction, nutritional values (e.g., protein, fat, sugar) and  recipe metadata (such as cook time and number of ingredients) would be available, so only these featues are used for modeling. Model performance is evaluated using **Root Mean Squared Error (RMSE)**. RMSE is appropriate because it penalizes large predicition errors more heavily, which is important when misestimating the calorie density of a recipe. 

## Baseline Model

All features are quantitative:

- `protein`
- `fat`
- `carbohydrates`
- `minutes`
- `n_ingredients`

These features were standardized using StandardScaler and combined with the regression model in a single `sklearn` pipeline

### Evaluation Metric 

I evaluated model performance using **Root Mean Squared Error**, which measures the typical magnitude of predicition errors and penalizes large mistakes more heavily. 

### Baseline Performance 

The baseline model achieved an RMSE of approximately:

**RMSE** ~ 43.5

This baseline provides a reasonable starting point, but its performance suggests that calorie density is not fully explained by linear relationships alone, motivating more expressive models in later steps. 

## Final Model

To improve upon the baseline linear regression model, I trained a more flexible **Random Forest Regressor** to predict calorie density. Unlike linear regression, random forests can capture non-linear relationships and interactions between nutritional features, which are common in recipe data.

### Features Used
The final model used the same features as the baseline model:
- Protein
- Fat
- Carbohydrates
- Cook time (minutes)
- Number of ingredients

These features are all known at the time a recipe is created, making them appropriate for prediction.

### Hyperparameter Tuning
I performed hyperparameter tuning using **GridSearchCV** with 3-fold cross-validation. The following hyperparameters were tuned:
- Number of trees (`n_estimators`)
- Maximum tree depth (`max_depth`)
- Minimum samples per leaf (`min_samples_leaf`)

These parameters control model complexity and help balance bias and variance.

### Model Performance
To evaluated model performance **Root Mean Squared Error (RMSE)** was utilized on a held-out test set.

- **Baseline Model RMSE:** 43.5  
- **Final Model RMSE:** 91.2  

Although the final model did not outperform the baseline model, this result suggests that the simpler linear model generalizes better for this prediction task. The random forest likely overfits extreme values in calorie density, which are heavily right-skewed in the data.

Despite the lack of improvement, this model demonstrates a thoughtful attempt to capture non-linear structure and explore more expressive modeling techniques.

## Fairness Analysis 
To evaluate whether my final regression model performs equitably across different types of recipes, I conducted a fairness analysis comparing **high-protein** and **low-protein recipes**.

### Groups 

- **Low-protein recipes:** bottom 25% of protein values
- **High-protein recipes:** top 25% of protein values

These groups were defined using protein quartiles computed from the full dataset, and fairness was evaluated **only on the held-out test set** to measure generalization. 

### Evaluation Metric 

Because this is a regression task, I used **Root Mean Squared Error (RMSE)** as the evaluation metric. RMSE is appropriate becaue it penalizes larger prediction errors more heavily and aligns with the metric used to evaluate overall model performance. 

### Hypotheses

- **Null Hypothesis (H₀):**
  The model is fair. Any difference in RMSE between high-protein and low-protein recipes is due to
  random chance. 
- **Alternative Hypothesis (H₁):**
  The model is unfair. The RMSE for high-protein recipes is larger than the RMSE for low-protein
  recipes.

### Test Statistic

I used the difference between groups:

T = RMSE(high-protein) - RMSE(low-protein)

Positive values indicate worse performance on high-protein recipes. 

### Observed Results 
- RMSE(low-protein): **48.75**
- RMSE(high-protein): **171.74**
- Observed Statistic: **122.99**

### Permutation Test

I performed a permutation test with **2,000** shuffles of the protein group labels while holding model predictions fixed. This simulates the distribution of the test statistic under the null hypothesis that group membership does not affect model performance.

- One-sided p-value: 0.0045

### Conclusion

Because the p-value is well below the 0.05 significance level, I reject the null hypothesis. There is strong evidence that my final model performs significantly worse on high-protein recipes than on low-protein recipes. This indicates a fairness concern, suggesting that the model’s predictions are less reliable for recipes with higher protein content.

<iframe 
    src="assets/fairness_perm_test.html" 
    width="800" 
    height="600" 
    frameborder="0" 
    ></iframe>
