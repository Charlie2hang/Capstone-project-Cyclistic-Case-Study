# Capstone project Cyclistic Case Study



## Introduction

​	I am the junior data analyst working on the marketing analyst team at Cyclistic, a bike-share company in Chicago. Cyclistic offers more than 5,800 bicycles and 600 docking stations by reclining bikes, hand tricycles, and cargo bikes. Cyclistic provides single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclitic members.

### Ask

The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Data team assigns me to ask the three questions to guide our future marketing program: 

1. How do annual members and casual riders use Cyclitic bikes differently? 
2. Why would casual riders buy Cyclitic annual memberships? 
3. How can Cyclitic use digital media to influence casual riders to become members?



# Prepare

​	To answer this question I will be analyzing historical Cyclistic bike trip data for all 12 months in 2021. The data is reliable, free of any bias, and has been collected by Cyclistic and stored on the company’s database separated by month in CSV format. For the purpose of this project, I have saved the 12 relevant CSV files in my local drive.

The data collection team at Cyclistic have outlined some key facts and constraints about the data:

1. Each month contains every single trip that took place during that period.

2. All personal customer information has been removed for privacy issues.

3. Classic bikes were previously labeled ‘docked bikes’, they refer to the same thing.

4. Classic bikes must start and end at a docking station, whereas electric bikes have a bike lock attached to them; thus, electric bikes can also start and end their trip locked up anywhere in the general vicinity of a docking station.

5. The data should have no trips shorter than 1 minute or longer than 1 day. Any data that does not fit these constraints should be removed as it is a maintenance trip carried out by the Cyclistic team, or the bike has been stolen.

   

   Note: Cyclistic dataset can be found https://divvy-tripdata.s3.amazonaws.com/index.html

   

   # Process

   ​	To combine and clean the data I used **SQL** on Google’s **BigQuery** platform. Below is an outline of my process:

   

   ## Data Combination

   ​	Created a dataset called ‘bike_tripdata_2021’ and imported the 12 CSV files as 12 separate monthly tables in the dataset.
   Combined all 12 monthly tables into one table containing all bike trip data from 2021. The combined table contains 5,595,063 rows and has 13 columns/fields. Below is the name of each column and its data type:

   1. Created a dataset called ‘bike_tripdata_2021’ and imported the 12 CSV files as 12 separate monthly tables in the dataset.

   2. Combined all 12 monthly tables into one table containing all bike trip data from 2021. The combined table contains **5,595,063 rows and has 13 columns/fields. Below is the name of each column and its data type:**

      

      <img src="https://miro.medium.com/v2/resize:fit:373/1*l1OIWBcpWyesBI7aH4JYxQ.png" alt="img" style="zoom:80%;" />

      

      3. Double-check that the amount of rows in the combined 2021 table matches the sum of the 12 separate monthly tables rows.

         

      ## Pre-Cleaning Data Exploration

   ​	I familiarized myself with the combined table by running queries on each column/field and making notes of data to be cleaned. My pre-cleaning data exploration process with SQL can be viewed on my Github. An example of me analyzing the first column ‘ride_id’ can be seen below:

   ```sql
   /*1.ride_id:
   - check length combinations for ride_id
   - and all values are unigue as ride_id is a primary key
   */
   
   SELECT LENGTH(ride_id), count(*)FROM 'divvy-bike-sharing-app-data.bike_tripdata_2021.combined_tripdata' GROUP BY LENGTH(ride_id);
   
   SELECT COUNT(DISTINCT ride_id) FROM 'divvy-bike-sharing-app-data.bike_tripdata_2021.combined_tripdata';
   
   # NOTES:
   # All ride_id strings are 16 characters long 
   # And they are all distinctNo cleaning neccesary on this column.
   
   ```

   Here is a quick summary of my findings:

   - **ride_id:** is a primary key, has no duplicates, and each ride_id string is exactly 16 characters long; no data cleaning is necessary. Further, around 5.6 million unique bike trips took place in 2021.

   - **rideable_type:** the data contains 3 types of bikes: classic, docked, and electric bikes; however, as specified by the data collection team, ‘docked bike’ is the old name for ‘classic bike’. Thus, we must change any occurrence of ‘docked bike’ to ‘classic bike’.

   - **started_at/ended_at:** these columns show the date and time that the bike trips started/ended. There are 163,630 trips where the ride-time of the trip was less than 1 minute or greater than 1 day. All of these trips will be removed during the cleaning process.

   - **start_station_name/end_station_name:** found various station names that are maintenance stations, these will be removed. Further, there are instances of leading and trailing spaces in the station names, these will be cleaned to ensure there are no duplicate station names. Lastly, there are 1,006,760 trips with null values in either the starting station name column or the ending station name. Of these trips, 9,334 pertain to classic bike trips. These trips will be removed as classic bike trips **must** start or end at a docking station. The remaining 997,426 trips pertain to electric bike trips and will not be removed as electric bikes do not have to start or end at a docking station, as they have the bike lock option.

   - **start_station_id/end_station_id:** there are various inconsistencies in string length and these columns do not add value to our analysis; thus, we will remove the columns for the final cleaned table.

   - **start_lat/end_lat & start_lng/end_lng:** these 4 columns show the starting and ending location of the bike trips. We will plot the locations on a map in Tableau later; thus, we will remove all 4,771 trips that have at least one null value.

   - **member_casual:** this column indicates if the customer of the trip was a casual customer or an annual member. I double-checked that ‘casual’ and ‘member’ are the only allowable strings in this column.

     

   ## Cleaning the Data

