# Are Meta Champions Truly Broken?

## Project Overview
The following project was completed for DSC80 at UCSD. This project aims to investigate the difference in winrate and performance of heavily banned champions vs other champions within league of legends tournaments.

**Author**: Kelly Yu

## Jargon Definitions
Throughout this project, I will be using some jargon particular to League of Legends. I have included a short description of what each word means below for better understanding. 

| Jargon | Definition |
|:-----------:|:-----------:|
| Champion | characters you play as in game |
| Meta | describes whether a champion is popular |
| KDA | calculated metric that describes a player's performance in game base off kills, deaths, and assists|

## Background
League of Legends is a game purely focused on game understanding, champion mastery, macro/micro gameplay, and teamwork. There are currently over 152 champions as of 2022, each with their own skillsets and playstyles. In professional matches, a team's chance of winning is not only heavily dependent on the champions they select, but also the champions they ban from the game, which reduces their enemy's ability to play such champions. Banning certain champions could heavily increase a team's winrate multiple ways, for example, banning strong champions could reduce a team's chance of losing lane due to having to play against a "broken" champion. 

With each season, we tend to see the same champions being banned in profession league matches, as generally these champions are known to be part of a meta, or just have great utility for teamfights overall. Therefore, in this project, we will explore whether the top most banned champions truly have a higher win rate and performance distribution compared to the rest of the champion pool. 

### Dataset Explaination
The dataset we will be using will be from Oracle's Elixir. The dataset itself contains League Esports data from 2022 and 2023, however we will be filtering out data from 2023 to only focus on competitive that occured in 2022. The overall dataset has a total of 123 columns. After filtering and cleaning, our final dataset for hypothesis tests has a total of 107257 rows, and 11 columns. For Missing Data investigations, we will be using subsets of the original dataframe, at 121010 rows and 4 columns.

**Column Focus and Descriptions**

For our analysis, we will only focus on the columns defined below. All other columns are not as relevant for our needs and will be discarded.

| Column Name | Description |
|:-----------:|:-----------:|
| gameid | id corresponding the played match |
| league | the tournament name |
| result | True if team win False if loss |
| year | year of tournament |
| position | the player's role in game |
| champion | the champion name that a player plays as |
| kills | number of opponents a player has slain |
| deaths | number of times a player has died |
| assists | number of times a player assisted in killing an opponent |
| ban1 ... ban5| the champion banned within a match|
| goldat10 | the gold a player or team has 10 minutes into the game |


Additionally, we will use two calculated columns we defined.


| Column Name | Description |
|:-----------:|:-----------:|
| kda | [(kills + assists)/deaths] which is a performance metric |
| meta | true if a champion is heavily banned false if not |


## Data Cleaning & EDA

### Data Cleaning

As our dataset contains a lot of extra columns, we will first drop all the columns that won't be used for our analysis. Additionally, the 2022 dataset we downloaded contains some matches from 2023 preseason, hence we will also drop any match data with the "year" 2023. After removing these matches, we have a total of 12101 rows. We will also conver the `"result"` column to boolean type. Our next steps will be to create new columns to measure performance and determine whether a champion is heavily banned or not. We define these columns to be "meta" and "kda". 

**Defining `"Meta"`**

As defined in our jargon, meta refers to the popularity status of a champion. Here, we will use the term `"meta"` as a reference to whether a champion is popularly banned. As there are 152 champions in League of Legends, if we take all champions as equal/fair, each champion would techinally have about a 1/152 chance of getting banned. As there can be up to 10 bans per game, the chance of a certain champion being banned within one game is 1 - (151/152) x (150/151) x ... (142/143), which is 5/76 or about 6.6% chance of getting banned. 

Hence, using this probability as a baseline, let's define a champion to be `"meta"` (banned a lot) if their ban rate is larger than 15%, about double the fair chance of a champion being banned. 15% is high for a champion if they are being banned in at least 15% of professional tourament matches given there are 152 other champions that could also be banned. Using this definition, we will find the `"meta"` champions. 


Here is the `"meta"` list of champions we found after taking a look at the ban rates of all champions.

`['Zeri', 'Gwen', 'LeBlanc', 'Lucian', 'Ahri', 'Kalista', 'Wukong', 'Akali', 'Yuumi', 'Caitlyn', 'Sylas', 'Renata Glasc', 'Lee Sin', 'Karma', 'Nautilus', 'Taliyah', 'Gangplank', 'Corki', 'Azir', 'Camille']`

As a league player myself, taking a quick glance at this ban list, we seem to have quite a good spread of champions from all positions, about 3-6 champions per each position (as some of these champions could be flexed onto other positions).

