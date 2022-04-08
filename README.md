# Insights From Data With Big Query

This exercise is part of my Data Engineering and Machine Learning Fundamental training provided by Google. ‘Insights From Data With Big Query Challenge Lab’ is a non-guided lab under the quest of ‘Insights From Data With Big Query’ from Google Skills Boost. 

# Scenario

You're part of a public health organization which is tasked with identifying answers to queries related to the Covid-19 pandemic. Obtaining the right answers will help the organization in planning and focusing healthcare efforts and awareness programs appropriately.

The dataset and table that will be used for this analysis will be : ‘bigquery-public-data.covid19_open_data.covid19_open_data’. This repository contains country-level datasets of daily time-series data related to COVID-19 globally. It includes data relating to demographics, economy, epidemiology, geography, health, hospitalizations, mobility, government response, and weather.

Lab: https://www.cloudskillsboost.google/focuses/11988?parent=catalog

Dataset: https://console.cloud.google.com/marketplace/product/bigquery-public-datasets/covid19-open-data?q=search&referrer=search&project=sacred-particle-340305


## QUERY 1: Total Confirmed Cases
Build a query that will answer "What was the total count of confirmed cases on May 15, 2020 ?" The query needs to return a single row containing the sum of confirmed cases across all countries. The name of the column should be total_cases_worldwide.
Columns to reference:
* cumulative_confirmed
* date

```
SELECT SUM(cumulative_confirmed) AS total_cases_worldwide
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date = '2020-05-15'
GROUP BY DATE
```

## QUERY 2: Worst Affected Areas
Build a query for answering "How many states in the US had more than 200 deaths on May 15, 2020 ?" The query needs to list the output in the field count_of_states.
Hint: Don't include NULL values.
Columns to reference:
* country_name
* subregion1_name (for state information)
* cumulative_deceased

```
WITH us_subregion_death AS (
    SELECT subregion1_name, SUM(cumulative_deceased)
    FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
    WHERE date = '2020-05-15' AND country_name = 'United States of America' AND subregion1_name IS NOT NULL
    GROUP BY subregion1_name
    HAVING SUM(cumulative_deceased) > 200
)
SELECT COUNT(DISTINCT subregion1_name) AS count_of_states
FROM us_subregion_death
```

## QUERY 3: Identifying Hotspots
Build a query that will answer "List all the states in the United States of America that had more than 1500 confirmed cases on May 10, 2020?" The query needs to return the State Name and the corresponding confirmed cases arranged in descending order. Name of the fields to return state and total_confirmed_cases.
Columns to reference:
* country_code
* subregion1_name (for state information)
* cumulative_confirmed

```
SELECT subregion1_name as state, sum(cumulative_confirmed) as total_confirmed_cases 
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
where country_code= 'US' and date='2020-05-10' and subregion1_name is NOT NULL
group by subregion1_name
having total_confirmed_cases > 1500
order by total_confirmed_cases desc
```

## QUERY 4: Fatality Ratio
Build a query that will answer "What was the case-fatality ratio in Italy for the month of june 2020?" Case-fatality ratio here is defined as (total deaths / total confirmed cases) * 100. Write a query to return the ratio for the month of june 2020 and containing the following fields in the output: total_confirmed_cases, total_deaths, case_fatality_ratio.
Columns to reference:
* country_name
* cumulative_confirmed
* cumulative_deceased

```
WITH fatality_ratio AS (
SELECT 
SUM(cumulative_confirmed) as total_confirmed_cases, 
SUM(cumulative_deceased) as total_deaths, 
SUM(cumulative_deceased)/sum(cumulative_confirmed)*100 as case_fatality_ratio
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date BETWEEN "2020-06-01" AND "2020-06-30" AND country_name = 'Italy'
)
SELECT *
FROM fatality_ratio
```

### QUERY 5: Identifying Specific Day
Build a query that will answer: "On what day did the total number of deaths cross 14000 in Italy?" The query should return the date in the format yyyy-mm-dd.
Columns to reference:
* country_name
* cumulative_deceased

```
SELECT date
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name = 'Italy' AND cumulative_deceased > 14000
ORDER BY date
```

## QUERY 6: FINDING DAYS WITH ZERO NET NEW CASES
The following query is written to identify the number of days in India between 23, Feb 2020 and 15, March 2020 when there were zero increases in the number of confirmed cases. However it is not executing properly. You need to update the query to complete it and obtain the result:

