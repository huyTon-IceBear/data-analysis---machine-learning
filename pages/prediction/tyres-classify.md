# Type Performance Classify
To address the query:

"During the race drivers can choose between 5 type of tyres (compounds). Soft, Medium, Hard, Intermediate and Wet. What is the best tyre given certain weather conditions (AirTemp, Humidity, and so on..) on a certain track?"

## Introduction
In this analysis, I aim to determine the best tire choice for specific weather conditions and track characteristics. I utilize the K-Nearest Neighbors (KNN) algorithm for classification. The performance of each tire type is assessed based on lap time measurements, aiming to find the optimal tire selection for given circumstances.

## Data Collection
- First, I extract weather conditions from the data to create a table containing static weather information for each race event.
    ```sql weathers_data
    SELECT 
        wd.EventDate,
        wd.EventName,
        wd.Avg_AirTemp,
        wd.Avg_Humidity,
        wd.Avg_Pressure,
        wd.Avg_TrackTemp,
        wd.Avg_WindDirection,
        wd.Avg_WindSpeed,
        c.CircuitName
    FROM (
        SELECT
            CAST(filename[7:10] || '-' || filename[11:12] || '-' || filename[13:14] AS date) AS EventDate,
            CASE 
                WHEN filename LIKE '%Practice_1%' THEN filename[15:instr(filename, 'Practice_1') - 2]
                WHEN filename LIKE '%Practice_2%' THEN filename[15:instr(filename, 'Practice_2') - 2]
                WHEN filename LIKE '%Practice_3%' THEN filename[15:instr(filename, 'Practice_3') - 2]
                WHEN filename LIKE '%Qualifying%' THEN filename[15:instr(filename, 'Qualifying') - 2]
                WHEN filename LIKE '%Sprint%' THEN filename[15:instr(filename, 'Sprint') - 2]
                WHEN filename LIKE '%Race%' THEN filename[15:instr(filename, 'Race') - 2]
            END AS EventName,
            AVG(COALESCE(AirTemp, 0)) AS Avg_AirTemp,
            AVG(COALESCE(Humidity, 0)) AS Avg_Humidity,
            AVG(COALESCE(Pressure, 0)) AS Avg_Pressure,
            AVG(COALESCE(TrackTemp, 0)) AS Avg_TrackTemp,
            AVG(COALESCE(WindDirection, 0)) AS Avg_WindDirection,
            AVG(COALESCE(WindSpeed, 0)) AS Avg_WindSpeed
        FROM src_weather
        GROUP BY EventDate, EventName
    ) AS wd
    JOIN (
        SELECT            
            date,
            REPLACE("name:1", ' ', '_') AS CircuitName
        FROM src_driver_results
        WHERE year >= 2020
        GROUP BY date, CircuitName
    ) AS c ON wd.EventDate = c.date
    ORDER BY wd.EventDate
    ```
- Next, I gather lap time data based on the tire compound used during each race event. 
    ```sql laps_data
    select
        CAST(filename[7:10] || '-' || filename[11:12] || '-' || filename[13:14] as date) as EventDate,
        case 
                when filename like '%Practice_1%' then filename[15:instr(filename, 'Practice_1') - 2]
                when filename like '%Practice_2%' then filename[15:instr(filename, 'Practice_2') - 2]
                when filename like '%Practice_3%' then filename[15:instr(filename, 'Practice_3') - 2]
                when filename like '%Qualifying%' then filename[15:instr(filename, 'Qualifying') - 2]
                when filename like '%Sprint%' then filename[15:instr(filename, 'Sprint') - 2]
                when filename like '%Race%' then filename[15:instr(filename, 'Race') - 2]
            end as EventName,
        AVG( CAST(SUBSTRING(CAST(LapTime[7:] as time), 1, 2) AS INTEGER) * 3600000 + 
                        CAST(SUBSTRING(CAST(LapTime[7:] as time), 4, 2) AS INTEGER) * 60000 +
                        CAST(SUBSTRING(CAST(LapTime[7:] as time), 7, 2) AS INTEGER) * 1000 ) AS Avg_LapTime_in_ms,
        Compound,
    from src_laps
    WHERE Compound is not null AND Compound <> 'test_unknown' AND Compound <> 'unknown' and LapTime is not null
    GROUP BY EventDate, EventName, Compound
    Order by EventDate
    ```