Hence, let's use this list to create the `"meta"` boolean column.

**Defining `"kda"`**

KDA, also known as Kills, Deaths and Assists, is a common metric used to quantify a player's performance in most competitive games. The formula that is universally used to calculate this metric is `("Kills" + "Assists")/"Deaths"`. Having a KDA higher than 1 usually means that a player had a positive impact in game and to their team. Having a KDA between 0 and 1 usually indicates poor performance, meaning that a player has died more times than they have helped, or killed enemies. This indicates negative impact to a team. Using this definition, we will create the `"kda"` column.

After some additional filtering conducted after univariate & bivariate analysis, our final dataframe `"cldf"`, a short term for cleaned_df, is created.

**Here is the head of our `"cldf"` dataframe**

| gameid                | league   |   year | result   | position   | champion   |   kills |   deaths |   assists | meta   |     kda |
|:----------------------|:---------|-------:|:---------|:-----------|:-----------|--------:|---------:|----------:|:-------|--------:|
| ESPORTSTMNT01_2690210 | LCKC     |   2022 | False    | top        | Renekton   |       2 |        3 |         2 | False  | 1.33333 |
| ESPORTSTMNT01_2690210 | LCKC     |   2022 | False    | jng        | Xin Zhao   |       2 |        5 |         6 | False  | 1.6     |
| ESPORTSTMNT01_2690210 | LCKC     |   2022 | False    | mid        | LeBlanc    |       2 |        2 |         3 | True   | 2.5     |
| ESPORTSTMNT01_2690210 | LCKC     |   2022 | False    | bot        | Samira     |       2 |        4 |         2 | False  | 1       |
| ESPORTSTMNT01_2690210 | LCKC     |   2022 | False    | sup        | Leona      |       1 |        5 |         6 | False  | 1.4     |



### Univariate Analysis

With our cleaned data, we can finally take a look at some overall distributions within our dataset. To begin, we can look at the distribution of kills and KDA scores per player.

**Distribution of Kills** 

Kills make up a important part of our kda metric, hence we will visualize how many kills players often get per game of League of Legends.


<iframe src="charts/kills_dist.html" width=800 height=600 frameBorder=0></iframe>

As expected, the kills distribution is right skewed. This is likely due to the fact that in professional tournaments, all players are in the top 0.01% of players in the world. All players have a full understanding of how League of Legends is meant to be played, meaning any small mistake, such as a death, could win or break a game. Therefore, because all players play extremely careful, there will be few instances of players getting large amounts of kills within a game, hence the explaination for right skewness we see in the distribution.

**KDA Distribution**

As kills only make up one part of our performance metric, let us take a look at the overall performance of a player per game. 

<iframe src="charts/kda_dist.html" width=800 height=600 frameBorder=0></iframe>

Similarly, we see the same right skew that was present in the kills distribution. By similar logic as kills, it will be hard to find players that do extremely well in games, hence the right skewness we see in the distribution. Interestingly enough, is that because we now account for assists in addition to kills, we see that kda has a larger spread compared to just kills by themselves. One more point to note is that because kda is a caluclated metric based off of integer values, we see a lot of 0 heigh bars within our graph as some values cannot appear from interger division. Taking a look at the values itself, we can generally say that most players have a kda around 1-5, which indicates good performance within a game.

**Further Filtering**

After observing these distributions, to keep our data consistent, we will drop data where kda is higher than 20. This is because it is extremely hard to not die or only die once in a professional match. If such instances happen, we see extremely high kdas within our dataset, which we do in the distribution. Hence, we will remove these potential outlier instances. This leaves us with 12101 games we can observe for our dataset. You can see the head of the dataframe in the **Data Cleaning** section above.


### Bivariate Analysis

**Meta vs Non-Meta Win Rate**

For this section, we will compare the winrate of champions by whether they are classified as heavily banned or not. We will use columns `"result"` and `"meta"` for this comparision.


<iframe src="charts/meta_win_dist.html" width=800 height=600 frameBorder=0></iframe>

This bar chart shows the percentage of wins by whether a champion is classified as `"meta"` (heavily banned champion) or not. From the bar plots, it seems that champions who have a high ban rate tend to have a higher win rate compared to other champions -- by about 0.03 to be more exact. We will view the aggregrate data in a later section to investigate further.


**KDA by Role**

From our univariate analysis, we see that KDA is an extremely skewed distribution. This brings us to question, do player `"positions"` have an effect on KDA? To understand the extent of the player role on our performance metric, kda, we can use boxplots to visualize the distribution of kda per different player positions. If we observe the boxplot, we can observe common trends of different role responsbilities.


