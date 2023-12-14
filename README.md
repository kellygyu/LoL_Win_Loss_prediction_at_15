# FF Angle -- Is it over?

## Project Overview
The following project was completed for DSC80 at UCSD. This project aims to create a prediction model to determine the outcome of professional leauge matches given the data collected 15 minutes into a game.

**Author**: Kelly Yu

## Jargon Definitions
Throughout this project, I will be using some jargon particular to League of Legends. I have included a short description of what each word means below for better understanding. 

| Jargon | Definition |
|:-----------:|:-----------:|
| Champion | characters you play as in game |
| Lead | being ahead of an opponent, such as having more resources |
| Snowball | increasing the lead a team has against the other team |
| XP | experience points -- a measurement of player progression within the game |
| Objectives | game quests, such as slaying a dragon, which can reward your team with gold or buffs |

## Background and Problem Identification
League of Legends is a game purely focused on game understanding, champion mastery, macro/micro gameplay, acquiring objectives, and teamwork. A game's result is determined by a multitude of factors such as strategy, objectives, resources, kills, xp, and gold differences between both teams. If by chance a team obtains a lead in any of these categories, they will have a chance to snowball and ultimately win the game. In League of Legends, games tend to last between 25-40 minutes, and a majority of objectives can be acquired pre-15 minute mark. Game objectives help teams create a gap between them and their opponent as these objectives tend to grant gold bonuses or buffs to teams. 

Players can usually tell which team is ahead and likely to win by the objectives taken and kills within the first 15 minutes. Additionally, League of Legends also has a surrender option, also known as the term 'ff'. Teams will typically try to ff if they think a game is unwinnable, and would like to save time and hop into the next game rather than play the current game out. The earliest a team can ff or forfeit the game is 15 minutes into the game. Hence for this project, we will try to predict the outcome of a game based off of team metrics 15 minutes into each game. This model will help players understand when a game can be considered as truly lost and save players from playing a lost game, or if there is a slim chance at a miracle comeback victory. 

### Model Formation and Explaination
To create this model, we will be using a binary classification method as our response variable, `result`, is a binary column of 1's and 0's to represent the outcome of a game -- win or loss. We will start off with Decision Tree Classification, and alter the model after. For our model evaluation metric, we will be using *Accuracy* as our metric of choice. Why Accuracy? Within our dataset, for each game, we have data from the red and blue team. Therefore, each game will always have a winner and loser, 1 and 0, hence the number of wins in our dataset should always equal the number of losses within our dataset. This means that our datset will not have any class imbalance between the number of wins and losses, hence accuracy will work well as a metric here as there is no cavaet (class imbalance) to skew the metric.

### Dataset Explaination
The dataset we will be using will be from Oracle's Elixir. The dataset itself contains League Esports data from 2022 and 2023, however we will be filtering out data from 2023 to only focus on competitive matches that occured in 2022 and only on matches that occured in the Major Leagues. The leagues we will focus on will be all the Tier 1 leagues, as well as the Mid Season Invitational and World Championships. We will not be using data from Tier 2 leagues and smaller tournaments as the skill disparity in those leagues can vary heavily, meaning the match data we receive from those games could heavily influence our model.

**Column Focus and Descriptions**

After cleaning the data, we are left with 2857 games, aka 5714 rows. We will rely on the following columns to build and train our models. These are all metrics that can be obtained 15 minutes into each game. All metrics are focused strictly on statistical measurements and resources acquired in the game.   

| Column Name | Description |
|:-----------:|:-----------:|
| side | `Red` or `Blue` for each team|
| result | 1 if team win 0 if loss |
| firstdragon | 1 if team acquired the first dragon 0 elsewise |
| firstherald | 1 if team acquired first herald 0 etherwise |
| golddiffat15 | gold difference between both teams at 15 minutes |
| xpddiffat15 | experience points difference between both teams at 15 minutes |
| killsat15 | total amount of kills a team has at 15 minutes |
| deathsat15 | total amount of deaths a team has at 15 minutes |
| assistsat15 | total assists a team has at 15 minutes | 
| firstblood | 1 if a team obtained the first kill in the game 0 elsewise |


Additionally, we will use two extra column we defined from the data.


| Column Name | Description |
|:-----------:|:-----------:|
| teammates_ahead_at15 | an integer between 0-5 representing the number of teammates with a gold or xp advantage compared to their opponent|
| turretplatesdiff | the difference in turret plates obtained by a team and their opponents|