## Data Preparation
- To determine the best compound for each race based on the average minimum lap time, I execute the following query
    ```sql best_compound_for_track
    SELECT 
        ld.EventDate,
        ld.EventName,
        ld.Avg_LapTime_in_ms,
        ld.Compound
    FROM (
        SELECT 
            EventDate,
            EventName,
            MIN(Avg_LapTime_in_ms) AS Min_Avg_LapTime
        FROM ${laps_data} 
        GROUP BY EventDate, EventName
    ) AS min_lap_times
    JOIN ${laps_data} AS ld ON min_lap_times.EventDate = ld.EventDate 
        AND min_lap_times.EventName = ld.EventName 
        AND min_lap_times.Min_Avg_LapTime = ld.Avg_LapTime_in_ms
    ```
- Then combine this data with the weather and track information. This combined dataset serves as a basis for validating the results of the KNN classification algorithm when determining the optimal value of K
    ```sql data_table
    SELECT 
        ROW_NUMBER() OVER () AS index,
        WD.CircuitName,
        WD.Avg_AirTemp,
        WD.Avg_Humidity,
        WD.Avg_Pressure,
        WD.Avg_TrackTemp,
        WD.Avg_WindDirection,
        WD.Avg_WindSpeed,
        LD.Avg_LapTime_in_ms,
        LD.Compound as Best_compound
    FROM ${weathers_data} as WD
    LEFT JOIN ${best_compound_for_track} as LD ON WD.EventName = LD.EventName and WD.EventDate = LD.EventDate
    order by CircuitName
    ```
## KNN Algorithm
### Data Preparation For Train & Testing
To prepare the data for training and testing, I partition it into two sets: 70% for training and 30% for testing.

First I extract 70% of the data for training
```sql train_data
select * from ${data_table} where index <=47
```

Next, I extract 30% of the data for testing:
```sql test_data
select * from ${data_table} where index >47
```

### Normalizing data
Before implementing the algorithm, I normalize the data using min-max scaling to ensure that all values fall within the range of 0.0 to 1.0.

- To determine the scale of min, max, I form a table
    ```sql min_max_value
    SELECT 
        MIN(Avg_AirTemp) AS min_AirTemp,
        MAX(Avg_AirTemp) AS max_AirTemp,
        MIN(Avg_Humidity) AS min_Humidity,
        MAX(Avg_Humidity) AS max_Humidity,
        MIN(Avg_Pressure) AS min_Pressure,
        MAX(Avg_Pressure) AS max_Pressure,
        MIN(Avg_TrackTemp) AS min_TrackTemp,
        MAX(Avg_TrackTemp) AS max_TrackTemp,
        MIN(Avg_WindDirection) AS min_WindDirection,
        MAX(Avg_WindDirection) AS max_WindDirection,
        MIN(Avg_WindSpeed) AS min_WindSpeed,
        MAX(Avg_WindSpeed) AS max_WindSpeed,
        FROM ${data_table}
    ```
- The formula for min-max scaling is:

    `normalized_x = (x - min_x) / (max_x - min_x)`

- Then I normalize the training data using the min-max scaling:
    ```sql norm_train_table
    SELECT 
        (tb.Avg_AirTemp - mm.min_AirTemp) / (mm.max_AirTemp - mm.min_AirTemp) AS Avg_AirTemp,
        (tb.Avg_Humidity - mm.min_Humidity) / (mm.max_Humidity - mm.min_Humidity) AS Avg_Humidity,
        (tb.Avg_Pressure - mm.min_Pressure) / (mm.max_Pressure - mm.min_Pressure) AS Avg_Pressure,
        (tb.Avg_TrackTemp - mm.min_TrackTemp) / (mm.max_TrackTemp - mm.min_TrackTemp) AS Avg_TrackTemp,
        (tb.Avg_WindDirection - mm.min_WindDirection) / (mm.max_WindDirection - mm.min_WindDirection) AS Avg_WindDirection,
        (tb.Avg_WindSpeed - mm.min_WindSpeed) / (mm.max_WindSpeed - mm.min_WindSpeed) AS Avg_WindSpeed,
        tb.Best_compound
    FROM ${train_data} as tb
    CROSS JOIN ${min_max_value} as mm
    ```

