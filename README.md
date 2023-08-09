# Introduction

This project aims to showcase the performance of teams in Europe's top 5 leagues from the 2008/2009 to 2015/2016 seasons. The analysis is based on various football metrics, including goals productivity, defense strength, number of wins and losses, and other important numbers. In addition to utilizing SQL queries, I will also be presenting the data in a visual format using Tableau Public.

# Content
1.	Dataset
2.	Data Understanding
3.	Constraint
4.	Tools
5.	Data Cleaning
6.	Exploratory Data Analysis
7.	Data Visualization
8.	Conclusion

# Dataset
I got the dataset from Kaggle, you can download it on this [link](https://www.kaggle.com/code/dimarudov/data-analysis-using-sql/input).

# Data Understanding

There are 7 tables inside the database. Those tables are Match, Country, League, Team, Team Attributes, Player, Player Attributes. The total rows and columns of each table can be seen in this picture:

<img width="400" alt="Untitled" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/600c1fb4-6193-4244-b986-f95494a044b2">


All tables have a relationship so they can be joined by each other. The ERD looks like this:

 <img width="400" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/9fca5ac4-4f47-45ff-ae89-b44fa774b352">

# Constraints

- The tables that will be used for this project are Match, Country, League, And Team.
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
  
  <img width="400" alt="Untitled (1)" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/f7e1ce50-df17-4fa1-a980-72e31a7aac97">

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

  <img width="400" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/9aca8946-ddab-4539-8cdd-856971c6cc4f">

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

  <img width="400" alt="Untitled (2)" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/53b6e8d7-6c67-40aa-9c7c-787901703a6c">
  
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
   <img width="550" alt="Untitled (3)" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/ffc21fd9-2a56-4a05-9d1f-b914e2b9df9e">

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
    <img width="600" alt="Untitled (4)" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/254b127c-4d53-4512-b4c1-40bf452e934e">

After creating a View table that displays total goals and conceded from the previous queries, I analyzed Team performances by breaking down the data.      
