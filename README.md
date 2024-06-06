# Google-Cyclistic-Project

## Cyclistic Bike Share Case Study - Analysis with SQL and Tableau
This data set is from a fictional Chicago-based bike-sharing company named Cyclistic. I'll be analyzing data with the SQL language and creating visuals with Tableau.

## Table of contents:

-Contextual Information 

-Ask 

-Prepare 

-Process 

-Analyze 

-Share 

-Act 

## Contextual Information

As a junior data analyst at Cyclistic, my role is to support the company’s goal of increasing annual memberships. We focus on analyzing the behavior and preferences of casual riders and annual members. Using data analysis and visualization, we aim to develop targeted marketing strategies to convert casual riders into long-term members. By following the six phases of the data analysis process, we will provide data-driven recommendations to secure executive approval and advance Cyclistic’s vision of leading urban mobility and promoting sustainable transportation in Chicago's bike-sharing industry.

## Ask Process

Questions:

How do annual members and casual riders use Cyclistic bikes differently? 

Why would casual riders buy Cyclistic annual memberships?

How can Cyclistic use digital media to influence casual riders to become members?

### Business Task

As a junior data analyst at Cyclistic, I will analyze historical bike trip data to identify patterns distinguishing casual riders from annual members. This analysis will form the basis of a marketing strategy to convert casual riders into annual members.

## Prepare Process

### Data Source:

I analyzed six months of data, which included 13 columns such as ride IDs, bike type, start and end times, station details, and rider type. After uploading to BigQuery, I combined the files into a single table for analysis.

<img width="1043" alt="Screenshot 2024-06-05 at 13 34 27" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/5f9f2741-f726-4113-aa18-7d4499f3b8b5">

### Using the ROCCC framework, the data's credibility is summarized as follows:

Reliable: Despite some missing values and inconsistent station names, the dataset is largely accurate, with minor ride time inaccuracies having minimal impact.

Original: The data is from a primary source.

Comprehensive: It contains over one million rows of complete data.

Current: It is based on the latest six months of cycling data.

Cited: The data is well-referenced and derived from actual primary records.

Thus, the data appears reliable, original, comprehensive, current, and cited.

### Data to be used

Monthly csv files are used for organizing the data. For this research, the most current six months of data (November 2023–April 2024) were used.

### Following metrics in the use of bicycles by cyclists:

Casual vs. annual rider percentages

Ride frequency by bike type

Weekly ride frequency

Average ride duration (minutes)

Daily ride counts

Station-to-station ride counts

## Process Phase 
Data Cleaning and Transformation carried out with Big Query:

--merging last 6 months of data into 1 table:

```sql
CREATE TABLE `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes`
AS
SELECT * FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.202311-divvy-tripdata`
UNION ALL
SELECT * FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.202312-divvy-tripdata`
UNION ALL
SELECT * FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.202401-divvy-tripdata`
UNION ALL
SELECT * FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.202402-divvy-tripdata`
UNION ALL
SELECT * FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.202403-divvy-tripdata`
UNION ALL
SELECT * FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.202404-divvy-tripdata`
```
<img width="1061" alt="Screenshot 2024-05-26 at 15 14 40" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/f251dcb3-c784-4a28-a513-a9aa9803b6d3">


--Removing rows with missing values in specific columns:

```sql
CREATE TABLE `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_nonull`
AS
SELECT * FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes`
WHERE NOT (start_station_id IS NULL OR
 end_station_name IS NULL OR
 end_station_id IS NULL)
```
<img width="1070" alt="Screenshot 2024-05-26 at 15 23 51" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/81d446c1-2606-4cd4-8a0a-437ee6738cca">

## Analyze Process
This SQL query creates a new table, "sixmonths_bikes_dateinfo," by combining data from "sixmonths_bikes_nonull" and adding columns that extract trip start and end dates, including months, days of the week, and times of day.

```sql
CREATE TABLE `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo` AS
SELECT
EXTRACT(MONTH FROM started_at) AS month_start,
EXTRACT(TIME FROM started_at) AS time_start,
EXTRACT(DAYOFWEEK FROM started_at) AS day,
EXTRACT(MONTH FROM ended_at) AS month_end,
EXTRACT(TIME FROM ended_at) AS time_end,
TIMESTAMP_DIFF(ended_at, started_at, second) AS difference_secs,
TIMESTAMP_DIFF(ended_at, started_at, minute) AS difference_mins,
start_station_name,
end_station_name,
member_casual,
rideable_type,
FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_nonull`
```
<img width="1068" alt="Screenshot 2024-05-26 at 15 52 17" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/8965ea9e-7343-4d66-b13b-5cd3645b0b97">


