# CASE STUDY 2 PIZZA RUNNER

All data and questions can be found at [Dannys Website.](https://8weeksqlchallenge.com/case-study-1/)
All solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) using POSTGRE SQL.

 


![image](https://github.com/user-attachments/assets/1d4b4f82-b066-482d-b2b0-8ff2ee036b67)


Below is the relationship diagram and tables for this dataset:

![image](https://github.com/user-attachments/assets/48b26cb4-4a1f-4989-976a-23a76fcda39c)

### Table 1: runners:    
![image](https://github.com/user-attachments/assets/a6872583-60f5-441a-8716-4b8a8ed40b34)

### Table 2: customer_orders:    

![image](https://github.com/user-attachments/assets/0b22a842-f3a8-4907-916a-8bed690f2036)

### Table 3: runner_orders:  

![image](https://github.com/user-attachments/assets/4d26d3a9-0317-4ca4-a089-a03172e36f42)

### Table 4: pizza_names:  

![image](https://github.com/user-attachments/assets/3429e4d1-fbbf-414a-bbbb-6239393e6b79)

### Table 5: pizza_recipes

![image](https://github.com/user-attachments/assets/6fade657-74c3-4c57-9f9b-b5e8175122eb)

### Table 6: pizza_toppings

![image](https://github.com/user-attachments/assets/afda148d-1490-4f1a-9c3a-98d9904dedce)


## Data Cleaning  

Below are the updated tabes:

### Table 2: updated_customer_orders

```SQL
WITH updated_customer_orders AS (SELECT order_id, customer_id, pizza_id,CASE
WHEN exclusions = '' OR exclusions ='null' THEN NULL
ELSE exclusions END AS exclusions,
CASE
WHEN extras = '' OR extras ='null' OR extras ISNULL THEN NULL 
ELSE extras END AS extras, order_time
FROM customer_orders)
```

| order_id |	customer_id |	pizza_id |	exclusions |	extras |	order_time |
| --- | --- | --- | --- | --- | --- |
| 1 |	101 |	1 |	null |	null |	2020-01-01 18:05:02 |
| 2 |	101 |	1 |	null |	null |	2020-01-01 19:00:52 |
| 3 |	102 |	1 |	null |	null |	2020-01-02 23:51:23 |
| 3 |	102 |	2 |	null |	null |	2020-01-02 23:51:23 |
| 4 |	103 |	1 |	4 |	null |	2020-01-04 13:23:46 |
| 4 |	103 |	1 |	4 |	null |	2020-01-04 13:23:46 |
| 4 | 103 |	2 |	4 |	null |	2020-01-04 13:23:46 |
| 5 |	104 |	1 |	null |	1 |	2020-01-08 21:00:29 |
| 6 | 101 |	2 |	null |	null |	2020-01-08 21:03:13 |
| 7 |	105 |	2 |	null |	1 |	2020-01-08 21:20:29 |
| 8 |	102 |	1 |	null |	null |	2020-01-09 23:54:33 |
| 9 |	103 |	1 |	4 |	1, 5 |	2020-01-10 11:22:59 |
| 10 |	104 |	1 |	null |	null |	2020-01-11 18:34:49 |
| 10 |	104 |	1 |	2, 6 |	1, 4 |	2020-01-11 18:34:49 |




### Table 3: updated_runner_orders

```SQL
,updated_runner_orders AS (SELECT order_id, runner_id,  CASE 
WHEN pickup_time = 'null' OR pickup_time = '' THEN NULL
ELSE pickup_time::TIMESTAMP END AS pickup_time,
CASE
WHEN distance = 'null' OR distance = '' THEN NULL
ELSE TRIM('km' FROM distance)::DECIMAL END AS distance,
CASE
WHEN duration = 'null' OR duration = ''  THEN NULL
WHEN duration LIKE '%mins' THEN TRIM ('mins' FROM duration)::INTEGER
WHEN duration LIKE '%minutes' THEN TRIM ('minutes' FROM duration)::INTEGER
WHEN duration LIKE '%minute' THEN TRIM ('minute' FROM duration)::INTEGER
ELSE duration::INTEGER
END AS duration,
CASE
WHEN cancellation = 'null' OR cancellation = '' THEN NULL
ELSE cancellation END AS cancellation
FROM runner_orders)
```


| order_id |	runner_id |	pickup_time |	distance |	duration |	cancellation |
| --- | --- | --- | --- | --- |---|
| 1 |	1 |	2020-01-01 18:15:34 |	20 |	32 |	null |
| 2 |	1 |	2020-01-01 19:10:54 |	20 |	27 |	null |
| 3 |	1 |	2020-01-03 00:12:37 |	13.4 |	20 |	null |
| 4 |	2 |	2020-01-04 13:53:03 |	23.4 |	40 |	null |
| 5 |	3 |	2020-01-08 21:10:57 |	10 |	15 |	null |
| 6 |	3 |	null |	null |	null |	Restaurant Cancellation |
| 7 |	2 |	2020-01-08 21:30:45 |	25 |	25 |	null | 
| 8 |	2 |	2020-01-10 00:15:02 |	23.4 |	15 |	null |
| 9 |	2 |	null |	null |	null |	Customer Cancellation |
| 10 |	1 |	2020-01-11 18:50:20 |	10 |	10 |	null |











