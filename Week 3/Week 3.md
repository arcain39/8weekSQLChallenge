# CASE STUDY 3 Foodie-Fi

All data and questions can be found at [Dannys Website.](https://8weeksqlchallenge.com/case-study-1/)
All solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) using POSTGRE SQL.


<img width="1080" height="1080" alt="image" src="https://github.com/user-attachments/assets/85e16328-c1c9-493a-97c4-7619c4ff15cd" />


Relationship Diagram:

<img width="698" height="290" alt="image" src="https://github.com/user-attachments/assets/e7a072b1-733b-46ae-a5b3-43914aeeb2b0" />


Table 1: Plans


<img width="277" height="242" alt="image" src="https://github.com/user-attachments/assets/f77be239-7f88-4eff-abf5-99fcef5a6034" />



Table 2: Subscriptions


<img width="323" height="862" alt="image" src="https://github.com/user-attachments/assets/a4e978e5-f74a-40f4-8556-5f252ac8f40a" />



Creation of CTE table (ONLY FIRST 10 ROWS SHOWN)

```SQL
WITH CTE AS (SELECT sub.*, plan_name, price 
FROM plans pl
JOIN subscriptions sub
ON pl.plan_id = sub.plan_id)
```



| customer_id |	plan_id |	start_date |	plan_name	price |
| --- | --- | --- | --- |
| 1 |	0 |	2020-08-01 |	trial |	0.00 |
| 1 |	1 |	2020-08-08 |	basic monthly |	9.90 |
| 2 |	0 |	2020-09-20 |	trial |	0.00 |
| 2 |	3 |	2020-09-27 |	pro annual |	199.00 |
| 3 |	0 |	2020-01-13 |	trial |	0.00 |
| 3 |	1 |	2020-01-20 |	basic monthly |	9.90 |
| 4 |	0 |	2020-01-17 |	trial |	0.00 |
| 4 |	1 |	2020-01-24 |	basic monthly |	9.90 |
| 4 |	4 |	2020-04-21 |	churn |	null |
| 5 |	0 |	2020-08-03 |	trial |	0.00 |




**1B. How many customers has Foodie-Fi ever had?**

```SQL
SELECT COUNT(DISTINCT customer_id) AS total_num_of_customers
FROM CTE
```

| total_num_of_customers |
| --- |
| 1000 |



**2B. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**

```SQL
SELECT EXTRACT(MONTH FROM start_date) as start_month,  COUNT(*) AS number_of_trail_plans
FROM CTE
WHERE plan_id = 0
GROUP BY start_month
ORDER BY start_month
```

| start_month |	number_of_trail_plans |
| --- | --- |
| 1 |	88 |
| 2 |	68 |
| 3 |	94 |
| 4 |	81 |
| 5 |	88 |
| 6 |	79 |
| 7 |	89 |
| 8 |	88 |
| 9 |	87 |
| 10 |	79 |
| 11 |	75 |
| 12 |	84 |



**3B. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**

```SQL
SELECT plan_name, COUNT(plan_id) AS start_date_values
FROM CTE
WHERE EXTRACT(YEAR FROM start_date) > 2020
GROUP BY plan_name
ORDER BY start_date_values
```

| plan_name |	start_date_values |
| --- | --- |
| basic monthly |	8 |
| pro monthly |	60 |
| pro annual |	63 |
| churn |	71 |



**4B. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

```SQL
SELECT COUNT(customer_id) AS churn_count, ROUND(100.0*COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions),1) AS churn_percentage
FROM CTE
WHERE plan_id = 4
```

| churn_count |	churn_percentage |
| --- | --- |
| 307 |	30.7 |



**5B. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

```SQL
, trial_churn_table AS(SELECT ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date) AS sub_order, *
FROM CTE)

SELECT ROUND(100.0 * SUM(CASE WHEN sub_order = 2 AND plan_id = 4 THEN 1 ELSE 0 END) / COUNT(DISTINCT customer_id), 1) AS percent_churned
FROM trial_churn_table
```

| percent_churned |
| --- |
| 9.2 |



**6B. What is the number and percentage of customer plans after their initial free trial?**

```SQL
, subscription_order_table AS (SELECT ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date) AS sub_order, *
FROM CTE)

SELECT plan_id, plan_name, COUNT(*) AS number_of_plans, 100 * COUNT(*) / (SELECT COUNT(*) FROM subscription_order_table WHERE sub_order = 2) AS percentage_of_plans
FROM subscription_order_table
WHERE sub_order = 2
GROUP BY plan_id, plan_name
```


| plan_id |	plan_name |	number_of_plans |	percentage_of_plans |
| --- | --- | --- | --- |
| 1 |	basic monthly |	546 |	54 |
| 2 |	pro monthly |	325 |	32 |
| 3 |	pro annual |	37 |	3 |
| 4 |	churn |	92 |	9 |



**7B. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

```SQL
, plan_values_date AS (SELECT ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date DESC) AS sub_order,*
FROM CTE
WHERE start_date <= '2020-12-31')

SELECT plan_id, plan_name, COUNT(plan_name) AS count_of_plans, ROUND(100.0*COUNT(plan_name)/(SELECT COUNT(DISTINCT customer_id)FROM subscriptions),1)
FROM plan_values_date 
WHERE sub_order = 1
GROUP BY plan_name, plan_id
ORDER BY plan_id
```

| plan_id |	plan_name |	count_of_plans |	round |
| --- | --- | --- | ---
| 0 |	trial |	19 |	1.9 |
| 1 |	basic monthly |	224 |	22.4 |
| 2 |	pro monthly |	326 |	32.6 |
| 3 |	pro annual |	195 |	19.5 |
| 4 |	churn |	236 |	23.6 |


**8B. How many customers have upgraded to an annual plan in 2020?**

```SQL
SELECT COUNT(*) AS customer_upgrade
FROM CTE
WHERE EXTRACT(YEAR FROM start_date) = 2020 AND plan_id = 3
```


| customer_upgrade |
| --- |
| 195 |

