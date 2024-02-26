# Hamilton Performance

To address the query "How did one of our drivers perform last weekend?", we'll focus on the performance of **Lewis Hamilton**.

Driver Details:
- Name: Lewis Hamilton
- Car Number: 44
- Race: Australian Grand Prix
- Race Date: April 2, 2023

In order to comprehensively assess the driver's performance, let's delve into specific sub-questions that will provide insights into various aspects of his racing performance throughout the weekend.

1. Do the results improve over time?

The driver's performance shows **significant improvement** from the practice session to the qualifying session and ultimately to the final race session. To substantiate this improvement, **lap time data and sector times** were analyzed. The table below presents the lap time data for Lewis Hamilton over the weekend, with all times converted to milliseconds.

```sql lap_time_result
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
    CAST(SUBSTRING(Sector3Time, 10, 3) AS INTEGER) AS Sector3Time
from 
    (
    select 
        LapNumber,
        CAST(LapTime[7:] as time) as LapTime, 
        CAST(Sector1Time[7:] as time) as Sector1Time,
        CAST(Sector2Time[7:] as time) as Sector2Time,
        CAST(Sector3Time[7:] as time) as Sector3Time
    from 
        src_laps
    where 
        filename like '%data\\20230402%' and DriverNumber = 44 and LapTime not null
    ) as lap_time_result
```

To gauge the improvement, average lap time and sector time tables were calculated for the driver's performance, serving as benchmarks to assess whether the driver's performance improved over time.

```sql avg_lap_time_result_ms
select 
    avg(LapTime) as avg_lap_time,
    avg(Sector1Time) as avg_sector_1,
    avg(Sector2Time) as avg_sector_2,
    avg(Sector3Time) as avg_sector_3,  
from 
    ${lap_time_result}
```
## Reasoning:
Analyzing the lap time data in detail, it becomes evident that the driver's performance evolves over the course of the weekend. Here's a breakdown:

- Practice Session:
    Initially, during the early races (around 30 laps), the driver seems to be acclimatizing to the vehicle, resulting in varying lap times. This phase typically involves adjustments and familiarization with the track and vehicle dynamics.
- Stable Performance:
    Subsequently, from approximately races 40 to 80, the driver demonstrates a stable performance, with lap times consistently below the average. This indicates that the driver has adapted well to the track conditions and optimized their driving techniques, resulting in improved performance.
- Final Race Session:
    - Remarkably, during the final race session, the driver delivers an exceptional performance, with most lap times consistently below the average. However, there are a few instances where the lap times deviate from the average, which could be attributed to unforeseen factors such as track conditions or strategic decisions during the race.
    - It's worth noting that anomalies in data, particularly during the final rounds, may occur due to hot restart moments or other race incidents, which could affect the consistency of lap times.

## Conclusion:
In conclusion, Lewis Hamilton demonstrates a remarkable improvement throughout the weekend, as evidenced by the consistent decrease in lap times and their ability to maintain performance levels below the average. While there are minor fluctuations, particularly in sector 1 times, sectors 2 and 3 remain relatively consistent with minimal changes observed. This suggests a progressive enhancement in the driver's performance and a strong adaptation to the race conditions over time.

Here are the charts depicting the lap time and sector time trends:

### Lap Time Chart
<LineChart data={lap_time_result} x=index y="LapTime">
    <ReferenceLine y={avg_lap_time_result_ms[0].avg_lap_time} label="Average Time in ms"/>
</LineChart>

### Sector 1 Time Chart
<LineChart data={lap_time_result} x=index y="Sector1Time">
    <ReferenceLine y={avg_lap_time_result_ms[0].avg_sector_1} label="Average Time in ms"/>
</LineChart>

### Sector 2 Time Chart
<LineChart data={lap_time_result} x=index y="Sector2Time">
    <ReferenceLine y={avg_lap_time_result_ms[0].avg_sector_2} label="Average Time in ms"/>
</LineChart>

### Sector 3 Time Chart
<LineChart data={lap_time_result} x=index y="Sector3Time">
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