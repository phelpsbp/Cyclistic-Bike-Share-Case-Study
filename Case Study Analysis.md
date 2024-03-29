# Cyclistic Bike-Share Case Study Analysis

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Results and Findings](#results-and-findings)
- [Recommendations](#recommendations)
- [Limitations](#limitations)
- [Deliverables](#deliverables)

  
### Project Overview
---

This data analysis project aims to produce insights into customer trends for a bike-share company, Cyclistic. By analyzing historical bike trip data, we seek to produce professional data visualizations, along with recommendations for marketing strategies, aimed at converting casual riders to annual members for future revenue and growth.

### Data Sources

The [data](https://divvy-tripdata.s3.amazonaws.com/index.html) is made available by Motivate International Inc. under this [license](https://divvybikes.com/data-license-agreement). 
This is a public data set and there is no personal user information being shared. 


### Tools

- Excel - Data Cleaning
- Google BigQuery - Data Analysis
- PowerBI - Dashboard

---
## Data Cleaning and Preparation

In the initial data preparation phase, we performed the following tasks:
#### Cleaning in Excel:
1. Checked for and removed duplicates and nulls. 
2. Removed spacing using the TRIM() function.
3. Changed time format in `ride_length` to 37:30:55.
4. Created new `day_of_week` column using the WEEKDAY() function and formatted it as a number with no decimals(1=Sunday, 7=Saturday).
5. Created `ride_month` with 1-12 representing January-December respectively.
6. Loaded each month's CSV file into Google Cloud as a bucket then imported them into BigQuery.

#### Preparation in BigQuery
1. Created a new dataset titled [alldata](https://github.com/phelpsbp/Project-Files/blob/main/SQL/GoogleCaseStudy/alldata.csv) using the UNION ALL function.
2. Created a new master dataset, [summary_table](https://github.com/phelpsbp/Project-Files/blob/main/SQL/GoogleCaseStudy/summary_table.csv), to get rid of unnecesary columns.
3. Made the following changes for a clearer understanding of column names:
   * `rideable_type` -> `bike_type`
   * `member_casual` -> `customer_type`
4. Used table aggregation using the WITH agg_table AS function to extract `ride_duration` in minutes through the DATETIME_DIFF function.
5. Replaced all numeric values in `day_of_week` and `ride_month` columns with their corresponding names using the CASE, WHEN function.

```sql
     WITH
          agg_table AS
          (SELECT
            * EXCEPT (day_of_week),
            DATETIME_DIFF(ended_at, started_at, minute) AS ride_duration,
            EXTRACT(month
            FROM
              started_at) AS ride_month,

       CASE
              WHEN day_of_week = 1 THEN "Sunday"
              WHEN day_of_week = 2 THEN "Monday"
              WHEN day_of_week = 3 THEN "Tuesday"
              WHEN day_of_week = 4 THEN "Wednesday"
              WHEN day_of_week = 5 THEN "Thursday"
              WHEN day_of_week = 6 THEN "Friday"
            ELSE
            "Saturday"
          END
            AS day_of_week
          FROM
            `capstone-403818.months_2021.alldata`),

       month_name as
          (select
            * EXCEPT (ride_month),
            CASE
              WHEN ride_month = 1 THEN "Jan"
              WHEN ride_month = 2 THEN "Feb"
              WHEN ride_month = 3 THEN "Mar"
              WHEN ride_month = 4 THEN "Apr"
              WHEN ride_month = 5 THEN "May"
              WHEN ride_month = 6 THEN "Jun"
              WHEN ride_month = 7 THEN "Jul"
              WHEN ride_month = 8 THEN "Aug"
              WHEN ride_month = 9 THEN "Sept"
              WHEN ride_month = 10 THEN "Oct"
              WHEN ride_month = 11 THEN "Nov"
              WHEN ride_month = 12 THEN "Dec"
          END
            AS month_name
          FROM
            `capstone-403818.VisualizationTables.summary_table`),
```

## Exploratory Data Analysis

EDA involved exploring the service usage data to answer key questions, such as:

- What are the overall bike usage trends?
- Which bikes are used most?
- When are the peak bike usage periods?
- Which bike stations are most frequented?

## Data Analysis

*Note: All data is grouped by customer type. There was a type within the data tables. `costumer_type` was corrected to `customer_type`*

### Examining the differences total trips taken 
#### Total Trips Overall: 

```sql
select costumer_type,
count(*) as total_trips
from `capstone-403818.VisualizationTables.summary_table`
group by costumer_type
order by costumer_type;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/7bc19cb7-2c13-4ecd-bd76-b7426e0a8ebc" width="425"/>

#### Total Trips per Bike Type
```sql
select costumer_type, bike_type,
count(*) as total_trips
from `capstone-403818.VisualizationTables.summary_table`
group by costumer_type, bike_type
order by costumer_type, total_trips;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/1c0a6397-68b6-47e1-b277-6eeb1ce2be11" width="425"/>

#### Monthly Trips 

```sql
select costumer_type, ride_month,
count(ride_id) as total_trips
from `capstone-403818.VisualizationTables.summary_table`
group by costumer_type, ride_month
order by ride_month;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/fd7fde56-793e-418c-b500-36c9078b3036" width="425"/>

#### Trips per Day

```sql
select costumer_type, day_of_week,
count(ride_id) as total_trips
from `capstone-403818.VisualizationTables.summary_table`
group by costumer_type, day_of_week
order by day_of_week;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/23575b16-2300-4424-8f25-821942f8fcdc" width="425"/>

#### Trips per Hour
```sql
select 
extract (HOUR from started_at) as hour,
count(started_at) as total_rides, costumer_type
from `capstone-403818.VisualizationTables.summary_table`
group by hour, costumer_type
order by hour;
```
| <img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/c841e258-d145-4a9e-a663-b71d1c855eb9" width="425"/> <img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/cb902219-5644-4e1c-9252-42488e2a5907" width="425"/> | 
|:--:| 
| *Hours are in a 24-hour clock format with 0 = midnight (12AM)* |


### Differences in Frequencies
#### Average Ride Lengths
```sql
select costumer_type,
avg(ride_duration) as avg_ride_length
from `capstone-403818.VisualizationTables.summary_table`
group by costumer_type;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/28922e31-f0bb-4d5c-a507-dad356c4d508" width="425"/>

#### Average Ride Lengths per Month
```sql
select costumer_type, ride_month,
avg(ride_duration) as avg_ride_length
from `capstone-403818.VisualizationTables.summary_table`
group by costumer_type, ride_month
order by ride_month;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/b7967069-c989-4505-b4f4-66a17099b177" width="425"/>

#### Average Ride Lengths per Day
```sql
select costumer_type, day_of_week,
avg(ride_duration) as avg_ride_length
from `capstone-403818.VisualizationTables.summary_table`
group by costumer_type, day_of_week
order by day_of_week;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/8c0cd415-a9c7-4fb4-b9f2-49785454d992" width="425"/>

### Most Popular Bike Stations
#### Top 10 Start and End Stations - Casual Riders
```sql
select costumer_type, start_station_name,
count(*) as visits_casual
from `capstone-403818.VisualizationTables.summary_table`
where costumer_type = 'casual'
group by costumer_type, start_station_name
order by count(*) desc
limit 10;

select costumer_type, end_station_name,
count(*) as visits_casual
from `capstone-403818.VisualizationTables.summary_table`
where costumer_type = 'casual'
group by costumer_type, end_station_name
order by count(*) desc
limit 10;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/48a660a2-1c08-4740-a9b7-104776fbb51e" width="425"/><img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/7b731fcc-38d4-4b29-8b52-8b2bb2fff41e" width="425"/>

#### Top 10 Start and End Stations - Members
```sql
select costumer_type, start_station_name,
count(*) as visits_member
from `capstone-403818.VisualizationTables.summary_table`
where costumer_type = 'member'
group by costumer_type, start_station_name
order by count(*) desc
limit 10;

select costumer_type, end_station_name,
count(*) as visits_member
from `capstone-403818.VisualizationTables.summary_table`
where costumer_type = 'member'
group by costumer_type, end_station_name
order by count(*) desc
limit 10;
```
<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/d52a8c29-8fc1-4e3a-b439-aca278d823d3" width="425"/><img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/a3f6b288-51de-40fb-a798-bfe900e30c24" width="425"/>


## Results and Findings

The analysis results are summarized as follows:
|Casual Riders|Annual Members|
|--------|--------|
|Preferred classic bikes. Had a greater preference for docked bikes|Preferred classic bikes. Showed a higher inclination for electric bikes|
|Favored the Summer: June-August were the busiest months|Rode most mid-Summer to early fall: July-September|
|Rode most frequently on weekends, specifically Saturdays|Rode consistently throughout the week, peaking on Wednesdays, indicating that members most likely use this service for purposes outside of leisure - like commuting or everyday errands|
|Usage increased steadily throughout the day, peaking at 5PM|Busiest hours coincided with school and working hours - 8AM, 12PM, and 5PM|
|Rode for longer ride durations, especially on weekends in warmer months|Had shorter ride lengths but rode at more consistent durations, suggesting that usage is linked to daily routines and commute|
|Visited bike stations primarily by large attractions and entertainment such as parks, theaters, and aquariums|Frequented stations that were located downtown - colleges, office buildings, and residential areas|

## Recommendations

Based on the analysis, my recommendations of a marketing strategy aimed at converting casual riders into annual subscription members are as follows:

1. ***Exclusive Discounts***
   - Casual riders used Cyclistic Bike-Share at large and in longer durations on weekends. Providing exclusive member-only discounts for weekends and longer rides can encourage casual riders to pay for annual memberships.
2. ***Area-Specific Promotions***
   - Advertise at bike stations most frequented by casual riders, especially at stations near large tourist attractions and entertainment.
3. ***Seasonal Marketing Campaign***
   - Use targeted marketing during peak activity hours, specifically on weekends and during Summer, to highlight the cost-savings advantages of annual membership.

## Limitations

Even after removing test rides (rides greater than 24 hours in duration), there were some outliers in casual riders. This could be a factor in casual riders having significantly longer average ride lengths.
There was no data on gender or age, both of which can play a huge role in biking trends, limiting possible marketing opportunities to specific groups. 

  
## Deliverables

The Full, interactive Power BI Dashboard can be viewed [here](https://app.powerbi.com/view?r=eyJrIjoiNDY5Y2NkYWYtY2M0Zi00YTJkLWE5MjQtMTBhMmU5ZjA0NGNiIiwidCI6IjM1NWI3MWIwLWEyMDQtNGMyMC05NzQ3LTVlYTU3OTQyNzkxZCIsImMiOjJ9)

<img src="https://github.com/phelpsbp/Cyclistic-Bike-Share-Case-Study/assets/150976820/fdd16710-dd57-4240-ba1c-4dbff6cb27c1" height="450"/>




---
