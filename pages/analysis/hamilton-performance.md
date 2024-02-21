# Driver Performance

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

```sql avarage_position
select carNumber, avg(position) as average_position, avg(milliseconds) as finish_time_in_milliseconds
from src_lap_times
where date = '2023-04-02' 
group by carNumber
order by average_position asc
```

The 

2. How are the results per session during the weekend?

3. Is the driver (and team) consistent with their pitstop times?

4. How does the position of the driver change during the race?

5. Has the maximum speed been improved during the weekend?

