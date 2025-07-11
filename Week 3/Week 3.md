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




