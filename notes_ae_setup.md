# Analytics Engineering

Disclaimer: Notes forked from section 4 README [2025 Notes by Manuel Guerra](https://github.com/ManuelGuerra1987/data-engineering-zoomcamp-notes/blob/main/4_Analytics-Engineering/README.md)

### Table of contents

- [Analytics Engineering](#analytics-engineering)
    - [Table of contents](#table-of-contents)
- [DBT and  BigQuery](#dbt-and--bigquery)
  - [Set up Environment](#set-up-environment)
  - [Create a BigQuery service account](#create-a-bigquery-service-account)
    - [Download JSON key](#download-json-key)
  - [Copy taxi\_rides\_ny folder](#copy-taxi_rides_ny-folder)
  - [Create a dbt project (Alternative A - dbt Cloud)](#create-a-dbt-project-alternative-a---dbt-cloud)
    - [Add a Connection in DBT Cloud](#add-a-connection-in-dbt-cloud)
    - [Set up a GitHub Repository](#set-up-a-github-repository)
    - [Start developing](#start-developing)
      - [Create new branch](#create-new-branch)
  - [Development of dbt Models](#development-of-dbt-models)
    - [Modular data modelling](#modular-data-modelling)
    - [Developing the first staging model](#developing-the-first-staging-model)
    - [Variables](#variables)
    - [Developing the second staging model](#developing-the-second-staging-model)
  - [Core Models](#core-models)
    - [Dim zones](#dim-zones)
    - [Fact trips](#fact-trips)
    - [Monthly zone revenue](#monthly-zone-revenue)
  - [Building the model](#building-the-model)
  - [Visualising the transformed data](#visualising-the-transformed-data)
    - [1: Process the full dataset](#1-process-the-full-dataset)
    - [2: Open Looker Studio.](#2-open-looker-studio)
    - [3: Create a BigQuery data source](#3-create-a-bigquery-data-source)
    - [4: Set Default aggregations](#4-set-default-aggregations)
    - [5: Add a date range control](#5-add-a-date-range-control)
    - [6: Add a time series chart](#6-add-a-time-series-chart)
    - [7: Add a scorecard](#7-add-a-scorecard)
    - [8: Add a pie chart](#8-add-a-pie-chart)
    - [9: Add a table with heatmap](#9-add-a-table-with-heatmap)
    - [10: Add a stacked column chart](#10-add-a-stacked-column-chart)
- [DBT and  BigQuery](#dbt-and--bigquery-1)
  - [Set up Environment](#set-up-environment-1)

# DBT and  BigQuery

**How are we going to use dbt?**

There are two ways to use dbt, and throughout the project, you'll see videos illustrating these approaches: version A and version B.

- **Version A** primarily uses BigQuery as the data warehouse. This method involves using the **dbt Cloud** Developer plan, which is free. You can create an account at no cost, and since this is cloud-based, there’s no need to install dbt Core locally.
  - **Version A2???** Could we do a combination of both in which we use dbt Core but we point to BigQuery?

- **Version B** uses PostgreSQL. In this approach, you'll perform development using your own IDE, such as VS Code, and install dbt Core locally connecting to the postgresql database. You will be running dbt models through the CLI

During the project you might already have data loaded into GCP buckets. This raw data will be loaded into tables in BigQuery. dbt will be used to transform the data, and finally, dashboards will be created to present the results.

## Set up Environment

- Created dedicated DBT service account in GCP with BigQuery Admin 
- Created developed account in DBT Cloutd
- Created project in DBT Cloud (following file `dbt_cloud_setup.md`):
  - By default it appeard "IFCO" as an account not sure why (cannot change the name)
  - Within that account I use the default project "Analytics"
  - Added in Account > Connections the connection to BigQuery by adding a specific created json credentials service account in the GCP project. Set location default to `europe-west2` as it is the location of the BigQuery dataset.
  - I created and added new brand repository in GitHub [zoomcamp2025-dbt-project]([zoomcamp2025-dbt-project](https://github.com/AlvaroPica/zoomcamp2025-dbt-project).
  - Added a DBT deploy key in GitHub Repo deploy keys
  - Created Credentials > Development with a specific dataset target in Bigquery: `zoomcamp_dbt`
  

## Create a BigQuery service account

In order to connect we need the service account JSON file generated from bigquery. Open the [BigQuery credential wizard](https://console.cloud.google.com/apis/credentials/wizard) to create a service account

Select BigQuery API and Application data

![ae7](images/ae7.jpg)

Next --> Continue --> Complete Service account name and description

![ae8](images/ae8.jpg)

Click on Create and continue

Select role --> BigQuery Admin

You can either grant the specific roles the account will need or simply use BigQuery admin, as you'll be the sole user of both accounts and data.

![ae9](images/ae9.jpg)

Click on continue --> Click on Done

### Download JSON key

Now that the service account has been created we need to add and download a JSON key, go to the keys section, select "create new key". Select key type JSON and once you click on create it will get inmediately downloaded for you to use.

In the navigation menu (the three horizontal lines in the top-left corner), go to IAM & Admin > Service Accounts.

Find the dbt service account:

![ae10](images/ae10.jpg)

Navigate to the Keys tab. Click on Add Key > Create New Key

![ae11](images/ae11.jpg)

select JSON as the key type --> Create. A Json file will be downloaded, keep it in a safe place.

## Copy taxi_rides_ny folder

Copy taxi_rides_ny folder from https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/04-analytics-engineering in your 04-analytics-engineering folder

This taxi_rides_ny folder contains macros, models, seeds. Cloning these elements gives you a strong foundation for your DBT project, enabling you to focus on building and improving your data pipeline rather than starting from scratch. Also saves time by reducing the need to recreate common logic or datasets.

```bash
git clone https://github.com/DataTalksClub/data-engineering-zoomcamp.git
```

Then copy the `taxi_rides_ny` folder in your own `04-analytics-engineering` folder

## Create a dbt project (Alternative A - dbt Cloud)

Create a dbt cloud account from [their website](https://www.getdbt.com/pricing/) (free for solo developers)
Once you have logged in into dbt cloud you will be prompt to create a new project.

You are going to need:

- Access to your data warehouse (bigquery)
- Admin access to your repo, where you will have the dbt project.

![ae12](images/ae12.jpg)

### Add a Connection in DBT Cloud

At account level:

- Connections --> Add new connection --> Select BigQuery
- Click on Upload a Service Account JSON file --> upload the json downloaded from the BigQuery Account Service
- Click on Save

At project level:

- Back on your project setup, select BigQuery
- Click on save
- Test the connection
- In Account > Credentials > Project > Development credentials make sure you have your dataset name set (e.g. `zoomcamp_dbt`)

![ae14](images/ae36.jpg)

### Set up a GitHub Repository

The repository that dbt Cloud will manage can be a dedicated one or the general you use for this project. Now its time to setup a repository for the Project. Within the Project sheet go to the repository part:

![ae15](images/ae15.jpg)

Select git clone and paste the SSH link from your repo. Import.

You will get a deploy key that you need to add to your deploy keys in your GitHub repository (Settings > Deploy keys > Add) (Select Write Access)

Back on dbt cloud, click on next, you should look this:

![ae17](images/ae17.jpg)

In this case we want the project to be created under a folder in a repository that already exists. Make sure you informed taxi_rides_ny as the project subdirectory in the Project details:

Project subdirectory: `04-analytics-engineering/taxi_rides_ny`

![ae17](images/ae18.jpg)

### Start developing

Once BigQuery and dbt Cloud project are configured you can start developing.

#### Create new branch

On the left sidebar, click on Develop --> Cloud IDE

Create a new branch: I will use "ze-zoocamp-alvaro" branch. After creating the new branch, you can go to your repo on github and see the new branch created.

Note: it is important to create a new branch, because if we had chosen to work on the master branch we would get stuck in read-only mode.

## Development of dbt Models

_[Video source](https://www.youtube.com/watch?v=ueVy2N54lyc)_

Let's start now with the development of those DBT models.

### Modular data modelling

To get started, we're going to use a modular data modeling approach. As we discussed in earlier lessons, we'll create fact tables and dimensional tables. The structure of our DBT project will look something like this:

- First, we have the tables we loaded (trips data). These are our sources.

- Then, we'll start building SQL scripts called "models" in DBT to perform transformations.

For example, we'll pick up the source data, clean it, deduplicate it, recast and rename columns, and typecast data. Afterward, we'll apply business logic to create fact and dimension tables. Finally, we'll create data marts to aggregate the data for our stakeholders.

We initially have tables that already exist outside our DBT project. These contain the data we will use, and we define them as sources. Additionally, we will use a file (e.g., taxi_zone_lookup) to create a table that will be incorporated into our transformations.

![ae22](images/ae22.jpg)

### Developing the first staging model

**schema.yml**

Under the models directory, there is a folder named staging. This will represent the initial layer of models responsible for cleaning the source data. Inside the staging folder, there is a schema.yml file for defining the sources:

```yaml

version: 2

sources:
  - name: staging
    database: zoomcamp-airflow-444903 
    schema: zoomcamp
      
    tables:
      - name: green_tripdata
      - name: yellow_tripdata

models:
    - name: stg_green_tripdata
    ...  
    - name: stg_yellow_tripdata
    ...
```      

> [!NOTE]  
> Make sure the values ​​in the YAML match the values ​​in your BigQuery!

Full code: [`schema.yml`](taxi_rides_ny/models/staging/schema.yml)

**model: stg_green_tripdata.sql**

Inside the staging folder, there is a stg_green_tripdata.sql file. This dbt model defines a SQL query that transforms and materializes data from a source table (green_tripdata) into a view in the database.

stg_green_tripdata.sql looks like this:

```sql

{{
    config(
        materialized='view'
    )
}}

with tripdata as 
(
  select *,
    row_number() over(partition by vendorid, lpep_pickup_datetime) as rn
  from {{ source('staging','green_tripdata') }}
  where vendorid is not null 
)
select
    -- identifiers
    {{ dbt_utils.generate_surrogate_key(['vendorid', 'lpep_pickup_datetime']) }} as tripid,
    {{ dbt.safe_cast("vendorid", api.Column.translate_type("integer")) }} as vendorid,
    {{ dbt.safe_cast("ratecodeid", api.Column.translate_type("integer")) }} as ratecodeid,
    {{ dbt.safe_cast("pulocationid", api.Column.translate_type("integer")) }} as pickup_locationid,
    {{ dbt.safe_cast("dolocationid", api.Column.translate_type("integer")) }} as dropoff_locationid,
    
    -- timestamps
    cast(lpep_pickup_datetime as timestamp) as pickup_datetime,
    cast(lpep_dropoff_datetime as timestamp) as dropoff_datetime,
    
    -- trip info
    store_and_fwd_flag,
    {{ dbt.safe_cast("passenger_count", api.Column.translate_type("integer")) }} as passenger_count,
    cast(trip_distance as numeric) as trip_distance,
    {{ dbt.safe_cast("trip_type", api.Column.translate_type("integer")) }} as trip_type,

    -- payment info
    cast(fare_amount as numeric) as fare_amount,
    cast(extra as numeric) as extra,
    cast(mta_tax as numeric) as mta_tax,
    cast(tip_amount as numeric) as tip_amount,
    cast(tolls_amount as numeric) as tolls_amount,
    cast(ehail_fee as numeric) as ehail_fee,
    cast(improvement_surcharge as numeric) as improvement_surcharge,
    cast(total_amount as numeric) as total_amount,
    coalesce({{ dbt.safe_cast("payment_type", api.Column.translate_type("integer")) }},0) as payment_type,
    {{ get_payment_type_description("payment_type") }} as payment_type_description
from tripdata
where rn = 1


-- dbt build --select <model_name> --vars '{'is_test_run': 'false'}'
{% if var('is_test_run', default=true) %}

  limit 100

{% endif %}
```

**Important**

```sql
{{ dbt_utils.generate_surrogate_key(['vendorid', 'lpep_pickup_datetime']) }} as tripid,
{{ dbt.safe_cast("vendorid", api.Column.translate_type("integer")) }} as vendorid,

```

- Generates a unique surrogate key (tripid) by combining vendorid and lpep_pickup_datetime using dbt's generate_surrogate_key utility.

- Safely casts vendorid to an integer using dbt's safe_cast() function. This type of cast is agnostic to the database used, meaning it should work in PosgeSQL or in BiGQuery.


### Variables

Now, let’s learn about variables. The concept of variables in DBT is similar to variables in any programming language. A variable acts like a container where you store a value that you want to use later, and you can access it whenever needed.

In DBT, variables can be defined at the project level within the dbt_project.yml file, allowing you to use them across various models or macros. For example, you might define a variable payment_type_values as a list of numbers:

```yaml

vars:
  payment_type_values: [1, 2, 3, 4, 5, 6]
```  

This list could be used in different scenarios, such as building a CASE statement by looping through the list. Running a test to check if the actual values in the table are part of the list or dynamically setting a variable's value within a macro using the var marker.

Additionally, you can pass a value for a variable during execution. This allows you to customize behavior dynamically at runtime. To access a variable, use the var() marker.

Here’s an example in stg_green_tripdata:

```sql

{% if var('is_test_run', default=true) %}

  limit 100

{% endif %}
```

A conditional execution checks if a variable is_test_run is True. If it’s True, it adds LIMIT 100 to the query. If it’s False, the query proceeds without the limit. The code also defines a default value for is_test_run, which is True. This means that unless specified otherwise, LIMIT 100 will always be added by default.

To test this, you can run the code as it is and confirm that the LIMIT 100 is applied. If you don’t want the limit, you can override the variable during execution by passing a dictionary of variables. For example:

```
dbt run --vars '{"is_test_run": false}'
```

When this command is executed, the code no longer adds the LIMIT 100. This is a useful technique for development, as it allows you to test with smaller datasets (faster and cheaper queries) while ensuring full production data is used during deployment by setting is_test_run to False.

This method, often referred to as a "dev limit," is highly recommended for optimizing development workflows. By default, you’ll have faster and cheaper queries during development, but the limit can easily be removed when working with the full production data.


### Developing the second staging model

The next task is to create the staging file for yellow_trip_data. This is very similar to the previous file, so we won't go into detail about its structure. The code is almost identical, using the same macro.

There are a few differences: the staging file typically includes a straightforward CTE and doesn't use the same variables as before. However, the core SQL logic remains largely the same.

Check `stg_yellow_tripdata.sql`

## Core Models

So far, our project looks like this: we have our two sources and a set of models. Now, we need to create our fact and dimensional tables

![ae40](images/ae40.jpg)

### Dim zones

The goal for dim_zones is to act as a master data table containing all the zone information where the taxis operate. These taxis move within specific zones, and we want to ensure we have accurate information about them.

Since we don’t have source data for this, we’ll use the seeds mentioned earlier. For this, we'll leverage the taxi_zone_lookup file. It’s unlikely that this data will change frequently.

We’ll copy this data, save it as a CSV file, and include it in our project under the seeds folder. The file is named taxi_zone_lookup.csv, and it can be downloaded directly from GitHub if needed. Once saved, the seed file will have a distinct icon in the project, and we can preview the data.

The seed contains fields like location_id, which is also present in both the green and yellow trip data. This will allow us to connect the data with the taxi_zone_lookup table for additional context. The dim_zones model is under the core folder.

dim_zones.dql looks like this:

```sql

{{ config(materialized='table') }}

select 
    locationid, 
    borough, 
    zone, 
    replace(service_zone,'Boro','Green') as service_zone 
from {{ ref('taxi_zone_lookup') }}
```

The dim_zones model will use data from taxi_zone_lookup. It will define fields like location, borough, and service_zone. Additionally, we’ll address an issue where all entries labeled as "Borough" were actually "Green Zones," which only green taxis operate in. We'll clean up the data by renaming those values for easier analytics.

So far, our project looks like this:

![ae31](images/ae42.jpg)

### Fact trips

With dim_zones, we are ready to create the next step: a fact table for trips (fact_trips). We will combine the green and yellow trip data, encase it with dimensional data, and materialize it as a table. Materializing it as a table ensures better performance for analytics since this table will be large due to unions and joins.

fact_trips.sql model goal is to:

- Combine both green and yellow trip data.
- Add a field to identify whether a record is from the green or yellow dataset for easier analysis.
- Join this data with the dim_zones model to enrich it with pickup and drop-off zone details.

Check `fact_trips.sql` looks like:

When we run the model with the full production dataset, the resulting table will contain millions of rows, representing a comprehensive and enriched fact table. This table is now ready for use in analysis or as a source for BI tools.

With all of this, the fact_trips table is complete, and we can proceed to testing and further analysis.

So far, our project looks like this:

![ae31](images/ae43.jpg)

We can check the lineage to see how the modular data modeling looks. Now, we can observe that fact_trips depends on all the required models. One of the great features of dbt is that it identifies all these connections. This means we can run fact_trips, but first, dbt will execute all its parent models. dbt will test the sources for freshness or other requirements, run any missing or outdated models, and only then build fact_trips.

The final step is to test these models to ensure that all rows and calculations—totaling 62.7 million—are correct before delivering the results.

### Monthly zone revenue

This is a dbt model that creates a table summarizing revenue-related metrics for trips data as a table selecting data from our previous dbt model called fact_trips.

This model creates a table with monthly revenue metrics per pickup zone and service type, including various fare components, trip counts, and averages.

It enables analysis of revenue trends, passenger patterns, and trip details across zones and services, giving a clear breakdown of monthly performance.

dm_monthly_zone_revenue.sql looks like:

```sql

{{ config(materialized='table') }}

with trips_data as (
    select * from {{ ref('fact_trips') }}
)
    select 
    -- Reveneue grouping 
    pickup_zone as revenue_zone,
    {{ dbt.date_trunc("month", "pickup_datetime") }} as revenue_month, 

    service_type, 

    -- Revenue calculation 
    sum(fare_amount) as revenue_monthly_fare,
    sum(extra) as revenue_monthly_extra,
    sum(mta_tax) as revenue_monthly_mta_tax,
    sum(tip_amount) as revenue_monthly_tip_amount,
    sum(tolls_amount) as revenue_monthly_tolls_amount,
    sum(ehail_fee) as revenue_monthly_ehail_fee,
    sum(improvement_surcharge) as revenue_monthly_improvement_surcharge,
    sum(total_amount) as revenue_monthly_total_amount,

    -- Additional calculations
    count(tripid) as total_monthly_trips,
    avg(passenger_count) as avg_monthly_passenger_count,
    avg(trip_distance) as avg_monthly_trip_distance

    from trips_data
    group by 1,2,3
```    

The main query groups the data by:

- Pickup zone (pickup_zone) → Labeled as revenue_zone.
- Month of the pickup date (pickup_datetime) → Labeled as revenue_month 
- Service type (service_type) → Such as economy, premium, etc.

For each group, the query calculates revenue-related metrics like revenue_monthly_fare (Sum of fare_amount), revenue_monthly_extra (Sum of additional fees), etc and other metrics like total_monthly_trips (Count of trips), avg_monthly_passenger_count (Average number of passengers per trip) and avg_monthly_trip_distance (Average distance per trip).

Finally, The GROUP BY 1, 2, 3 clause organizes the results by the specified dimensions (pickup zone, revenue month, and service type). Each calculation is applied within these groups. 1 refers to pickup_zone, 2 refers to the truncated month of pickup_datetime, 3 refers to service_type.

So far, our project looks like this:

 <br>

![ae31](images/ae47.jpg)
<br><br>


## Building the model

**1: schema.yml values**

Open VS Code, go to taxi_rides_ny --> models --> staging --> schema.yml

Make sure that project id, dataset name and tables matches your project id, dataset and tables name in BigQuery!

```yaml

sources:
  - name: staging
    database: zoomcamp-airflow-444903 # project id
    schema: zoomcamp # dataset name
     
    tables:
      - name: green_tripdata #table name
      - name: yellow_tripdata #table name
```

Values check:

 <br>

![ae22](images/ae29.jpg)
<br><br>

**2: dbt build**

In the dbt cloud console, run:

```
dbt build
```

You should look something like this:
 <br>

![ae31](images/ae20.jpg)
<br><br>

 <br>

After running the model (dbt build), the process should complete successfully, creating a view. You can check the view details to confirm the output, including the trip_id and other data fields. You can also examine the compiled SQL in the target/compiled folder for additional troubleshooting.

When you run dbt build in dbt Cloud, it does the following:

- Builds Models: Executes the SQL transformations defined in your project to create or update tables and views in your target data warehouse.

- Runs Tests: Validates data quality by executing both custom tests (defined in .yml files) and standard tests (like unique or not null constraints).

- Updates Snapshots: Captures historical changes in your source data for versioning and time-based analytics.

- Loads Seeds: Loads any seed files (like .csv files) defined in your project into the target data warehouse.


By default, only 100 rows are processed in the query for testing purposes. This is controlled by the is_test_run variable, which defaults to true in stg_green_tripdata.sql and stg_yellow_tripdata models:

```python

{% if var('is_test_run', default=true) %}

  limit 100

{% endif %}
```


To run the query without this limit and process the full dataset in production, you need to explicitly set the variable to false by using the following command:

```
dbt build --select +fact_trips.sql+ --vars '{is_test_run: false}'
```

This ensures that the model processes the entire dataset

**3: Check BigQuery**

Head over to BigQuery and check the views that dbt generated:

 <br>

![ae31](images/ae32.jpg)
<br><br>

 <br>

![ae31](images/ae33.jpg)
<br><br>

 <br>

![ae31](images/ae34.jpg)
<br><br>

Dim_zones:

 <br>

![ae31](images/ae41.jpg)
<br><br>

Fact_trips:

 <br>

![ae31](images/ae44.jpg)
<br><br>

dm_monthly_zone_revenue:

 <br>

![ae31](images/ae48.jpg)
<br><br>

## Visualising the transformed data 

_[Video source](https://www.youtube.com/watch?v=39nLTs74A3E)_

Now that we created our models and transformed our data, we are now going to visualize this data.

### 1: Process the full dataset

Before you start with the visualization, make sure that you have processed all the data without the limit of 100
 in the stg_green_tripdata.sql and stg_yellow_tripdata.sql models.

To run the query without this limit and process the full dataset in production, you need to explicitly set the 
variable to false by using the following command: 

```
dbt build --select +fact_trips.sql+ --vars '{is_test_run: false}'
```

### 2: Open [Looker Studio](https://lookerstudio.google.com/).

Looker Studio (formerly known as Google Data Studio) is a free tool by Google that allows you to create interactive dashboards and visualizations of data. It enables users to connect various data sources—such as Google Sheets, Google Analytics, BigQuery, and more—and transform raw data into visually engaging reports that are easy to share and understand.

### 3: Create a BigQuery data source 

On the left sidebar, click on Create --> Data source

 <br>

![ae49](images/ae49.jpg)
<br><br>

Select BigQuery (it may be necessary to authorize Google Data Studio to access BigQuery)

Then select fact_trips from your project

 <br>

![ae50](images/ae50.jpg)
<br><br>

Click on Connect

### 4: Set Default aggregations

 in the next screen, we can see that the tool already suggests some aggregations for us. In this example, we set
  all of them to None, except for passenger_count, for which we keep the "Sum" aggregation.

   <br>

![ae51](images/ae51.jpg)
<br><br>

Then click on CREATE REPORT --> Add report

### 5: Add a date range control 

First, eliminate the default chart. Then click on Add a control --> Date range control

- Select start date Jan 1, 2019
- Select end date Dec 31, 2020

   <br>

![ae52](images/ae52.jpg)
<br><br>

### 6: Add a time series chart

Select Add a chart --> Time series chart

   <br>

![ae53](images/ae53.jpg)
<br><br>

Let's add a breakdown dimension

On the rigth menu, click on add dimension --> drag and drop service_type

   <br>

![ae53](images/ae54.jpg)
<br><br>

The chart now should look like this:

   <br>

![ae55](images/ae55.jpg)
<br><br>

If you look at the graph, the drop that you can see in march because of covid


### 7: Add a scorecard

Select Add a chart --> Scorecard with compact numbers

   <br>

![ae56](images/ae56.jpg)
<br><br>

### 8: Add a pie chart

Select Add a chart --> pie chart

<br>

![ae57](images/ae57.jpg)
<br><br>


### 9: Add a table with heatmap

Select Add a chart --> table with heatmap

Select pickup_zone as dimension

<br>

![ae58](images/ae58.jpg)
<br><br>

The table should look like this:

<br>

![ae59](images/ae59.jpg)
<br><br>

### 10: Add a stacked column chart

Select Add a chart --> stacked column chart

We will also add a Stacked Column Bar showing trips per month. Since we do not have that particular dimension, 
what we can do instead is to create a new field that will allow us to filter by month:

1. In the Available Fields sidebar, click on Add a field at the bottom --> Add calculated field
2. Name the new field pickup_month.
3. In the Formula field, type MONTH(pickup_datetime).
4. Click on Save and then on Done.
5. Back in the main page, drag the new pickup_month field  to the Dimension field. 
6. Get rid of all breakdown dimensions.

Our bar chart will now display trips per month but we still want to discriminate by year:

7. Add a new field and name it pickup_year.
8. Type in the formula YEAR(pickup_datetime).
9. Click on Save and Done.
10. Add the pickup_year field as a breakdown dimension for the bar chart.
11. Change the Sort dimension to pickup_month and make it ascending.

<br>

![ae60](images/ae60.jpg)
<br><br>

The table should look like this:

<br>

![ae61](images/ae61.jpg)
<br><br>

# DBT and  BigQuery

## Set up Environment

- Created dedicated DBT service account in GCP with BigQuery Admin 
- Created developed account in DBT Cloutd
- Created project in DBT Cloud (following file `dbt_cloud_setup.md`):
  - By default it appeard "IFCO" as an account not sure why (cannot change the name)
  - Within that account I use the default project "Analytics"
  - Added in Account > Connections the connection to BigQuery by adding a specific created json credentials service account in the GCP project. Set location default to `europe-west2` as it is the location of the BigQuery dataset.
  - I created and added new brand repository in GitHub [zoomcamp2025-dbt-project]([zoomcamp2025-dbt-project](https://github.com/AlvaroPica/zoomcamp2025-dbt-project).
  - Added a DBT deploy key in GitHub Repo deploy keys
  - Created Credentials > Development with a specific dataset target in Bigquery: `zoomcamp_dbt`
  

