## Recipes & Ratings Model Prediction
Analysis done by Paul J. Losiewicz
### **Framing the Problem**
   Websites such as food.com have changed the landscape of the culinary-arts world, making it incredibly easy to share and try new-recipes on the go.  Whether you are a first-time amateur or a seasoned professional, home-cooks across the world turn to food.com for its accesibility and extensive selections of recipes.  On the other hand, the same people who turn to food.com for accesibility are usually the same people who are looking for quick-and-easy recipes.  Not everyone has the time to cut out hour-long blocks of their day to cook a meal, so as a result, this project aims to build a model in order to predict how many 'minutes' a recipe takes based on its other parameters.  For example, using the >80,000 unique recipes submitted across our dataset, we are looking to see if trends in certain parameters such as 'nutritional value' or 'average rating' are able to semi-accurately predict how long a recipe is going to take.

   For this project, we are going to be building a regression model since we are predicting a numerical value, and the response value, as mentioned, is going to be the 'minutes' column. The metric we are using to assess the quality of our model is the R^2 score since it explains the variability of our predictions as opposed to the real, observed values in our response variable.  We chose R^2 as the suitable metric for our prediction problem due to the simplicity of comparison (R^2 is unitless hence it can be easy to compare models) and its clear-interpretation in terms of assessing the quality of fit in regards to our model.  In regards to the parameters we are using to predict the 'minutes' variable, we found all variables appropriate to be used for fitting-our-model.  Our problem is being analyzed in the context that all of the other information has already been collected but minutes were not provided, so we are trying to use all information available to make a suitable prediction for the 'minutes' column.

### **Baseline Model**
   For our baseline model, the features that we are using to predict 'minutes' are purely quantitative.  Some of these features were created during the data-cleaning portion of our project, but the extensive list of variables includes: 'average-rating', 'n-steps (number of steps)', 'n_ingredients (number of ingredients)', 'calories', 'sugar', 'total_fat_pdv', 'protein', and 'saturated_fat_pdv'; all of such are quantitative.  Afterwords, we processed 'average_rating', 'n_ingredients', and 'saturated_fat_pdv' using a Binarizer, and we used the mean-value in each respective column as the threshold.  Additionally, a standard-scaler was used to standardize the rest of the columns used in our model, primarily to enhance comparability and interpretability since all of these variables are in completely different scales.  Afterwords, we fit our pipeline using the preprocessing steps labeled above and a standard linear-regressor, and after using this model on our testing data, we achieved an R^2 score of ~0.117. We feel that its notable to mention that we are only training and assessing our model with recipes that take <=300 minutes our 5 hours to make, since the presencce of outliers muddys our model and shifts all of our predictions higher than the observed 'minutes' values.  Based off both the features we used to predict our response and the results we recieved from our scoring-criteria, this baseline model is obviously not able to accurately predict our response variable.  The first reason for why I believe this is we only used quantitative variables to predict our response while the nominal variables included in our original data-set have tons of information which can dictate the 'minutes' column.  For example, the 'recipe name', 'description', and 'tags' columns often include indicators that let food.com users know initially whether the recipe is quick and easy; however, it is difficult to use these variables in a baseline model without creating features to assess value (more on this later).  As much information can be pulled from the quantitative variables in our dataset, there is loads of context to be extracted from the words attached to each respective recipe.  These trends can be easily observed through the poor performance of our model in relation to our R^2 metric.  A score of ~0.117 tells us that there is an incredibly weak correlation between our predicted and observed response variables, so moving forward, we aim to create features out of our nominal variables in order to better predict the 'minutes' column accurately and efficiently.

   <iframe src="assets/comparison.html" width=800 height=600 frameBorder=0></iframe>
   This visualization is meant to demonstrate the difference in distributions between our observed 'minutes' variable and our predicted values from our baseline model.  While the distributions of each variable may seem similar, the empirical distributions do not necessarily indicate any sort of quality in our model.

