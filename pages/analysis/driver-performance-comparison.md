# Performance Comparison

To address the query:

"What is the difference in performance between the two drives in our team?"

Regarding the difference in performance between the two drivers in Mercedes team, Lewis Hamilton (car number 44) and George Russell (car number 63), we will delve into specific sub-questions. 

These sub-questions will provide insights into various aspects of their racing performance.
## Sector Performance Comparison
1. Which driver is best in which sector?

To evaluate the performance of Lewis Hamilton and George Russell in each sector, we examined their average sector times across all races.

Lewis Hamilton's average sector times are as follows:

```sql hamilton_sector_time_result
select 
    avg(Sector1Time) as avg_sector_1,
    avg(Sector2Time) as avg_sector_2,
    avg(Sector3Time) as avg_sector_3
from 
    (
    select 
        CAST(SUBSTRING(Sector1Time, 1, 2) AS INTEGER) * 3600000 + 
        CAST(SUBSTRING(Sector1Time, 4, 2) AS INTEGER) * 60000 +
        CAST(SUBSTRING(Sector1Time, 7, 2) AS INTEGER) * 1000 AS Sector1Time,  
        CAST(SUBSTRING(Sector2Time, 1, 2) AS INTEGER) * 3600000 + 
        CAST(SUBSTRING(Sector2Time, 4, 2) AS INTEGER) * 60000 +
        CAST(SUBSTRING(Sector2Time, 7, 2) AS INTEGER) * 1000  AS Sector2Time,  
        CAST(SUBSTRING(Sector3Time, 1, 2) AS INTEGER) * 3600000 + 
        CAST(SUBSTRING(Sector3Time, 4, 2) AS INTEGER) * 60000 +
        CAST(SUBSTRING(Sector3Time, 7, 2) AS INTEGER) * 1000 AS Sector3Time,  
    from 
        (
        select 
            CAST(Sector1Time[7:] as time) as Sector1Time,
            CAST(Sector2Time[7:] as time) as Sector2Time,
            CAST(Sector3Time[7:] as time) as Sector3Time
        from 
            src_laps
        where 
            DriverNumber = 44 and Sector1Time is not null and Sector2Time is not null and Sector3Time is not null
        ) as hamilton_sector_time_result
    ) as avg_hamilton_sector_time_result
```
On the other hand, George Russell's average sector times are:
```sql russell_sector_time_result
select 
    avg(Sector1Time) as avg_sector_1,
    avg(Sector2Time) as avg_sector_2,
    avg(Sector3Time) as avg_sector_3
from 
    (
    select 
        CAST(SUBSTRING(Sector1Time, 1, 2) AS INTEGER) * 3600000 + 
        CAST(SUBSTRING(Sector1Time, 4, 2) AS INTEGER) * 60000 +
        CAST(SUBSTRING(Sector1Time, 7, 2) AS INTEGER) * 1000 AS Sector1Time,  
        CAST(SUBSTRING(Sector2Time, 1, 2) AS INTEGER) * 3600000 + 
        CAST(SUBSTRING(Sector2Time, 4, 2) AS INTEGER) * 60000 +
        CAST(SUBSTRING(Sector2Time, 7, 2) AS INTEGER) * 1000  AS Sector2Time,  
        CAST(SUBSTRING(Sector3Time, 1, 2) AS INTEGER) * 3600000 + 
        CAST(SUBSTRING(Sector3Time, 4, 2) AS INTEGER) * 60000 +
        CAST(SUBSTRING(Sector3Time, 7, 2) AS INTEGER) * 1000 AS Sector3Time,  
    from 
        (
        select 
            CAST(Sector1Time[7:] as time) as Sector1Time,
            CAST(Sector2Time[7:] as time) as Sector2Time,
            CAST(Sector3Time[7:] as time) as Sector3Time
        from 
            src_laps
        where 
            DriverNumber = 63 and Sector1Time is not null and Sector2Time is not null and Sector3Time is not null
        ) as russell_sector_time_result
    ) as avg_russell_sector_time_result
```
### Line Chart Analysis:
The line chart below illustrates the average sector times of Lewis Hamilton and George Russell in each sector.

<LineChart 
    data={[
        {driver: 'Hamilton', sector: 'Sector 1', time: 30.0}, 
        {driver: 'Hamilton', sector: 'Sector 2', time: 36.0}, 
        {driver: 'Hamilton', sector: 'Sector 3', time: 29.3}, 
        {driver: 'Russell', sector: 'Sector 1', time: 29.8}, 
        {driver: 'Russell', sector: 'Sector 2', time: 35.7}, 
        {driver: 'Russell', sector: 'Sector 3', time: 29.2}
    ]} 
    x="sector" 
    y="time" 
    series="driver" 
