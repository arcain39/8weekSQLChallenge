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

```SQL
SELECT customer_id, SUM (CASE
WHEN cancellation ISNULL AND exclusions ISNULL AND extras ISNULL THEN 1 ELSE 0 END) AS orders_no_change,
SUM (CASE WHEN  (exclusions NOTNULL OR extras NOTNULL) AND cancellation ISNULL THEN 1 ELSE 0 END) AS modified_orders
FROM customer_runner_orders
GROUP BY customer_id
ORDER BY customer_id
```


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




**1B. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

```SQL
SELECT DENSE_RANK() OVER(ORDER BY week) AS week_number, COUNT(*) AS runners_signed_up, week
FROM (SELECT runner_id, DATE_TRUNC('WEEK',(registration_date + interval '4 day')) - interval '3 day'  AS week
FROM runners) cte2 
GROUP BY week
```

| week_number |	runners_signed_up |	week |
| --- | --- | ---|
| 1 |	2 |	2021-01-01 00:00:00 |
| 2 |	1 |	2021-01-08 00:00:00 |
| 3 |	1 |	2021-01-15 00:00:00 |


**2B. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```SQL
SELECT runner_id, ROUND(AVG(pickup_order_time)) AS average_TIME
FROM (SELECT runner_id, order_id, EXTRACT(MINUTE FROM pickup_time - order_time) AS pickup_order_time
FROM customer_runner_orders
WHERE pickup_time NOTNULL
GROUP BY runner_id, order_id, pickup_order_time
order by order_id) cte2
GROUP BY runner_id
ORDER BY runner_id
```

| runner_id |	average_time |
| --- | --- |
| 1 |	14 |
| 2 |	20 |
| 3 |	10 |



**3B. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

```SQL
, pizza_time AS (SELECT COUNT(order_id) AS number_of_pizzas, AVG(EXTRACT(MINUTE FROM pickup_time - order_time)) AS prep_time
FROM customer_runner_orders
WHERE pickup_time NOTNULL
GROUP BY order_id)

SELECT number_of_pizzas, AVG(prep_time) AS prep_time
FROM pizza_time
GROUP BY number_of_pizzas
```

| number_of_pizzas |	prep_time |
| --- | --- |
| 1 |	12 |
| 2 |	18 |
| 3 |	29 |


**4B. What was the average distance travelled for each customer?**

```SQL
SELECT customer_id, ROUND(AVG(distance),2) AS avg_distance
FROM customer_runner_orders
GROUP BY customer_id
ORDER BY customer_id
```

| customer_id |	avg_distance |
| --- | --- |
| 101 |	20.00 |
| 102 |	16.73 |
| 103 |	23.40 |
| 104 |	10.00 |
| 105 |	25.00 |



**5B. What was the difference between the longest and shortest delivery times for all orders?**

```SQL
SELECT MAX(duration) - MIN(duration) AS diff_between_durations
FROM customer_runner_orders
```

| diff_between_durations |
| --- |
| 30 |



**6B. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

```SQL
SELECT runner_id, ROUND(AVG(km_per_hr),2) AS avg_km_per_hr
FROM(
SELECT runner_id, order_id, (AVG(distance/duration)*60) AS km_per_hr
FROM customer_runner_orders
WHERE cancellation ISNULL
GROUP BY runner_id, order_id
ORDER BY runner_id) cte2
GROUP BY runner_id
```

| runner_id |	avg_km_per_hr |
| --- | --- |
| 1 |	45.54 |
| 2 |	62.90 |
| 3 |	40.00 |



**7B. What is the successful delivery percentage for each runner?**

```SQL
, cancellation_perc AS (SELECT order_id, runner_id, cancellation
FROM customer_runner_orders
GROUP BY order_id, runner_id, cancellation)

SELECT runner_id,  ROUND(100.0*SUM(CASE 
WHEN cancellation ISNULL THEN 1 ELSE 0 END) / COUNT(*),2) AS successfull_delivery_percentage
FROM cancellation_perc
GROUP BY runner_id
ORDER BY runner_id
```

| runner_id |	successfull_delivery_percentage |
| --- | --- |
| 1 |	100.00 |
| 2 |	75.00 |
| 3 |	50.00 |




## Creation of CTE table pizza_ingredients for part C, including data from pizza_names, pizza_toppings, and pizza_recipes

```SQL
, pizza_ingredients AS (SELECT pn.*, ingredient, topping_name
FROM pizza_names pn
JOIN 
(SELECT pizza_id, unnest(STRING_TO_ARRAY(toppings, ','))::INTEGER AS ingredient
FROM pizza_recipes) i
ON pn.pizza_id = i.pizza_id
JOIN pizza_toppings pt
ON pt.topping_id = i.ingredient 
ORDER BY pizza_id)
```

