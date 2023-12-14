# League-pro-predictions-
Used in dsc80 project 5
# Problem Identification
League of Legend games often seem very unpredictable; a seemingly easy game can be lost at the flip of a coin. We want to use data to explore the factors that could help us predict the outcome of a game early on.

In our model, we used data from all the professional matches from 2017 until 2022 to predict the winner of the game. As such, we will use a **Binary Classification** algorithm to predict the response variable, ``result`` which is ``0`` if a team *loses*, and ``1`` if a team *wins*. This directly corresponds to predicting the winning team in a game.

We then chose accuracy as a metric to evaluate our model, seeing as how each game of LoL must have exactly 1 winner and 1 loser, which means that the data is very balanced. As such, accuracy is a better metric than precision, recall, and the f-1 score.

Our focus is to predict the outcome of a game only 15 minutes in, as such we will only take data that can be measured/calculated around that time. That means we cannot use features that are only available after the first 15 minutes of the game, such as ``firstbaron``, ``elder``, or ``inhibitor``. The data that would be available to us, however, are the gold and exp earned by each player at 15 minutes, early objectives like ``firstdragon``, ``firstherald``, ``turretplates``, and ``opp_turretplates``, and very early fights that determine which team has ``firstblood``: the first kill of the game.

To be more specfic, the ``firstdragon`` spawns at 5:00 and is an important objective that gives powerful bonuses, so most professional teams would take it before 15:00. The ``firstherald`` spawns at 8:00 and can greatly help in destroying enemy turrets, so it is also a high priority objective. ``turretplates`` and ``opp_turretplates`` are only available until 14:00, after which they disappear, so that data is available at 15:00. Similarly, turrets are often quite quickly after turretplates disappear at 14:00, sometimes even before. It is rare that all turrets are intact after that, so this data should also be available. 


# Baseline Model

Our baseline model takes in some game statistics available before 15:00 of the game, and makes a prediction on whether a team with those given stats will win or not.

**Quantitative Features**: ``turretplates``, ``opp_turretplates``, ``xpdiffat15``, ``golddiffat15``

**Nominal Features**: ``firstherald`` (1/0), ``firstdragon`` (1/0), ``firsttower`` (1/0), ``firstblood`` (1/0), ``side`` (red/blue)

**Response Variable**: ``result`` (1,0) where
1 means win, 0 means lose

We first encoded ``side`` with a OneHotEncoder, as it is the only nominal feature that is not binary. We then used a FunctionTransformer inside a ColumnTransformer on ``turretplates`` and ``opp_turretplates`` to calculate a new feature, ``platediff`` that indicates the difference between the plates earned by the team and their opponent. All other features are passed through as-is into our Logistic Regressor, which we used GridSearchCV to tune the hyperparameters of.

### Model evaluation


Our accuracy score for the baseline model was 0.76, or 76%, which we think is fairly reliable seeing as we only had the first 15 minutes of a game that usually takes 31 minutes, meaning we only have less than half of the available data that can be gathered throughout the whole game. Also, an important aspect of League of Legend matches is the snowball effect, where a player earning an early advantage leads to more advantages for their team as a whole, which eventually cascades into their team winning the game from them continually extending their advantage. All of the metrics we used (other than ``side``) contributes to this snowball effect by either providing more xp, more gold, or numerical increases to the teams' combat ability.


# Final Model

For our final model, we added several new features:


1.  Direct additions: ``killsat15``, ``deathsat15``, ``assistsat15``, ``cspm``, ``vspm``, ``dpm``

These features were available in the original dataset, and we found them to be possibly correlated with the ``result`` of the game.


2. Engineered features: ``top_golddiffat15``, ``jng_golddiffat15``, ``mid_golddiffat15``, ``bot_golddiffat15``, ``sup_golddiffat15``, ``top_xpdiffat15``, ``jng_xpdiffat15``, ``mid_xpdiffat15``, ``bot_xpdiffat15``, ``sup_xpdiffat15``


These features break down the existing ``golddiffat15`` and ``xpdiffat15`` more specifically into the positions in LoL: top, jungle, mid, support, and bot. Different positions are typically strong at very different points in the game, so being more specific with which player has an advantage can be important to the model's accuracy.


3. Scraped features: ``team_mobility``

This feature was scraped from the official Riot API and the well-established CommunityDragon API, indicating how many champions in the team has some form of mobility skill in their kit. We believe this is important, as champions without mobility, especially the fragile bot laners, are much more vulnerable to assassins since they cannot run away as quickly. Knowing the number of "mobile" champions can tell us whether the team is able to manoeuvre quickly around the map and more importantly, their enemies.

### Model selection

We tested many different models, but we had the highest train and test accuracy with ``LogisticRegression``. We used ``GridSearchCV`` to tune our hyperparameters: 
1. the ``C: [0.001, 0.01, 0.1, 1, 10, 100]``, 
2. ``solver: ['liblinear', 'saga']``
3.  ``penalty: ['l1', 'l2', 'elasticnet']`` 

We found that the best hyperparameters were ``C = 10``, ``solver = 'liblinear'``, and ``penalty = l1``.  

### Model evaluation (Final)

Our test accuracy score for the final model is 0.82, or 82%, which is an improvement over our baseline model.


# Fairness Analysis

<<<<<<< HEAD
### Grouping choice
***Does the model perform differently for games played before/after the introduction of mythic items in Season 11?***

The introduction of mythic items in S11 marked a major change in the game's design and strategies. We took the Season number from the ``patch`` column by taking the numbers before the comma (i.e. ``patch 12.13`` indicates ``Season = 12``). 

We split the dataset into games where ``Season < 11`` (S7-S10) and groups where ``Season >= 11`` (S11-S13)

### Null Hypothesis
Our model's accuracy is the same for both groups, and any differences in the observed statistic is due to random chance

### Alternative Hypothesis
Our model performs differently for games with mythic items, compared to games without.

### Test Statistic
The absolute difference in model accuracy for the two groups

### Significance level
0.01

### Permutation Distribution


![Permutation Distribution]()


 Our null hypothesis was that the accuracy before and after mythic items were introduced comes from the same distribution, and our alt hypothesis is that they come from different distributions. The test statistic we used was the absolute difference in accuracy, and the p-value we got was 0.00. As such
=======
We chose to do our fairness analysis test to check if our model preformed similarly before and after the introduction of mythic items, which marked a major change in the game's design and was introduced in season 11. These items are typically purchased first and provide a massive advantage, meaning the value of an early gold lead may have changed significantly after their introduction. Our group x was before season 11 (pre mythic), and our group y was during and after season 11. Our null hypothesis was that the accuracy before and after mythic items were introduced comes from the same distribution, and our alt hypothesis is that they come from different distributions. The test statistic we used was the absolute mean difference, and our p value we got was 0.0. This provides strong evidence that we should reject the null hypothesis, meaning that our classification model likely is not fair when comparing matches before and after the introduction of mythic items. 
>>>>>>> 33b077109a16cdc554407de293b91c70ca2fb92c