This SQL query computes statistical metrics for trip durations in the "sixmonths_bikes_dateinfo". It calculates the average, minimum, and maximum trip durations in minutes, providing valuable insights into the distribution of trip lengths:

```sql
SELECT
 AVG(difference_mins) AS avg_mins,
 MIN(difference_mins) AS minimum_mins,
 MAX(difference_mins) AS maximum_mins
 FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo`
```

<img width="1072" alt="Screenshot 2024-05-26 at 16 00 44" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/22a757f8-cab1-42ad-be6b-e2c0a5771088">

This SQL query calculates the quantity of rides with negative durations (less than zero) from the "sixmonths_bikes_dateinfo":

```sql
SELECT COUNT (*) as negative_rides_length
FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo`
WHERE difference_mins < 0
```

<img width="1041" alt="Screenshot 2024-05-26 at 16 03 30" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/6be7533b-aec8-49d5-83b8-98717f50b53c">


This SQL query counts the number of trips that lasted more than 24 hours (1,440 minutes) based on the data from the "sixmonths_bikes_dateinfo" table:

```sql
SELECT COUNT (*) rides_longer_than_day
FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo`
WHERE difference_mins > 1440
```

<img width="1016" alt="Screenshot 2024-05-26 at 16 06 39" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/7a38659d-cd81-4f29-9360-75fd64d155f8">

This SQL query creates a new table, "sixmonths_bikes_dateinfo_cleanedtimes," containing trips lasting between 1 and 1,439 minutes, excluding zero, negative, and over 24-hour trips:

```sql
CREATE TABLE logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo_cleanedtimes AS
SELECT *
FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo`
WHERE difference_mins > 0
AND difference_mins < 1440
```
<img width="1070" alt="Screenshot 2024-06-05 at 14 20 36" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/e294c6e6-8ca7-4488-92d0-44cbb5b04ddb">

This SQL query calculates the number of trips made by users (members and customs) on each day of the week:

```sql
SELECT COUNT (rideable_type) AS rides_taken,
day,
member_casual,
FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo_cleanedtimes`
GROUP BY member_casual, day
ORDER BY member_casual
```
<img width="506" alt="Screenshot 2024-06-05 at 14 24 05" src="https://github.com/esengok/Google-Cyclistic-
  Project/assets/169263106/be9c74cc-8a1f-4269-b1f4-716f0fa925d2">

This query returns the number of trips made by users on different days of the week and hours of the day, including the day name and hour:
```sql
SELECT CASE
 WHEN day = 1 THEN 'sunday'
 WHEN day = 2 THEN 'monday'
 WHEN day = 3 THEN 'tuesday'
 WHEN day = 4 THEN 'wednesday'
 WHEN day = 5 THEN 'thursday'
 WHEN day = 6 THEN 'friday'
 WHEN day = 7 THEN 'saturday' END AS day_of_the_week,
 EXTRACT(hour from time_start) AS hour,
 COUNT(*) AS rides_taken,
 member_casual,
 FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo_cleanedtimes`
 GROUP BY member_casual, day, hour
 ORDER BY member_casual, day, hour
```

<img width="706" alt="Screenshot 2024-06-05 at 14 28 04" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/fa0671af-f453-415e-b891-192687672cc5">


This SQL query combines the names of the departure and arrival stations to create a route name. Then it counts the number of trips for each route created depending on the user type (member_casual). The result is sorted by the number of trips in reverse order (from highest to lowest):

```sql
SELECT
 CONCAT (start_station_name, ' to ', end_station_name) AS route,
 COUNT (rideable_type) AS number_of_rides,
 member_casual
 FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo_cleanedtimes`
