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


## CTE Table customer_runner_orders: 
```SQL
, customer_runner_orders AS (SELECT co.*, runner_id, pickup_time, distance, duration, cancellation, pn.pizza_name
FROM updated_customer_orders co 
JOIN updated_runner_orders ro 
ON co.order_id = ro.order_id
JOIN pizza_names pn
ON pn.pizza_id = co.pizza_id
ORDER BY order_id)
```
 
| order_id |	customer_id |	pizza_id |	exclusions |	extras |	order_time |	runner_id |	pickup_time |	distance |	duration |	cancellation |	pizza_name |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 |	101 |	1 |	null |	null |	2020-01-01 18:05:02 |	1 |	2020-01-01 18:15:34 |	20 |	32 |	null |	Meatlovers |
| 2 |	101 |	1 |	null |	null |	2020-01-01 19:00:52 |	1 |	2020-01-01 19:10:54 |	20 |	27 |	null |	Meatlovers |
| 3 |	102 |	1 |	null |	null |	2020-01-02 23:51:23 |	1 |	2020-01-03 00:12:37 |	13.4 |	20 |	null |	Meatlovers |
| 3 |	102 |	2 |	null |	null |	2020-01-02 23:51:23 |	1 |	2020-01-03 00:12:37 |	13.4 |	20 |	null |	Vegetarian |
| 4 |	103 |	2 |	4 |	null |	2020-01-04 13:23:46 |	2 |	2020-01-04 13:53:03 |	23.4 |	40 |	null |	Vegetarian |
| 4 |	103 |	1 |	4 |	null |	2020-01-04 13:23:46 |	2 |	2020-01-04 13:53:03 |	23.4 |	40 |	null |	Meatlovers |
| 4 |	103 |	1 |	4 |	null |	2020-01-04 13:23:46 |	2 |	2020-01-04 13:53:03 |	23.4 |	40 |	null |	Meatlovers |
| 5 |	104 |	1 |	null |	1 |	2020-01-08 21:00:29 |	3 |	2020-01-08 21:10:57 |	10 |	15 |	null |	Meatlovers |
| 6 |	101 |	2 |	null |	null |	2020-01-08 21:03:13 |	3 |	null |	null |	null |	Restaurant Cancellation |	Vegetarian |
| 7 |	105 |	2 |	null |	1 |	2020-01-08 21:20:29 |	2 |	2020-01-08 21:30:45 |	25 |	25 |	null	| Vegetarian |
| 8 |	102 |	1 |	null |	null |	2020-01-09 23:54:33 |	2 |	2020-01-10 00:15:02 |	23.4 |	15 |	null |	Meatlovers |
| 9 |	103 |	1 |	4 |	1, 5 |	2020-01-10 11:22:59 |	2 |	null |	null |	null |	Customer Cancellation |	Meatlovers |
| 10 |	104 |	1 |	2, 6 |	1, 4 |	2020-01-11 18:34:49 |	1 |	2020-01-11 18:50:20 |	10 |	10 |	null |	Meatlovers |
| 10 |	104 |	1 |	null |	null |	2020-01-11 18:34:49 |	1 |	2020-01-11 18:50:20 |	10 |	10 |	null |	Meatlovers |



**1A. How many pizzas were ordered?**

```SQL
SELECT COUNT(*) AS total_pizza_ordered
FROM customer_orders
```

| total_pizza_ordered |
| --- |
| 14 |


**2A. How many unique customer orders were made?**

```SQL
SELECT COUNT(*) AS unique_customer_orders
FROM (SELECT DISTINCT customer_id, order_id
FROM updated_customer_orders) cte2
```

| unique_customer_orders |
| --- |
| 10 |



**3A. How many successful orders were delivered by each runner?**

```SQL
SELECT runner_id, COUNT(pickup_time) AS successful_orders
FROM updated_runner_orders  
WHERE pickup_time NOTNULL
GROUP BY runner_id
ORDER BY runner_id
```

| runner_id |	successful_orders |
| --- | --- |
| 1 |	4 |
| 2 |	3 |
| 3 |	1 |



**4A. How many of each type of pizza was delivered?**

```SQL
SELECT pizza_name, COUNT(*) AS pizza_delivered
FROM customer_runner_orders 
WHERE cancellation ISNULL
GROUP BY pizza_name
```

| pizza_name |	pizza_delivered |
| --- | --- |
| Meatlovers |	9 |
| Vegetarian |	3 |



**5A. How many Vegetarian and Meatlovers were ordered by each customer?**


```SQL
SELECT customer_id, SUM(CASE
WHEN pizza_name = 'Vegetarian' THEN 1 ELSE 0 END) AS num_of_vegetarian,
SUM(CASE WHEN pizza_name = 'Meatlovers' THEN 1 ELSE 0 END) AS num_of_meatlovers
FROM customer_runner_orders c
GROUP BY customer_id
ORDER BY customer_id
```


| customer_id |	num_of_vegetarian |	num_of_meatlovers |
| --- | --- | ---|
| 101 |	1 |	2 |
| 102 |	1 |	2 |
| 103 |	1 |	3 |
| 104 |	0 |	3 |
| 105 |	1 |	0 |



**6A. What was the maximum number of pizzas delivered in a single order?**

```SQL
SELECT COUNT(*) AS max_delivered_pizzas
FROM customer_runner_orders
WHERE cancellation ISNULL
GROUP BY order_id
ORDER BY max_delivered_pizzas DESC
LIMIT 1
```

| max_delivered_pizzas |
| --- |
| 3 |




**7A. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

SELECT customer_id, SUM (CASE
WHEN cancellation ISNULL AND exclusions ISNULL AND extras ISNULL THEN 1 ELSE 0 END) AS orders_no_change,
SUM (CASE WHEN  (exclusions NOTNULL OR extras NOTNULL) AND cancellation ISNULL THEN 1 ELSE 0 END) AS modified_orders
FROM customer_runner_orders
GROUP BY customer_id
ORDER BY customer_id


| customer_id |	orders_no_change |	modified_orders |
| --- | --- | --- |
| 101 |	2 |	0 |
| 102 |	3 |	0 |
| 103 |	0 |	3 |
| 104 |	1 |	2 |
| 105 |	0 |	1 |



**8A. How many pizzas were delivered that had both exclusions and extras?**

```SQL
SELECT SUM (CASE
WHEN cancellation ISNULL AND exclusions NOTNULL AND extras NOTNULL THEN 1 ELSE 0 END) AS modified_pizzas_delivered
FROM customer_runner_orders
```


| modified_pizzas_delivered |
| --- |
| 1 |




**9A. What was the total volume of pizzas ordered for each hour of the day?**

```SQL
SELECT EXTRACT (HOUR FROM order_time) AS hour, COUNT(*) AS pizzas_ordered
FROM customer_runner_orders
GROUP BY hour
ORDER BY hour
```

| hour |	pizzas_ordered |
| --- | --- |
| 11 |	1 |
| 13 |	3 |
| 18 |	3 |
| 19 |	1 |
| 21 |	3 |
| 23 |	3 |




**10A. What was the volume of orders for each day of the week?**

```SQL
SELECT TO_CHAR(order_time, 'day') AS day, COUNT(*) AS pizzas_ordered
FROM customer_runner_orders
GROUP BY DAY
```

| day |	pizzas_ordered |
| --- | --- |
| wednesday |	5 |
| thursday |	3 |
| friday |	1 |
| saturday |	5 |







