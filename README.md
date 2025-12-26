# NBA 2024-25: Utilizing Roles

---

## **Introduction**

For any NBA fan, checking a player's game box score usually begins with points, rebounds, and assists. These are the primary statistics that measure raw on-court *output*. Superstars are expected to put up big numbers, but what about players who have smaller roles in a team's offense? Traditional box scores may fail to capture their true impact.

This study analyzes how NBA players **maximize output** relative to their **offensive role size** during the 2024-25 regular season. Instead of focusing on raw volume statistics alone, this analysis examines how **efficiently** and **consistently** players convert usage → output.

---

## **Table of Contents**

1. [Introduction](#introduction)
2. [About the Data](#about-the-data)
3. [Project Goals](#project-goals)
4. [Tools Used](#tools-used)
5. [Project Files](#project-files)
6. [Notebook 01: Data Acquisition, Pt. 1](#notebook-01-data-acquisition-pt-1)
7. [Notebook 02: Data Acquisition, Pt. 2](#notebook-02-data-acquisition-pt-2)
8. [Notebook 03: Data Acquisition, Pt. 3](#notebook-03-data-acquisition-pt-3)
9. [Notebook 04: Calculating All-Star Baselines](#notebook-04-calculating-all-star-baselines-usg-and-pra)
10. [Notebook 05: Merge 2024-25 Game Logs & Season Averages](#notebook-05-merge-2024-25-game-logs--2024-25-season-averages)
11. [Notebook 06 + Dashboard: Role-Based Analysis](#notebook-06--dashboard-role-based-analysis)
12. [Tableau Dashboard](#tableau-dashboard)
13. [Conclusion](#conclusion)
14. [Dashboard Link](#tableau-dashboard-link)

---

## **About the Data**

The primary data for this study is a combination of player-level and team-level game logs from the 2024-25 NBA regular season. They were sourced from the **NBA API** (via Python), which is the official data interface that powers statistics on NBA.com.

The secondary data consists of season-level per-game statistics for the previous five regular seasons (2019-20 through 2023-24). These were sourced from [Basketball-Reference](https://www.basketball-reference.com/).

> For example, for the 2023-24 regular season:
> - [Standard Per-Game Statistics](https://www.basketball-reference.com/leagues/NBA_2024_per_game.html)
> - [Advanced Statistics](https://www.basketball-reference.com/leagues/NBA_2024_advanced.html)

---

## **Project Goals**

1. How often does a player produce *all-star-level* output within their role?
2. How much output does a player produce within their role?
3. Who combines high output (relative to their role) with strong game-to-game consistency?

> <sub> Players were evaluated both league-wide and within usage cohorts (low/medium/high usage) to ensure fair role-based comparisons.

---

## **Tools Used**

- **Python** (via **Jupyter Notebook):** Data loading, cleaning, merging, and analysis ![Python](https://img.shields.io/badge/Python-orange)
- **Tableau:** Dashboard design and final visualizations ![Tableau](https://img.shields.io/badge/Tableau-blue)

---

## **Project Files**

### **Jupyter Notebooks**
- [`01_acquire_player_game_logs.ipynb`](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/notebooks/01_acquire_player_game_logs.ipynb)
Raw data collection (via NBA API), cleaning, and consolidation of all 2024-25 player game logs
- [`02_acquire_team_game_logs.ipynb`](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/notebooks/02_acquire_team_game_logs.ipynb)
Raw data collection (via NBA API), cleaning, and consolidation of all 2024-25 team game logs
- [`03_acquire_previous_5_seasons_stats.ipynb`](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/notebooks/03_acquire_previous_5_seasons_stats.ipynb)
Raw data import (from Basketball-Reference), cleaning, merging, and consolidation of all player per-game statistics for 2024-25 and the previous 5 seasons (2019-20 through 2023-24)
- [`04_calculating_historical_all_star_baselines.ipynb`](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/notebooks/04_calculating_historical_all_star_baselines.ipynb)
Using 2019-20 through 2023-24 per-game data, calculations of average USG% and PRA for all all-stars combined
- [`05_merge_player_game_logs_and_season_averages.ipynb`](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/notebooks/05_merge_player_game_logs_and_season_averages.ipynb)
Merging of 2024-25 player game logs and season-level 2024-25 per-game statistics
- [`06_role_based_analysis.ipynb`](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/notebooks/06_role_based_analysis.ipynb)
Analysis of maximizing output relative to role size

### **CSV Exports**
- [`NBA_2024_25_game_logs_final.csv`](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/csv/NBA_2024_25_game_logs_final.csv)
Finalized player game logs for the 2024-25 NBA regular season
- [`season_summary.csv`](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/csv/season_summary.csv)
Season-level averages for all players

### **Documentation**
- `README.md`: Project documentation

---

## **Notebook 01: Data Acquisition, Pt. 1**

Of all NBA players across the 2024-25 regular season, 569 players registered at least one statistic (a point, a turnover, ..., anything of a statistical nature). All regular season game logs for these 569 players were retrieved from the NBA API endpoints `LeagueDashPlayerStats` and `PlayerGameLog`. These endpoints included player IDs (e.g., 1630639), but not the actual player names. Therefore, the names were retrieved from the NBA API endpoint `players`. After appropriate cleaning and merging, the player game logs DataFrame was created.

> <sub> **Note:** "API" stands for Application Programming Interface. It's like a vending machine where you can request data. "Endpoints" are like selections inside the vending machine.

---

## **Notebook 02: Data Acquisition, Pt. 2**

**Usage Rate (USG%)** estimates the **percentage of team possessions a player finishes** while on the court. A possession is considered "finished" or "used" when a player either:
- attempts a shot
- draws free throws
- commits a turnover

$$
\text{USG\\\%} = 100 \times \frac{(FGA + 0.44 \times FTA + TOV) \times (Team\ Minutes / 5)}{Minutes \times (Team\ FGA + 0.44 \times Team\ FTA + Team\ TOV)}
$$

> In this study, USG% refers to **role size** (or offensive role size). In other words, how much was a player involved in the offense when on the court?

**Points + Rebounds + Assists (PRA)** is simply a measure of **output**, combining a player's scoring, playmaking, and rebounding contributions.

$$
PRA = PTS + REB + AST
$$

In the USG% calculation above, there are several team-related inputs that the player game logs (from Notebook 01) do not have. Therefore, to get these necessary inputs, team game logs were retrieved from the NBA API endpoint `LeagueGameLog`. After cleaning, player game logs and team game logs were merged into one DataFrame, where USG% and PRA were calculated for every game.

> One could make the argument that rebounding doesn't impact offensive output as directly as points or assists, so why include it? It's a fair point, but **every offensive possession has a lifecycle**. A rebound creates (or extends) an offensive possession, and then points and/or assists finish it (at least in successful offensive possessions). PRA captures this full arc, which is why rebounding remains a key component of measuring offensive output.

> <sub> **Note:** This analysis uses USG% and USG interchangeably; both refer to "Usage Rate".

---

## **Notebook 03: Data Acquisition, Pt. 3**

For USG% and PRA, reference points were needed so role sizes and output could be properly evaluated.

Using Basketball-Reference, both per-game and advanced season-level statistics were loaded from the **previous 5 seasons (2019-20 through 2023-24)**. This was cleaned, merged, and combined into one DataFrame.

---

## **Notebook 04: Calculating All-Star Baselines (USG% and PRA)**

The historical data from Notebook 03 was then filtered so that it **only** included season statistics from **all-stars**. The goal was to eventually compare current players' statistics with these historical all-star players' statistics.

For the previous five seasons, the **average USG% was 29.3%** and the **average PRA was 37.8 points + rebounds + assists**.

---

## **Notebook 05: Merge 2024-25 Game Logs & 2024-25 Season Averages**

Before the analysis, I wanted to make sure that all player USG% and PRA values were appropriately aggregated. For each player's season, should I take the average of all USG% values across all games? Or should I use the season-level USG% that Basketball-Reference already calculated?

I decided to use the latter so that every player was anchored to a single, stable USG% value, avoiding any game-to-game fluctuations. Therefore, these Basketball-Reference season-level values were merged into the player game logs DataFrame.

> Before the merge took place, there was a player name issue since the player game logs (from NBA API) had some names that differed from the Basketball-Reference names. For example, in NBA API, the name is Pacôme Dadiet, but in Basketball-Reference, the name appears as Pacome Dadiet (without an accent). For the merge to work, all names must be exactly the same, so I decided to use the NBA API names.
>
> After exporting all names from both sources, I manually scanned through all 569 player names across 2 text files, and found 22 names that didn't match. These names were "cleaned" so that all 569 names matched. Then a successful merge occurred.

---

## **Notebook 06 + Dashboard: Role-Based Analysis**

### **Filters**

To ensure all metrics reflect meaningful on-court roles as opposed to random, short-term appearances, the following filters were applied:
- A player must play **at least 12 minutes** in a game for that game to be included
- A player must play **at least 20 games** during the regular season for that player to be included

### **Regression**

The first objective was to measure **how output (PRA) changed** in response to **a change in role size (USG%)**.

> <sub> In other words, if a player's role size increased by 1%, *by what percentage* did their output increase?

To find this, **log-log regression** was used in Python:

$$
\log(\text{PRA}) = \alpha + \beta \log(\text{USG}) + \varepsilon
$$

Where:
- $\alpha$ = $\text{baseline output level}$
- $\beta$ = $\text{elasticity of PRA with respect to USG}$ = $\text{the \\\% change in PRA associated with a 1\\\% change in USG}$
- $\varepsilon$ = $\text{random game-to-game fluctuations}$

According to the regression results, $\beta$ = 0.898.

This means that a **1% increase in role size (USG%)** was associated with a **0.898% increase in output (PRA)**.

### **PRA Signal**

Some players have heavy offensive responsibilities while others play smaller roles. **PRA Signal** was designed **to adjust a player's output (PRA)** based on their role size, whether it's high usage, low usage, or somewhere in between.

> These adjustments are based on the "all-star usage baseline" of **29.3%** (the average USG% of every all-star across the previous 5 seasons). PRA Signal adjusts each game's output *as if* it were produced at this common usage baseline.
>
> **Note:** For any game where a player has a USG% of 29.3% or higher, there is NO adjustment. In this case, PRA Signal simply equals the raw PRA value.

$$
\text{PRA Signal} = \text{PRA} \cdot \left( \frac{29.3\\\%}{\text{USG}} \right)^{\beta}
$$

Where:
- $29.3\\\%$ = $\text{average USG of all-stars over the previous 5 seasons}$
- $\beta$ = 0.898 = $\text{elasticity of PRA with respect to USG}$

With this, all game outputs were comparable.

For example, if Player A has a role size of 30% and Player B has a role size of 15%, their outputs can be fairly compared (via PRA Signal) even though their role sizes are significantly different.

> It's very important to point out that these are **signals** (or indicators), NOT projections.
>
> **Example:**
> In a game, Player X records 16 PRA with a 20% usage rate. 20% is less than the all-star baseline of 29.3%, so using the PRA Signal formula above, this output (16 PRA) adjusts to a PRA *Signal* of 30.9.
> 
> This does NOT mean that Player X *would have* produced 30.9 PRA if their USG% was increased from 20% to 29.3%.
>
> It simply means that Player X produced a high level of output (30.9) **relative to their role size**.
> 
> A high usage player with a PRA Signal of 30.9 is the **same adjusted output** as a low usage player with a PRA Signal of 30.9, even though their raw output (PRA) values may have been different.

### **All-Star Output Rate** 

With these output signals introduced, it was time for classification.

> This is where the "all-star output baseline" of **37.8 PRA** (the average PRA of every all-star across the previous 5 seasons) came into play.

$$
\text{All-Star Output Rate} = \frac{\text{Number of games where PRA Signal} \ge 37.8}{\text{Total games played}}
$$

Where:
- $37.8$ = $\text{average PRA of all-stars over the previous 5 seasons}$

The **All-Star Output Rate** is the **percentage of a player's games** where the **PRA signal was at least 37.8** (i.e., all-star-level).

This metric indicates **how consistently** a player produced **all-star quality output** game-to-game across the regular season, **regardless of role size**.

> For example, if Player A played 60 games, and in 30 of those games their PRA Signal was at least 37.8 (all-star quality), then their All-Star Output Rate is 50%.

The **average All-Star Output Rate** across all 373 players in this study was **26.2%**. The highest was Nikola Jokić with a rate of 95.7%.

### **Output per Role**

To recap, PRA measures raw output, PRA Signal measures adjusted output (i.e., if all players had the same role size), but both of these measures don't say much about output efficiency.

**Output per Role** addresses this concern:

$$
\text{Output per Role} = \frac{\text{PRA}}{\text{USG}}
$$

Output per Role measures **how efficiently** a player **produced output relative to their role size**. In other words, how efficiently does a player convert their role into output?

> For example, Player X had a raw PRA of 16 with a USG of 15%. Player Y had a raw PRA of 28 with a USG of 30%. Using the formula, Player X's output per role is 1.07, while for Player Y, it's 0.93.
>
> Even though Player Y's raw output was higher, Player X was more efficient. Here, efficiency is more important than raw volume.

This metric was measured for every game, but for a player's entire regular season, simply take the average value across all games.

The **average Output per Role** across all 373 players in this study was **0.95**. The highest belonged to Josh Hart and Rudy Gobert, both with an average of 1.90.

### **Output Consistency (OC) Grade**

All-Star Output Rate and Output per Role are the two primary metrics of this analysis, but the most presentable option would be to combine these two into one primary metric.

$$
\text{Output Consistency} = \sqrt{{\text{All-Star Output Rate}} \times {\text{Output per Role}}}
$$

**Output Consistency (OC)** measures **how consistently** and **how efficiently** a player produced **high output relative to their role size** across all regular season games.

> <sub> To combine All-Star Output Rate and Output per Role, I took the geometric mean of these two metrics. This was done to regulate any potential extreme values. That's the purpose of the square root.

For further interpretability, these OC values were rescaled to a **0-100 grade**:

$$
\text{Output Consistency Grade} = \frac{\text{OC} - \min(\text{OC})}{\max(\text{OC}) - \min(\text{OC})}
$$

Where: 
- $\text{OC}$ = $\text{Output Consistency value}$
- $\min(\text{OC})$ = $\text{minimum observed OC value}$ = $0$
- $\max(\text{OC})$ = $\text{maximum observed OC value}$ = $1.305$

Now each player's season can be summed up with one grade. The **average OC grade** across all 373 players in this study was **36.7**. The highest was Nikola Jokić with a grade of 100.0.

---

### **Tableau Dashboard**

![Dashboard Screenshot](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/images/NBA_2024_25_utilizing_roles_dashboard.png)

For the dashboard, all 373 players in this study were placed into cohorts based on role size:
- **Low Usage:** The player's season USG% is 15% or lower (94 players)
- **Medium Usage:** The player's season USG% is between 15% and 23% (189 players)
- **High Usage:** The player's season USG% is 23% or higher (90 players)

![Top Players Under 24](https://github.com/dylanbarrett-analytics/nba-2024-25-utilizing-roles/blob/main/images/NBA_2024_25_utilizing_roles_rising_talent.png)

As a bonus, here are the highest OC grades for *players under 23*. This study is not intended to predict future stars, but it's worth sharing for additional context.

---

## **Conclusion**

The full arc of a successful offensive possession begins with a rebound and ends with points and assists. PRA captures this entire sequence.

Notably, **8 of the top 11 OC Grades** belong to **centers** even though their offensive role sizes are relatively small. Their impact often looks routine and effortless, as opposed to flashy or highlight-worthy, especially compared to smaller positions. Because of this, their contributions are easy to take for granted despite being foundational to team success.

Not everyone can be *the star*, but this study illustrates that you don't need to be *the star* to have *star impact*.

---

## **Tableau Dashboard Link**

🔗 [View the Dashboard on Tableau Public](https://public.tableau.com/app/profile/dylan.barrett1539/viz/---inprogress/Dashboard)

---

<small> <mark> **Note:** </mark> This study was for exploratory purposes only, not for "reinventing the wheel". This data can be used for so many different creative directions. This was one of them.

---

<sub> On an unrelated note, I am so excited about the future of the Brooklyn Nets with head coach Jordi Fernández. The development of this team is so fun to watch. ⚫️⚪️🏗️
