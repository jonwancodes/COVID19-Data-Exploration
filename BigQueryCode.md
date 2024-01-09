## This dataset was obtained on March 7th, 2023 from the Coronavirus (COVID-19) Vaccinations 

```sql
SELECT *
FROM `covid-19-data-exploration.covid_deaths.covid_deaths`
ORDER BY 3,4;

-- Select Data that we are going to be using
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM `covid-19-data-exploration.covid_deaths.covid_deaths`
ORDER BY 1,2;

-- Looking at Total Cases vs Total Deaths
-- Shows likelihood of dying if you were to contract COVID-19 in specified country
SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS DeathPercentage
FROM `covid-19-data-exploration.covid_deaths.covid_deaths`
WHERE location like '%United States%'
ORDER BY 1,2 DESC;

-- Looking at Total Cases vs Population
-- Shows what percent of population in US got COVID-19
SELECT location, date, total_cases, population, (total_cases/population)*100 AS PopulationPercentInfected
FROM `covid-19-data-exploration.covid_deaths.covid_deaths`
WHERE location like '%United States%'
ORDER BY 1,2 DESC;

-- Looking at Countries with Highest Infection Rate Compared to Population
SELECT location, population, MAX(total_cases) As HighestInfectionCount, MAX((total_cases/population))*100 AS PopulationPercentInfected
FROM `covid-19-data-exploration.covid_deaths.covid_deaths`
GROUP BY location, population
ORDER BY PopulationPercentInfected DESC; 

--Showing Countries with Highest Death Count Per Population
SELECT location, MAX(total_deaths) AS TotalDeathCount
FROM `covid-19-data-exploration.covid_deaths.covid_deaths`
WHERE continent IS NOT NULL
GROUP BY location
ORDER BY TotalDeathCount DESC;

--Showing Countries with Highest People Vaccinated Count
SELECT location, MAX(people_vaccinated) AS TotalPeopleVaccinated
FROM `covid-19-data-exploration.covid_deaths.covid_vaccinations`
WHERE continent IS NOT NULL
GROUP BY location
ORDER BY TotalPeopleVaccinated DESC;

--Showing Countries with Highest People Fully Vaccinated Count
SELECT location, MAX(people_fully_vaccinated) AS TotalPeopleFullyVaccinated
FROM `covid-19-data-exploration.covid_deaths.covid_vaccinations`
WHERE continent IS NOT NULL
GROUP BY location
ORDER BY TotalPeopleFullyVaccinated DESC;


-- BREAKING DATA DOWN BY CONTINENT
--Showing Continents with Highest Death Count Per Population
SELECT continent, MAX(total_deaths) AS TotalDeathCount
FROM `covid-19-data-exploration.covid_deaths.covid_deaths`
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC;


-- Global Numbers

SELECT date, SUM(new_cases) AS TotalCases, SUM(new_deaths) AS TotalDeaths, SUM(new_deaths)/SUM(new_cases)*100 AS DeathPercentage
FROM `covid-19-data-exploration.covid_deaths.covid_deaths`
WHERE continent IS NOT NULL
GROUP BY date
ORDER BY 1,2;

-- Join covid_deaths & covid_vaccinations table
-- Using CTE, get a rolling count of people vaccinated and percent of population vaccinated
-- Note: new_vaccinations may also account for boosters

WITH PopvsVac
AS(
SELECT DEA.continent, DEA.location, DEA.date, DEA.population, VAC.new_vaccinations, SUM(VAC.new_vaccinations) OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS RollingPeopleVaccinated
FROM `covid-19-data-exploration.covid_deaths.covid_deaths` AS DEA
JOIN `covid-19-data-exploration.covid_deaths.covid_vaccinations` AS VAC
  ON DEA.location = VAC.location
  AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
ORDER BY 2, 3
)
SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentPopulationVaccinated
FROM PopvsVac;

-- Same as above, except using TEMP Table

CREATE TEMP TABLE PercentPopulationVaccinated
(
  continent string,
  location string,
  date datetime,
  population numeric,
  new_vaccinations numeric,
  RollingPeopleVaccinated numeric
);

INSERT INTO PercentPopulationVaccinated
SELECT DEA.continent, DEA.location, DEA.date, DEA.population, VAC.new_vaccinations, SUM(VAC.new_vaccinations) OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS RollingPeopleVaccinated
FROM `covid-19-data-exploration.covid_deaths.covid_deaths` AS DEA
JOIN `covid-19-data-exploration.covid_deaths.covid_vaccinations` AS VAC
  ON DEA.location = VAC.location
  AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
ORDER BY 2, 3;

SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentPopulationVaccinated
FROM PercentPopulationVaccinated;

-- Percent of Population Fully Vaccinated by Country

WITH PopvsFullVac
AS(
SELECT DEA.continent, DEA.location, MAX(DEA.population) AS Population, MAX(VAC.people_fully_vaccinated) AS PeopleFullyVaccinatedCount
FROM `covid-19-data-exploration.covid_deaths.covid_deaths` AS DEA
JOIN `covid-19-data-exploration.covid_deaths.covid_vaccinations` AS VAC
  ON DEA.location = VAC.location
  AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
GROUP BY DEA.continent, DEA.location
ORDER BY 2, 3
)
SELECT *, (PeopleFullyVaccinatedCount/Population)*100 AS PercentPopulationFullyVaccinated
FROM PopvsFullVac;
--WHERE location = 'United States';

-- Percent of Population who received at least one vaccine by Country

WITH PopvsVac
AS(
SELECT DEA.continent, DEA.location, MAX(DEA.population) AS Population, MAX(VAC.people_vaccinated) AS PeopleVaccinatedCount
FROM `covid-19-data-exploration.covid_deaths.covid_deaths` AS DEA
JOIN `covid-19-data-exploration.covid_deaths.covid_vaccinations` AS VAC
  ON DEA.location = VAC.location
  AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
GROUP BY DEA.continent, DEA.location
ORDER BY 2, 3
)
SELECT *, (PeopleVaccinatedCount/Population)*100 AS PercentPopulationVaccinated
FROM PopvsVac;
--WHERE location = 'United States';
```