**Splitting the Data**
To train and test our models, we split the data into a 80/20 split for train and test data. For our basline model, we will take the same train and test data and keep the columns used for our model. For the final model, we will use the train and test data with all columns. 


## Baseline Model
For the baseline model, we will start off with a simple `DecisionTreeClassifier` model to predict `result` with 3 attributes. No parameter adjustments were made to the model. The columns of interest here will be `side`, `xpdiffat15`, and `golddiffat15`. 

**Response Variable**: `Result` -- 1 (Win) or 0 (Loss)

**Features**

| Column Name | Feature Type | Encodings |
|:-----------:|:-----------:|:-----------:|
| side | Nominal | One Hot Encoded|
| xpdiffat15 | Quantitative | StandardScaler() |
| golddiffat15 | Quantitative | StandardScaler() |

`side` was one hot encoded as it is a nominal data type of `Red`s and `Blue`s. All other quantative features were transformed with StandardScaler() as their values are quite large.

Why these features? Gold and xp difference are big factors of the game. Gold helps players purchase more items and increase their damage output while reducing damage taken. The same also apply to xp difference. A higher positive xp difference mea that players can output more damage by leveling up their skills and getting higher base stats (armor, damage, magic reduction, etc). Therefore, these two variables help teams in creating leads and gaps, which could ultimately allow teams to snowball into a victory. `Side` plays a smaller role in this model, however the side a team plays on may allow teams to have a slight advantage over certain objectives. Hence, why `side` is considered in our baseline model. All together, these 3 features form a decent foundation for our baseline model.

### Result
After training our model and running the test predicitions, our model achieved an accuracy of 0.698. If we look at it from a model assessment standpoint, this is a terrible model, as this means that we predicted about 69.8% of the game results correctly for our test set. However, in reality, 0.698 is actually pretty good if we considered the data we used. To reiterate, the whole purpose of this project is to figure out whether it is possible to predict a game's outcome with only game data from the first 15 minutes of the game. Considering each game lasts around and over 30 minutes, we techinally only have data for about the first half of each game. The other half of the game is completely variable, and unknown. Therefore having an accuracy of over 0.50 is impressive given that we barely know data from half the game.


## Final Model
In our final model, we will keep the same features used in our *Baseline Model* (see Baseline Model Features section for explaination of features). We will add additional features to improve our accuracy. 

**New Features**
Our baseline model had pretty significant features that created a sound foundation for our final model. As mentioned, gold and xp difference provide great insights on how a team is performing 15 minutes into the game. However, gold and xp are not the only determinants of a team's advantage 15 minutes into the game.

`turretplatediff` - Quantative
Turrets play a crucial role in the first 15 minutes of each League Game. Turrets act as a barrier to stop enemies from progressing into a player's base/territory, but in the first 14 minutes of the game, turrets have a feature called turret plating, which grants players gold with milestones for damage dealt to the turret. Each turret plate grants a player or multiple players a spilt gold pool of 160 gold. Therefore, the more turretplates a team has compared to their opponent, the more gold they over their opponent, which could be used to buy items and increase damage output. Hence adding this feature in will help us improve our knowledge about a team's gold gap over their opponents. We will not be applying any transformations on this feature as it's distribution is rougly normal.

`firstdragon` - Nominal Feature (Binary 1/0)
Dragons act to provide permananent buffs to the teams that slay the dragon. Each dragon type has different buffs that can be given to its slayers. To name some examples, slaying an inferno drake can provide a team extra 6% damage. Additionally, killing a dragon can grant a team between 75 to 300 gold as an objective. The first dragon spawns 5 minutes into a League of Legends game, and is usually a target objective as soon as it spawns, meaning first dragons are usually killed before 15 minutes into a game. Adding this feature would help our model accuracy as this feature allows us to determine which team received extra gold and a buff to their stats, garnering them an advantage over their opponent. 

`firstherald` - Nominal Feature (Binary 1/0)
The Rift Herald plays a similar role as dragons do. Rift Herald spawns 8 minutes into each game and is similarly usually killed before 15 minutes as it is also a heavily prioritized objective. By slaying a herald, a team gains 100 gold and 200 experience points. Additionally, a player will be able to summon the herald on any lane, where it will target a turret and deal 2 plates worth of turret damage in exchange for a percentage of the herald's health. Therefore, adding this feature will do the same as adding `firstdragon`, which can help us predict which team has a xp and gold lead over their opponents.

`firstblood` - Nominal Feature (Binary 1/0)
First blood, aka first kill, grants the killer 400 additional gold and 42-990 xp to all allies who helped with the kill. Adding this feature in will help our model in determining gold and team advantage. 

