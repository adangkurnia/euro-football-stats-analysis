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

<img width="550" alt="Untitled" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/600c1fb4-6193-4244-b986-f95494a044b2">


All tables have a relationship so they can be joined by each other. The ERD looks like this:

 <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/9fca5ac4-4f47-45ff-ae89-b44fa774b352">

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
  <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/ec073161-1825-489c-a6ad-be83509a217c">

- Add country_id column in Team table and set Foreign Key of country id from country table to team table

  To connect the Team table and the Country table, I added a new column and set Foreign Key to the Team table using this query:
  <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/625b2c26-b456-4de3-88c7-e00f0f7903a0">

- Update values country_id column in Team table

  Since there are only five countries, then I update the values of the country_id column using a single query like this:

  <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/ac1a9ca4-f503-4c76-88e9-fad55bfd2299">

- Delete teams that are not part of the top 5 leagues in the team table

  Since the analysis will be done only for teams from top 5 leagues in Europe, the list of teams from out of that area will be deleted. To delete those rows of data, I used this query:
  
  <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/6a8f5aaa-b651-4308-8a57-e46c2385b323">
  
- Delete teams that are not from the top 5 leagues in the Match table

  The same thing also will be done in Match table with this query:
  <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/66a377f6-fbb2-4538-b943-d7f4c7d5aec9">

# Exploratory Data Analysis

In general, my analysis will consist of three main components: identifying the team participants followed by constraints criteria, summarizing the goals, and summarizing the wins. 

- Total team participants in each league in a season

  I want to see the total team participants from each league in a season just to compare it to those teams that have never been relegated and how many teams have been relegated for the entire season. To do that, I use this query:

  <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/5b98ae43-f9ea-4760-ba87-ec912e69ae57">
  
  <img width="550" alt="Untitled (1)" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/f7e1ce50-df17-4fa1-a980-72e31a7aac97">

  We can see from the result that there are 20 team participants in all countries except Germany that only has 18 teams.

- Find a list of teams that the country has never relegated for the entire season

  To make the analysis more objective, the teams that will be analyzed are only those who never got relegated for all seasons from 2008 to 2016. To do that, I used this query to filter the list of teams following the criteria. I also set this query as the view, so I can use it later to join with other tables.

  <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/87b790f7-1e9c-430d-95dc-185d42686b42">

  <img width="550" alt="image" src="https://github.com/adangkurnia/euro-football-analysis/assets/65482851/9aca8946-ddab-4539-8cdd-856971c6cc4f">

  We can see from the result there are 50 teams from all the leagues that never relegated from 2008 to 2016.