/>

### Conclusion:
- Sector 1: Russell demonstrates faster times than Hamilton, with a difference of 0.2 milliseconds (0.2k milliseconds), indicating better performance in Sector 1.
- Sector 2: Russell also exhibits faster performance compared to Hamilton, with a difference of 0.3 milliseconds (0.3k milliseconds) in Sector 2.
- Sector 3: Similarly, Russell showcases faster times than Hamilton, with a difference of 0.1 milliseconds (0.1k milliseconds) in Sector 3.

Upon analyzing the data, it's evident that George Russell outperforms Lewis Hamilton in all three sectors. Russell consistently displays lower average times across all sectors, indicating superior performance throughout the race.

2. Which driver does the most overtakes?
## Overtake Comparison
To determine which driver executed the most overtakes during the race, we analyzed the lap position data for both Lewis Hamilton and George Russell. Additionally, we examined the number of overtakes in races where both drivers participated, providing a more direct comparison.

### Data Overview
- Lewis Hamilton (Car Number 44)
    - Records of overtakes in career (till 02/04/2023):
    ```sql hamilton_overtakes
    WITH hamilton_overtakes AS (
        SELECT date, lap, position
        FROM src_lap_times
        WHERE carNumber = 44
    ),
    hamilton_overtakes_count AS (
        SELECT 
            date,
            COUNT(*) AS overtakes_within_race
        FROM (
            SELECT 
                date, 
                lap, 
                position,
                CASE WHEN position - LAG(position) OVER (PARTITION BY date ORDER BY lap) < 0 THEN 1 ELSE 0 END AS overtakes
            FROM hamilton_overtakes
        ) AS hamilton_overtakes_count
        WHERE overtakes = 1
        GROUP BY date
        ORDER BY date
    )
    SELECT * FROM hamilton_overtakes_count
    ```
    - Statistic of overtakes:
    The total number of overtakes by Lewis Hamilton:
    ```sql total_hamilton_overtakes
    SELECT 
        SUM(overtakes_within_race) AS total_overtake,
        avg(overtakes_within_race) AS average_overtake_per_race
    FROM ${hamilton_overtakes}
    ```
- George Russell (Car Number 63)
    - Records of overtakes in career (till 02/04/2023):
    ```sql russell_overtakes
    WITH russell_overtakes AS (
        SELECT date, lap, position
        FROM src_lap_times
        WHERE carNumber = 63
    ),
    russell_overtakes_count AS (
        SELECT 
            date,
            COUNT(*) AS overtakes_within_race
        FROM (
            SELECT 
                date, 
                lap, 
                position,
                CASE WHEN position - LAG(position) OVER (PARTITION BY date ORDER BY lap) < 0 THEN 1 ELSE 0 END AS overtakes
            FROM russell_overtakes
        ) AS russell_overtakes_count
        WHERE overtakes = 1
        GROUP BY date
        ORDER BY date
    )
    SELECT * FROM russell_overtakes_count
    ```
    - Statistic of overtakes:
    ```sql total_russell_overtakes
    SELECT 
        SUM(overtakes_within_race) AS total_overtake,
        avg(overtakes_within_race) AS average_overtake_per_race
    FROM ${russell_overtakes}
    ```

### Analysis:
- Lewis Hamilton's Performance
    <LineChart data={hamilton_overtakes} x=date y=overtakes_within_race >
        <ReferenceLine y={4.59} label="Average Overtake"/>
    </LineChart>
    The line chart illustrates Lewis Hamilton's overtakes within each race over time. Hamilton's overtaking performance varies across races, with some witnessing higher frequencies than others. Notably, there are peaks and troughs in the graph.

- George Russell's Performance
    <LineChart data={russell_overtakes} x=date y=overtakes_within_race >
        <ReferenceLine y={6.19} label="Average Overtake"/>
    </LineChart>
    The line chart showcases George Russell's overtakes within each race over time. Similar to Hamilton, Russell's overtaking performance fluctuates across races.