```
WITH india_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="India"
    AND date between '2020-02-23' and '2020-03-15'
  GROUP BY
    date
  ORDER BY
    date ASC
 )
, india_previous_day_comparison AS
(SELECT
  date,
  cases,
  LAG(cases) OVER(ORDER BY date) AS previous_day,
  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases
FROM india_cases_by_date
)
SELECT count(*)
FROM india_previous_day_comparison 
WHERE net_new_cases = 0
```

## QUERY 7: DOUBLING RATE
Using the previous query as a template, write a query to find out the dates on which the confirmed cases increased by more than 5 % compared to the previous day (indicating doubling rate of ~ 7 days) in the US between the dates March 22, 2020 and April 20, 2020. The query needs to return the list of dates, the confirmed cases on that day, the confirmed cases the previous day, and the percentage increase in cases between the days. Use the following names for the returned fields: Date, Confirmed_Cases_On_Day, Confirmed_Cases_Previous_Day and Percentage_Increase_In_Cases.

```
WITH us_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="United States of America"
    AND date between '2020-03-22' and '2020-04-20'
  GROUP BY
    date
  ORDER BY
    date ASC 
 )
, us_previous_day_comparison AS 
(SELECT
  date,
  cases,
  LAG(cases) OVER(ORDER BY date) AS previous_day,
  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases,
  (cases - LAG(cases) OVER(ORDER BY date))*100/LAG(cases) OVER(ORDER BY date) AS percentage_increase
FROM us_cases_by_date
)
SELECT Date, 
cases as Confirmed_Cases_On_Day, 
previous_day as Confirmed_Cases_Previous_Day, 
percentage_increase as Percentage_Increase_In_Cases
from us_previous_day_comparison
where percentage_increase > 10
```

## QUERY 8: Recovery Rate
Build a query to list the recovery rates of countries arranged in descending order (limit to 5 ) upto the date May 10, 2020. Restrict the query to only those countries having more than 50K confirmed cases. The query needs to return the following fields: country, recovered_cases, confirmed_cases, recovery_rate.
Columns to reference:
* country_name
* cumulative_confirmed
* cumulative_recovered

```
WITH covid_cases AS
(
SELECT
    country_name AS country,
    sum(cumulative_confirmed) AS confirmed_cases,
    sum(cumulative_recovered) AS recovered_cases
FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
    
WHERE date = '2020-05-10'
GROUP BY country_name
), covid_recovery_rate AS
(
SELECT 
    country,
    confirmed_cases,
    recovered_cases,
    recovered_cases / confirmed_cases *100 AS recovery_rate
FROM covid_cases
)
SELECT *
FROM covid_recovery_rate 
WHERE confirmed_cases > 50000
ORDER BY recovery_rate DESC 
LIMIT 10
```

## QUERY 9 : CUMULATIVE DAILY GROWTH
The following query is trying to calculate the CDGR on May 10, 2020 (Cumulative Daily Growth Rate) for France since the day the first case was reported. The first case was reported on Jan 24, 2020. The CDGR is calculated as:
((last_day_cases/first_day_cases)^1/days_diff)-1)
Where :
* last_day_cases is the number of confirmed cases on May 10, 2020
* first_day_cases is the number of confirmed cases on Jan 24, 2020
* days_diff is the number of days between Jan 24 - May 10, 2020
The query isn’t executing properly. Can you fix the error to make the query execute successfully?

```
WITH
  france_cases AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS total_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="France"
    AND date IN ('2020-01-24',
      '2020-05-10')
  GROUP BY
    date
  ORDER BY
    date)
, summary as (
SELECT
  total_cases AS first_day_cases,
  LEAD(total_cases) OVER(ORDER BY date) AS last_day_cases,
  DATE_DIFF(LEAD(date) OVER(ORDER BY date),date, day) AS days_diff
FROM
  france_cases
LIMIT 1
)
select first_day_cases, last_day_cases, days_diff, POW((last_day_cases/first_day_cases),(1/days_diff))-1 as cdgr
from summary
```

## QUERY 10 : DATA STUDIO REPORT
Create a Google Data Studio report that plots the following for the United States:
* Number of Confirmed Cases
* Number of Deaths
* Date range : 2020-03-23 to 2020-04-28

```
SELECT 
date, 
sum(cumulative_confirmed) AS confirmed_cases,
sum(cumulative_deceased) AS confirmed_death
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date BETWEEN '2020-03-23' AND '2020-04-28'
AND country_name = 'United States of America'
GROUP BY date
```
