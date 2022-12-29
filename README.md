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

### Q4 : What is the global death percentage rate per date?
```sql
SELECT date, sum(new_cases) as total_NewCases, SUM(new_deaths) as total_deaths, (SUM(new_deaths)/sum(new_cases))*100 AS DeathPercentage 
from coviddeaths
where continent<>''
GROUP BY date
order by 1,2
```
![image](https://user-images.githubusercontent.com/39070251/209875739-5ddc6086-c39f-4f03-9d7a-4d091b9d3e8e.png)
![image](https://user-images.githubusercontent.com/39070251/209875792-816ef969-7373-469c-b363-383a237a2635.png)
![image](https://user-images.githubusercontent.com/39070251/209875859-a191ca0b-afcb-40b7-a521-6b4c5435d93d.png)

**Insights:** As we can see, covid´s death percentage is increasing through time, it would be interesting to see what was the peak of the trend and hoy many people were affected by the pandemic.

### Q5: ¿How many people were affected by COVID-19?
```sql
SELECT sum(new_cases) as total_NewCases, SUM(new_deaths) as total_deaths, (SUM(new_deaths)/sum(new_cases))*100 AS DeathPercentage 
from coviddeaths
where continent<>''
order by 1,2
```
![image](https://user-images.githubusercontent.com/39070251/209969640-433395d8-87e7-4244-bb80-fada2f6a68ab.png)

**Insights**: More than +650M people got sick from COVID-19 around the world, from this aprox 1% died, that´s more than 6.5 million people. 

### Q6: ¿How many people got vaccinated per country and per date? ¿ What was the accumulative number of vaccinated people?
```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(vac.new_vaccinations) OVER(PARTITION by dea.location ORDER BY dea.location, dea.date) as RollingPeopleVaccinated
FROM coviddeaths dea
JOIN covidvaccines vac ON dea.location = vac.location
and dea.date = vac.date
where dea.continent<>''
ORDER BY 2,3
```
![image](https://user-images.githubusercontent.com/39070251/209988784-42570bba-4842-4556-894c-eaaaf5e696a7.png)

**Insights:** Here we can see the levels of vaccinated people per conuntry and the evolution of this over time, it would be interesting to use this information to analyze how effective was the vaccination campaign over different countries and which got the best deploy strategy.

### Q7: Over time, it was neccesary to delivery more than 1 vaccine to every person, so ¿ How many vaccines the mean population of every country had?
```sql
 With PopvsVac(Continent, Location, Date, Population, New_vaccinations, RollingPeopleVaccinated)
 AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(vac.new_vaccinations) OVER(PARTITION by dea.location ORDER BY dea.location, dea.date) as RollingPeopleVaccinated
FROM coviddeaths dea
JOIN covidvaccines vac ON dea.location = vac.location
and dea.date = vac.date
where dea.continent<>''
GROUP BY dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
ORDER BY 2,3
)
Select *, (RollingPeopleVaccinated/Population) AS vacs_pp
from PopvsVac
ORDER BY vacs_pp DESC
```
![image](https://user-images.githubusercontent.com/39070251/209971799-b1f9c8ab-64d2-4ac2-8eb3-f1521783c7d4.png)

**Insights:** With this query we can see the evolution of the vaccines deploy for every country, with CUBA leading the board with more than 3 vaccines per person.

### Q8: Which Countries had the most effective vaccine campaign, measure by how many vaccines the population got?
```sql
 With PopvsVac(Continent, Location, Date, Population, New_vaccinations, RollingPeopleVaccinated)
 AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS SIGNED)) OVER(PARTITION by dea.location ORDER BY dea.location, dea.date) as RollingPeopleVaccinated
FROM coviddeaths dea
JOIN covidvaccines vac ON dea.location = vac.location
and dea.date = vac.date
where dea.continent<>''
GROUP BY dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
ORDER BY 2,3
)

Select Location, Population, MAX(RollingPeopleVaccinated) as Total_vaccinated ,MAX((RollingPeopleVaccinated/Population)) as vacs_pp
from PopvsVac
GROUP BY location, population
ORDER BY vacs_pp DESC
```
![image](https://user-images.githubusercontent.com/39070251/209989192-7917fa9e-2b15-4d16-a1e9-db35b2770a6f.png)


**Insights:** The countries with the most effective vaccine campaign deploy were Chile, Cuba and Japan, with a mean of 3 vaccines per person in every country.