<iframe src="charts/boxplot.html" width=800 height=600 frameBorder=0></iframe>

 From our boxplots, we see that mid and bot players have a larger box size, and the median kda is higher than all other roles. This reflects the general playstyle of these roles. In league of legends, bot and mid players tend to be the "carry" roles of the game. The champions often played in these roles are consistent damage dealers such as asassians, mages, and gunsmans/archers, while for other roles such as top, they tend to be tankier champions, which aim to take damage, or help intiate fights. 
 
 Hence, we see that roles such as support and top, tend to have a smaller 25th - 75th quantile compared to carry roles such as mid and bot. Bot has the largest 25th - 75th quantile spread as bot is an extremely important role within the game. Teams will typically aim to camp or kill carry players (ie: bot) as much as possible to put these players behind in the game so that they will be less of a threat to the enemy team. On the other hand, if a bot player is able to play their role out and get ahead of their opponents, these players will end up 'snowballing' -- essentially dominating the game. Therefore bot players will have a larger kda spread because of these situations. 

 There is a lot more we can infer about player roles and performance from this chart, but for the sake of keeping this analysis portion simple and not getting to far into game specific details, we will move on.


### Interesting Aggregrates

 As mentioned earlier in the bivariate analysis about meta vs non-meta win rates, this `groupby()` chart shows us the statistics behind meta vs non-meta champions. 

 | meta   |   count |   result |   kills |   deaths |   assists |     kda |
|:-------|--------:|---------:|--------:|---------:|----------:|--------:|
| False  |   74657 | 0.434494 | 2.67675 |  3.29798 |   6.25052 | 4.14552 |
| True   |   32600 | 0.456748 | 2.90426 |  3.19387 |   6.23767 | 4.39084 |


 We see that on average, meta champions have a have a higher winrate than non-meta champions. Additionally, we see the average amount of kills is higher, alongside a lower amount of average deaths. This is reflected as well on our kda calcualtion, displaying a 0.25 kda gap between meta and non-meta champions. Interestingly enough, the average number of assists for non-meta champions is slightly higher than meta champions. We will contiune to break this data down a bit further by observing meta champions statistics within specific player positions.


 | position   | meta   |   count |   winrate |    kills |   deaths |   assists |     kda |
|:-----------|:-------|--------:|----------:|---------:|---------:|----------:|--------:|
| bot        | False  |   16218 |  0.416574 | 3.90165  |  2.97996 |   5.19731 | 4.57961 |
| bot        | True   |    4525 |  0.454586 | 4.50541  |  2.88862 |   5.06387 | 4.96331 |
| jng        | False  |   16791 |  0.433923 | 2.85474  |  3.50664 |   6.60437 | 4.17357 |
| jng        | True   |    4905 |  0.479715 | 3.09867  |  3.29684 |   6.67339 | 4.60127 |
| mid        | False  |   11365 |  0.424901 | 3.15205  |  3.15952 |   5.88218 | 4.32954 |
| mid        | True   |    9702 |  0.442383 | 3.5269   |  2.89322 |   5.37683 | 4.63536 |
| sup        | False  |   14153 |  0.458207 | 0.86434  |  3.54907 |   8.83657 | 4.17355 |
| sup        | True   |    7816 |  0.44652  | 0.809493 |  3.60427 |   8.80514 | 4.17271 |
| top        | False  |   16130 |  0.439058 | 2.51525  |  3.27774 |   4.93156 | 3.52563 |
| top        | True   |    5652 |  0.477353 | 3.28167  |  3.29742 |   4.72647 | 3.63179 |

Looking at the above position by meta champion `groupby()`, we can see that meta champions tend to have a higher winrate for all positions except support. Additionally, it seems that on average meta support champions do worse than their counterparts. This is extremely interesting given that these champions are heavily banned, usually due to their skillsets being extremely good for teamfights, or champion statistics are extremely good. Because of this, we should've expected to see meta supports either having higher kills or assists compared to non-meta supports. There is no proper explaination for this finding, we can only confirm it with more data and further tests. We unfortunately will not be looking into this case for this project.


## Assessment of Missingness

### NMAR Analysis

