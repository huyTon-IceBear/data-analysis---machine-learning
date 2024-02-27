# Performance Comparison

To address the query:

"What is the difference in performance between the two drives in our team?"

Regarding the difference in performance between the two drivers in Mercedes team, Lewis Hamilton (car number 44) and George Russell (car number 63), we will delve into specific sub-questions. 

These sub-questions will provide insights into various aspects of their racing performance.
## Sector Performance
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
## Overtake Analysis
To determine which driver executed the most overtakes during the race, we analyzed the lap position data for both Lewis Hamilton and George Russell. Here are the lap position data for each driver:

Lewis Hamilton (Car Number 44):
```sql hamilton_position
select lap, position from 
    src_lap_times
where 
    date = '2023-04-02' and carNumber = 44
```
George Russell (Car Number 63):
```sql russell_position
select lap, position from 
    src_lap_times
where 
    date = '2023-04-02' and carNumber = 63
```
### Analysis:
We represented the lap positions of both drivers throughout the race using line charts. In these visualizations, the x-axis denotes the lap number, while the y-axis indicates the position on the track.

Upon reviewing the lap position data:

- Hamilton's Line Chart: Hamilton's line illustrates his race performance, showing how his position fluctuated over each lap. Consistently high positions, such as positions 1 or 2, indicate periods when Hamilton was leading or closely following the race leader.
<LineChart 
    data={hamilton_position}
    x="lap" 
    y="position"
/>

- Russell's Line Chart: George Russell's line graph illustrates his race progression, highlighting his initial lead for the first seven laps of the race, followed by notable declines in position, each succeeded by recoveries. Notably, he dropped to 15th position at lap 17due to a Power Unit failure, which forced him to retire on lap 18, triggering a virtual safety car.
<LineChart 
    data={russell_position}
    x="lap" 
    y="position"
/>

### Conclusion:
Examining the lap position data, it's evident that George Russell executed more overtakes compared to Lewis Hamilton. Russell's varied positions throughout the race, including regaining positions lost due to the Power Unit failure. Therefore, George Russell is considered to have executed the most overtakes among the two drivers.

3. Which driver scores the most points?


4. Which driver is better during qualification, and which is better during the race?

5. Which driver improves best from `free practice 1` till the race?

```sql avarage_position
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