| pizza_id |	pizza_name |	ingredient |	topping_name |
| --- | --- | --- | --- |
| 1 |	Meatlovers |	2 |	BBQ Sauce |
| 1 |	Meatlovers |	8 |	Pepperoni |
| 1 |	Meatlovers |	4 |	Cheese |
| 1 |	Meatlovers |	10 |	Salami |
| 1 |	Meatlovers |	5 |	Chicken |
| 1 |	Meatlovers |	1 |	Bacon |
| 1 |	Meatlovers |	6 |	Mushrooms |
| 1 |	Meatlovers |	3 |	Beef |
| 2 |	Vegetarian |	12 |	Tomato Sauce |
| 2 |	Vegetarian |	4 |	Cheese |
| 2 |	Vegetarian |	6 |	Mushrooms |
| 2 |	Vegetarian |	7 |	Onions |
| 2 |	Vegetarian |	9 |	Peppers |
| 2 |	Vegetarian |	11 |	Tomatoes |



**1C. What are the standard ingredients for each pizza?**

```SQL
SELECT pizza_name, STRING_AGG(topping_name,', ') AS ingredients
FROM pizza_ingredients
GROUP BY pizza_name
```

| pizza_name |	ingredients |
| --- | --- |
| Meatlovers |	BBQ Sauce, Pepperoni, Cheese, Salami, Chicken, Bacon, Mushrooms, Beef |
| Vegetarian |	Tomato Sauce, Cheese, Mushrooms, Onions, Peppers, Tomatoes |



**2C. What was the most commonly added extra?**

```SQL
, unnest_customer_orders AS (SELECT order_id, unnest(STRING_TO_ARRAY(exclusions, ',')) AS exclusions, unnest(STRING_TO_ARRAY(extras, ',')) AS extras
FROM customer_RUNNER_ORDERS)

SELECT topping_NAME AS most_ordered_extra
FROM
(SELECT extras::INTEGER, COUNT(*) AS extras_ordered
FROM unnest_customer_orders
WHERE extras NOTNULL
GROUP BY extras
ORDER BY extras_ordered DESC
LIMIT 1) e
JOIN pizza_toppings pt
ON extras = topping_id
```

| most_ordered_extra |
| --- |
| Bacon |



**3C. What was the most common exclusion?**

```SQL
, unnest_customer_orders AS (SELECT order_id, unnest(STRING_TO_ARRAY(exclusions, ',')) AS exclusions, unnest(STRING_TO_ARRAY(extras, ',')) AS extras
FROM customer_RUNNER_ORDERS)

SELECT topping_NAME AS most_ordered_extra
FROM
(SELECT exclusions::INTEGER, COUNT(*) AS exclusions_ordered
FROM unnest_customer_orders
WHERE exclusions NOTNULL
GROUP BY exclusions
ORDER BY exclusions_ordered DESC
LIMIT 1) e
JOIN pizza_toppings pt
ON exclusions = topping_id
```


| most_ordered_extra |
| --- |
| Cheese |



**4C. Generate an order item for each record in the customers_orders table in the format of one of the following:**