- Similarly, I normalize the test data using the min-max scaling:
    ```sql norm_test_table
    SELECT 
        ROW_NUMBER() OVER () AS index,
        (te.Avg_AirTemp - mm.min_AirTemp) / (mm.max_AirTemp - mm.min_AirTemp) AS Avg_AirTemp,
        (te.Avg_Humidity - mm.min_Humidity) / (mm.max_Humidity - mm.min_Humidity) AS Avg_Humidity,
        (te.Avg_Pressure - mm.min_Pressure) / (mm.max_Pressure - mm.min_Pressure) AS Avg_Pressure,
        (te.Avg_TrackTemp - mm.min_TrackTemp) / (mm.max_TrackTemp - mm.min_TrackTemp) AS Avg_TrackTemp,
        (te.Avg_WindDirection - mm.min_WindDirection) / (mm.max_WindDirection - mm.min_WindDirection) AS Avg_WindDirection,
        (te.Avg_WindSpeed - mm.min_WindSpeed) / (mm.max_WindSpeed - mm.min_WindSpeed) AS Avg_WindSpeed,
        te.Best_compound
    FROM ${test_data} as te
    CROSS JOIN ${min_max_value} as mm
    ```

### Calculate Euclidean Distances and Determine K
To implement the KNN algorithm effectively, I need to compute the Euclidean distances between data points and determine the optimal value of K, which represents the number of nearest neighbors considered for classification. To do this:

- I start by joining each row from the normalized training data with the first row of the normalized test data. This allows me to compare the features of the test data with those of the training data, facilitating the computation of Euclidean distances and the determination of the most suitable value for K.
    ```sql first_row_norm_test_data
    select
        *
    from ${norm_test_table}
    Limit 1
    ```

- Then I start calculating the Euclidean distances between each data point in the test set and all data points in the training set. This helps in measuring the similarity between different data points. I try different values of K, ranging from 3 to 10, to find the value that provides the highest classification accuracy. K represents the number of nearest neighbors that will be considered when classifying a new data point.

- With K = 3, here is the distance
    ```sql distance_first_K_3
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound as predicted,
        fr.Best_compound as expected
    from 
        ${norm_train_table} AS nt
    CROSS JOIN 
        ${first_row_norm_test_data} as fr
    ORDER by euclidean_distance
    Limit 3
    ```
- With K = 4, here is the distance
    ```sql distance_first_K_4
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound as predicted,
        fr.Best_compound as expected
    from 
        ${norm_train_table} AS nt
    CROSS JOIN 
        ${first_row_norm_test_data} as fr
    ORDER by euclidean_distance
    Limit 4
    ```
- With K = 5, here is the distance
    ```sql distance_first_K_5
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound as predicted,
        fr.Best_compound as expected
    from 
        ${norm_train_table} AS nt
    CROSS JOIN 
        ${first_row_norm_test_data} as fr
    ORDER by euclidean_distance
    Limit 5
    ```
- With K = 6, here is the distance
    ```sql distance_first_K_6
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound as predicted,
        fr.Best_compound as expected
    from 
        ${norm_train_table} AS nt
    CROSS JOIN 
        ${first_row_norm_test_data} as fr
    ORDER by euclidean_distance
    Limit 6
    ```
    - With K = 7, here is the distance
    ```sql distance_first_K_7
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound as predicted,
        fr.Best_compound as expected
    from 
        ${norm_train_table} AS nt
    CROSS JOIN 
        ${first_row_norm_test_data} as fr
    ORDER by euclidean_distance
    Limit 7
    ```

- With K = 8, here is the distance
    ```sql distance_first_K_8
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound as predicted,
        fr.Best_compound as expected
    from 
        ${norm_train_table} AS nt
    CROSS JOIN 
        ${first_row_norm_test_data} as fr
        ORDER by euclidean_distance

    Limit 8
    ```

- With K = 9, here is the distance
    ```sql distance_first_K_9
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound as predicted,
        fr.Best_compound as expected
    from 
        ${norm_train_table} AS nt
    CROSS JOIN 
        ${first_row_norm_test_data} as fr
        ORDER by euclidean_distance

    Limit 9
    ```

- With K = 10, here is the distance
    ```sql distance_first_K_10
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound as predicted,
        fr.Best_compound as expected
    from 
        ${norm_train_table} AS nt
    CROSS JOIN 
        ${first_row_norm_test_data} as fr
        ORDER by euclidean_distance
    Limit 10
    ```

