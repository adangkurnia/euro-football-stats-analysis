# Introduction

![euro football stats](https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/9c0707ea-a579-4a12-b412-5f347dd23c7f)

This project aims to showcase the performance of teams in Europe's top 5 leagues from the 2008/2009 to 2015/2016 seasons. The analysis is based on various football metrics, including goals productivity, defense strength, number of wins and losses, and other important numbers. In addition to utilizing SQL queries, I will also be presenting the data in a visual format using Tableau Public.

# Content
1.	[Dataset](https://github.com/adangkurnia/euro-football-stats-analysis#dataset)
2.	[Data Understanding](https://github.com/adangkurnia/euro-football-stats-analysis#data-understanding)
3.	[Constraints](https://github.com/adangkurnia/euro-football-stats-analysis#constraints)
4.	[Tools](https://github.com/adangkurnia/euro-football-stats-analysis#tools)
5.	[Data Cleaning](https://github.com/adangkurnia/euro-football-stats-analysis#data-cleaning)
6.	[Exploratory Data Analysis](https://github.com/adangkurnia/euro-football-stats-analysis#exploratory-data-analysis)
7.	[Data Visualization](https://github.com/adangkurnia/euro-football-stats-analysis#data-visualization)
8.	[Conclusion](https://github.com/adangkurnia/euro-football-stats-analysis#conclusion)

# Dataset
I got the dataset from Kaggle, you can download it on this [link](https://www.kaggle.com/code/dimarudov/data-analysis-using-sql/input). You can also download all SQL queries syntax for this project on this [SQL Query](https://github.com/adangkurnia/euro-football-stats-analysis/blob/main/eurofootballScript.sql)

# Data Understanding
There are 7 tables inside the database. Those tables are **Match, Country, League, Team, Team Attributes, Player, Player Attributes**. The total rows and columns of each table can be seen in this picture:

<img width="400"  alt="Untitled" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/36f2ce8c-ba14-4af4-b9c3-844c5ffae289">

All tables have a relationship so they can be joined by each other. The **Entity Relationship Diagram** looks like this:
<img width="550"  alt="ERD (1)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/83c52490-02c5-4452-af7c-568c3f38aed7">



# Constraints

- The tables that will be used for this project are **Match, Country, League, And Team**.
- The analysis will be done only for teams from the top 5 leagues in Europe which are **England, Spain, France, Germany, and Italy**.
- The analysis will be done only for teams that **never got relegated** for the period of season data (2008-2016).
- The output of the query will only be shown for the top 5-10 data of the case.

# Tools

- SQL
- DBeaver
- Tableau Public

# Data Cleaning

Before doing the data analysis process, I will do some data cleaning first to adjust the data content to the needs of the analysis that will be done. These are the things that I will do:

- Delete columns (goal, shoton, shotoff, foulcommit, card, "cross",	corner, possession) from the Match table

  I will delete these columns since they only contain NULL and unwanted values. to delete them, I used this query:
  ```SQL
  ALTER TABLE Match
  DROP COLUMN goal, shoton, shotoff, foulcommit, card, "cross",	corner, possession;
  ```

- Add country_id column in Team table and set Foreign Key of country id from country table to team table

  To connect the Team table and the Country table, I added a new column and set Foreign Key to the Team table using this query:
  ```SQL
  ALTER TABLE TEAM
  ADD COUNTRY_ID INT4, 
  ADD CONSTRAINT FK_COUNTRY_TEAM FOREIGN KEY (COUNTRY_ID) REFERENCES COUNTRY (ID)
  ON DELETE CASCADE ON UPDATE CASCADE;
  ```

- Update values country_id column in Team table

  Since there are only five countries, then I update the values of the country_id column using a single query like this:
  ```SQL
  /*ENGLAND*/
  UPDATE TEAM SET COUNTRY_ID = 1729
  WHERE TEAM_API_ID IN (SELECT HOME_TEAM_API_ID FROM "MATCH" WHERE LEAGUE_ID  = 1729);
  
  /*FRANCE*/
  UPDATE TEAM SET COUNTRY_ID = 4769
  WHERE TEAM_API_ID IN (SELECT HOME_TEAM_API_ID FROM "MATCH" WHERE LEAGUE_ID  = 4769);
  
  /*GERMANY*/
  UPDATE TEAM SET COUNTRY_ID = 7809
  WHERE TEAM_API_ID IN (SELECT HOME_TEAM_API_ID FROM "MATCH" WHERE LEAGUE_ID  = 7809);
  
  /*ITALY*/
  UPDATE TEAM SET COUNTRY_ID = 10257
  WHERE TEAM_API_ID IN (SELECT HOME_TEAM_API_ID FROM "MATCH" WHERE LEAGUE_ID  = 10257);
  
  /*SPAIN*/
  UPDATE TEAM SET COUNTRY_ID = 21518
  WHERE TEAM_API_ID IN (SELECT HOME_TEAM_API_ID FROM "MATCH" WHERE LEAGUE_ID  = 21518);
  ```
  
- Delete teams that are not part of the top 5 leagues in the team table

  Since the analysis will be done only for teams from top 5 leagues in Europe, the list of teams from out of that area will be deleted. To delete those rows of data, I used this query:
  ```SQL
  DELETE FROM Team
  WHERE country_id IS NULL;
  
  DELETE FROM Team
  WHERE country_id IN (
  SELECT c.id
  FROM Country c
  WHERE c.name NOT IN ('Spain', 'France', 'England', 'Germany', 'Italy'));
  ```
  
- Delete teams that are not from the top 5 leagues in the Match table

  The same thing also will be done in Match table with this query:
  ```SQL
  DELETE FROM "Match"  
  WHERE country_id IN (
  SELECT c.id 
  FROM Country c
  WHERE c.name NOT IN ('Spain', 'France', 'England', 'Germany', 'Italy'));
  ```

# Exploratory Data Analysis

In general, my analysis will consist of three main components: identifying the team participants followed by constraints criteria, summarizing the goals, and summarizing the wins. 

- Total team participants in each league in a season

  I want to see the total team participants from each league in a season just to compare it to those teams that have never been relegated and how many teams have been relegated for the entire season. To do that, I use this query:
  ```SQL
  WITH TEAMS_BY_COUNTRY
  AS (
  SELECT M.SEASON,  C.NAME AS COUNTRY, COUNT(DISTINCT M.HOME_TEAM_API_ID) AS TOTAL_TEAMS 
  FROM TEAM T 
  JOIN COUNTRY C 
  ON T.COUNTRY_ID = C.ID
  JOIN "MATCH" M 
  ON M.COUNTRY_ID = C.ID
  GROUP BY M.SEASON, C.NAME
  ORDER BY M.SEASON)
  
  SELECT COUNTRY, AVG(TOTAL_TEAMS) AS TOTAL_PARTICIPANTS
  FROM TEAMS_BY_COUNTRY
  GROUP BY COUNTRY;
  ```
  <img width="400" alt="Untitled (1)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/b2db5135-62bf-45df-aa3d-1880761b26c3">

  We can see from the result that there are 20 team participants in all countries except Germany that only has 18 teams.

- Find a list of teams that the country has never relegated for the entire season

  To make the analysis more objective, the teams that will be analyzed are only those who never got relegated for all seasons from 2008 to 2016. To do that, I used this query to filter the list of teams following the criteria. I also set this query as the view, so I can use it later to join with other tables.
  ```SQL
  CREATE VIEW NOT_RELEGATED_TEAMS AS
  WITH CTE
  AS (
  SELECT T.TEAM_API_ID, SEASON, C.NAME AS COUNTRY, T.TEAM_LONG_NAME  AS TEAM
  FROM "MATCH" M  
  FULL JOIN TEAM T
  ON M.HOME_TEAM_API_ID = T.TEAM_API_ID
  JOIN COUNTRY C 
  ON T.COUNTRY_ID = C.ID 
  GROUP BY SEASON, T.TEAM_API_ID, C.NAME, T.TEAM_LONG_NAME
  )
  SELECT TEAM
  FROM CTE
  GROUP BY TEAM 
  HAVING COUNT(*) = 8
  ORDER BY TEAM;
  
  SELECT *
  FROM NOT_RELEGATED_TEAMS;
  ```
  <img width="400" alt="image" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/a7fb34cd-cec1-4d04-bd67-61f665e3d815">

  We can see from the result there are 50 teams from all the leagues that never relegated from 2008 to 2016.

- Find total teams that have never been relegated by country

  after filtering the list of teams that have never been relegated for the entire season, I want to take a look at the total teams that passed the criteria using this query:
  ```SQL
  SELECT C.NAME AS COUNTRY, COUNT(*) AS TOTAL_TEAMS 
  FROM TEAM T 
  JOIN COUNTRY C 
  ON T.COUNTRY_ID = C.ID
  WHERE T.TEAM_LONG_NAME IN (SELECT * FROM NOT_RELEGATED_TEAMS)
  GROUP BY C.NAME
  ORDER BY COUNT(*) DESC;
  ```
  <img width="400" alt="Untitled (2)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/d01377c4-6b7c-4c32-b227-5e77403d9075">
  
  We can see that from the total of 50 teams, there are variations of how many teams can survive in the highest caste of their respective leagues. The country that has the most teams surviving is Germany with 11 teams, followed by England, France, and Italy with 10 teams. Meanwhile, Spain is the country with the least number of teams that have never been relegated with a total of 9 teams.

**Summary of Team Performances by Goals**

At first, I aim to assess each team's performance by analyzing their total goals scored, total goals conceded, and other relevant data. However, it can be challenging to obtain this information when the match table includes separate columns for home and away games. To overcome this issue, I conducted two separate queries to gather the necessary data on the total goals and goals conceded for each team. The query used is as follows:

- Create a view of the total goals scored by each team
    
    I am interested in examining the total number of goals produced by each team per season, taking into account both home and away matches. To accomplish this, I have split the query into two parts: one for counting the goals created in the home side, and another for counting those created in the away side. Here are the queries I have created:
   ```SQL
   CREATE VIEW TOTAL_TEAM_GOALS
   AS 
   SELECT T1.SEASON,
   T1.TEAM,
   T1.COUNTRY,
   T1.LEAGUE,
   T1.TOTAL_HOME_GOAL,
   T2.TOTAL_AWAY_GOAL,
   T1.TOTAL_HOME_GOAL+T2.TOTAL_AWAY_GOAL AS TOTAL_GOALS
   FROM
   	(SELECT SEASON,
   	TEAM_LONG_NAME AS TEAM,
   	C.NAME AS COUNTRY,
   	L.NAME AS LEAGUE,
   	SUM(HOME_TEAM_GOAL) AS TOTAL_HOME_GOAL
   	FROM TEAM T
   	JOIN "MATCH" M
   	ON T.TEAM_API_ID = M.HOME_TEAM_API_ID
   	JOIN COUNTRY C
   	ON M.COUNTRY_ID = C.ID
   	JOIN LEAGUE L
   	ON M.LEAGUE_ID = L.ID
   	GROUP BY SEASON, TEAM_LONG_NAME, C.NAME, L.NAME
   ) T1
   JOIN
   	(SELECT SEASON,
   	TEAM_LONG_NAME AS TEAM,
   	C.NAME AS COUNTRY,
   	L.NAME AS LEAGUE,
   	SUM(AWAY_TEAM_GOAL) AS TOTAL_AWAY_GOAL
   	FROM TEAM T
   	JOIN "MATCH" M
   	ON T.TEAM_API_ID = M.AWAY_TEAM_API_ID
   	JOIN COUNTRY C
   	ON M.COUNTRY_ID = C.ID
   	JOIN LEAGUE L
   	ON M.LEAGUE_ID = L.ID
   	GROUP BY SEASON, TEAM_LONG_NAME, C.NAME, L.NAME
   ) T2
   ON T1.SEASON = T2.SEASON
   AND T1.TEAM = T2.TEAM
   AND T1.COUNTRY = T2.COUNTRY
   ORDER BY TOTAL_GOALS DESC;
   
   SELECT * FROM TOTAL_TEAM_GOALS;
   ```
   <img width="550" alt="Untitled (3)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/e3b65176-289d-46d9-bb84-e27e1f7d86a6">

- Create a view of the total goals conceded by each team
    
    In addition to the total number of goals scored by each team, I am also interested in evaluating their defensive capabilities. In a similar scenario, I needed to differentiate my queries to determine the number of goals conceded in both home and away matches. To accomplish this, I utilized the following query:
    ```SQL
    CREATE VIEW TOTAL_TEAM_CONCEDED
    AS
    SELECT
    T1.SEASON,
    T1.TEAM,
    T1.COUNTRY,
    T1.LEAGUE,
    T1.TOTAL_HOME_CONCEDED,
    T2.TOTAL_AWAY_CONCEDED,
    SUM(T1.TOTAL_HOME_CONCEDED+T2.TOTAL_AWAY_CONCEDED) AS TOTAL_GOALS_CONCEDED
    FROM
    	(SELECT SEASON,
    	T.TEAM_API_ID, 
    	T.TEAM_LONG_NAME AS TEAM,
    	C.NAME AS COUNTRY,
    	L.NAME AS LEAGUE,
    	SUM(AWAY_TEAM_GOAL) AS TOTAL_HOME_CONCEDED
    	FROM "MATCH" M 
    	JOIN TEAM T 
    	ON M.HOME_TEAM_API_ID = T.TEAM_API_ID 
    	JOIN COUNTRY C 
    	ON M.COUNTRY_ID = C.ID 
    	JOIN LEAGUE L
    	ON M.LEAGUE_ID = L.ID
    	GROUP BY SEASON, T.TEAM_LONG_NAME, C.NAME, L.NAME, T.TEAM_API_ID
    	) T1
    JOIN 
    	(SELECT SEASON,
    	T.TEAM_API_ID, 
    	T.TEAM_LONG_NAME AS TEAM,
    	C.NAME AS COUNTRY,
    	L.NAME AS LEAGUE,
    	SUM(HOME_TEAM_GOAL) AS TOTAL_AWAY_CONCEDED
    	FROM "MATCH" M 
    	JOIN TEAM T 
    	ON M.AWAY_TEAM_API_ID = T.TEAM_API_ID 
    	JOIN COUNTRY C 
    	ON M.COUNTRY_ID = C.ID 
    	JOIN LEAGUE L
    	ON M.LEAGUE_ID = L.ID
    	GROUP BY SEASON, T.TEAM_LONG_NAME, C.NAME, L.NAME, T.TEAM_API_ID
    	) T2
    ON 	T1.SEASON = T2.SEASON
    AND T1.TEAM = T2.TEAM
    AND T1.COUNTRY = T2.COUNTRY
    GROUP BY T1.SEASON, T1.TEAM, T1.COUNTRY, T1.LEAGUE, T1.TOTAL_HOME_CONCEDED, T2.TOTAL_AWAY_CONCEDED
    ORDER BY TOTAL_GOALS_CONCEDED;
    
    SELECT * FROM TOTAL_TEAM_CONCEDED;
    ```
    <img width="600" alt="Untitled (4)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/aac25ff1-ad83-49fa-9a43-178012c83a49">

After creating a View table that displays total goals and conceded from the previous queries, I analyzed Team performances by breaking down the data.      

- Summary table
    
    I will merge the two view tables into a summary table to make data analysis easier. Here is the query I have created:
  ```SQL
  CREATE VIEW SUMMARY_TABLE
  AS
  SELECT 
  TG.SEASON,
  TG.TEAM,
  TG.COUNTRY,
  TG.LEAGUE,
  TG.TOTAL_HOME_GOAL,
  TOTAL_AWAY_GOAL,
  TG.TOTAL_GOALS,
  TC.TOTAL_HOME_CONCEDED,
  TC.TOTAL_AWAY_CONCEDED,
  TC.TOTAL_GOALS_CONCEDED 
  FROM TOTAL_TEAM_GOALS TG
  JOIN TOTAL_TEAM_CONCEDED TC
  ON TG.TEAM = TC.TEAM
  AND TG.SEASON = TC.SEASON
  JOIN NOT_RELEGATED_TEAMS NRT
  ON TG.TEAM = NRT.TEAM
  GROUP BY TG.SEASON, 
  TG.TEAM, 
  TG.COUNTRY, 
  TG.LEAGUE, 
  TG.TOTAL_HOME_GOAL,
  TOTAL_AWAY_GOAL,
  TG.TOTAL_GOALS,
  TC.TOTAL_HOME_CONCEDED,
  TC.TOTAL_AWAY_CONCEDED,
  TC.TOTAL_GOALS_CONCEDED 
  ORDER BY TG.SEASON, TG.TEAM;
  
  SELECT * FROM SUMMARY_TABLE;
  ```
  <img width="800" alt="Untitled (5)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/de868a82-6633-46f6-84ac-562757a81db1">

After running the query, we now have the total number of goals scored and conceded during each home and away match throughout the season. Let's analyze each set of data individually to evaluate the performance of each team.

- Teams with the most goals in the entire season (The most productive teams)
    
    The first finding will be to see the total goals scored by each team throughout the entire season. To see that, I created the following query:
  ```SQL
  SELECT TEAM, SUM(TOTAL_GOALS) AS ALL_SEASON_GOALS
  FROM SUMMARY_TABLE 
  GROUP BY TEAM
  ORDER BY SUM(TOTAL_GOALS) DESC;
  ```
  <img width="400" alt="Untitled (6)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/7063254d-6f05-4858-b959-3ac4526c3387">

  Based on the query result, it's evident that **FC Barcelona** was the highest-scoring team from 2008 to 2016, with a total of **849 goals**. This is five times more than the team in second place. The team in second place is also from Spain, **Real Madrid**, with **843 goals** scored in the same period. The remaining top 10 teams comprise four teams from England, two from Germany, and one each from France and Italy. It's worth noting that, except for FC Barcelona and Real Madrid, no other team managed to score 700 goals. This leaves a gap of over 100 goals between the Spanish teams and the others.

- Teams with the most goals each season
  ```SQL
  SELECT SEASON, TEAM, COUNTRY, LEAGUE, TOTAL_GOALS
  FROM SUMMARY_TABLE 
  ORDER BY TOTAL_GOALS DESC;
  ```
  the result of the query will be visualized like this:
  
  <img width="400" alt="Untitled (7)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/55df0c58-0441-4c33-a2d8-fdcf938f4c96">
  
  <img width="960" alt="Untitled (8)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/3b41d587-9957-471f-b4cc-0705e508fef5">

  From the chart, we can see that in 8 seasons **FC Barcelona** can score goals more than 90 goals per season. The highest number of goals that they ever scored for a season was in the **2012/2013** season with **115 goals**. This season also took part in Lionel Messi putting up an astonishing **91** goals, shattering the old record of 85 that was held by Gerd Muller (Bayern Munich, Germany - 1972). Over a total of 69 games, Messi scored 79 for Barcelona and another 12 for his home country of Argentina.
  Back to the context, the same thing also did by Real Madrid to score 100+ goals in each season except in 2008/2009 when we knew **Cristiano Ronaldo** hadn’t come yet to Santiago Bernabeu. To see the data more clearly, let's take a look at the following visualization:

  <img width="959" alt="Untitled (9)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/c666ffc1-8a72-4786-acb0-dc75473a34fb">

  The peak performance of the **Real Madrid** team and also as the **#1 team** with the most goals scored in a single season was in **2011/2012** producing **121 goals**. **The rest of the teams** were scoring goals in the majority **less than 90 goals**.

- Goals scored trendline each season
    
    To see how the trendline of each team's goal-scoring from 2008/2009 to 2015/2016, I created a line chart for the top 5 teams with the most goals scored. the line chart will look like this:

  <img width="960" alt="Untitled (10)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/30b02c2f-47ac-4a3f-8320-94180453137d">

    From the chart we can see that, the **top 2 teams** have **up and down performances** during each season but they still can keep the performance at a high level. that case also applied to **FC Bayer Munich** and **Manchester City** which can still consistently score goals around **70 to 90**. Meanwhile, Chelsea only had peak performance in terms of scoring goals in the **2009/2010 season** which scored **103 goals**, after that they just dropped the number of goals **below 75 goals** and keep dropping for the **2015/2016 season**. **The lowest number of goals** scored by the top 5 teams is **Manchester City 2008/2009** with **58 goals** only.

- Average goals produced by each team
    
    After looking at the number of goals created, let's take a look at how many goals on average can every team score in a season. I used this query:
  ```SQL
  SELECT 
  TEAM,
  COUNTRY,
  LEAGUE,
  ROUND(AVG(TOTAL_GOALS),1) AS AVERAGE_GOALS
  FROM SUMMARY_TABLE
  GROUP BY TEAM, COUNTRY, LEAGUE
  ORDER BY AVERAGE_GOALS DESC;
  ```
  <img width="400" alt="Untitled (11)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/e315509d-209a-450c-b34d-3ac0deb39593">

  From the chart, we can see that both **FC Barcelona** and **Real Madrid** have an average of goals per season of **more than 100**. Meanwhile, **the rest of the top 10 teams didn’t reach 85 goals** and most of them only scored around 60s to 70s goals per season.

- Teams with the most home goals each season
    
    Go a little further, I want to see which teams scored the most goals while playing as the home team. To do that, I used this query:
  ```SQL
  SELECT 
  SEASON,
  TEAM,
  COUNTRY,
  LEAGUE,
  TOTAL_HOME_GOAL
  FROM SUMMARY_TABLE 
  ORDER BY TOTAL_HOME_GOAL DESC;
  ```
  <img width="400" alt="Untitled (12)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/2351f030-6e36-4d57-92b4-948f75bffbfa">
  
  <img width="960" alt="Untitled (13)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/fb2a1a10-4ad7-4d8c-ba10-226bd048f4f8">
  
  <img width="960" alt="Untitled (14)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/8287e59c-4688-428d-8bce-1c4e8efb53b0">

  The team that scored the most goal when playing on the home ground was **FC Barcelona** in the season **2011/2012 with 73 goals**. Not far from that, the rival also almost reached the same number as FC Barcelona achieved, **Real Madrid scored 70 goals** in a single season **twice in 2011/2012 and 2015/2016**. The rest of the total home goals scored not reaching 70 goals afterwards.

- Teams with the most away goals each season
    
    the same thing also I did see the goals scored by the teams while playing as visitors. To do that, I used this query:
  ```SQL
  SELECT 
  SEASON,
  TEAM,
  COUNTRY,
  LEAGUE,
  TOTAL_AWAY_GOAL
  FROM SUMMARY_TABLE 
  ORDER BY TOTAL_AWAY_GOAL DESC;
  ```
  <img width="426" alt="Untitled (15)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/e80e6c34-4748-4343-8a77-434a92fe06d2">

  <img width="960" alt="Untitled (16)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/991b1462-e57a-4332-8dbe-58211722509a">

  <img width="960" alt="Untitled (17)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/7db6b289-4434-44f0-a1a4-636160ec42c3">

  The most away goals scored was done by **Real Madrid with 53 goals in the season 2014/2015**. They also ever scored almost the same as the first one in the **season 2011/2012 with 51 goals**. The only team that can reach more than 50 away goals in a single season was **FC Barcelona** with **52 goals** in the season **2012/2013**.

- Goal distributions (%) from the top 10 teams by total goals
    
    I also want to take a look at the goal distribution percentage between the home and away sides. The chart will look like this:

  <img width="960" alt="Untitled (18)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/6b920ab2-1468-4807-9f59-9370c8ad0106">

  From the top 10 teams, we can see that more than **50%** of the total goals were scored by the home side with **Manchester City** coming on top 1 as they scored **60.23%** of total goals.  Meanwhile, the team with the most goals scored when played as a **visitor** was **Arsenal **which can score** 46.60%** of total goals. It's important to mention that the information presented is focused on the top 10 teams with the highest number of goals scored. Specifically, we're looking for the team that scored the most goals overall, both when playing at home and away.

- Teams with the least conceded goals in the entire season (Teams with the best defense)
    
    The next part will be about the record of goals conceded by each team. I want to take a look at the teams with the best defense from the goals they have conceded. To do that, I used this query:
  ```SQL
  SELECT 
  TEAM,
  COUNTRY,
  LEAGUE,
  SUM(TOTAL_GOALS_CONCEDED) AS ALL_SEASON_CONCEDED
  FROM SUMMARY_TABLE
  GROUP BY TEAM, COUNTRY, LEAGUE
  ORDER BY ALL_SEASON_CONCEDED;
  ```
  <img width="400" alt="Untitled (19)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/2169f69a-9a54-49c8-a385-6798169ec76c">

  <img width="960" alt="Untitled (20)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/b7e7b6e6-70c7-40ff-b81c-752a98288d63">

  Based on the chart, it appears that **FC Bayern Munich** had the strongest defense among the teams. Over the course of 8 seasons, they only conceded a total of **211 goals**, resulting in an average of **26.38% goals per season**. In the second place, **FC Barcelona**, the team with the highest number of goals scored, also performed well by only conceding **232 goals** in 8 years. Afterward, some teams struggled to defend and ended up conceding an average of over 30 goals per season.

- Teams with the least conceded goals each season
    
    To gather more information about the conceded data, I will utilize a query that displays the number of goals conceded by each team per season. Below is the query that I have formulated:
  ```SQL
  SELECT 
  SEASON,
  TEAM,
  COUNTRY,
  LEAGUE,
  TOTAL_HOME_CONCEDED,
  TOTAL_AWAY_CONCEDED, 
  TOTAL_GOALS_CONCEDED
  FROM SUMMARY_TABLE
  ORDER BY TOTAL_GOALS_CONCEDED;
  ```
  <img width="711" alt="Untitled (21)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/400d0c0d-8541-44b4-a327-8c7ad3373e91">

  <img width="960" alt="Untitled (22)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/3c1e4377-10eb-4fb4-9a05-89a7a1d6686c">

  From the chart, we can see that **FC Bayern Munich** become the team with the fewest goals conceded in a single season. In the season **2015/2016**, they only conceded **17 goals**. They also did almost the same thing in **2012/2013 and 2014/2015** with only conceded **18 goals**. There are several teams that also only conceded goals less than 20 which are PSG 2015/2016, Juventus 2011/2012, and Atletico Madrid 2015/2016.

- Teams with the most goal difference each season
    
    In the last part of team performances by goals, I want to see the teams with the most goal difference. To do that, I used this query:
  ```SQL
  SELECT 
  SEASON,
  TEAM,
  COUNTRY,
  LEAGUE,
  TOTAL_GOALS-TOTAL_GOALS_CONCEDED AS GOAL_DIFF
  FROM SUMMARY_TABLE
  ORDER BY TOTAL_GOALS-TOTAL_GOALS_CONCEDED DESC;
  ```
  <img width="400" alt="Untitled (23)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/295c7e29-8c39-462a-a69c-bae06acd30ab">

  <img width="960" alt="Untitled (24)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/c8440d29-99d7-491d-a696-07db5a340510">

  From the chart, we can see both **FC Barcelona and Real Madrid** have ever booked the highest goal difference with an **89** gap. **Real Madrid** accomplished it first In the **2011/2012** season, while **FC Barcelona** followed suit three years later.

**Summary team performances by total wins and losses**

I would like to view more than just the number of goals a team has scored and conceded. It would be helpful to also see their total number of wins and losses, their average number of wins, and their win rate.

- Create a view of team performances by total wins and lost
    
    First of all, I want to join some tables like I did before. In this part, I also combined the join tables by using UNION ALL function as we know that I must find the total wins and losses of each team when competing at home and away. to do that, I used this query:
  ```SQL
  CREATE VIEW TEAM_WIN_LOSE
  AS 
  WITH CTE AS
  (SELECT 
  M.SEASON,
  T.TEAM_LONG_NAME AS TEAM,
  C."NAME" AS COUNTRY,
  L."NAME" AS LEAGUE,
  CASE WHEN HOME_TEAM_GOAL > AWAY_TEAM_GOAL 
  THEN 'WIN' ELSE 'LOSE' END AS RESULT
  FROM "MATCH" M 
  JOIN TEAM T 
  ON M.HOME_TEAM_API_ID = T.TEAM_API_ID
  JOIN COUNTRY C 
  ON M.COUNTRY_ID = C.ID 
  JOIN LEAGUE L 
  ON M.LEAGUE_ID = L.ID 
  UNION ALL
  SELECT 
  M.SEASON,
  T.TEAM_LONG_NAME AS TEAM, 
  C."NAME" AS COUNTRY,
  L."NAME" AS LEAGUE,
  CASE WHEN  AWAY_TEAM_GOAL > HOME_TEAM_GOAL
  THEN 'WIN' ELSE 'LOSE' END AS RESULT
  FROM "MATCH" M 
  JOIN TEAM T 
  ON M.AWAY_TEAM_API_ID  = T.TEAM_API_ID
  JOIN COUNTRY C 
  ON M.COUNTRY_ID = C.ID 
  JOIN LEAGUE L 
  ON M.LEAGUE_ID = L.ID)
  
  SELECT
  SEASON,
  TEAM,
  COUNTRY,
  LEAGUE,
  COUNT(*) AS TOTAL_MATCH,
  SUM(CASE WHEN RESULT = 'WIN' THEN 1 ELSE 0 END) AS TOTAL_WIN,
  SUM(CASE WHEN RESULT = 'LOSE' THEN 1 ELSE 0 END) AS TOTAL_LOSE
  FROM CTE
  WHERE TEAM IN (SELECT TEAM FROM NOT_RELEGATED_TEAMS NRT)
  GROUP BY SEASON, TEAM, COUNTRY, LEAGUE
  ORDER BY SEASON, TEAM;
  
  SELECT * FROM TEAM_WIN_LOSE ;
  ```
  <img width="550" alt="Untitled (25)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/95f63aaf-a802-407a-b818-07f3a4641e03">

- The team with the most wins in all season
    
    After creating the view, I then want to see the team with the most wins for the entire season. To do that, I used this query:
  ```SQL
  SELECT
  TEAM,
  COUNTRY,
  LEAGUE,
  SUM(TOTAL_WIN) AS TOTAL_WIN
  FROM TEAM_WIN_LOSE
  GROUP BY TEAM, COUNTRY, LEAGUE
  ORDER BY TOTAL_WIN DESC;
  ```
  <img width="400" alt="Untitled (26)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/b2cd5ba9-8988-42a7-b6d7-f088fe97d4a7">

  From the result, we can see that **FC Barcelona** ranks number 1 in the most wins in the entire season with **234 wins**. The second also comes from the same country which is **Real Madrid** with **228 wins**. The rest of the teams were not even reaching 200 wins with **FC Bayern Munich** from Germany being the closest one with **193 wins**.

- Team with the fewest losses in all seasons
    
    The next part will be to see the team with the least defeats for the entire season. The query will be like this:
  ```SQL
  SELECT
  TEAM,
  COUNTRY,
  LEAGUE,
  SUM(TOTAL_LOSE) AS TOTAL_LOSE
  FROM TEAM_WIN_LOSE
  GROUP BY TEAM, COUNTRY, LEAGUE
  ORDER BY TOTAL_LOSE;
  ```
  <img width="400" alt="Untitled (27)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/e439d9b3-cdd4-4bb8-bae7-05b31718ed48">

  From the result, we can see that the teams in the top five rankings for this are also the same teams that are in the teams with the most wins just discussed. They are **FC Barcelona** with only **70 defeats**, **Real Madrid** with **76 defeats**, and **FC Bayern Munich** with **79 defeats**. They are only three teams that got lost less than 100 times, other than team, all the teams had more than 110 times lost.

- The winning percentage of each team
    
    to see the winning percentage of each team, I used this query:
  ```SQL
  SELECT TEAM, 
  ROUND(SUM(TOTAL_WIN)*100.0/SUM(TOTAL_MATCH),2) AS WINRATE
  FROM TEAM_WIN_LOSE TWL
  GROUP BY TEAM
  ORDER BY WINRATE DESC;
  ```
  <img width="200" alt="Untitled (28)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/e7f2ad58-d513-4134-9605-aea8419fdf81">

  From the query, as we have known the list of the team with the most wins. The **top 3 teams** have a winning percentage **above 70%**. meanwhile, **the rest** of them only have a win rate of **less than 63%**.

- Average win by each team per season
    
    to see the average win done by every team in a single season, I used this query:
  ```SQL
  SELECT TEAM, 
  ROUND(AVG(TOTAL_WIN),2) AS AVG_WIN,
  floor(sum(total_match)/8) as total_match_per_season
  FROM TEAM_WIN_LOSE TWL
  GROUP BY TEAM
  ORDER BY AVG_WIN DESC;
  ```
  <img width="300" alt="Untitled (29)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/0ebeccbb-57a5-403c-9dd3-3cdca5b39b17">

  To make the data more clear, I also added the total match per season just to see the difference between the average win and total match. From the result, we can see that **FC Barcelona** has an average win of **29.25** from 38 matches in a season. Seemingly not much different from the leaders, **Real Madrid** recorded a total average of** 28.5** wins per season. That's a difference of at least 4 wins compared to the teams below them who average **20 to 24 wins per season**.

- Total goals and wins correlation
    
    It's just a curiosity to see the relationship between the number of goals scored and the total wins.
  
  <img width="960" alt="Untitled (30)" src="https://github.com/adangkurnia/euro-football-stats-analysis/assets/65482851/f0605941-a090-4ce6-8da6-426d30bcffa4">

  Based on the above diagram, there seems to be a positive correlation between the number of goals scored and the number of victories obtained.

# Data Visualization

I have created a summary of charts using Tableau Public to gain a deeper understanding of the data. You can view it by clicking on this [link](https://public.tableau.com/views/EuroFootball/EuroFootballStats?:language=en-US&publish=yes&:display_count=n&:origin=viz_share_link).

# Conclusion

Based on the analysis conducted, it seems that two Spanish teams, FC Barcelona and Real Madrid, dominated in almost every aspect of the sport. Their remarkable record each season, particularly in terms of goals and total wins, outshone most other teams, with only a few, such as FC Bayern Munich, coming close to their numbers. Although the number of matches played and the level of opponent strength varied across different leagues, there is a clear correlation between the data analyzed and the achievements of Spanish teams in the European arena that season. FC Barcelona won three UCL titles, while Real Madrid won two, meaning that they won more than half of all UCL titles during that period.

That's the result of analyzing the performance of each team from the top five leagues in Europe using SQL and Tableau Public. I'm sure there are still many shortcomings such as ineffective queries and inappropriate data presentation. But I will try to improve it in the future. Peace!
