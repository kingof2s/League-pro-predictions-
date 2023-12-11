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
Dragon and rift herald come spawn at 5 and 8 minutes in, and typically taken close to the time they spawn, and first blood is the first kill in a game and is awarded extra gold. While there is a chance that first blood, first dragon, and first rift herald occur after 15 minutes, we believe there are only a very small and statistically insignificant number of games where this happens, so we decided to include them in our data. We avoided looking at objectives like baron and elder dragon however, since those do not come into play by until after 15 minutes. 
We chose to look at accuracy because in this example false negatives and false positives are weighted equally. If we predict a team to win and they lose or if we predict a team to lose and they win are equally as wrong, unlike a covid test where a false negative is much more harmful than a false positive. 

# Baseline Model
The features we used have been listed above. Most of our features were quantitative, and we left them the same except we used stdScale to transform the gold and xp differences. Our side data was nominal, so we used onehotencoding to transform them. 