- Here, I'm computing the accuracy of the KNN algorithm for different values of K (number of nearest neighbors). Each query calculates the accuracy for a specific value of K by comparing the predicted values with the expected values. The accuracy is computed as the number of correct predictions divided by the total number of predictions. These results are then combined using UNION ALL to create a unified view of accuracy for different K values. Finally, I filter out any accuracy values that are equal to 0, as they do not contribute to the evaluation.
    ```sql k_accuracy_results
    WITH knn_results AS (
        -- Query for k = 3
        SELECT
            3 AS k,
            predicted,
            SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) / 3.0 AS accuracy
        FROM ${distance_first_K_3}
        GROUP BY 
            predicted

        UNION ALL

        -- Query for k = 4
        SELECT
            4 AS k,
            predicted,
            SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) / 4.0 AS accuracy
        FROM ${distance_first_K_4}
        GROUP BY 
            predicted

        UNION ALL

        -- Query for k = 5
        SELECT
            5 AS k,
            predicted,
            SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) / 5.0 AS accuracy
        FROM ${distance_first_K_5}
        GROUP BY 
            predicted

        UNION ALL

        -- Query for k = 6
        SELECT
            6 AS k,
            predicted,
            SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) / 6.0 AS accuracy
        FROM ${distance_first_K_6}
        GROUP BY 
            predicted

        UNION ALL

        -- Query for k = 7
        SELECT
            7 AS k,
            predicted,
            SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) / 7.0 AS accuracy
        FROM ${distance_first_K_7}
        GROUP BY 
            predicted

        UNION ALL

        -- Query for k = 8
        SELECT
            8 AS k,
            predicted,
            SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) / 8.0 AS accuracy
        FROM ${distance_first_K_8}
        GROUP BY 
            predicted
        
        UNION ALL

        -- Query for k = 9
        SELECT
            9 AS k,
            predicted,
            SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) / 9.0 AS accuracy
        FROM ${distance_first_K_9}
        GROUP BY 
            predicted

        UNION ALL

        -- Query for k = 9
        SELECT
            10 AS k,
            predicted,
            SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) / 10.0 AS accuracy
        FROM ${distance_first_K_10}
        GROUP BY 
            predicted
    )

    SELECT k, accuracy 
    FROM knn_results
    where accuracy > 0
    GROUP BY k, accuracy
    ORDER BY k
    ```
- I use a line chart visualizing the accuracy of the KNN algorithm for different values of K is provided below
    <LineChart data={k_accuracy_results} x=k y=accuracy/>

- Finally, I'm selecting the K value with the highest accuracy from the KNN results obtained above. This helps in identifying the optimal K value for the KNN algorithm. 
    ```sql best_k_value
    SELECT k, accuracy
    FROM ${k_accuracy_results}
    ORDER BY accuracy DESC
    LIMIT 1
    ```
After all, k = 5 have the highest accuracy is 0.6.

### Validate the classification Knn
In this section, I'm validating the classification by applying the KNN algorithm to the second row of the test data. The Euclidean distance is calculated between the features of the second row in the test data and those of the training data.The KNN algorithm then predicts the compound type based on the nearest neighbors (in this case, the 5 nearest neighbors). Finally, the predicted compound type is compared with the expected compound type, and the accuracy of the prediction is determined.

```sql knn_result_second_row_test
WITH knn_results AS (
    SELECT 
        SQRT(
            POWER(nt.Avg_AirTemp - fr.Avg_AirTemp, 2) +
            POWER(nt.Avg_Humidity - fr.Avg_Humidity, 2) +
            POWER(nt.Avg_Pressure - fr.Avg_Pressure, 2) +
            POWER(nt.Avg_TrackTemp - fr.Avg_TrackTemp, 2) +
            POWER(nt.Avg_WindDirection - fr.Avg_WindDirection, 2) +
            POWER(nt.Avg_WindSpeed - fr.Avg_WindSpeed, 2) 
        ) AS euclidean_distance,
        nt.Best_compound AS predicted,
        fr.Best_compound AS expected
    FROM 
        ${norm_train_table} AS nt
    CROSS JOIN 
        (SELECT * FROM ${norm_test_table} WHERE index = 2) AS fr
    ORDER BY 
        euclidean_distance
    LIMIT 5
)
SELECT 
    predicted,
    COUNT(*) AS frequency,
    COUNT(*) / 5.0 AS accuracy
FROM 
    knn_results
GROUP BY 
    predicted
ORDER BY 
    frequency DESC
LIMIT 1
```