A lot of the columns within our dataset seemed to have missing data. I'd believe quite a few of these columns likely have NMAR data as well. For example, in the data taken from specific leagues (ex: DCup (2022 Demacia cup) in Asia) there multiple columns of data such as `"goldat15"` and `"xpat15"` which are null. This makes sense as some tournaments might simply not have the tools/means to record this data, or will only record this data for important matches, such as quarterfinals/finals. That means for games such as playoffs, many of these playoff games are conducted at once and there might not be enough resources to collect these datapoints for all thes games. Additionally for smaller tournaments, officials might also choose to not record this data due to a belief that these tournamentss are less significant compared to larger ones, and that there might be not much to learn from collecting this data. Hence, columns such as `"goldat15"` and `"xpat15"` are NMAR as the missingness can depend on the type of match the data is from, or also the tournament the match data is from.

To change our this missingess from NMAR to MAR, we can just collect the data that was missing from the matches by watching the VODs/videos and recording down the missing metrics that leagues/touraments might have not recorded. This will change the data so that the missingness is only dependent on the duration of the game (missing if a game ends before 15 minutes) and not dependent on match or tournament not recording the data.


### Missing Dependency

In this section we will investigate the dependency of `"goldat10"` on `"league"` and verify if the missing values of `"goldat10"` is influenced by the `"league"` the match data came from.

There are 18190 missing goldat10 values out of 121010 rows. 

Here is the distribution of `"league"` when `"goldat10"` is missing and not missing.

| league     |   goldat10_not_missing |   goldat10_missing |
|:-----------|-----------------------:|-------------------:|
| CBLOL      |             0.0236335  |         0          |
| CBLOLA     |             0.0210076  |         0          |
| CDF        |             0.00709979 |         0          |
| CT         |             0.00252869 |         0          |
| DCup       |             0          |         0.042331   |
| DDH        |             0.020035   |         0          |
| EBL        |             0.0156584  |         0          |
| EL         |             0.012838   |         0          |
| ESLOL      |             0.0214939  |         0          |
| EUM        |             0.0259677  |         0          |
| GL         |             0.0152694  |         0          |
| GLL        |             0.0180899  |         0          |
| HC         |             0.0157557  |         0          |
| HM         |             0.0140051  |         0          |
| IC         |             0.0072943  |         0          |
| LAS        |             0.0220774  |         0          |
| LCK        |             0.0454192  |         0          |
| LCKC       |             0.0383194  |         0          |
| LCL        |             0.00155612 |         0          |
| LCO        |             0.0206186  |         0          |
| LCS        |             0.0297607  |         0          |
| LCSA       |             0.052519   |         0          |
| LDL        |             0          |         0.517867   |
| LEC        |             0.0236335  |         0          |
| LFL        |             0.0221747  |         0          |
| LFL2       |             0.0216884  |         0          |
| LHE        |             0.0232445  |         0          |
| LJL        |             0.0208131  |         0          |
| LJLA       |             0.00369578 |         0          |
| LLA        |             0.0175063  |         0          |
| LMF        |             0.0304415  |         0          |
| LPL        |             0          |         0.432106   |
| LPLOL      |             0.0186734  |         0          |
| LVP SL     |             0.0218829  |         0          |
| MSI        |             0.00778059 |         0          |
| NEXO       |             0.0112819  |         0          |
| NLC        |             0.0354017  |         0          |
| PCS        |             0.0263567  |         0          |
| PGC        |             0.0546586  |         0          |
| PGN        |             0.0135188  |         0          |
| PRM        |             0.0344291  |         0          |
| SL (LATAM) |             0.0149776  |         0          |
| TAL        |             0.0199378  |         0          |
| TCL        |             0.0214939  |         0          |
| UL         |             0.0250924  |         0          |
| UPL        |             0.04007    |         0          |
| VCS        |             0.0314141  |         0          |
| VL         |             0.0151721  |         0          |
| WLDs       |             0.0137133  |         0.00769654 |


We can see from the missing and not missing columns that the distribution of `"league"` varies greatly. Hence, we will run a permutation test with 1000 reps using Total Variation Distance (TVD) to confirm this finding.

**Null Hypothesis:** The missingess of `"goldat10"` does not depend on `"league"` 
**Alternative Hypothesis:** The missingess of `"goldat10"` depends on `"league"` 

**Our results are as listed:**

| Statistic | Value |
|:----------:|:------------------:|
| significance level | 0.05 |
| p-value | 0.00 |
| observed TVD| 0.99|

The corresponding empirical total variation distance distribution:

<iframe src="charts/mis_dep.html" width=800 height=600 frameBorder=0></iframe>

Based off our test, our p-value is 0.00, hence we will reject the null. We can conclude that thers is a high chance that the missingness of `"goldat10"` is dependent on `"league"`. This makes sense as some leagues, may simply decide to not track these stats, or only track them for certain matches (ie: finals).


