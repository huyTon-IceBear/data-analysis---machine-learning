# Hamilton Performance

This page use to answer the question: **How did one of our drivers do last weekends?**

Driver name: Lewis Hamilton
Car number: 44
Race: Australian Grand Prix
Race Date: 2023-04-02
Race Round: 2

To answer the question above, let's validate the performance of the driver through sub questions.
1. Do the results improve over time?
This question can be answer by analyze the finishing position and points earned in each race during the weekends

```sql hamilton_lap_results
select lap, position, time, milliseconds
from src_lap_times
where date = '2023-04-02' and carNumber = 44
```
<LineChart data={hamilton_lap_results} x=lap y=position />

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

2. How are the results per session during the weekend?
For the practice round,

For the qualifying round,

For the race,

3. Is the driver (and team) consistent with their pitstop times?

4. How does the position of the driver change during the race?

5. Has the maximum speed been improved during the weekend?

