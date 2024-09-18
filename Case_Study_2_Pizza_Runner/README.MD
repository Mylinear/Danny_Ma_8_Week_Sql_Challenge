# Data Cleaning

## Customer Orders Table

In the `customer_orders` table, the `exclusions` and `extras` columns contain both empty strings and the word "null" stored as a string. Our goal is to replace these with actual `NULL` values. To achieve this, we use the `CASE WHEN` structure.

```sql
SELECT order_id,
       customer_id,
       pizza_id,
       CASE WHEN exclusions = 'null' OR exclusions = '' THEN NULL ELSE exclusions END AS exclusions,
       CASE WHEN extras = 'null' OR extras = '' THEN NULL ELSE extras END AS extras,
       order_time
FROM customer_orders;
``` 
output 
![image](https://github.com/user-attachments/assets/c5775798-a74d-4bc9-85c7-bf79f6b170f9)

Once I verified that the output matches the desired table structure, I proceed to update the table using the `UPDATE` and `SET` commands.

```sql
UPDATE customer_orders
SET exclusions = CASE
    WHEN exclusions = 'null' OR exclusions = '' THEN NULL ELSE exclusions
END; 

UPDATE customer_orders
SET extras = CASE
    WHEN extras = 'null' OR extras = '' THEN NULL ELSE extras
END;
```

## Runner Orders Table

When inspecting the `runner_orders` table, I noticed that the `pickup_time` column contains the word "null" stored as a string. The `distance` column also has "null" strings and includes the "km" suffix, despite it being a numeric field. We need to remove the "km" part and convert the column to a numeric data type. Lastly, the `cancellation` column also has string "null" values that need to be converted to `NULL`.

```sql
UPDATE runner_orders
SET pickup_time = CASE
    WHEN pickup_time = 'null' THEN NULL ELSE pickup_time
END,
    distance = CASE
    WHEN distance = 'null' THEN NULL
    WHEN distance LIKE '%km' THEN TRIM(distance, 'km') ELSE distance
END,
    duration = CASE
    WHEN duration LIKE 'null' THEN NULL
    WHEN duration LIKE '%mins' THEN TRIM(duration, 'mins')
    WHEN duration LIKE '%minute' THEN TRIM(duration, 'minute')
    WHEN duration LIKE '%minutes' THEN TRIM(duration, 'minutes') ELSE duration
END,
    cancellation = CASE
    WHEN cancellation = 'null' OR cancellation = '' THEN NULL ELSE cancellation
END;
```

### Changing Data Types

To allow for time-based operations, I converted the `pickup_time` column to a `timestamp` format.

```sql
ALTER TABLE runner_orders
ALTER COLUMN pickup_time
TYPE timestamp 
USING to_timestamp(pickup_time, 'YY/MM/DD HH24:MI');
```

Next, I converted the `distance` and `duration` columns to numeric values to enable mathematical operations.

```sql
ALTER TABLE runner_orders
ALTER COLUMN distance
TYPE float  -- You can also use numeric or real here
USING distance::double precision;
```

```sql
ALTER TABLE runner_orders
ALTER COLUMN duration
TYPE int  
USING duration::integer;
```
