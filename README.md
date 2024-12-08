# League of Legends Gold Difference Analysis

## Introduction
League of Legends (aka League or LoL) is a multiplayer online battle arena video game developed and published by Riot Games. It has millions of players in its community as well as a thriving competitive Esports scene. Esports is a form of professional competitive video games that has often been framed as the next big thing in sports. In fact, League boasts some of the most watched Esports events ever, with the recent Worlds 2024 championship event.

As an avid watcher of competitive League, there are often times that my favorite team is losing, yet I hold out hope that they will be able to comeback. Or, my favorite team might be winning, and I hope that they don't "throw" the game (make major mistakes to lose in a completely winnable situation).

How are teams identified as winning or losing in League? Casters and analysts often analyze this with the gold value. Teams that have more gold will have the ability to buy more items that increase their team's power, and with increased power they will be able to accumulate more gold through acquiring kills or objectives (e.g. towers, dragons, heralds, barons). In essence, gold theoretically has a snowballing effect in League. So, I wanted to see this pattern statistically: **What degree of impact does gold difference have on the result of a game? (i.e. What are the chances that a team can come back from a gold deficit?)**

To answer this question, I will be looking at file records from competitive League matches in the 2022 season. This dataset contains 150,180 rows. For this question, I will be looking at the following columns:

* `team`: What team is playing. There are 2 teams in each game, with 5 players per team.
* `position`: What role that player is playing, i.e. `top`, `mid`, `jng`, `bot`, `sup`, or `team`. `team` indicates that this row displays the aggregate result for that specific team.
* <mark> `result`: The result of the game, `1` for a win, `0` for a loss
* `gamelength`: How long the game lasted, in seconds
* `golddiffat10`, `golddiffat15`, `golddiffat20`, `golddiffat25`: Gold differences at the 10, 15, 20, and 25 minute marks of the game, respectively. Gold difference is the difference between this team's gold amount and the enemy team's gold amount.
* `killsat25`: The number of enemy kills a certain team got by the 25 minute mark. Each kill is worth 100-1000 gold, depending on how "valuable" the kill was.
* `assistsat25`: The number of assists a certain team accumulated by the 25 minute mark. An assist occurs when a player helps deal damage that leads to the killing of an enemy, whilst not dealing the final blow. For example, a kill could have multiple assists if multiple players helped out with it, but would only count as one kill. Each assist is worth 50-150 gold, depending on how "valuable" the kill was.
* `deathsat25`: The number of deaths a certain team accumulated by the 25 minute mark. A death occurs when a player's health bar is depleted. This can be due to damage from multiple sources, like other players, monsters, or structures.
* <mark> `firstdragon`: Whether this team was the first to slay a dragon, `1` if they were first and `0` otherwise. Dragons grant some additional power to the team that slayed it, as well as some gold.
* <mark> `firstherald`: Whether this team was the first to slay a rift herald, `1` if they were first and `0` otherwise. Rift heralds grant some additional structure-taking power to the team that slayed it (which can help take down towers), as well as some gold.
* <mark> `firstbaron`: Whether this team was the first to slay a baron, `1` if they were first and `0` otherwise. Barons grant some additional structure-taking power to the team that slayed it (which can help take down towers), as well as some gold.
* <mark> `firsttower`: Whether this team was the first to take down a tower. `1` if they were first and `0` otherwise. Towers grant 50-250 gold per player and provide strategical advantages to the team that took it down.
* <mark> `firstmidtower`: Whether this team was the first to take down the mid lane (most strategically advantage) tower. `1` if they were first and `0` otherwise. The mid tower provides additional strategical advantages to the team that took it down.

## Data Cleaning and Exploratory Analysis

### Data Cleaning

In order to clean the data, I first indexed the data to only include the rows with `team` in the `position` column. These were the rows holding a team's summary statistics. I then filtered for only the needed columns (thos listed out in the previous section). Next, there were many columns that I wanted to convert into `boolean` types. These columns are highlighted in the previous section's column analysis. Additionally, I dropped rows that were missing gold difference data. I did that by filtering for rows had `NaN` values in the `golddiffat10` column. Finally, I created additional `boolean` type columns that represents if a team has a gold advantage at 10, 15, 20, and 25 minute marks of the game. These were represented by `goldadv10`, `goldadv15`, `goldadv20`, `goldadv25` columns and were derived by the `golddiffat10`, `golddiffat15`, `golddiffat20`, and `golddiffat25` columns, respectively. Below is the head of the cleaned data.

| gameid                | result   |   gamelength |   golddiffat10 |   golddiffat15 |   golddiffat20 |   golddiffat25 |   killsat25 |   deathsat25 |   assistsat25 | result   | firsttower   | firstmidtower   | firstblood   | firstbloodkill   | firstbloodassist   | firstbloodvictim   | firstherald   | firstdragon   | firstbaron   | goldadvat10   | goldadvat15   | goldadvat20   | goldadvat25   |
|:----------------------|:---------|-------------:|---------------:|---------------:|---------------:|---------------:|------------:|-------------:|--------------:|:---------|:-------------|:----------------|:-------------|:-----------------|:-------------------|:-------------------|:--------------|:--------------|:-------------|:--------------|:--------------|:--------------|:--------------|
| ESPORTSTMNT01_2690210 | False    |         1713 |           1523 |            107 |           -944 |             88 |           6 |            7 |            12 | False    | True         | True            | True         | True             | True               | True               | True          | False         | False        | True          | True          | False         | True          |
| ESPORTSTMNT01_2690210 | True     |         1713 |          -1523 |           -107 |            944 |            -88 |           7 |            6 |            22 | True     | False        | False           | False        | True             | True               | True               | False         | True          | False        | False         | False         | True          | False         |
| ESPORTSTMNT01_2690219 | False    |         2114 |          -1619 |          -1763 |          -5140 |          -7280 |           1 |            8 |             1 | False    | False        | False           | False        | True             | True               | True               | True          | False         | False        | False         | False         | False         | False         |
| ESPORTSTMNT01_2690219 | True     |         2114 |           1619 |           1763 |           5140 |           7280 |           8 |            1 |            13 | True     | True         | True            | True         | True             | True               | True               | False         | True          | True         | True          | True          | True          | True          |
| ESPORTSTMNT01_2690227 | True     |         1972 |           -103 |           1191 |           1744 |           4145 |           5 |            2 |            12 | True     | True         | True            | False        | True             | True               | True               | False         | True          | True         | False         | True          | True          | True          |

