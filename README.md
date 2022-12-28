# COVID-19 Analysis

### Analyzing COVID-19 stats to date 22/12/2022.
### COVID-19 Dataset
* The dataset came from https://ourworldindata.org/covid-deaths
* For simplicity porpouses, I've split the dataset into 2 tables, covid_deaths and covid_vaccinations.
* The aim for this study is to analyze the impact of covid-19 in different countries using SQL.

### First Step: Get to know your data
First of all, we will run a query to familiarize ourselves with the dataset. This dataset has already been cleaned up, if you want to see how to analyze and clean your dataset, check out this repository HERE!
```sql
SELECT *
FROM coviddeaths
ORDER BY 1,2
```
![image](https://user-images.githubusercontent.com/39070251/209868408-0c7a554a-2ae7-44f6-b09b-e922a8d80172.png)

### Q1: How many people died from COVID-19 and what is the percentage of people that got infected in 'CHILE'?
```sql
SELECT location, date, total_cases, population, (total_cases/population)*100 AS Infectious_rate
FROM coviddeaths
WHERE location LIKE '%Chile%'
ORDER BY date DESC
```
![image](https://user-images.githubusercontent.com/39070251/209868979-8ca85146-2314-46fe-931c-7b9fd72003a8.png)

**Insights:** From the query we can see that from the total population of chile, 25% has been infected, almost 5 million people.

### Q2: Which are the countries with the highest infectious rates?
```sql
SELECT location, population, MAX(CAST(total_cases AS SIGNED)) as Highest_infection_count, MAX((total_cases/population)*100) as Max_infectious_rate
from coviddeaths
group by location, population
order by Max_infectious_rate DESC
```
![image](https://user-images.githubusercontent.com/39070251/209870742-07bef72a-7a47-4b70-b272-7776ee2e4f88.png)

**Insights:** Cyprus, San Marino and Faeroe Islands leads the infectious rate per zone of the world, with a peak of 67% of infectious rate, that´s 2/3 of the total population that got COVID-19 at some point of the pandemic.

### Q3: Which are the countries with the highest death count compared to the population?
```sql
SELECT continent, location, MAX(CAST(total_deaths AS SIGNED)) as Total_deathCount
from coviddeaths
group by continent, location
order by Total_deathCount DESC
```
![image](https://user-images.githubusercontent.com/39070251/209872837-f209fba0-0440-4631-b308-eb9798702aa5.png)

Here we face a *problem*. The location column also has calculated values for some groups, as south america or per income groups. I noticed that this calculated values don´t belong to any continent, so 1 encountered solution its to filter for all the values that has a not null value for 'continent' column.

```sql
SELECT continent, location, MAX(CAST(total_deaths AS SIGNED)) as Total_deathCount
FROM coviddeaths
WHERE continent<>''
GROUP BY continent, location
ORDER BY Total_deathCount DESC
```
![image](https://user-images.githubusercontent.com/39070251/209873262-d8571819-f0c6-4711-84c2-56232ae270e8.png)

**Insights:** Now we can see that almost all the biggest countries of each continent were the most affected by the pandemic in terms of total deaths, with the USA leading the board with more than +1 million deaths.