GROUP BY route, member_casual
ORDER BY number_of_rides DESC
```

<img width="581" alt="Screenshot 2024-06-05 at 14 31 37" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/6017f2dc-5d5d-4278-91c8-7b0e4f1d909d">

This SQL query calculates the number of trips by bike type and user type (member or casual), revealing bike popularity among different user categories:

```sql
SELECT
 rideable_type, member_casual,
 COUNT (rideable_type) as rides_taken
 FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo_cleanedtimes`
GROUP BY member_casual, rideable_type
ORDER BY rides_taken DESC
```
<img width="584" alt="Screenshot 2024-06-05 at 14 34 26" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/a6e53904-83c7-48e3-b42e-391deebc2e17">

This SQL query computes the average trip duration per day of the week, categorized by user type (members or casual users) and hour:
```sql
SELECT CASE
WHEN day = 1 THEN 'sunday'
 WHEN day = 2 THEN 'monday'
 WHEN day = 3 THEN 'tuesday'
 WHEN day = 4 THEN 'wednesday'
 WHEN day = 5 THEN 'thursday'
 WHEN day = 6 THEN 'friday'
 WHEN day = 7 THEN 'saturday' END AS day_of_the_week,
 ROUND(AVG(difference_mins),2) AS avg_ride_length,
 member_casual,
 EXTRACT(hour FROM time_start) AS hour,
 FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo_cleanedtimes`
 GROUP BY member_casual, day, hour
 ORDER BY member_casual, day, hour
```
<img width="755" alt="Screenshot 2024-06-05 at 14 37 10" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/7f82f681-9ed1-47ba-88e0-ea9e47f3ad44">

This SQL query calculates the average trip length in minutes by day of the week, route, bike type, and user type (member or casual user). The results are grouped by these four aspects and displayed sorted by the number of trips in descending order:

```sql
SELECT CASE
            WHEN day = 1 THEN 'Sunday'
            WHEN day = 2 THEN 'Monday'
            WHEN day = 3 THEN 'Tuesday'
            WHEN day = 4 THEN 'Wednesday'
            WHEN day = 5 THEN 'Thursday'
            WHEN day = 6 THEN 'Friday'
            WHEN day = 7 THEN 'Saturday' END AS day_of_the_week,
            CONCAT (start_station_name,' to ', end_station_name) as route,
            COUNT (rideable_type) as number_of_rides,
            rideable_type,
            member_casual,
            EXTRACT(hour FROM time_start) AS hour,
FROM `logical-acumen-422709-f9.cyclistic_bike_sharing.sixmonths_bikes_dateinfo_cleanedtimes`
GROUP BY member_casual, day, hour, route, rideable_type
ORDER BY number_of_rides DESC
```

Tables were saved as CSV files and imported into Tableau for visualization creation.

## Share Process

<img width="970" alt="Screenshot 2024-06-05 at 19 11 53" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/1de6ee19-18b3-4cc7-bbaa-acdf1350cdb6">
<img width="997" alt="Screenshot 2024-06-05 at 19 13 41" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/81478d0b-ce85-4f59-a72c-6a2720be0347">
<img width="720" alt="Screenshot 2024-06-05 at 19 15 07" src="https://github.com/esengok/Google-Cyclistic-Project/assets/169263106/4f075b18-1305-4dfc-b4ac-8b5fa5643203">

Members predominantly ride the bikes during morning and afternoon rush hours, occurring between 7-9am and 4-6pm. In contrast, casual riders gradually increase their usage throughout the day until it drops significantly around 5pm.

Overall, the data shows that casual riders use the bikes more for leisure, while members primarily use them for commuting.

The destinations of riders vary by membership type. Members frequently use routes near colleges and universities, especially around the Illinois Institute of Technology and the University of Chicago. In contrast, casual riders prefer routes near water and recreational areas

##  Act Process

Based on these findings, I recommend the marketing team:

-Revise the marketing plan to target periodic student riders. Launch a digital campaign highlighting the financial benefits of Cyclistic memberships for students. Start this promotion before the academic year to boost awareness and encourage annual memberships.

-Use digital OOH ads in popular Harborside areas and key Cyclistic routes to attract recreational casual riders. Promote the electric bikes as a fun way to explore the city. Display these ads during daylight hours in the summer, when casual rides peak.