### Univariate Analysis

I performed univariate analysis on the gold difference values at the 25 minute mark.

<iframe
  src="assets/fig25.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram shows that the distribution of gold difference is normal. It is completely symmetric. This is due to the fact that a team's gold difference is the difference between their gold amount and the enemy team's gold amount. Thus, a team and their opponent will have complementary gold difference values. That is, they will add up to 0. What's important to note here is that most gold differences seem to be between the 10k amount at the 25 minute mark. In the context of the research question, this graph is helpful to understand what gold differences usually look like at different points of the game.

### Bivariate Analysis

I performed a bivariate analysis on the gold advantage and result statistics in the dataset. This is to visualize among teams with gold disadvantages at a certain point of the game, what percentage are able to come back and win the game.

<iframe
  src="assets/pie25.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

In regards to the initial question, this plot shows how likely it is that a team will be able to come back from a deficit at the 25 minute mark. According to the plot, teams with gold deficits at the 25 minute mark have a win rate of about 20%. This shows that it is very unlikely that a team will be able to come back from such a gold deficit, but not wholly impossible.

### Interesting Aggregates

Here are some more interesting aggregates about the gold difference amounts.

|   golddiffat10 |   golddiffat15 |   golddiffat20 |   golddiffat25 |
|---------------:|---------------:|---------------:|---------------:|
|       -691.987 |       -1726.75 |       -3006.34 |        -4768.9 |
|        691.987 |        1726.75 |        3006.34 |         4768.9 |

I first grouped by the game result and then calculated the average gold differences at the 10, 15, 20, and 25 minute marks of the game. In regards to the research question, this is helpful to understand what gold differences usually look like at different points of the game.

## Framing a Prediction Problem

From the last section, we learned the importance of gold advantages to the game. We found that coming back from gold deficits are rare but not impossible. So, my prediction problem is: **Are we able to predict if a team will be able to come back from a gold deficit based on other game statistics?** This is a **binary classification** problem. The response variable is an additional column, **comeback**, a `boolean` type that signifies if a team that is at a gold disadvantage by the 25 minute mark managed to come back and win. I chose it because of the impact that gold disadvantages usually have on the game. I chose the 25 minute mark because, as I saw from past analysis, this is a point where gold difference amounts become very significant.

I will be using accuracy and F1-score to evaluate my model. The reason that I wanted to use F1-score in addition to accuracy is because the data is unbalanced. In the DataFrame, 20% of teams at a gold disadvantage manage to come back, and 80% of teams at a gold disadvantage don't end up pulling it off. So, the accuracy score alone won’t give us a representative evaluation of the model.

At the time of prediction, I am assuming that we would be 25 minutes into the game. Thus, the only known values at this point of the game would be `golddiffat25`, `killsat25`, `deathsat25`, `assistsat25`, `firstdragon`, `firstherald`, `firstbaron`, `firsttower`, `firstmidtower`.

## Baseline Model

For the baseline model, I used a Decision Tree Classifier, with the following four features: `golddiffat25`, `killsat25`, `deathsat25`, `assistsat25`. All four features are quantitative, so I utilized a StandardScaler Transformer to transform them into a standard scale.

After fitting the model, our accuracy score on unseen data is 0.7347. This means that this model is able to correctly predict 73.47% of data. This accuracy score may sound decent, but it is quite misleading since our data is unbalanced. This model has many false negatives, i.e. comebacks that the model does not identify as comebacks. So, the F1-score for comebacks is 0.37, which is rather low.

## Final Model

In this final model, I added five more features: `firstdragon`, `firstherald`, `firstbaron`, `firsttower`, and `firstmidtower`. I added these features into the model because in League, other than gold, the other important items are objectives (e.g. dragons, heralds, barons) and structures (e.g. towers). Thus, it might be useful to identify which team is first to reach these goals in order to predict whether they're able to comeback.

This final model also uses a Decision Tree Classifier in alignment with the baseline model. The five additional features I added are all boolean values. They are all nominal categorical variable already in binary form, so I did not need to perform more encodings. In terms of tuning hyperparameters, the two hyperparameters I chose are: max depth and the minimum number of samples split. I'm testing max depth of up to 10. For the min number of samples split, I'm testing 2, 5, and 10 sample split. Using the technique of grid search to find the best hyperparameters, I found out that the best max depth is 5 and the best minimum sample split is 2.

The accuracy score is now 0.8816, meaning our model is able to correctly predict 88.17% of unseen data. This is much better now. This model also has much less false negatives. Taking a look at f1-score for True comebacks, it is 0.73, meaning both precision and recall are much closer to 1 compared to before. I have achieved huge improvement in both evaluation metrics, and this improvement suggests that our adjustment to the model is effective in terms of prediction power.