### Missingness Non-Dependency

In this section, we will investigate if the number of kills that a player gets in a game affects the missingness of 'ban1'. 

There are 1905 missing ban1 values out of 121010 rows.

|   kills |   ban1_not_missing |   ban1_missing |
|--------:|-------------------:|---------------:|
|       0 |        0.190949    |    0.192651    |
|       1 |        0.197255    |    0.198425    |
|       2 |        0.159792    |    0.16063     |
|       3 |        0.123311    |    0.111811    |
|       4 |        0.0951513   |    0.0871391   |
|       5 |        0.0712397   |    0.0661417   |
|       6 |        0.0532471   |    0.0535433   |
|       7 |        0.037278    |    0.048294    |
|       8 |        0.0264305   |    0.0225722   |
|       9 |        0.017111    |    0.0225722   |
|      10 |        0.0112674   |    0.0125984   |
|      11 |        0.00666639  |    0.0104987   |
|      12 |        0.00451702  |    0.00577428  |
|      13 |        0.00247681  |    0.00367454  |
|      14 |        0.00140212  |    0.00209974  |
|      15 |        0.00070526  |    0.000524934 |
|      16 |        0.000503757 |    0.000524934 |
|      17 |        0.000251879 |    0.000524934 |
|      18 |        0.000125939 |    0           |
|      19 |        0.000151127 |    0           |
|      20 |        6.71676e-05 |    0           |
|      21 |        2.51879e-05 |    0           |
|      22 |        2.51879e-05 |    0           |
|      23 |        8.39595e-06 |    0           |
|      24 |        1.67919e-05 |    0           |
|      25 |        8.39595e-06 |    0           |
|      27 |        8.39595e-06 |    0           |
|      28 |        8.39595e-06 |    0           |

The two distributions of kills seems similar between ban1 missing and not missing. We will run a permutation test to confirm this finding.

**Null Hypothesis:** The missingess of `"ban1"` does not depend on `"kills"` 
**Alternative Hypothesis:** The missingess of `"ban1"` depends on `"kills"` 


**Our results are as follows:**

| Statistic | Value |
|:----------:|:------------------:|
| significance level | 0.05 |
| p-value | 0.453 |
| observed TVD| 0.03|

The corresponding empirical total variation distance distribution:

<iframe src="charts/non_dep.html" width=800 height=600 frameBorder=0></iframe>

From our test, our p-value is 0.452, hence we fail to reject the null. We can conclude that it is highly possible that the missingness of column `"ban1"` is not dependent on `"kills"`. This is likely as bans occur before the game even starts, hence the amount of kills a player gets in game should not affect the chances of bans being missing. 


## Hypothesis Testing

Now that we assessed the possibilities of missing data within the bigger dataset, we can revert back to studying whether popularly banned champions tend to have higher win rates and performance compared to the rest of the champion pool. We will conduct two different permutation tests to test whether `"meta"` positively affects `"result"` and `"performance"`.

### `meta` on `result`

**Null Hypothesis**: Meta champions have the same winrate as other champions.

**Alternative Hypothesis**: Meta champions have a higher winrate compared to other champions.

**Type of Test**: One-sided permutation test with 10,000 simulations

**Test Statistic**: Difference in Means statistic for `"result"`.

**Significance Level**: 0.05, the base level for most hypothesis tests.

**Results**
Our test resulted in a p-value of 0.00, hence we will reject the null hypothesis. 

<iframe src="charts/win_md.html" width=800 height=600 frameBorder=0></iframe>


### `meta` on `kda`

**Null Hypothesis**: Meta champions have the same kda as other champions.

**Alternative Hypothesis**: Meta champions have a higher kda compared to compared to other champions.

**Type of Test**: One-sided permutation test with 10,000 simulations

**Test Statistic**: Difference in Means statistic for `"kda"`.

**Significance Level**: 0.05, the base level for most hypothesis tests.

**Results**
Our second test also resulted in a p-value of 0.00, hence we will reject the null hypothesis. 

<iframe src="charts/kda_md.html" width=800 height=600 frameBorder=0></iframe>

### Conclusion

From both our tests, we reject the null hypothesis, as our p-value, 0.0 is less than our significance level. We can conclude that our dataset shows there is a statistically significant difference between meta champions' and non-meta champions' winrate (`result`) and performance in game (`kda`). However, we have to keep in mind we cannot conclude casuation from these tests as this is not experimental data. We can only conclude there is only a trend between `meta` and our metrics `result` and `kda`.

