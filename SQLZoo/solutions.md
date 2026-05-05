# Soluções SQLZoo
## Sumário
1. SELECT basics
2. SELECT from world
3. SELECT from nobel
4. [SELECT in SELECT](#select-within-select-tutorial)
5. [SUM and COUNT](#sum-and-count)
6. [JOIN](#join)
7. More JOIN
8. Using NULL
9. Self JOIN

## SELECT within SELECT Tutorial
### world(name, continent, area, population, gdp)
1. List each country name where the population is larger than that of 'Russia'.
```sql
SELECT name
FROM world
WHERE population > (SELECT population FROM world WHERE name = 'Russia')
```

2. Show the countries in Europe with a per capita GDP greater than 'United Kingdom'.
```sql
SELECT name
FROM world
WHERE continent = 'Europe' AND
	  gdp/population > (SELECT gdp/population FROM world WHERE name = 'United Kingdom')
```

3. List the name and continent of countries in the continents containing either Argentina or Australia. Order by name of the country.
```sql
SELECT name, continent
FROM world
WHERE continent IN (SELECT continent FROM world WHERE name IN ('Argentina', 'Australia'))
ORDER BY name
```

4. Which country has a population that is more than United Kingdom but less than Germany? Show the name and the population.
```sql
SELECT name, population
FROM world
WHERE population > (SELECT population FROM world WHERE name = 'United Kingdom') AND
	  population < (SELECT population FROM world WHERE name = 'Germany')
```

5. Show the name and the population of each country in Europe. Show the population as a percentage of the population of Germany.
```sql
SELECT name, CONCAT(ROUND(100*population/(SELECT population FROM world WHERE name = 'Germany')), '%') percentage
FROM world
WHERE continent = 'Europe'
```

6. Which countries have a GDP greater than every country in Europe? [Give the name only.] (Some countries may have NULL gdp values)
```sql
SELECT name
FROM world
WHERE gdp > ALL(SELECT gdp FROM world WHERE gdp>0 AND population>0 AND continent = 'Europe')
```

7. Find the largest country (by area) in each continent, show the continent, the name and the area:
```sql
SELECT continent, name, area
FROM world a
WHERE area >= ALL(SELECT area FROM world b WHERE a.continent = b.continent)
```

8. List each continent and the name of the country that comes first alphabetically.
```sql
SELECT continent, name
FROM world a
WHERE name <= ALL(SELECT name FROM world b WHERE b.continent = a.continent)
```

9. Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.
```sql
SELECT name, continent, population
FROM world a
WHERE population >= ALL(SELECT population FROM world b WHERE b.continent=a.continent) AND population <= 25000000
```

10.  Some countries have populations more than three times that of all of their neighbours (in the same continent). Give the countries and continents.
```sql
SELECT name, continent
FROM world a
WHERE a.population >= ALL(SELECT 3*population FROM world b WHERE b.continent=a.continent AND a.name <> b.name)
```

## SUM and COUNT
### world(name, continent, area, population, gdp)
1. Show the total population of the world.
```sql
SELECT SUM(population)
FROM world
```

2. List all the continents - just once each.
```sql
SELECT DISTINCT continent
FROM world
```

3. Give the total GDP of Africa
```sql
SELECT SUM(gdp)
FROM world
WHERE continent = 'Africa'
```

4. How many countries have an area of at least 1000000
```sql
SELECT COUNT(*)
FROM world
WHERE area >= 1000000
```

5. What is the total population of ('Estonia', 'Latvia', 'Lithuania')
```sql
SELECT SUM(population)
FROM world
WHERE name IN ('Estonia', 'Latvia', 'Lithuania')
```

6. For each continent show the continent and number of countries.
```sql
SELECT continent, COUNT(name) number_of_countries
FROM world
GROUP BY continent
```

7. For each continent show the continent and number of countries with populations of at least 10 million.
```sql
SELECT continent, COUNT(name)
FROM world
WHERE population >= 10000000
GROUP BY continent
```

8. List the continents that have a total population of at least 100 million.
```sql
SELECT continent
FROM world
GROUP BY continent
HAVING SUM(population) >= 100000000
```

## JOIN
### game(id, mdate, stadium, team1, team2)
### goal(matchid, teamid, player, gtime)
### eteam(id, teamname, coach)

1. Show the matchid and player name for all goals scored by Germany.
```sql
SELECT matchid, player
FROM goal
WHERE teamid = 'GER'
```

2. Show id, stadium, team1, team2 for just game 1012.
```sql
SELECT id, stadium, team1, team2
FROM game 
WHERE id = 1012
```

3. Show the player, teamid, stadium and mdate for every German goal.
```sql
SELECT player, teamid, stadium, mdate
FROM goal JOIN game ON (id=matchid)
WHERE teamid = 'GER'
```

4. Show the team1, team2 and player for every goal scored by a player called Mario.
```sql
SELECT team1, team2, player
FROM goal JOIN game ON (id=matchid)
WHERE player LIKE 'Mario%'
```

5. Show player, teamid, coach, gtime for all goals scored in the first 10 minutes.
```sql
SELECT player, teamid, coach, gtime
FROM goal JOIN eteam ON (teamid=id)
WHERE gtime <= 10
```

6. List the dates of the matches and the name of the team in which 'Fernando Santos' was the team1 coach.
```sql
SELECT mdate, teamname
FROM game GM, eteam T
WHERE T.id = GM.team1 AND T.coach = 'Fernando Santos'
```

7. List the player for every goal scored in a game where the stadium was 'National Stadium, Warsaw'
```sql
SELECT player
FROM goal G, game GM
WHERE G.matchid = GM.id AND GM.stadium = 'National Stadium, Warsaw'
```

8. Instead show the name of all players who scored a goal against Germany.
```sql
SELECT DISTINCT G.player
FROM goal G, game GM
WHERE G.matchid = GM.id AND 'GER' IN (team1, team2) AND G.teamid <> 'GER'
```

9.  Show teamname and the total number of goals scored.
```sql
SELECT T.teamname, COUNT(*) goals
FROM eteam T, goal G
WHERE G.teamid = T.id
GROUP BY T.teamname
```

10. Show the stadium and the number of goals scored in each stadium.
```sql
SELECT stadium, COUNT(*) goals
FROM game JOIN goal ON id=matchid
GROUP BY stadium
```

11. For every match involving 'POL', show the matchid, date and the number of goals scored.
```sql
SELECT matchid, mdate, COUNT(*) goals
FROM game JOIN goal ON id=matchid
WHERE team1 = 'POL' OR team2 = 'POL'
GROUP BY matchid, mdate
```

12. For every match where 'GER' scored, show matchid, match date and the number of goals scored by 'GER'.
```sql
SELECT matchid, mdate, COUNT(*) goals
FROM game JOIN goal ON id=matchid
WHERE teamid = 'GER'
GROUP BY matchid, mdate
```

13. List every match with the goals scored by each team for all ENG games as shown.
```sql
SELECT mdate,
       team1, SUM(CASE WHEN teamid=team1 THEN 1 ELSE 0 END) AS score1, 
       team2, SUM(CASE WHEN teamid=team2 THEN 1 ELSE 0 END) AS score2
FROM game LEFT JOIN goal ON game.id = goal.matchid
WHERE 'ENG' IN (team1, team2)
GROUP BY mdate, game.id, team1, team2;
```