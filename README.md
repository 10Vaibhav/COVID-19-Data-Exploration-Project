# COVID-19 Data Exploration Project

## Overview
This project analyzes COVID-19 mortality and vaccination trends across countries and continents to identify infection rates, death percentages, and vaccination progress using SQL Server.

**File:** `Covid Deaths Report.sql`  
**Database:** PortfolioProject.CovidDeaths & CovidVaccinations  
**Source Data:** CovidDeaths.xlsx, CovidVaccinations.xlsx

## Objective
Explore COVID-19 pandemic data to uncover insights about death rates, infection spread, and vaccination rollout across different geographic locations and time periods.

## Analysis Components

1. **Death Analysis**
   - Calculated death percentages by country
   - Analyzed total deaths vs total cases
   - Identified countries with highest death counts
   - Created continental death statistics

2. **Infection Analysis**
   - Calculated infection rates relative to population
   - Identified countries with highest infection rates
   - Tracked total cases vs population percentage

3. **Vaccination Analysis**
   - Created rolling counts of vaccinations
   - Analyzed population vs vaccination rates
   - Used CTE and Temp Tables for complex calculations

4. **Visualization Preparation**
   - Created views for storing transformed data
   - Prepared datasets for later visualization

## Detailed Analysis

### 1. Mortality Analysis

#### Death Percentage by Country
```sql
-- Shows likelihood of dying if you contract COVID in your country
SELECT location, date, total_cases, total_deaths, 
    (total_deaths/total_cases)*100 AS DeathPercentage
FROM CovidDeaths
WHERE location LIKE 'india' AND continent IS NOT NULL
ORDER BY 1,2
```

#### Highest Death Counts
```sql
-- Countries with highest death counts
SELECT location, MAX(CAST(total_deaths AS INT)) AS TotalDeathCount
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location
ORDER BY TotalDeathCount DESC
```

#### Continental Death Statistics
```sql
-- Continents with highest death counts
SELECT continent, MAX(CAST(total_deaths AS INT)) AS TotalDeathCount
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC
```

### 2. Infection Rate Analysis

#### Population Infection Percentage
```sql
-- Shows what percentage of population got COVID
SELECT location, date, total_cases, population, 
    (total_cases/population)*100 AS InfectedPercentage
FROM CovidDeaths
WHERE location LIKE 'india' AND continent IS NOT NULL
ORDER BY 1,2
```

#### Highest Infection Rates
```sql
-- Countries with highest infection rate compared to population
SELECT location, population, 
    MAX(total_cases) AS HighestInfectionCount,
    MAX((total_cases/population)*100) AS PercentPopulationInfected
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location, population
ORDER BY PercentPopulationInfected DESC
```

### 3. Vaccination Tracking

#### Rolling Vaccination Count
```sql
-- Cumulative vaccinations by location over time
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(CONVERT(INT, vac.new_vaccinations)) OVER (
        PARTITION BY dea.location 
        ORDER BY dea.location, dea.date
    ) AS RollingPeopleVaccinated
FROM CovidDeaths AS dea
JOIN CovidVaccinations AS vac
    ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
ORDER BY 2,3
```

### 4. Advanced Calculations

#### Using CTE
```sql
-- Calculate vaccination percentage using CTE
WITH PopVsVac (continent, location, date, population, new_vaccinations, RollingPeopleVaccinated)
AS (
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
        SUM(CONVERT(INT, vac.new_vaccinations)) OVER (
            PARTITION BY dea.location 
            ORDER BY dea.location, dea.date
        ) AS RollingPeopleVaccinated
    FROM CovidDeaths AS dea
    JOIN CovidVaccinations AS vac
        ON dea.location = vac.location AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated/population) * 100 
FROM PopVsVac
```

#### Using Temp Table
```sql
-- Alternative approach using temporary table
DROP TABLE IF EXISTS #PercentPopulationVaccinated
CREATE TABLE #PercentPopulationVaccinated (
    continent NVARCHAR(255),
    location NVARCHAR(255),
    date DATETIME,
    population NUMERIC,
    new_vaccinations NUMERIC,
    RollingPeopleVaccinated NUMERIC
)

INSERT INTO #PercentPopulationVaccinated
-- [Same query as CTE approach]

SELECT *, (RollingPeopleVaccinated/population) * 100 
FROM #PercentPopulationVaccinated
```

### 5. Global Statistics
```sql
-- Worldwide aggregated cases and deaths
SELECT SUM(new_cases) AS total_cases, 
    SUM(CAST(new_deaths AS INT)) AS total_deaths,
    SUM(CAST(new_deaths AS INT))/SUM(new_cases) * 100 AS DeathPercentage
FROM CovidDeaths
WHERE continent IS NOT NULL
```

### 6. View Creation for Visualization
```sql
-- Persistent view for BI tools (Tableau, Power BI)
CREATE VIEW PercentPopulationVaccinated AS 
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(CONVERT(INT, vac.new_vaccinations)) OVER (
        PARTITION BY dea.location 
        ORDER BY dea.location, dea.date
    ) AS RollingPeopleVaccinated
FROM CovidDeaths AS dea
JOIN CovidVaccinations AS vac
    ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
```

## Technical Components Used
- CTEs (Common Table Expressions)
- Temporary Tables
- Window Functions
- Views
- Joins
- Aggregate Functions
- Data Type Conversion
- String Functions

## Technical Skills Demonstrated
- **Multi-table joins** (INNER JOIN on location and date)
- **Window functions** (SUM OVER with PARTITION BY and ORDER BY)
- **CTEs** for query organization and reusability
- **Temporary tables** for intermediate calculations
- **Views** for persistent data access
- **Aggregate functions** (SUM, MAX, COUNT)
- **Type casting** (CAST, CONVERT)
- **Filtering** (WHERE, GROUP BY, ORDER BY)
- **Calculated fields** for percentage metrics

## Key Insights Enabled
- Country-specific death likelihood and infection rates
- Comparative analysis across continents
- Vaccination progress tracking over time
- Global pandemic impact metrics
- Population-adjusted infection and death rates

## Use Cases
This analysis supports:
- Public health policy decisions
- Pandemic trend visualization
- Vaccination campaign effectiveness
- Geographic risk assessment
- Time-series forecasting

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
