# Point Prediction
To address the query:

"Will the driver score any points (finish as one of the first 10 drivers) or no points in the upcoming race?"

## Introduction
In this analysis, I aim to predict the likelihood of Lewis Hamilton, driver number 44, scoring points in future races using the Naive Bayes algorithm. This prediction is based on historical race data and employs the assumption of independence among the features.

## Data Collection
I begin by collecting all data relevant to Lewis Hamilton's racing performances since 2008. This includes race dates, circuit names, and the points scored by Hamilton in each race.

```sql driver_result_all_races
select
    date, 
    REPLACE("name:1", ' ', '_') AS CircuitName,
    points
from src_driver_results 
WHERE CarNumber = 44 and year >= 2008
order by date
```

## Data Preparation
Once the data is collected, I organize it into a format suitable for Naive Bayes classification. This involves creating a table with columns for race dates, circuit names, indicators for track familiarity, indicators for finishing in the top 10 in the last race, and a categorical variable indicating whether Hamilton scored points (Yes) or not (No).

```sql data_table_for_naive_bayes
select
    date,
    CircuitName,
    CASE 
        WHEN LAG(Date) OVER (PARTITION BY CircuitName ORDER BY Date) < Date THEN true
        ELSE false
    END AS "Track_familiar",
    CASE 
        WHEN LAG(points) OVER (PARTITION BY CircuitName ORDER BY Date) > 0 THEN true
        ELSE false
    END AS "Top_10_last_race",
    CASE 
        WHEN points > 0 THEN 'Yes'
        ELSE 'No'
    END AS "Top_10",
from ${driver_result_all_races} 
order by date
```

## Naive Bayes Prediction
With the data prepared, I  apply the Naive Bayes algorithm to predict the likelihood of Hamilton scoring points in future races. The upcoming race this season I will predict is the Azerbaijan Grand Prix, the race happen on 30 April of 2023 at Baku City Circuit.

### Data Study
To facilitate the prediction, I need to gather insights from previous races held at the Baku City Circuit.

```sql previous_data_baku_circuit
select * 
from ${data_table_for_naive_bayes} 
where CircuitName = 'Baku_City_Circuit' 
order by date desc
limit 1
```
Based on our analysis of the latest available race data, the most recent race at the Baku City Circuit occurred on 12/06/2022. In this race, Lewis Hamilton finished in the top 10, demonstrating familiarity with the track. Consequently, we focus our prediction on scenarios where Hamilton is familiar with the track and finished in the top 10 in the last race.

### Calculation
- Probability of finishing in the top 10 (P(Yes))
    ```sql top10_probability
    SELECT 
        COUNT(*) AS total_count,
        SUM(CASE WHEN Top_10 = 'Yes' THEN 1 ELSE 0 END) AS top_10_true_count,
        (top_10_true_count::FLOAT / total_count) AS probability
    FROM ${data_table_for_naive_bayes}
    ```

- Conditional probability of being familiar with the track given finishing in the top 10 (P(Track_familiar = true|Yes))
    ```sql top10_probability_with_familiar_track
    SELECT 
        COUNT(*) AS total_count,
        SUM(CASE WHEN Track_familiar = true AND Top_10 = 'Yes' THEN 1 ELSE 0 END) AS track_familiar_and_top_10_true_count,
        SUM(CASE WHEN Top_10 = 'Yes' THEN 1 ELSE 0 END) AS top_10_true_count,
        (track_familiar_and_top_10_true_count::FLOAT/ total_count) / (top_10_true_count::FLOAT / total_count) AS probability
    FROM ${data_table_for_naive_bayes}
    ```

- Conditional probability of finishing in the top 10 in the last race given finishing in the top 10 (P(Top_10_last_race = true|Yes))
    ```sql top10_probability_with_top_10_last_time
    SELECT 
        COUNT(*) AS total_count,
        SUM(CASE WHEN Top_10_last_race = true AND Top_10 = 'Yes' THEN 1 ELSE 0 END) AS top_10_last_race_and_top_10_true_count,
        SUM(CASE WHEN Top_10 = 'Yes' THEN 1 ELSE 0 END) AS top_10_true_count,
        (top_10_last_race_and_top_10_true_count::FLOAT/ total_count) /(top_10_true_count::FLOAT / total_count) AS probability
    FROM ${data_table_for_naive_bayes}
    ```

- Combined probability of finishing in the top 10 given being familiar with the track and finishing in the top 10 in the last race 

    `P(Yes|Track_familiar = true, Top_10_last_race= true) = P(Yes) * P(Track_familiar = true|Yes) * P(Top_10_last_race= true|Yes)`

    ```sql prediction_next_race
    SELECT 
        tp.probability * tf.probability * ttl.probability AS combined_probability
    FROM 
        ${top10_probability} as tp
    JOIN 
        ${top10_probability_with_familiar_track} as tf ON 1=1
    JOIN 
        ${top10_probability_with_top_10_last_time} as ttl ON 1=1;
    ```
