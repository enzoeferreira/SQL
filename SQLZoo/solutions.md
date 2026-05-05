# Soluções SQLZoo
## Sumário
1. SELECT basics
2. SELECT from world
3. SELECT from nobel
4. [SELECT in SELECT](#select-within-select-tutorial)
5. SUM and COUNT
6. JOIN
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