`killsat15`, `deathsat15`, `assistsat15` - Quantative
Kills grant 300 gold if it is not the first kill, and additional 42-990 xp to the player and their allies that helped. 
In contrast, deaths will grant the same bonuses as kills, except to the enemy team. Assist tells us the amount of times players have partipicated in kills, which means the amount of times players have also received gold and xp for helping their ally. Therefore all together, these features will also help our model in determining how much ahead or behind players are strictly from kills.

`teammates_ahead_at15` - Quantative
This is a new feature that we created earlier when data cleaning, which tells us how many players on the team have an level or gold advantage over their opponent. This essentially quantifies the extent of how ahead a team can be over their opponent. Having higher number of teammates who have an advantage means a higher chance those teammates win their lane and can help other teammates out, or win team fights. Hence this will also be a good feature to include in our model.


## Finalizing the Model
In our baseline model, we chose to use a Decision Tree Classifier model without any parameters. For our final model, we will switch to using a Random Forest Classifier model which trains more decision trees and can reduce overfitting. For the features we added above, we will apply StandardScaler() on the quantitative features `golddiffat15` and `xpdiffat15` and QuantileTransformer() to `killsat15` and `deathsat15` to account for the negative skewness of these two columns. Else, we will OneHotEconder() on `side` similar to the baseline model. As for the rest of the features, we will leave them as it is. 

On the base RandomForestClassifier model without any parameters, training it on our train and predicting with our test set saw an improvement of 0.05. Hence, we will choose this model and run GridSearchCV to find the best parameters on it. After running GridSearch on our defined hyperparameters list, the best parameters that worked out were with criterion `gini` with no max depth and a min sample split of 50, and n_estimators of 20. After including these new features and running the model, our final model model accuracy skyrocketed to 0.757, a 0.06 improvement from our baseline model. It seems that optimizing model parameters and adding the new features helped our model generate better predictions. As mentioned, this can be considered as a pretty good model, due to the information we still lack about more than half duration of the game which could be used to predict the outcome of the game. 

## Fairness Analysis
To analyze whether our model performs equally as well across different data groups, we will run a fairness analysis test on our data. We will attempt check if our model performs better for teams with more than 7 kills within 15 minutes or teams with less kills within 15 minutes. Why 7? In profession League of Legends, players tend to play extremely carefully and together, always weary of where their opponents are. Additionally, kills grant gold bonuses from a range of 150 gold to nearly 1000 (for killing fed enemies) to the slayer and their team. Hence most players would want to avoid giving out free kills to their enemies. We can additionally see from the distribution of kills below, that the median number of kills before 15 minutes hover around 5 kills. This could infer that more kills tend to give a team a bigger advantage in wining, so we'd like to see if our model actually performs better or worse for teams that may have a high amount of kills at 15 minutes.

<iframe src="charts/kills_dist.html" width=800 height=600 frameBorder=0></iframe>

### Framing the Test
We will run a permutation test with 10,000 simulations for fairness with difference in accuracy.

**Null Hypothesis**: The model is fair and the accuracy for teams with greater than 7 kills is the same as teams with less than 7 kills at 15 minutes.

**Alternative Hypothesis**: Meta champions have a higher kda compared to compared to other champions.

**Type of Test**: The model is unfair and the accuracy for teams with greater than 7 kills is higher than its counterpart.

**Test Statistic**: Difference in accuracy

**Significance Level**: 0.05, the base level for most hypothesis tests.

**Results**
From this test, we reject the null, as our p-value, 0.0 is less than the significance level of 0.05. We can conclude that our dataset shows there is a statistically significant difference between the the Accuracy of our model for teams with larger that 7 kills at 15 minutes.

<iframe src="charts/fairness.html" width=800 height=600 frameBorder=0></iframe>


## Conclusion
To wrap up this project, it seems that with enough data 15 minutes into any league game, we are able to predict the outcome of the game semi accuractely. However, this model is still far from perfect as there are still many factors to consider when it comes to analyzing League of Legends game data. For example, because of the scope and timeframe of this project, I was unable to focus on champion matchups or team compositions. However, champion matchups and compositions would be interesting to investigate, as League of Legends has over 152 champions, many of which can be classified as early or late game champions. Team comps consisting heavily of early game champions may have a larger advantage over other compositions in objectives or kills. If I were to improve on this model in the future, these are some ideas that I will keep in mind and implement for future editions. 