![image](https://github.com/user-attachments/assets/c501ba05-14fd-4aad-bb7b-e99beefb2076)


```SQL
, order_rank AS (SELECT ROW_NUMBER() OVER (ORDER BY order_id) AS order_number, order_id, customer_id, pizza_id, exclusions, extras
FROM updated_customer_orders uco)

, unnest_order_rank AS (SELECT order_number, order_id, unnest(STRING_TO_ARRAY(exclusions, ','))::INTEGER AS unnest_exclusions, unnest(STRING_TO_ARRAY(extras, ','))::INTEGER AS unnest_extras
FROM order_rank)


, order_name AS (SELECT o.order_number,  STRING_AGG(pt.topping_name, ', ') AS exclusions_name, STRING_AGG(ptt.topping_name, ', ') AS extras_name
FROM unnest_order_rank u
LEFT JOIN pizza_toppings pt
ON u.unnest_exclusions = pt.topping_id
LEFT JOIN pizza_toppings ptt
ON u.unnest_extras = ptt.topping_id
RIGHT JOIN order_rank o
ON o.order_number = u.order_number
GROUP BY o.order_number
ORDER BY o.order_number)

SELECT order_id, 
CASE WHEN exclusions_name ISNULL AND extras_name ISNULL THEN pizza_name
WHEN extras_name ISNULL THEN CONCAT(pizza_name, ' - Exclude ', exclusions_name)
WHEN exclusions_name ISNULL THEN CONCAT(pizza_name, ' - Extra ', extras_name)
ELSE CONCAT(pizza_name, ' - Exclude ', exclusions_name, ' - Extra ', extras_name)
END AS orders
FROM order_rank o
JOIN order_name n
ON o.order_number = n.order_number 
LEFT JOIN pizza_names pn 
ON o.pizza_id = pn.pizza_id
ORDER BY o.order_number
```


| order_id |	orders |
| --- | --- |
| 1 |	Meatlovers |
| 2 |	Meatlovers |
| 3 |	Meatlovers |
| 3 |	Vegetarian |
| 4 |	Meatlovers - Exclude Cheese |
| 4 |	Meatlovers - Exclude Cheese |
| 4 |	Vegetarian - Exclude Cheese |
| 5 |	Meatlovers - Extra Bacon |
| 6 |	Vegetarian |
| 7 |	Vegetarian - Extra Bacon |
| 8 |	Meatlovers |
| 9 |	Meatlovers - Exclude Cheese - Extra Bacon, Chicken |
| 10 |	Meatlovers |
| 10 |	Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |




**5C.Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"**

```SQL


, order_rank AS (SELECT ROW_NUMBER() OVER (ORDER BY order_id) AS order_number, order_id, customer_id, pizza_id, exclusions, extras
FROM updated_customer_orders uco)

, unnest_order_rank AS (SELECT order_number, order_id, unnest(STRING_TO_ARRAY(exclusions, ','))::INTEGER AS unnest_exclusions, unnest(STRING_TO_ARRAY(extras, ','))::INTEGER AS unnest_extras
FROM order_rank)

,order_with_extras AS (SELECT o.order_id, o.pizza_id, order_number, 
CASE WHEN extras NOTNULL THEN CONCAT(toppings, ',', extras)
ELSE toppings END AS new_order
FROM order_rank o
RIGHT JOIN pizza_recipes pr
ON pr.pizza_id  = o.pizza_id)

,exclusions AS (SELECT DISTINCT u.order_number, u.unnest_exclusions
FROM unnest_order_rank u
JOIN unnest_order_rank un
ON u.order_number = un.order_number
WHERE u.unnest_exclusions NOTNULL)

, final_table AS (SELECT i.*, topping_name, unnest_exclusions
FROM 
(SELECT *, unnest(STRING_TO_ARRAY(new_order, ','))::INTEGER AS new_ingredient
FROM order_with_extras) i 
JOIN pizza_toppings pt 
ON pt.topping_id = i.new_ingredient
LEFT JOIN exclusions e 
ON e.order_number = i.order_number AND e.unnest_exclusions = i.new_ingredient
WHERE e.order_number ISNULL)

, two_times as (SELECT order_number, STRING_AGG(two_times,', ') AS extras_name
FROM (SELECT DISTINCT order_number,
CASE WHEN COUNT(new_ingredient) = 2 THEN CONCAT('2X: ', topping_name) END AS two_times
FROM final_table
GROUP BY order_number, topping_name) tt
GROUP BY order_number)

, plain_orders AS (SELECT f.order_number, STRING_AGG(topping_name, ',') AS ingredient_list
FROM final_table f
LEFT JOIN unnest_order_rank uo
ON f.order_number = uo.order_number AND f.new_ingredient = uo.unnest_extras
WHERE uo.unnest_extras ISNULL
GROUP BY f.order_number)

SELECT po.order_number, 
CASE WHEN extras_name ISNULL THEN ingredient_list
ELSE CONCAT(extras_name, ', ', ingredient_list) END AS ingredient_ist
FROM plain_orders po
JOIN two_times tt
ON po.order_number = tt.order_number
```

| order_number |	ingredient_ist |
| --- | --- |
| 1 |	Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 2 |	Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 3 |	Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 4 |	Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce |
| 5 |	Bacon,BBQ Sauce,Beef,Chicken,Mushrooms,Pepperoni,Salami |
| 6 |	Bacon,BBQ Sauce,Beef,Chicken,Mushrooms,Pepperoni,Salami |
| 7 |	Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce |
| 8 |	2X: Bacon, BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 9 |	Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce |
| 10 |	Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce |
| 11 |	Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 12 |	2X: Bacon, 2X: Chicken, BBQ Sauce,Beef,Mushrooms,Pepperoni,Salami |
| 13 |	Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 14 |	2X: Bacon, 2X: Cheese, Beef,Chicken,Pepperoni,Salami |



**6C. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**

-Using final table CTE from above question 

``` SQL
SELECT topping_name, COUNT(new_ingredient) AS ingredients_ordered
FROM final_table ft
GROUP BY topping_name
ORDER BY ingredients_ordered DESC
```


| topping_name |	ingredients_ordered |
| --- | --- |
| Bacon |	14 |
| Mushrooms |	13 | 
| Chicken |	11 |
| Cheese |	11 |
| Pepperoni |	10 |
| Salami |	10 |
| Beef |	10 |
| BBQ Sauce |	9 |
| Tomato Sauce |	4 |
| Onions |	4 |
| Tomatoes |	4 |
| Peppers |	4 |





**1C. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

```SQL
SELECT SUM(CASE WHEN pizza_id = 1 THEN 12
ELSE 10 END) AS total_in_sales
FROM  customer_runner_orders
WHERE cancellation ISNULL
```


| total_in_sales |
| --- |
| 138 |



**2C. What if there was an additional $1 charge for any pizza extras?
Add cheese is $1 extra**

```SQL
SELECT SUM(CASE WHEN pizza_id = 1 THEN 12
ELSE 10 END)  + 
(SELECT SUM(CASE WHEN 
unnest_extras = 4 THEN 2 ELSE 1 END) AS extra_charge
FROM unnest_order_rank
WHERE unnest_extras NOTNULL) AS total_with_extras
FROM  customer_runner_orders
WHERE cancellation IS NULL
```


| total_with_extras |
| --- |
| 145 |



**3C. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**

```SQL

CREATE TABLE runner_ratings (
  runner_id INTEGER,
  order_id INTEGER,
  rating INTEGER,
  feedback TEXT);
  
  
INSERT INTO runner_ratings (order_id, runner_id, rating, feedback) values
  
 (1,1,2, 'Pizza took a long time to arrive and was cold'),
 (2,1,4, 'Good pizza, delivery driver looked like my grandson'),
(3,1,5, 'Fast delivery, pizza was delicious'),
(4,2,2, NULL),
(5,3,5, 'Good pizza, arrived in no time!'),
(7,2,3,'Food was meh'),
(8,2,4, 'Yum'),
(10,1,5, NULL);

SELECT *
FROM runner_ratings
```


| runner_id |	order_id |	rating	feedback |
| --- | --- | --- |
| 1 |	1 |	2 |	Pizza took a long time to arrive and was cold |
| 1 |	2 |	4 |	Good pizza, delivery driver looked like my grandson |
| 1 |	3 |	5 |	Fast delivery, pizza was delicious |
| 2 |	4 |	2 |	null |
| 3 |	5 |	5 |	Good pizza, arrived in no time! |
| 2 |	7 |	3 |	Food was meh |
| 2 |	8 |	4 |	Yum |
| 1 |	10 |	5 |	null |



**4C. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas**

```SQL
SELECT customer_id, cro.order_id, cro.runner_id, rating, order_time, pickup_time, ROUND(AVG(EXTRACT(MINUTE FROM pickup_time - order_time))) AS avg_time_for_pickup, duration AS duration_in_mins, ROUND(AVG((distance/duration)*60)) AS avg_km_per_hr, COUNT(*) as num_of_pizzas
FROM customer_runner_orders cro
JOIN runner_ratings rr
ON rr.order_id = cro.order_id
GROUP BY customer_id, cro.order_id, cro.runner_id, rating, order_time, pickup_time, duration_in_mins
ORDER BY customer_id, cro.order_id
```

| customer_id |	order_id |	runner_id |	rating |	order_time |	pickup_time |	avg_time_for_pickup |	duration_in_mins |	avg_km_per_hr |	num_of_pizzas |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 101 |	1 |	1 |	2 |	2020-01-01 18:05:02 |	2020-01-01 18:15:34 |	10 |	32 |	38 |	1 |
| 101 |	2 |	1 |	4 |	2020-01-01 19:00:52 |	2020-01-01 19:10:54 |	10 |	27 |	44 |	1 |
| 102 |	3 |	1 |	5 |	2020-01-02 23:51:23 |	2020-01-03 00:12:37 |	21 |	20 |	40 |	2 |
| 102 |	8 |	2 |	4 |	2020-01-09 23:54:33 |	2020-01-10 00:15:02 |	20 |	15 |	94 |	1 |
| 103 |	4 |	2 |	2 |	2020-01-04 13:23:46 |	2020-01-04 13:53:03 |	29 |	40 |	35 |	3 |
| 104 |	5 |	3 |	5 |	2020-01-08 21:00:29 |	2020-01-08 21:10:57 |	10 |	15 |	40 |	1 |
| 104 |	10 |	1 |	5 |	2020-01-11 18:34:49 |	2020-01-11 18:50:20 |	15 |	10 |	60 |	2 |
| 105 |	7 |	2 |	3 |	2020-01-08 21:20:29 |	2020-01-08 21:30:45 |	10 |	25 |	60 |	1 |



**5C. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**

```SQL
SELECT SUM(CASE WHEN pizza_id = 1 THEN 12
WHEN pizza_id = 2 THEN 10 END) - (SELECT SUM(distance*.3) AS cost_per_km
FROM 
(SELECT DISTINCT order_id, runner_id, distance
FROM customer_runner_orders) i) AS money_left_over
FROM customer_runner_orders
WHERE cancellation ISNULL
```


| money_left_over |
| --- |
| 94.44 |
