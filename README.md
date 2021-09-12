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
- Data is located on this [link](https://divvy-tripdata.s3.amazonaws.com/index.html)
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
- Combined 12 table for each month into 1  single table
- Added ride duration column as duration 
- Removed start_station_id and end_station_id because I allready used station name column
- Removed start_lat, start_lng, end_lat,and end_lng because data inconsistency on these column

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
After combined data was created, we check for null on this data

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
![null](https://user-images.githubusercontent.com/90141628/132937672-2acef97f-9c40-4b97-9d6d-54f888af5c3e.PNG)

```
SELECT
    (SUM(CASE WHEN start_station_name IS NULL THEN 1 ELSE 0 END)) / Count(start_station_name) * 100 end_station_null_percentage,
    (SUM(CASE WHEN end_station_name IS NULL THEN 1 ELSE 0 END)) / Count(start_station_name) * 100 start_station_null_percentage 
FROM  analyzing-data-319917.bike_capstone_project.year_trip
```
![nul_percentage](https://user-images.githubusercontent.com/90141628/132937724-bc799a96-188e-42a3-bac6-cd08b028616d.PNG)

From those query, we know that only start_station_name (5,62%) and end_station_name (6.35%) that has null value

Next we check for duplicated data by using ride_id as an unique constraint

```
SELECT ride_id, count (ride_id)
FROM analyzing-data-319917.bike_capstone_project.year_trip
GROUP BY ride_id
HAVING count(ride_id) > 1
```
From this query we can knwow that there is 209 duplicates data

From data source we know that trip duration must be at least 60 seconds/1 minute. There is 59.575 data on trip_duration column that is below 1 minute.
```
SELECT COUNT(duration_second)
FROM analyzing-data-319917.bike_capstone_project.year_trip
WHERE duration_second < 60
```
Next we do cleaning process with a single query into a new clean table. I also add ride_day column to identify whether the trip is done on weekdays or weekends.
```
CREATE TABLE analyzing-data-319917.bike_capstone_project.year_trip_clean AS 
    SELECT 
        ride_id,
        rideable_type,
        CAST(started_at AS DATE) started_date, 
        CAST(started_at AS TIME) started_time,
        start_station_name,
        end_station_name,
        member_casual,
        duration_minute,
        CASE WHEN day_of_week BETWEEN 2 AND 6 THEN "weekday" ELSE "weekend" END ride_day
    FROM 
    (
        SELECT 
            *,
            EXTRACT(DAYOFWEEK FROM started_at) day_of_week,
            ROW_NUMBER()
            OVER (PARTITION BY ride_id) AS row_number 
        FROM analyzing-data-319917.bike_capstone_project.year_trip )
    WHERE 
        row_number = 1 AND
        duration_minute >= 1 AND
        start_station_name IS NOT NULL AND 
        end_station_name IS NOT NULL
```
And then, we double check to ensure is our data really clean or not with previous query.
```
-- Check for Null --

SELECT
    SUM(CASE WHEN ride_id IS NULL THEN 1 ELSE 0 END) AS ride_id_null,
    SUM(CASE WHEN rideable_type IS NULL THEN 1 ELSE 0 END) AS rideable_type_null,
    SUM(CASE WHEN started_at IS NULL THEN 1 ELSE 0 END) AS started_at_null,
    SUM(CASE WHEN ended_at IS NULL THEN 1 ELSE 0 END) AS ended_at_null,
    SUM(CASE WHEN start_station_name IS NULL THEN 1 ELSE 0 END) AS start_station_null,
    SUM(CASE WHEN end_station_name IS NULL THEN 1 ELSE 0 END) AS end_station_null,
    SUM(CASE WHEN member_casual IS NULL THEN 1 ELSE 0 END) AS start_station_null
FROM  analyzing-data-319917.bike_capstone_project.year_trip_clean

-- Check for Duplicates --

SELECT ride_id, count (ride_id)
FROM analyzing-data-319917.bike_capstone_project.year_trip_clean
GROUP BY ride_id
HAVING count(ride_id) > 1

-- Check for Duration Below 1 Minute --

SELECT COUNT(duration_minute)
FROM analyzing-data-319917.bike_capstone_project.year_trip_clean
WHERE duration_minute < 1
```
## ANALYZE
For interactive dashboard in Tableau Public click on this [link](https://public.tableau.com/app/profile/rizky.ramadhan5281/viz/cyclistic_16312552520290/Dashboard1#1)

```
SELECT 
    COUNT(ride_id) num_of_ride,
    AVG(duration_minute) avg_duration_minute,
    SUM(duration_minute)/60 total_duration_hour,
    SUM(CASE WHEN ride_day = "weekday" THEN 1 ELSE 0 END)/ COUNT(ride_day) * 100 weekday_ride_percentage,
    SUM(CASE WHEN ride_day = "weekend" THEN 1 ELSE 0 END)/ COUNT(ride_day) * 100 weekend_ride_percentage 
FROM analyzing-data-319917.bike_capstone_project.year_trip_clean
GROUP BY member_casual
```
Result 

![member_casual_diff](https://user-images.githubusercontent.com/90141628/132940802-5f2d6b35-b8e6-43f7-a8fc-2b186330b907.PNG)

- **Number of Ride**

![num_of_ride](https://user-images.githubusercontent.com/90141628/132942971-9be9fa9a-3eb2-46e0-bdf7-7bf210bef754.PNG)
 
- **Total Duration (Hour)**

![total_duration](https://user-images.githubusercontent.com/90141628/132943105-5d04bbb2-4e8d-4ae3-8e55-1518fd635e78.PNG)

- **Average Duration (Minute)**

![avg_duration](https://user-images.githubusercontent.com/90141628/132943158-4e1d2b59-69b6-4f13-b87e-750deac64378.PNG)

- **Day Ride**

![ride_day](https://user-images.githubusercontent.com/90141628/132943046-6549c2a2-e39e-4fe6-99ca-7a24fa8fa91b.PNG)

- **Day to Day Ride**

![day to day](https://user-images.githubusercontent.com/90141628/132943652-f9a181c3-be5d-4dbf-a8ce-25e16fea9ebc.PNG)

### SUMMARY
**How do annual members and casual riders use Cyclistic bikes differently?**
- Member has more trip rather than casual, but if we looks in Total Duration and Average Duration chart, casual has more duration in using bike with 44 minutes for single trip in avareage
- Ratio of bicycles types used tends to be the same between Casual and Member.
- Casual rider trip in weekend has a significant increase, while the number of member rider trips tends to stay the same from day to day
- If we look on day to day charts, Member rider trip significantly increased on 5am to 9am and on 3pm to 7pm. 
- We can conclude that most of member rider used cyclistic as transportaion for going to work on weekdays, while most of casual rider used cyclistic only for pleasure. 

**Why would casual riders buy Cyclistic annual memberships?**

- There is similar trend on casual rider, with increasing ride on 3pm to 9pm. Casual rider who use cyclistic for work can be the first target to convert as a member, because it is very simmilar to characteristic of member rider. 
- Second target to be converted to become annual member is casual rider who use cyclistic for pleasure.
- If a rider use cyclistic as a transportation for going to work, it must be a single trip not rounded trip, in other word started station must be different with ended station. We can look how many rider who match these critheria:

```
SELECT 
    member_casual,
    count(member_casual) AS count_all,
    SUM(CASE WHEN start_station_name NOT LIKE end_station_name THEN 1 ELSE 0 END) AS count_one_trip
FROM analyzing-data-319917.bike_capstone_project.year_trip_clean
WHERE
    started_time >= '15:00:00' AND 
    started_time <= '19:00:00'
GROUP BY member_casual
```
![target](https://user-images.githubusercontent.com/90141628/132947809-c91c42a3-42aa-491c-b53e-e97fb4aa18da.PNG)

We can also look for trip distributions for each stationns:

```
SELECT 
    start_station_name,
    SUM(CASE WHEN start_station_name NOT LIKE end_station_name THEN 1 ELSE 0 END) AS count_one_trip,
    member_casual
FROM analyzing-data-319917.bike_capstone_project.year_trip_clean
WHERE
    started_time >= '15:00:00' AND 
    started_time <= '19:00:00'
GROUP BY 
    start_station_name,
    member_casual
ORDER BY count_one_trip DESC 
```

![trip distribution](https://user-images.githubusercontent.com/90141628/132949122-74c6253b-d5a4-4eae-9dc3-57142a882936.PNG)

From this char we know that distribution of casual rider who tend to used cylistic for going to work in every stations.

### RECOMMENDATION
- Cyccistic can make a marketing campaign that targeted casual rider who use cyclistic for going to work. Give them education about what is benefits if they want to become membership.
- For casual rider who use cyclistic for pleasure, Cyclistic can give an insentif rent fee on weekends if they want to become annual membership.
- For a long term campaign, cyclistic can make a long term education about what is benefits of using bike for going to work, because the more users who use bicycles to go to work, the greater the potential for increasing the number of annual memberships. This refers to the characteristics of the annual membership that uses cyclistic as transportaion for going to work on weekdays.
- We can make a deeper investigation by look at data about each rider behaviour.



