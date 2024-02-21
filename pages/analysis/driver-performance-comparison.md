# Driver Performance Comparison

This page use to answer the question: **What is the difference in performance between the two drives in our team?**

between 2 racer of team Mercedes:
- Lewis Hamilton (car number 44) 
- George Russell (car number 63)

First let's examine the result of each racer between 2 racer

```sql avarage_position
select carNumber, avg(position) as average_position, avg(milliseconds) as finish_time_in_milliseconds
from src_lap_times
where date = '2023-04-02' 
group by carNumber
order by average_position asc
```