### **Final Model**
  As mentioned in the previous section, in our final model, we aim to attach quantitative meaning to each of our nominal variables: 'name (recipe name)', 'tags', 'ingredients', and 'description'.  In order to do this, we start by performing a TF-IDF analysis on the description column, and we create a new variable containing the five-best words to summarize each description.  We do this since in order to efficently process a large DataFrame of descriptions, it is unreailstic to be able to read through the entire descriptions of >80,000 recipes.  As a result, we further analyze these summaries by hand-picking 10 of the most-found words in all of the summaries that appear to be relevant in potentially predicting the 'minutes' variable.  Examples of such 'relevant words' are 'original', 'easy', and 'healthy'.  Finally, we attach meaning to this feature by one-hot-encoding each of the "10-relevant words" found in the tf-idf summaries, meaning that for each respective recipe we create 10 columns, each of which corresponding to one of our "10 relevant words." For each word, if it appears in the description of the given recipe, we make the value for that column 1 (and 0 for the opposite).  Additionally, for the "name" column, we use a bag-of-words technique to hand-pick another 10 of the most used, relevant words found in the "name" column (i.e. 'chicken', 'salad', 'easy') and proceeded with the same encoding steps as described above for the 'description' column.  Since ingredients and tags are list columns, only containing "relevant" words, we combined each respective column and used the 10 most-popular words found in each column to once again create features using one-hot encoding.  This essentially created 40 new features to train our final model and hopefully improve the accuracy in which we predict the 'minutes' column.  I feel that these features improved my baseline model since they are attaching some sort of real-world context to each of our respective recipes.  For example, in our baseline model, we used strictly quantitative features, which to a degree worked, but none of which can predict the preparation-time of a recipe better than when 'easy' is in the recipe-name or '60-minutes-or-less' is in the tags.  Additionally, it is known that certain type of foods taking longer to prepare than others, and while the nutritional-values of each respective recipe may help in predicting preparation-time, none of such quantitative features can definitively tell you whether a given recipe is a 'salad' or not.  

  After encoding all of the features mentioned above, I decided to switch our modeling algorithm from LinearRegression to a RandomForestRegressor.  There were a few reasons for this; first of all, using strictly quantitative values in our baseline model, to a degree, encourages the use of linear-regression.  Now, with the 40+ features that we are adding to our model, this introduces a ton of noise in our data.  While some of the encoded columns may provide heavy correlation to our minutes-column, there are likely just as many that do not.  As a result, I decided to use the DecisionTreeRegressor object, and more specifically the RandomForestRegressor, since the DecisionTree is more robust to irrelevant features.  Additionally, I further decided to use the RandomForestRegressor to help with the variability of a single DecisionTreeRegressor object.

  Now that we are using a RandomForestRegressor, a GridSearchCV was used to determine the best hyperparameters to fit our model.  The hyperparameters analyzed were 'max_depth' and 'min_samples_leaf', each of which were chosen since they dictate the complexity of our tree and the option for our forest to either overfit or underfit our model.  After performing the grid-search, it was observed that a 'max_depth' of 10 and a 'min_samples_leaf' of 250 most accurately predicted preparation-time through our model.  Combining such hyper-parameters with all of the encoding done above completes our final model, leading to the results of our predictions.  The R^2 of our final model using the same dataset as before was a ~0.426, a significant increase from the metrics collected in our baseline-model.  While 0.426 is still a fairly low value in terms of correlation between our predicted and observed response-variable, this can be attributed to the variability of our response variable and the difficulty of predicting preparation time from the data provided.  

  <iframe src="assets/comparison2.html" width=800 height=600 frameBorder=0></iframe>
  The visualization above once again plots the respective empirical distributions of our observed response variable and our predicted response variable.  I find it notable to discuss how the empirical distribution of our model's preidctions are now heavily skewed to a couple given bins, but the R^2 value of our final model significantly outscores the metrics of our baseline model.  Additionally, our final model predicted a couple of outliers in the dataset, while in the baseline, almost all of the predicted values hovered arond the same general area.

### **Fairness Analysis**
   Now that we have completed the implementation of our final-model, we look to conduct a fairness analysis of our model using a specific partition of our training-data.  In this section, we look to analyze if our model predicts more accurately for recipes with an average-rating of 5-stars as opposed to all other recipes.  Surprisintly, recipes with an average score of 5-stars make up ~58.9% of our data-set, so there is a significant amount of data to be used in both groups for our permutation test. Our null-hypothesis is that our final-model performs just as well for recipes with an average-rating of 5-stars in comparison to all other recipes, and our alternative suggests that our model performs better for non-5-star recipes.  The test-statistic we are using is the difference in R^2 values between 5-star recipes and all other recipes.  Using a significance value of 0.05, our permutation tests resulted in a p-value of 0.003, suggesting that only 0.3% of the simulated test-statistics resulted in a greater difference in performance between 5-star recipes and all other recipes.  This test suggests that our model predicts preparation-time more accurately for recipes not rated 5-stars under a significance level of 0.05.