​	After carrying out the pre-cleaning data exploration process, I now know what data needs to be cleaned/removed and which columns can be created from the existing data to assist us in our analysis. My data cleaning query can be viewed on Github and below is an example of one part of the query:

```sql
# 1. Find classic/docked bike trips which do not begin or end at a docking station
null_station_names AS (
	SELECT ride_id As bad_ride_id
    FROM (SELECT ride_id, start_station_name, start_station_id,
          end_station_name, end_station_id 
          FROM combined_data
          WHERE rideable_type ='docked_bike' OR rideable_type ='classic_bike'
         )
	WHERE start_station_name IS NULL AND start_station id IS NULL OR
    end_station_name IS NULL AND end_station_id IS NULL
),

# 2. Remove these rows as classic/docked bikes have to start/end at a docking station
# Also remove rows that do not have a starting/ending latitude and longitude
null station names _cleaned AS (
    SELECT * FROM combined_data cd
    LEFT JOIN nu1l_station_names nsn ON cd.ride id = nsn.bad_ride _id
    WHERE nsn.bad ride_id IS NULL AND
    cd.start_lat IS NOT NULL AND
    cd.start_Ing IS NOT NULL AND
    cd.end lat IS NOT NULL AND
    cd.end_Ing IS NOT NULL   
)
```

A summary of the cleaning steps I took is as follows:

- Removed trips where the type of bike used was a classic bike and either the start or end station was null.
- Removed trips that had null values in the starting/ending latitude or longitude columns.
- Replaced occurrences of ‘docked bike’ with ‘classic bike’.
- Cleaned up the start and end station name columns by trimming leading and trailing spaces.
- Replaced null values in the starting & ending station name columns with the string ‘On Bike Lock’ for **only** electric bikes.
- Created day of the week, month, year, and ride time length columns to bolster analysis.
- Removed trips where the ride time length was less than/equal to 1 minute or greater than/equal to 1 day.
- Removed trips that contained bike maintenance station names.
- In total, I removed 170,733 rows to be left with a clean combined table with 5,424,330 rows.



# Analyze

​	Now that the data is completely clean it is time to analyze the data and answer the question: **“How do annual members and casual riders use Cyclistic bikes differently?”**

​	To analyze my data I used **SQL** to sort, filter, and aggregate my data before importing it into **Tableau** to create data visualizations. 

<img src="https://miro.medium.com/v2/resize:fit:826/1*FuLGGLr6qfZSqGF1vIYp-A.png" alt="img" style="zoom:80%;" />

​	As we can see, annual members made up 54.5% of all trips taken in 2021, whereas casual riders accounted for 45.5% of rides. Further, both groups prefer classic bikes over electric bikes, but casual riders use electric bikes at a higher proportion in relation to classic bikes than annual members (37.3% vs 34.9% respectively). This amount is not statistically significant though.

​	Next, I examined the total number of bike trips taken in 2021 by month and time of the day.

<img src="https://miro.medium.com/v2/resize:fit:826/1*cdbqU0CVbWNZaC-Q-Ub8xA.png" alt="img" style="zoom:80%;" />

​	From the line graphs above, it is evident that casual members prefer riding bikes between late spring to the end of summer and refrain from biking in the winter months (Jan/Feb), likely due to the worse weather conditions. Annual members prefer the spring/summer months as well; however, their ridership levels are more consistent and steady over the year. Furthermore, annual member ridership levels rose significantly between 6am and 8am and peak between 4pm and 6pm, before dropping considerably after 7pm. Whereas, casual ridership levels are more consistent during the day with no abrupt spikes.

