# Insights from Data with BigQuery: [Challenge Lab](https://www.qwiklabs.com/focuses/11988?parent=catalog)

## Challenge scenario
You're part of a public health organization which is tasked with identifying answers to queries related to the Covid-19 pandemic. Obtaining the right answers will help the organization in planning and focusing healthcare efforts and awareness programs appropriately.

The dataset and table that will be used for this analysis will be : **bigquery-public-data.covid19_open_data.covid19_open_data**. This repository contains country-level datasets of daily time-series data related to COVID-19 globally. It includes data relating to demographics, economy, epidemiology, geography, health, hospitalizations, mobility, government response, and weather.

- Total Confirmed Cases
- Worst Affected Areas
- Identifying Hotspots
- Fatality Ratio
- Identifying specific day
- Finding days with zero net new cases
- Doubling rate
- Recovery rate
- CDGR - Cumulative Daily Growth Rate
- Create a Datastudio report

### Task 1: Total Confirmed Cases
Build a query that will answer "What was the total count of confirmed cases on Apr 15, 2020?" The query needs to return a single row containing the sum of confirmed cases across all countries. The name of the column should be **total_cases_worldwide**.

Navigate to the BigQuery console, select Navigation menu > BigQuery. And put SQL Query on the blank field.

SQL Query:
```
SELECT SUM(cumulative_confirmed) AS total_cases_worldwide
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date = '2020-04-15'
```

### Task 2: Worst Affected Areas
Build a query for answering "How many states in the US had more than 100 deaths on Apr 10, 2020?" The query needs to list the output in the field **count_of_states**. Hint: Don't include NULL values.

SQL Query:
```
SELECT COUNT(*) AS count_of_states
FROM (
  SELECT subregion1_name AS state, SUM(cumulative_deceased) AS deaths
  FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE country_name = 'United States of America' AND date = '2020-04-10' AND subregion1_name IS NOT NULL
  GROUP BY subregion1_name
)
WHERE deaths > 100
```

### Task 3: Identifying Hotspots
Build a query that will answer "List all the states in the United States of America that had more than 1000 confirmed cases on Apr 10, 2020?" The query needs to return the State Name and the corresponding confirmed cases arranged in descending order. Name of the fields to return **state** and **total_confirmed_cases**.

SQL Query:
```
SELECT subregion1_name AS state, SUM(cumulative_confirmed) AS total_confirmed_cases
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name = 'United States of America' AND date = '2020-04-10' AND subregion1_name is NOT NULL
GROUP BY subregion1_name
HAVING total_confirmed_cases > 1000
ORDER BY total_confirmed_cases DESC
```

### Task 4: Fatality Ratio
Build a query that will answer "What was the case-fatality ratio in Italy for the month of April 2020?" Case-fatality ratio here is defined as (total deaths / total confirmed cases) * 100. Write a query to return the ratio for the month of April 2020 and containing the following fields in the output: **total_confirmed_cases, total_deaths, case_fatality_ratio**.

SQL Query:
```
SELECT SUM(cumulative_confirmed) AS total_confirmed_cases, SUM(cumulative_deceased) AS total_deaths, (SUM(cumulative_deceased) / SUM(cumulative_confirmed)) * 100 AS case_fatality_ratio
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name = 'Italy' AND date BETWEEN '2020-04-01' AND '2020-04-30'
```

### Task 5: Identifying specific day
Build a query that will answer: "On what day did the total number of deaths cross 10000 in Italy?" The query should return the date in the format **yyyy-mm-dd**.

SQL Query:
```
SELECT date
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name = 'Italy' AND cumulative_deceased > 10000
ORDER BY date
```

### Task 6: Finding days with zero net new cases
The following query is written to identify the number of days in India between 21 Feb 2020 and 15 March 2020 when there were zero increases in the number of confirmed cases. However it is not executing properly. You need to update the query to complete it and obtain the result.

SQL Query:
```
WITH india_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="India"
    AND date between '2020-02-21' and '2020-03-15'
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
SELECT COUNT(date)
FROM india_previous_day_comparison
WHERE net_new_cases = 0
```

### Task 7: Doubling rate
Using the previous query as a template, write a query to find out the dates on which the confirmed cases increased by more than 10% compared to the previous day (indicating doubling rate of ~ 7 days) in the US between the dates March 22, 2020 and April 20, 2020. The query needs to return the list of dates, the confirmed cases on that day, the confirmed cases the previous day, and the percentage increase in cases between the days. Use the following names for the returned fields: **Date, Confirmed_Cases_On_Day, Confirmed_Cases_Previous_Day and Percentage_Increase_In_Cases**.

