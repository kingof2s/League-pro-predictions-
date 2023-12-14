# League-pro-predictions-
Used in dsc80 project 5
# Framing the problem
For our problem, we wanted to look at data to try and predict which team would win the game by 15 minutes. This is a binary classification and our response variable is the match result, where a value of 1 is a team winning and a value of 0 is a team losing. We made sure to only include information which is only available during the first 15 minutes. 
We looked at 

<ol>
  <li>gold difference@15min</li>
  <li>xp difference@15min</li>
  <li>First blood</li>
  <li>First Dragon</li>
  <li>First Rift Herald</li>
  <li>Turret plates taken</li>
  <li>Side</li>
</ol>  
Dragon and rift herald come spawn at 5 and 8 minutes in, and typically taken close to the time they spawn, and first blood is the first kill in a game and is awarded extra gold. While there is a chance that first blood, first dragon, and first rift herald occur after 15 minutes, we believe there are only a very small and statistically insignificant number of games where this happens, so we decided to include them in our data. 

We avoided looking at objectives like baron and elder dragon however, since those do not come into play by until after 15 minutes. 
We chose to look at accuracy because in this example false negatives and false positives are weighted equally. If we predict a team to win and they lose or if we predict a team to lose and they win are equally as wrong, unlike a covid test where a false negative is much more harmful than a false positive. 

# Baseline Model
The features we used have been listed above. Most of our features were quantitative, and we left them the same except we used stdScale to transform the gold and xp differences. Our side data was nominal, so we used onehotencoding to transform them. 
Our accuracy score for this baseline model was 0.75. We believe this model is fairly "good", as it is 50% more accurate than it would be if you guessed the winner at random. Intuitively it also makes sense. In League of Legends a teams advantages often create what's called a snowball effect, meaning getting an advantage leads to more advantages leading to eventually leading to winning the game. All of our metrics we used are likely to contribute to this snowball effect. 
However the accuracy can never reach 100, since sometimes a team will come back to win a game, even when being behind in every trackable metric in the game, and a model cannot account for that. 

# Final Model
For our final model, our biggest change was calculating our stats by team position. In a game of league of legends, having more xp and gold is more important on certain positions than others. The support role is generally designed to function well without gold, so a team with a gold advantage on the face value but all of that gold is on the support may not be in a better position to win. Knowing this, we believed that a role specific cs and xp differences would be a better indicator of success than overall differences. In particular, we believed the stats on the bot lane and mid lane to be most impactful, as those roles are typically "carries" who need a lot of gold in order to build their items and deal a lot of damage. 

We tested many different models, but we had the most success with LogisticRegression. We used a GridSearch to tune our hyperparameters. We c values of [0.001, 0.01, 0.1, 1, 10, 100], liblinear and saga solvers w different l1_ratio, and penalties of [l1(abs of coeff), l2(sq of coeff), and various combinations of the two]. We found that the best hyperparameters were C = 10,solver = 'liblinear', and penalty = l1(abs).  

In the end we had a slight but noticible improvement, with around .816 and .815 values for our testing and training accuracy as opposed to .758 and .756 from the baseline model. 
# Fairness Analysis

We chose to do our fairness analysis test to check if our model preformed similarly before and after the introduction of mythic items, which marked a major change in the game's design and was introduced in season 11. These items are typically purchased first and provide a massive advantage, meaning the value of an early gold lead may have changed significantly after their introduction. Our group x was before season 11 (pre mythic), and our group y was during and after season 11. Our null hypothesis was that the accuracy before and after mythic items were introduced comes from the same distribution, and our alt hypothesis is that they come from different distributions. The test statistic we used was the absolute mean difference, and our p value we got was 0.0. This provides strong evidence that we should reject the null hypothesis, meaning that our classification model likely is not fair when comparing matches before and after the introduction of mythic items. 
