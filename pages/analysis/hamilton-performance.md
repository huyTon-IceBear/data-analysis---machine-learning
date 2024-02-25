# Hamilton Performance

This page use to answer the question: **How did one of our drivers do last weekends?**

Driver name: Lewis Hamilton
Car number: 44
Race: Australian Grand Prix
Race Date: 2023-04-02
Race Round: 2

To answer the question above, let's validate the performance of the driver through sub questions.
1. Do the results improve over time?

This question can be answer by analyze the different results of the driver during weekends to figure out if the result is actually improve during the race. The data for the analyzing will be from the Laps tables where will will look at the lap time to complete the lap and the time for each sector.

```sql lap_time_result
select 
    CAST(LapTime[7:] as time) as LapTime, 
    CAST(Sector1Time[7:] as time) as Sector1Time,
    CAST(Sector2Time[7:] as time) as Sector2Time,
    CAST(Sector3Time[7:] as time) as Sector3Time, 
from 
    src_laps
where 
    filename like '%data\\20230402%' and DriverNumber = 44 and LapTime not null
```

```sql lap_time_result_ms
select 
    ROW_NUMBER() OVER () AS index,
    CAST(SUBSTRING(LapTime, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(LapTime, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(LapTime, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(LapTime, 10, 3) AS INTEGER) AS LapTime,  
    CAST(SUBSTRING(Sector1Time, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(Sector1Time, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(Sector1Time, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(Sector1Time, 10, 3) AS INTEGER) AS Sector1Time,  
    CAST(SUBSTRING(Sector2Time, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(Sector2Time, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(Sector2Time, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(Sector2Time, 10, 3) AS INTEGER) AS Sector2Time,  
    CAST(SUBSTRING(Sector3Time, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(Sector3Time, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(Sector3Time, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(Sector3Time, 10, 3) AS INTEGER) AS Sector3Time,  
from 
    ${lap_time_result}
```

```sql avg_lap_time_result_ms
select 
    avg(LapTime) as avg_lap_time,
    avg(Sector1Time) as avg_sector_1,
    avg(Sector2Time) as avg_sector_2,
    avg(Sector3Time) as avg_sector_3,  
from 
    ${lap_time_result_ms}
```
So regarding the lap time, the driver have an amazing performance where 
<LineChart data={lap_time_result_ms} x=index y="LapTime">
    <ReferenceLine y={avg_lap_time_result_ms[0].avg_lap_time} label="Average Time in ms"/>
</LineChart>

<LineChart data={lap_time_result_ms} x=index y="Sector1Time">
    <ReferenceLine y={avg_lap_time_result_ms[0].avg_sector_1} label="Average Time in ms"/>
</LineChart>

<LineChart data={lap_time_result_ms} x=index y="Sector2Time">
    <ReferenceLine y={avg_lap_time_result_ms[0].avg_sector_2} label="Average Time in ms"/>
</LineChart>

<LineChart data={lap_time_result_ms} x=index y="Sector3Time">
    <ReferenceLine y={avg_lap_time_result_ms[0].avg_sector_3} label="Average Time in ms"/>
</LineChart>

2. How are the results per session during the weekend?

For the practice round,

```sql practice_lap_time_result
select 
    CAST(LapTime[7:] as time) as LapTime, 
from 
    src_laps
where 
    filename like '%data\\20230402%' and filename like '%Practice%' and DriverNumber = 44 and LapTime not null
```
```sql practice_lap_time_result_ms
select 
    ROW_NUMBER() OVER () AS index,
    CAST(SUBSTRING(LapTime, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(LapTime, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(LapTime, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(LapTime, 10, 3) AS INTEGER) AS LapTime,  
from 
    ${practice_lap_time_result}
```
```sql avg_practice_lap_time_result_ms
select 
    avg(LapTime) as avg_lap_time,
from 
    ${practice_lap_time_result_ms}
```
<LineChart data={practice_lap_time_result_ms} x=index y=LapTime >
    <ReferenceLine y={avg_practice_lap_time_result_ms[0].avg_lap_time} label="Average Time in ms"/>
</LineChart>

For the qualifying round,

```sql qualify_lap_time_result
select 
    CAST(LapTime[7:] as time) as LapTime, 
from 
    src_laps
where 
    filename like '%data\\20230402%' and filename like '%Qualifying%' and DriverNumber = 44 and LapTime not null
```
```sql qualify_lap_time_result_ms
select 
    ROW_NUMBER() OVER () AS index,
    CAST(SUBSTRING(LapTime, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(LapTime, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(LapTime, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(LapTime, 10, 3) AS INTEGER) AS LapTime,  
from 
    ${qualify_lap_time_result}
```
```sql avg_qualify_lap_time_result_ms
select 
    avg(LapTime) as avg_lap_time,
from 
    ${qualify_lap_time_result_ms}
```
<LineChart data={qualify_lap_time_result_ms} x=index y=LapTime >
    <ReferenceLine y={avg_qualify_lap_time_result_ms[0].avg_lap_time} label="Average Time in ms"/>
