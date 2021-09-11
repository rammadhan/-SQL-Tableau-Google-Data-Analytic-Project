# Cyclistic Case Study
## INTRODUCTION
In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. Until now, Cyclistic’s marketing strategy relied on building general awareness and appealing to broad consumer segments. One approach that helped make these things possible was the flexibility of its pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as Casuals. Customers who purchase annual memberships are Members.

Although the pricing flexibility helps Cyclistic attract more customers, Director of Marketing Lily Moreno believes that maximizing the number of annual members will be key to future growth. She has set a clear goal: design marketing strategies aimed at converting Casuals into Members. In order to do that, however, the marketing analyst team needs to better understand how Members and Casuals differ. Moreno and her team are interested in analyzing the Cyclistic historical bike trip data to identify trends.

## ASK
Three questions will guide the future marketing program: 
- How do annual members and casual riders use Cyclistic bikes differently?
- Why would casual riders buy Cyclistic annual memberships? 
- How can Cyclistic use digital media to influence casual riders to become members?

**Business Problem**
“How to influence causal riders to become members using digital media”

## PREPARE
- Data is located on this link
- The size of this data is 473.98 MB
- Overall there is 3,651,474 rows and 12 column on this data
- There is 12 .csv file that separated by months and the most update data was on may 2021

**Data Preview**
![data_preview](https://user-images.githubusercontent.com/90141628/132163032-99b5383d-ea7b-4b22-b565-92de1623280b.PNG)

## PROCESS
**Tools**
- SQL (BigQuery) for cleaning and format data.
- R studio to analyze and make data visualization.
- Tableau to make interactive dashboard

**Data Cleaning Process**
- I combined 12 table for each month into 1  single table
- I added ride duration column as duration 
- I removed start_station_id and end_station_id because I allready used station name column
- I removed start_lat, start_lng, end_lat,and end_lng because data inconsistency on these column

```
CREATE TABLE analyzing-data-319917.bike_capstone_project.year_trip AS
SELECT *,
    date_diff(ended_at, started_at, minute) duration_second
FROM (
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2020_05 UNION ALL 
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2020_06 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2020_07 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2020_09 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2020_10 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2020_11 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2020_12 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2021_01 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2021_02 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2021_03 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2021_04 UNION ALL
    SELECT * EXCEPT (start_station_id, end_station_id, start_lat, end_lat, start_lng, end_lng) FROM analyzing-data-319917.bike_capstone_project.2021_05 )
```
After combined data was created we check for null on this data

```
SELECT
    SUM(CASE WHEN ride_id IS NULL THEN 1 ELSE 0 END) AS ride_id_null,
    SUM(CASE WHEN rideable_type IS NULL THEN 1 ELSE 0 END) AS rideable_type_null,
    SUM(CASE WHEN started_at IS NULL THEN 1 ELSE 0 END) AS started_at_null,
    SUM(CASE WHEN ended_at IS NULL THEN 1 ELSE 0 END) AS ended_at_null,
    SUM(CASE WHEN start_station_name IS NULL THEN 1 ELSE 0 END) AS start_station_null,
    SUM(CASE WHEN end_station_name IS NULL THEN 1 ELSE 0 END) AS end_station_null,
    SUM(CASE WHEN member_casual IS NULL THEN 1 ELSE 0 END) AS start_station_null
FROM  analyzing-data-319917.bike_capstone_project.year_trip
```
