```SQL
SELECT *
FROM COVID_Project..COVID_DEATHS
ORDER BY 3, 4

SELECT *
FROM COVID_Project..COVID_Vaccinations
ORDER BY 3, 4

-- Select Data that we are going to be using 
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2

-- COVID Cases vs. Death
SELECT location, date, total_cases, total_deaths, round((total_deaths/total_cases), 2) AS DeathPercentage
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2

-- COVID Cases vs. Death: Using Implicit "Casting"
SELECT location, date, total_cases, total_deaths, (1.00*total_deaths/total_cases)*100
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2
--|The '1.00' is used to do implicit casting| 


SELECT location, date, total_cases, total_deaths, round((1.00*total_deaths/total_cases)*100, 2, 4)
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2


SELECT location, date, total_cases, total_deaths, cast((1.00*total_deaths/total_cases)*100 AS decimal(4,2)) 
    AS DeathPercentage
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2
-- Causes Integer Overflow --
SOLUTION:
SELECT location, date, total_cases, total_deaths, cast((1.00*total_deaths/total_cases)*100 AS decimal(10,2)) 
    AS DeathPercentage
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2


SELECT location, date, total_cases, total_deaths, total_deaths / CAST(total_cases AS DECIMAL) * 100 AS DeathPercentage
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2


SELECT location, date, total_cases, total_deaths, cast(total_deaths / CAST(total_cases AS DECIMAL) * 100 AS Decimal (10,2))
    AS DeathPercentage
FROM COVID_Project..COVID_DEATHS
ORDER BY 1, 2
--Increased the width of the variable to store this number i.e. making calculation DECIMAL (10,2) 
    -->solved Arithmetic overflow problem


SELECT location, date, total_cases, total_deaths, cast((1.00*total_deaths/total_cases)*100 AS decimal(10,2)) 
    AS DeathPercentage
FROM COVID_Project..COVID_DEATHS
WHERE location LIKE '%States%'
ORDER BY 1, 2


SELECT location, date, total_cases, total_deaths, cast((1.00*total_deaths/total_cases)*100 AS decimal(10,2)) 
    AS DeathPercentage
FROM COVID_Project..COVID_DEATHS
WHERE location IN ('pakistan', 'india')
ORDER BY 1 desc, 2


SELECT location, date, total_cases, total_deaths, cast((1.00*total_deaths/total_cases)*100 AS decimal(10,2)) 
    AS DeathPercentage
FROM COVID_Project..COVID_DEATHS
WHERE location = 'Pakistan'
ORDER BY 1 desc, 2
--shows likelihood of dying in Pakistan (my country)


--COVID CASES AS A PERCENTAGE OF POPULATION IN PAKISTAN to Two Decimal Places
SELECT date, location, total_cases, population, cast((1.00*total_cases/population)*100 AS Decimal(20,2)) AS InfectionPercentage
FROM COVID_Project..COVID_DEATHS
WHERE [location] = 'Pakistan'
ORDER BY 1

SELECT date, location, total_cases, population, cast(total_cases/cast(population AS decimal)*100 AS decimal(20,2)) AS InfectionPercentage
FROM COVID_Project..COVID_DEATHS
WHERE [location] = 'Pakistan'
ORDER BY 1

-- Interger Problem occurrs (0.00)
SELECT date, location, total_cases, population, cast((total_cases/population)*100 AS decimal(20,2)) AS InfectionPercentage
FROM COVID_Project..COVID_DEATHS
WHERE [location] = 'Pakistan'
ORDER BY 1

--Highest Percentage of Population Infected
SELECT location, population, MAX(total_cases) AS HighestInfectionCount, MAX(cast((1.00*total_cases/population)*100 AS Decimal (20,2))) 
    AS InfectionPercent
FROM COVID_Project..COVID_DEATHS
GROUP BY location, population
ORDER BY 4 desc

-- Shows countries with the highest death count per population
SELECT location, max(total_deaths) AS TotalDeathCount
FROM COVID_Project..COVID_DEATHS
GROUP BY LOCATION
ORDER BY 2 desc
-- Includes continent numbers too (cf. below)

-- Where Continent is "null," the continent is listed in location
SELECT *
FROM COVID_Project..COVID_DEATHS
WHERE continent IS NOT null 
ORDER BY 3,4

-- Shows countries with the highest death count per population (accurate)
SELECT location, max(total_deaths) AS TotalDeathCount
FROM COVID_Project..COVID_DEATHS
WHERE continent is not null 
GROUP BY LOCATION
ORDER BY 2 desc

-- Death Count by Continent 
SELECT continent, max(total_deaths) AS TotalDeathCount
FROM COVID_Project..COVID_DEATHS
WHERE continent is not null 
GROUP BY continent
ORDER BY 2 desc
-- doesn't seem very corrent (Alex); the query below seems more accurate

SELECT location, max(total_deaths) AS TotalDeathCount
FROM COVID_Project..COVID_DEATHS
WHERE continent is null
GROUP BY LOCATION
ORDER BY 2 desc
--includes high- and middle-income countries strings 

SELECT location, max(total_deaths) AS TotalDeathCount
FROM COVID_Project..COVID_DEATHS
WHERE continent is null
AND location NOT LIKE '%income'
GROUP BY LOCATION
ORDER BY 2 desc
--- doesn't include high- and middle-income countries strings 




--WORLD ANALYSIS-- 

SELECT date, sum(new_cases) AS TotalNewCases
FROM COVID_Project..COVID_DEATHS
GROUP BY date
ORDER BY date 
--faulty numbers: includes income-type location & "world" figures
--Accurate 
SELECT date, sum(new_cases) AS TotalNewCases
FROM COVID_Project..COVID_DEATHS
WHERE continent is not null 
GROUP BY date
ORDER BY date 

--New Deaths/New Cases & Percent 
SELECT date, sum(new_cases) AS TotalNewCases, sum(new_deaths) AS TotalNewDeaths, cast(1.00*sum(new_deaths)/sum(new_cases)*100 AS Decimal(12,2)) AS DeathPercent
FROM COVID_Project..COVID_DEATHS
WHERE continent is not null 
GROUP BY date
ORDER BY 1,2 

-- Global Death Rate (Total Deaths/Cases)
SELECT sum(total_cases) AS TotalCases, sum(total_deaths) AS TotalDeaths, cast(1.00*sum(total_deaths)/sum(total_cases)*100 AS Decimal(12,2)) AS DeathPercent
FROM COVID_Project..COVID_DEATHS
WHERE continent is not null 
ORDER BY 1,2 


--JOINS

SELECT*
FROM COVID_Project..COVID_DEATHS dea --"dea" is the alias
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location] --joining them on two things
    and dea.[date] = vac.[date]
Order by 3,4

--Selecting specific columns from the Join 
SELECT dea.continent, dea.location, dea.date, dea.population, vac.total_vaccinations --specifying the table column
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
Order by 2,3 

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
    sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION) 
    AS RollingPeopleVaccinated--specifying the table column, sum
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
Order by 2,3 
--Doesn't give a running total of new vaccines but an absolute total of vaccinations in every country in every column
-- Use ORDER BY cf. below 


--A running total of new vaccines 
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
    sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION ORDER BY dea.LOCATION, dea.date) -- ORDER BY this clause gives us the running total
    AS RollingPeopleVaccinated--specifying the table column, sum
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
Order by 2,3 
-- The ORDER BY (dea.LOCATION, dea.DATE) is telling SQL that the partitions created are to be ordered 
-- in ascending order based on location then date
-- dea.LOCATION allows SQL to restart sum when a new country comes on 
-- Omitting the dea.LOCATION in ORDER BY clause gives the same result, but why? 



--CTE 
    --The problem of Derived Columns: Erroneous Query 

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
    sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION ORDER BY dea.date)
    AS RollingPeopleVaccinated,(RollingPeopleVaccinated/population)*100--cannot use derived column to do calucation
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
Order by 2,3 


WITH PopvVac (continent, location, date, population, new_vaccinations, RollingVaccinatedPeople) --as columns in the query must also be up here 
AS 
( --all columns in the query must also be up here
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
SELECT *, (RollingPeopleVaccinated/population)*100
FROM PopvVac
ORDER BY [location], [date] 

--PercentPopVaccinated to 2 dec places
WITH PopvVac (continent, location, date, population, new_vaccinations, RollingVaccinatedPeople) --as columns in the query must also be up here 
AS 
( --as columns in the query must also be up here
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
--A CTE is not stored so you have to run it with the query 
-- DO more queries 



--SUBQUERY 
SELECT *, cast(round((1.00*RollingPeopleVaccinated/population)*100, 2) AS Decimal(10,2)) AS PercentagePopVaccinated
FROM
( --as columns in the query must also be up here
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
    sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION ORDER BY dea.location, dea.date)
    AS RollingPeopleVaccinated
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
--Order by 2,3 --not allowed in the CTE 
) AS SUBQUERY 


--TEMPTABLE 
DROP TABLE IF EXISTS PopvsVac

--always use '#' if you wanna create a temp table 
CREATE TABLE #PopvsVac ( --**datatypes with columns need to be inserted and all columns from query below must be here**
    continent nvarchar(255),
    Location nvarchar(255),    
    date datetime, 
    Population bigint, 
    New_vaccinations bigint, 
    RollingPeopleVaccinated numeric)

INSERT INTO #PopvsVac --**inserting this into above table 

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION ORDER BY dea.location, dea.date)
AS RollingPeopleVaccinated
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 

--Creating PercentPopVaccinated From TempTable
SELECT *, cast(round((1.00*RollingPeopleVaccinated/population)*100, 4) AS DECIMAL (10,4)) AS PercentPopVaccinated 
FROM #PopvsVac 

  

--Creating Views 


--CREATE VIEW PopvsVac 
--AS 
--(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
sum(vac.new_vaccinations) OVER (PARTITION BY dea.LOCATION ORDER BY dea.location, dea.date)
AS RollingPeopleVaccinated
FROM COVID_Project..COVID_DEATHS dea
JOIN COVID_Project..COVID_Vaccinations vac
    ON dea.[location] = vac.[location]
    and dea.[date] = vac.[date]
WHERE dea.continent IS NOT null 
--)

DROP VIEW PopvsVac 


```