- Head-to-Head Comparison: The number of overtakes between Lewis Hamilton and George Russell for races they participated in together. 
    ```sql overtakes_comparison
    SELECT 
        ho.date,
        ho.overtakes_within_race AS hamilton_overtakes,
        ro.overtakes_within_race AS russell_overtakes
    FROM 
        ${hamilton_overtakes} as ho
    JOIN 
        ${russell_overtakes} as ro ON ho.date = ro.date
    ```
    The total number of overtakes for each driver is as follows::
    ```sql total_overtakes_comparison
    SELECT 
        SUM(hamilton_overtakes) AS hamilton_total_overtakes,
        SUM(russell_overtakes) AS russell_total_overtakes
    FROM 
        ${overtakes_comparison}
    ```
    <LineChart data={overtakes_comparison} x=date y={["hamilton_overtakes","russell_overtakes"]} />
    The line chart compares the number of overtakes between Lewis Hamilton and George Russell for races they participated in together. Variations between the two lines highlight differences in their overtaking performances in specific races.

### Conclusion:
Based on the evidence presented, it's apparent that: 
- Lewis Hamilton has executed more overtakes than George Russell throughout their careers. However, it's worth mentioning that Lewis Hamilton began racing in 2014 compared to George Russell's start in 2019, which may have influenced the total overtakes. 
- Despite Hamilton's overall lead in total overtakes, George Russell demonstrates a competitive edge in average overtakes per race. 
- Additionally, in races where both drivers participated together, Russell often matches or surpasses Hamilton's overtaking numbers, indicating his potential as a formidable opponent.

## Scored Points Comparison
3. Which driver scores the most points?
To determine which driver scores the most points between Lewis Hamilton and George Russell, let's start by comparing the total points earned by each driver.
### Total Point Overall
Here's the data provided for Lewis Hamilton's points and total points:
```sql hamilton_point
select name, date, points from src_driver_results where CarNumber = 44 and year >= 2014
```
```sql total_hamilton_point
select sum(points) from ${hamilton_point}
```
And for George Russell's points and total points:
```sql russell_point
select name, date, points from src_driver_results where CarNumber = 63 and year >= 2019
```
```sql total_russell_point
select sum(points) from ${russell_point}
```
It's clear that Lewis Hamilton scored a much higher point 3333 compared to 299 of George Russell

### Total Point in Shared Race
To make the comparison more accurate, let's compare the points between 2 races in the race they participate together, here is the data table: 
```sql point_comparison
    SELECT 
        hp.date,
        hp.points AS hamilton_point,
        rp.points AS russell_point
    FROM 
        ${hamilton_point} as hp
    JOIN 
        ${russell_point} as rp ON hp.date = rp.date
```
The total number of overtakes for each driver is as follows:
```sql total_point_comparison
SELECT 
    SUM(hamilton_point) AS hamilton_total_point,
    SUM(russell_point) AS russell_total_point
FROM 
    ${point_comparison}
```
From the calculations, it's clear that Lewis Hamilton scored a total of 2487 points, while George Russell scored a total of 88 points.

The line chart below illustrates the comparison of points earned by Lewis Hamilton and George Russell over the specified period. Each line represents the points earned by each driver in the races where they participated together. The X-axis represents the dates (when the race happened), while the Y-axis represents the total points earned in that race.
<LineChart
    data={point_comparison} 
    x=date 
    y={["hamilton_point", "russell_point"]}
/>

### Conclusion
Lewis Hamilton has consistently outperformed George Russell in terms of total points earned. Despite both drivers participating in races together, Hamilton's points significantly outweigh those of Russell. This analysis reaffirms Hamilton's dominance in the sport compared to Russell, indicating Hamilton as the driver who scores the most points.

## Qualification & Race Comparison
4. Which driver is better during qualification, and which is better during the race?

## Improvement Comparison
5. Which driver improves best from `free practice 1` till the race?

```sql average_position
select carNumber, avg(position) as average_position, avg(milliseconds) as finish_time_in_milliseconds
from src_lap_times
where date = '2023-04-02' 
group by carNumber
order by average_position asc
```

```sql average_lap_results
select 
    lap_data.lap,
    lap_data.milliseconds,
    avg_time.average_finish_time
from 
    src_lap_times as lap_data
join 
    (select 
         lap,
         avg(milliseconds) as average_finish_time
     from 
         src_lap_times
     where 
         date = '2023-04-02'
     group by lap) as avg_time on lap_data.lap = avg_time.lap
where 
    lap_data.date = '2023-04-02' and lap_data.carNumber = 44;
```
<LineChart data={average_lap_results} x=lap y={["milliseconds", "average_finish_time"]} />