SQL Query:
```
WITH us_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'United States of America'
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
  (cases - LAG(cases) OVER(ORDER BY date)) * 100 / LAG(cases) OVER(ORDER BY date) AS percentage_increase
FROM us_cases_by_date
)
SELECT date AS Date, cases AS Confirmed_Cases_On_Day, previous_day AS Confirmed_Cases_Previous_Day, percentage_increase AS Percentage_Increase_In_Cases
FROM us_previous_day_comparison
WHERE percentage_increase > 10
```

### Task 8: Recovery rate
Build a query to list the recovery rates of countries arranged in descending order (limit to 10) on the date May 10, 2020. Restrict the query to only those countries having more than 50K confirmed cases. The query needs to return the following fields: **country, recovered_cases, confirmed_cases, recovery_rate**.

SQL Query:
```
WITH cases_by_country AS (
  SELECT country_name AS country, SUM(cumulative_recovered) AS recovered_cases, SUM(cumulative_confirmed) AS cases
  FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE date = '2020-05-10'
  GROUP BY country_name
)
, country_recovery_rate AS (
  SELECT country, recovered_cases, cases, (recovered_cases * 100) / cases AS recovery_rate
  FROM cases_by_country
)
SELECT country, recovered_cases, cases AS confirmed_cases, recovery_rate
FROM country_recovery_rate 
WHERE cases > 50000
ORDER BY recovery_rate DESC
LIMIT 10
```

### Task 9: CDGR - Cumulative Daily Growth Rate
The following query is trying to calculate the CDGR on May 10, 2020(Cumulative Daily Growth Rate) for France since the day the first case was reported. The first case was reported on Jan 24, 2020. 

The CDGR is calculated as:
```
((last_day_cases/first_day_cases)^1/days_diff)-1)
```

Where :
- **last_day_cases** is the number of confirmed cases on May 10, 2020
- **first_day_cases** is the number of confirmed cases on Feb 02, 2020
- **days_diff** is the number of days between Feb 02 - May 10, 2020

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
  LEAD(total_cases) AS last_day_cases,
  DATE_DIFF(LEAD(date) OVER(ORDER BY date),date, day) AS days_diff
FROM
  france_cases
LIMIT 1
)

select first_day_cases, last_day_cases, days_diff, SQRT((last_day_cases/first_day_cases),(1/days_diff))-1 as cdgr
from summary
```

**Note:** Refer to the following [page](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators) to learn more about the SQL function referenced **LEAD()**.

SQL Query:
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

select first_day_cases, last_day_cases, days_diff, POWER((last_day_cases/first_day_cases),(1/days_diff))-1 as cdgr
from summary
```

### Task 10: Create a Datastudio report
Create a Datastudio report that plots the following for the United States:
- Number of Confirmed Cases
- Number of Deaths
- Date range : 2020-03-15 to 2020-04-30

SQL Query:
```
SELECT date, SUM(cumulative_confirmed) AS country_cases, SUM(cumulative_deceased) AS country_deaths
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date BETWEEN '2020-03-15' AND '2020-04-30' AND country_name = 'United States of America'
GROUP BY date
```

After run the above query, click on **EXPLORE DATA > Explore with Data Studio**.
![Screenshot](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Insights-from-Data-with-BigQuery-1.png)

**Get started** Data Studio and **AUTHORIZE** access to BigQuery.
![Screenshot](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Insights-from-Data-with-BigQuery-2.png)

In the Data Studio report, select **Add a chart > Time series**.
![Screenshot](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Insights-from-Data-with-BigQuery-3.png)

Add **country_cases** and **country_deaths** to the Metric field.
![Screenshot](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Insights-from-Data-with-BigQuery-4.png)

Then **Save** changes on the report.
![Screenshot](https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Insights-from-Data-with-BigQuery-6.png)


## Congratulations!
<img src="https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/BigQuery-Basics-for-Data-Analysts.png" height="150" /> <img src="https://github.com/kkkkk317/qwiklabs-gcp/blob/main/img/Insights-from-Data-with-BigQuery.png" height="150" />

### Finish your Quest
This self-paced lab is part of the Qwiklabs [Insights from Data with BigQuery](https://google.qwiklabs.com/quests/123) Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge public and link to them in your online resume or social media account. [Enroll in this Quest](https://google.qwiklabs.com/quests/123/enroll) and get immediate completion credit if you've taken this lab. [See other available Qwiklabs Quests.](https://google.qwiklabs.com/catalog)
