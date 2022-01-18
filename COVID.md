# Portfolio-Project
Global COVID Data

use PortfolioProject

--Display Total Results from Deaths and Vaccinations

SELECT * 
FROM PortfolioProject..['Covid deaths$']
ORDER BY 3,4

SELECT * 
FROM PortfolioProject..['Covid Vaccinations$']
ORDER BY 3,4

--Display Cases by Location (in this case South Africa)
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..['Covid deaths$']
WHERE location like 'South Africa'

SELECT * 
FROM PortfolioProject..['Covid Vaccinations$']
WHERE location like 'South Africa'

--Display Total Vaccinations in South Africa
Select location, sum(cast(new_vaccinations as bigint)) as Total_Vaccinations
FROM PortfolioProject..['Covid Vaccinations$']
Where location like 'south Africa'
Group By Location

--Display New Vaccinations by Date in South Africa
Select date, location, new_vaccinations
FROM PortfolioProject..['Covid Vaccinations$']
Where location like 'South Africa' AND new_vaccinations is not null

-- Select Data that we are going to be using

Select location, date, total_cases, new_cases, total_deaths, population
From PortfolioProject..['Covid deaths$']
order by 1,2

--Looking at total cases vs total deaths

Select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as Death_Rate
From PortfolioProject..['Covid deaths$']
Where total_deaths is not null AND location like 'South Africa'
order by 1,2

-- Looking at cases and deaths before and after vaccination in South Africa

Select Distinct d.location, d.date, d.total_cases, d.total_deaths, v.new_vaccinations, (total_deaths/total_cases)*100 as Death_Percentage
From PortfolioProject..['Covid deaths$'] d
JOIN PortfolioProject..['Covid Vaccinations$'] v on d.date = v.date
Where total_deaths is not null AND d.location like 'South Africa'
Group by d.location, d.date, d.total_cases, d.total_deaths, v.new_vaccinations
order by 1,2

-- Looking at Total Cases vs Population (Case Rate)

Select location, date, total_cases, population, (total_cases/population)*100 as Case_Rate
From PortfolioProject..['Covid deaths$']
Where total_deaths is not null AND location like 'South Africa'
order by total_cases

--Looking at Total Cases and Deathe Rate vs Population per Country

Select location, date, total_cases, population, (total_deaths/population)*100 as Death_Percentage
From PortfolioProject..['Covid deaths$']
--Where total_deaths is not null AND location like 'South Africa'
order by 1,2

--Highest Infection Rate per Country

Select location, population, MAX(total_cases) as Highest_Infection_Count, MAX(total_cases/population)*100 as Percent_Population_Infected
From PortfolioProject..['Covid deaths$']
--Where total_deaths is not null AND location like 'South Africa'
Group by location, population
order by Percent_Population_Infected

--Showing Countries with Highest Death Count per Population

Select distinct location, MAX(cast(total_deaths as int)) as Total_Death_Count, MAX(total_deaths/total_cases)*100 as Infection_Death_Rate
From PortfolioProject..['Covid deaths$']
Where continent is not null
Group by location
order by Infection_Death_Rate desc

--Looking at Infection Rate by Income

Select location, population, MAX(total_cases) as Highest_Infection_Count, MAX(total_cases/population)*100 as Percent_Population_Infected
From PortfolioProject..['Covid deaths$']
Where location like'%income'
Group by location, population
order by Percent_Population_Infected

--Total Deaths by Continent

Select continent, MAX(cast(total_deaths as bigint)) as Total_Death_Count
From PortfolioProject..['Covid deaths$']
Where continent is not null and location not like '%income'
Group by continent
order by Total_Death_Count desc

--GLOBAL NUMBERS totals and percentage by day

Select date, SUM(new_cases) as totalcases, SUM(cast(new_deaths as int)) as totaldeaths, SUM(cast(new_deaths as int))/sum(new_cases)*100 as DeathRate
From PortfolioProject..['Covid deaths$']
where continent is not null
group by date
order by 1,2

--Global Numbers Totals and percentage

Select SUM(new_cases) as totalcases, SUM(cast(new_deaths as int)) as totaldeaths, SUM(cast(new_deaths as int))/sum(new_cases)*100 as DeathRate
From PortfolioProject..['Covid deaths$']
where continent is not null
--group by date
order by 1,2

-- Total Population vs Vaccination

SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations
	FROM PortfolioProject..['Covid deaths$'] d
	Join PortfolioProject..['Covid Vaccinations$'] v 
			on d.location = v.location
			and d.date = v.date
Where d.continent is not null
order by 2,3

SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations, 
	SUM(CONVERT(bigint,v.new_vaccinations)) OVER (Partition by d.location Order by d.location, d.date) as RollingPeopleVaccinated
FROM PortfolioProject..['Covid deaths$'] d
Join PortfolioProject..['Covid Vaccinations$'] v 
		on d.location = v.location
		and d.date = v.date
Where d.continent is not null
order by 2,3

--USE CTE

With PopvsVAC (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations, 
	SUM(CONVERT(bigint,v.new_vaccinations)) OVER (Partition by d.location Order by d.location, d.date) as RollingPeopleVaccinated
FROM PortfolioProject..['Covid deaths$'] d
Join PortfolioProject..['Covid Vaccinations$'] v 
		on d.location = v.location
		and d.date = v.date
Where d.continent is not null
)
Select *, (RollingPeopleVaccinated/Population)*100 VaccinationRate
FROM PopvsVAC
Where Location like 'South Africa'


--TEMP TABLE

DROP Table IF exists #PercentPopulationVaccinated
CREATE TABLE #PercentPopulationVaccinated
(
Continent NVarchar(255),
Location NVarchar (255),
Date datetime,
Population numeric,
New_Vaccinations numeric,
RollingPeopleVaccinated numeric
)

	INSERT INTO #PercentPopulationVaccinated
	SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations, 
		SUM(CONVERT(bigint,v.new_vaccinations)) OVER (Partition by d.location Order by d.location, d.date) as RollingPeopleVaccinated
	FROM PortfolioProject..['Covid deaths$'] d
	Join PortfolioProject..['Covid Vaccinations$'] v 
			on d.location = v.location
			and d.date = v.date
	Where d.continent is not null and d.date > '2021-11-14' 
	order by 2,3

	SELECT *, (RollingPeopleVaccinated/Population)*100 as Vaccination_Rate
	FROM #PercentPopulationVaccinated
