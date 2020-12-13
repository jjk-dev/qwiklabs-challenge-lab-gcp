# Build and Optimize Data Warehouses with BigQuery: [Challenge Lab](https://www.qwiklabs.com/focuses/14341?parent=catalog)

## Challenge scenario
You are part of an international public health organization which is tasked with developing a machine learning model to predict the daily case count for countries during the Covid-19 pandemic. As a junior member of the Data Science team you've been assigned to use your data warehousing skills to develop a table containing the features for the machine learning model. You receive the following request to complete the following tasks:

- Create a new dataset and table
- Add columns for population, country_area and a record column (named mobility) 
- Modify SQL template before populate the population data 
- Extract average values for the the six component fields that comprise the mobility 
- Run a query that returns a combined list of the DISTINCT countries that do not have any population data and countries which do not have country area information, ordered by country name

Some standards and tips you should follow:
- Exclude United Kingdom **(GBR)** and the United States **(USA)** data from your initial table.
- When updating the schema for a BigQuery table you can use the console to add the columns and record elements or you can use the command line **bq** utility to update the schema by providing a JSON file.
- The **covid19_ecdc** table contains a **population** column that you can use to populate the populationcolumn in your dataset.
- The **country_names_area** table does not contain a three letter country code column, but you can join it to your table using the full text **country_name** column that exists in both tables.
- When updating the mobility record remember that you must select (and average) a number of records for each country and date combination so that you get a single average of each child column in the mobility record.

### Task 1: Create a table partitioned by date
Create a new dataset and create a table in that dataset partitioned by date, with an expiry of 90 days. The table should initially use the schema defined for the **oxford_policy_tracker** table in the [COVID 19 Government Response public dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_govt_response&page=dataset).

You must also populate the table with the data from the source table for all countries except the United Kingdom **(GBR)** and the United States **(USA)**.
```

```

### Task 2: Add new columns to your table
Update your table to add new columns to your table with the appropriate data types to ensure alignment with the specification provided to you.
```

```

### Task 3: Add country population data to the population column
Add the country population data to the population column in your table with **covid_19_geographic_distribution_worldwide** table data from the [European Center for Disease Control COVID 19 public dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_ecdc&page=dataset) table.

```

```

### Task 4: Add country area data to the country_area column
Add the country area data to the country_area column in your table with **country_names_area** table data from the [Census Bureau International public dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=census_bureau_international&page=dataset).


### Task 5: Populate the mobility record data
Populate the mobility record in your table with data from the [Google COVID 19 Mobility public dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_govt_response&page=dataset) .

### Task 6: Query missing data in population & country_area columns
Run a query to find the missing countries in the population and country_area data. The query should list countries that do not have any population data and countries that do not have country area information, ordered by country name. If a country has neither population or country area it must appear twice.


## Congratulations!
![Badge Image]() ![Badge_cl Image]()