</LineChart>

For the race, by comparing the result

```sql race_lap_time_result
select 
    CAST(LapTime[7:] as time) as LapTime, 
from 
    src_laps
where 
    filename like '%data\\20230402%' and filename like '%Race%' and DriverNumber = 44 and LapTime not null
```
```sql race_lap_time_result_ms
select 
    ROW_NUMBER() OVER () AS index,
    CAST(SUBSTRING(LapTime, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(LapTime, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(LapTime, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(LapTime, 10, 3) AS INTEGER) AS LapTime,    
from 
    ${race_lap_time_result}
```
```sql avg_race_lap_time_result_ms
select 
    avg(LapTime) as avg_lap_time,
from 
    ${race_lap_time_result_ms}
```
<LineChart data={race_lap_time_result_ms} x=index y=LapTime >
    <ReferenceLine y={avg_race_lap_time_result_ms[0].avg_lap_time} label="Average Time in ms"/>
</LineChart>

3. Is the driver (and team) consistent with their pit stop times?

```sql pit_in_time_result
select
    CAST(PitInTime[7:] as time) as PitInTime 
from 
    src_laps
where 
    filename like '%data\\20230402%' and DriverNumber = 44 and PitInTime not null 
```

```sql pit_out_time_result
select
    CAST(PitOutTime[7:] as time) as PitOutTime,  
from 
    src_laps
where 
    filename like '%data\\20230402%' and DriverNumber = 44 and PitOutTime not null
```

```sql pit_in_time_result_ms
select  
    ROW_NUMBER() over () as index,
    CAST(SUBSTRING(PitInTime, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(PitInTime, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(PitInTime, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(PitInTime, 10, 3) AS INTEGER) AS PitInTime
from 
    ${pit_in_time_result}
```

```sql pit_out_time_result_ms
select
    ROW_NUMBER() over () as index,
    CAST(SUBSTRING(PitOutTime, 1, 2) AS INTEGER) * 3600000 + 
    CAST(SUBSTRING(PitOutTime, 4, 2) AS INTEGER) * 60000 +
    CAST(SUBSTRING(PitOutTime, 7, 2) AS INTEGER) * 1000 + 
    CAST(SUBSTRING(PitOutTime, 10, 3) AS INTEGER) AS PitOutTime
from 
    ${pit_out_time_result}
```

Scatter plot: Plot the pit stop times (PitInTime vs. PitOutTime) to visualize the consistency and distribution of pit stop durations.
<ScatterPlot 
    data={pit_in_time_result_ms} 
    x=index 
    y=PitInTime
/>

<ScatterPlot 
    data={pit_out_time_result_ms} 
    x=index 
    y=PitOutTime
/>

Histogram: Show the distribution of pit stop times to identify any patterns or outliers in the pit stop durations.
<Histogram
    data={pit_in_time_result_ms} 
    x=PitInTime 
/>

<Histogram
    data={pit_out_time_result_ms} 
    x=PitOutTime 
/>

4. How does the position of the driver change during the race?
```sql position_results
select lap, position
from src_lap_times
where date = '2023-04-02' and carNumber = 44
```

Line chart: Plot the positions of the driver over time (laps) to visualize the changes in position throughout the race.
<LineChart data={position_results} x=lap y=position />

Bar chart: Show the distribution of positions over time to understand the frequency of position changes.
<BarChart  data={position_results} x=lap y=position />

```sql average_position
select avg(position) as avg_position from ${position_results}
```

The position of the racer seems good and in the lead with mostly stay at the 2nd position and some laps even get to 1st position. The {fmt(average_position[0].avg_position,'#,##0.0')} is relative high which show that the driver very persistent with the 2nd place in the race.

5. Has the maximum speed been improved during the weekend?

```sql speed_result
select 
    ROW_NUMBER() over () as index, RPM, Speed
from 
    src_car_data
where 
    filename like '%data\\20230402%' and DriverNumber = 44 and Speed <> 0
```
Line chart: Display the speed of the driver over time to identify any trends or improvements in maximum speed.
<LineChart data={speed_result} x=index y=Speed />

Histogram: Show the distribution of speed measurements to understand the range and variability of speeds recorded during the weekend.
<Histogram 
    data={speed_result} 
    x=Speed 
    xAxisTitle="Speed in km/h"
/>