​	From these findings, we can begin to hypothesize that **annual members use Cyclistic bikes for commuting purposes** as their ridership levels peak during the typical commuting times for work (8am/5pm) and drop outside of typical work hours. Furthermore, annual member ridership levels show less of a trend of dropping in the winter months in comparison to casual members. Thus, we can also hypothesize that **casual members use Cyclistic bikes more for leisure purposes.**

​	To further test this hypothesis, I then analyzed the ridership levels per day of the week and by the average ride time (in minutes)

<img src="https://miro.medium.com/v2/resize:fit:826/1*zsTB1GNfFmuoZ0vYb9YVoA.png" alt="img" style="zoom:80%;" />

​	From the line graphs above, we can see that annual member ridership levels are very steady over the workweek before dropping slightly on the weekends; whereas, casual ridership levels are lower during the week and significantly rise during the weekend. Further, the average ride time for annual members is 13.23 minutes vs 25.98 minutes for casual riders (almost 2x longer).

​	**Both of these insights strongly support our hypothesis and we can conclude that annual members are typically commuters using Cyclistic’s bikes for short trips during the weekdays to get to work and casual riders typically use Cyclistic’s bikes on the weekend for leisure.**

​	Now let's dive deeper and look at a map of Chicago with the starting locations of the bike trips plotted. The bike trips are grouped by member type and exclude electric bike trips where a bike lock was used. I have also filtered the map to only show the top 10 locations with the largest frequencies of starting trips per member type.

<img src="https://miro.medium.com/v2/resize:fit:826/1*fW6clzor1S-WT5xHrEn1qA.png" alt="img" style="zoom:80%;" />

​	As we can see above, the 10 most popular starting stations for casual members are generally near the water and big parks; in contrast, annual members usually start their trips more inland, near Chicago’s financial district, office buildings, and apartment buildings.

​	Now let's analyze the top 10 ending locations, excluding trips where a bike lock was used.

<img src="https://miro.medium.com/v2/resize:fit:826/1*gUyz0RPia7jkEfoEGCUeNA.png" alt="img" style="zoom:80%;" />

​	The map above shows a similar trend of casual riders frequenting areas near parks and the water/pier and annual members frequenting commercial areas. Further, casual riders’ top 5 most visited stations are the same for both starting and ending stations. These include Streeter Dr & Grand Ave (the pier), Millennium Park, Theater on the Lake, and Shedd Aquarium. All very big tourist attractions and recreational areas. None of these stations are in the top 5 most visited for annual members, indicating that recreation and sightseeing are not as big of a priority.



# Summary of Insights

## **Casual Riders:**

- Tend to use the Cyclistic bikes for leisure; preferring to ride during the midday hours on the weekend, concentrated in the late spring and early summer months.
- Their average ride time (25.98 minutes) is around 2x longer than annual members.
- Tend to start and end their trips near the water, city parks, and other tourist/recreational attractions.

## **Annual Members:**

- Tend to use the Cyclistic bikes for commuting; preferring to ride all-year-round on the weekdays, with a drop off in the winter months, and during peak work commuting times (8am/5pm).
- Their average ride time (13.23 minutes) is around half the average trip time of casual riders.
- Tend to start and end their trips in the city, near office and apartment buildings.



# **Recommendations**

​	Having defined the differences between casual riders and annual members, the marketing department can start to develop its strategy of converting casual members to annual members. With my knowledge of the differences between the two customer segments, I have brought up one potential issue below:

​	**Issue:** A large percentage of the casual rider segment are most likely tourists visiting Chicago who cannot be converted into annual members. However, I have developed some **recommendations below** targeted at converting Chicago residents who are casual members into long-term members:

- Cyclistic could offer a new subscription plan called ‘Fair-Weather Member’ or one called ‘Weekend Warrior’. The Fair-Weather Member plan would offer unlimited use of Cyclistic bikes during the 6 months of spring and summer at a discounted price. The Weekend Warrior plan would offer unlimited use of Cyclistic bikes every Friday, Saturday, and Sunday of the year.
- The advertising campaign should be run in the summer months (highest casual ridership levels) in order to capture the most attention. Furthermore, informational and promotional booths can be set up on weekends near the most popular starting and ending stations for casuals (near the Chicago Pier, Millenium Park, etc).
