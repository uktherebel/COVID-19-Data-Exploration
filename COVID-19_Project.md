```SQL

-- Select Data that we are going to be using 

SELECT location, date, total_cases, new_cases, total_deaths, population
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2



-- COVID Cases vs. Death Using 
-- Shows liklihood of death if you contract COVID-19 in your country

SELECT location, date, total_cases, total_deaths, cast((1.00*total_deaths/total_cases)*100 AS decimal(10,2)) 
    AS DeathPercentage
    -- 1.00 is used to avoid the integer floor problem 
FROM COVID_Project..COVID_DEATHS
WHERE location LIKE '%States%'
AND continent IS NOT NULL
ORDER BY 1, 2



-- COVID CASES AS A PERCENTAGE OF POPULATION IN PAKISTAN to Two Decimal Places

SELECT date, location, total_cases, population, cast((1.00*total_cases/population)*100 AS Decimal(20,2)) AS InfectionPercentage
FROM COVID_Project..COVID_DEATHS
WHERE [location] = 'Pakistan'
ORDER BY 1, 2



-- Country Analysis: Countries with Highest Infection Rate compared to Population

-- Highest Percentage of Population Infected
SELECT location, population, MAX(total_cases) AS HighestInfectionCount, MAX(cast((1.00*total_cases/population)*100 AS Decimal (20,2))) 
    AS InfectionPercent
FROM COVID_Project..COVID_DEATHS
GROUP BY location, population
ORDER BY 4 desc

-- Where Continent is "null," the continent is listed in location
-- Also removing income-class numbers 
SELECT *
FROM COVID_Project..COVID_DEATHS
WHERE continent IS NOT null 
AND [location] NOT LIKE '%income%'
ORDER BY 3,4

-- Shows highest death count for each country 
SELECT location, max(total_deaths) AS TotalDeathCount
FROM COVID_Project..COVID_DEATHS
WHERE continent is not null 
AND [location] NOT LIKE '%income%'
GROUP BY LOCATION
ORDER BY 2 desc



-- Continental Analysis: 


-- Showing contintents with the highest death count per population
SELECT location, max(total_deaths) AS TotalDeathCount
FROM COVID_Project..COVID_DEATHS
WHERE continent is null
AND [location] NOT LIKE '%income%'
GROUP BY LOCATION
ORDER BY 2 desc
-- We're doing this since the continent names are present in the 
	--location column as well
    


-- Global Numbers

SELECT 
    SUM(new_cases) AS TotalCases,
    SUM(new_deaths) AS TotalDeaths, 
    CAST(1.00*sum(new_cases)/sum(new_deaths)*100 AS Decimal(12,2)) AS DeathPercent
FROM COVID_Project..COVID_DEATHS
WHERE continent is not null 
ORDER BY 1,2 




-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
    sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION ORDER BY dea.LOCATION, dea.date) 
    AS RollingPeopleVaccinated--specifying the table column, sum
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
Order by 2,3 



-- Using CTE to perform Calculation on Partition By in previous query

WITH PopvVac (continent, location, date, population, new_vaccinations, RollingVaccinatedPeople) 
AS 
( 
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
    sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION ORDER BY dea.location, dea.date)
    AS RollingPeopleVaccinated
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
--Order by 2,3 --not allowed in the CTE 
) 
SELECT *, cast(round((1.00*RollingVaccinatedPeople/population)*100, 2) AS Decimal(10,2)) AS PercentagePopVaccinated
FROM PopvVac
ORDER BY [location], [date]  




-- Using TEMP TABLE to perform Calculation on Partition By in previous query

DROP TABLE IF EXISTS PopvsVac

CREATE TABLE #PopvsVac 
		(									)
    continent nvarchar(255),
    Location nvarchar(255),    
    date datetime, 
    Population bigint, 
    New_vaccinations bigint, 
    RollingPeopleVaccinated numeric)

INSERT INTO #PopvsVac 

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
sum(convert(BIGINT,vac.new_vaccinations)) OVER (PARTITION BY dea.LOCATION ORDER BY dea.location, dea.date)
AS RollingPeopleVaccinated
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null; 

SELECT *, cast(round((1.00*RollingPeopleVaccinated/population)*100, 4) AS DECIMAL (10,4)) AS PercentPopVaccinated 
FROM #PopvsVac



-- Creating View to store data for later visualizations

CREATE VIEW PopvsVac 
AS 
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION ORDER BY dea.location, dea.date)
AS RollingPeopleVaccinated
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
)

