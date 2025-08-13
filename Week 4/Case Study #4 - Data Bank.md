# CASE STUDY 4 Data Bank

All data and questions can be found at [Dannys Website.](https://8weeksqlchallenge.com/case-study-1/)
All solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) using POSTGRE SQL.


<img width="1080" height="1080" alt="image" src="https://github.com/user-attachments/assets/24dbbac8-f6e1-41a7-aa15-709201c11434" />



Relationship Diagram:

<img width="698" height="290" alt="image" src="https://github.com/user-attachments/assets/e7a072b1-733b-46ae-a5b3-43914aeeb2b0" />


Table 1: Regions


<img width="247" height="260" alt="image" src="https://github.com/user-attachments/assets/dd1f2f6b-f08c-44e3-8467-f9bace1d1c1e" />


Table 2: Customer Nodes


<img width="572" height="482" alt="image" src="https://github.com/user-attachments/assets/f713b290-f32d-4233-af54-99365692354c" />


Table 3: Customer Transactions


<img width="481" height="483" alt="image" src="https://github.com/user-attachments/assets/ab567179-21c0-4695-b438-42f944b09dd9" />





**1A. How many unique nodes are there on the Data Bank system?**

```SQL
SELECT COUNT(DISTINCT node_id) AS unique_nodes 
FROM customer_nodes
```

| unique_nodes |
| --- |
| 5 |



**2A. What is the number of nodes per region?**

```SQL
SELECT region_id, COUNT(DISTINCT node_id) AS nodes_per_region
FROM customer_nodes
GROUP BY region_id
```


| region_id |	nodes_per_region |
| --- | --- |
| 1 |	5 |
| 2 |	5 |
| 3 |	5 |
| 4 |	5 |
| 5 |	5 |


**3A. How many customers are allocated to each region?**

```SQL
SELECT r.region_id, region_name, COUNT(DISTINCT  customer_id) AS nodes_per_region
FROM customer_nodes cn
JOIN regions r
ON cn.region_id = r.region_id
GROUP BY region_name, r.region_id
ORDER BY r.region_id
```

| region_id |	region_name |	nodes_per_region |
| --- | --- | --- |
| 1 |	Australia |	110 |
| 2 |	America |	105 |
| 3 |	Africa |	102 |
| 4 |	Asia |	95 |
| 5 |	Europe |	88 |


**4A. How many days on average are customers reallocated to a different node?**

```SQL
WITH cte1 AS (
SELECT * ,LEAD(node_id) OVER (ORDER BY customer_id, start_date) AS new_node,
LEAD(region_id) OVER (ORDER BY customer_id, start_date) AS new_region, RANK () OVER (PARTITION BY customer_id ORDER BY customer_id, start_date) AS rank_order,
RANK () OVER (PARTITION BY customer_id ORDER BY customer_id, start_date DESC) AS rev_rank_order
FROM customer_nodes)

, cte2 AS (SELECT *, CASE WHEN end_date = '9999-12-31' THEN 0
ELSE end_date - start_date + 1 END AS days
, CASE WHEN 
rank_order = 1 THEN 0 
WHEN rev_rank_order = 1 THEN 1
WHEN node_id = new_node AND region_id = region_id THEN 0
ELSE 1 END AS node_change
FROM cte1)

SELECT ROUND(SUM(days) / SUM(node_change)::decimal,2) AS average_node_change_in_days
FROM cte2
```

| average_node_change_in_days |
| --- |
| 18.82 |





**1B. What is the unique count and total amount for each transaction type?**

```
SELECT txn_type, COUNT(txn_type) AS number_of_transactions, SUM(txn_amount) AS total_transactions
FROM customer_transactions
GROUP BY txn_type
```

| txn_type |	number_of_transactions |	total_transactions |
| --- | --- | --- |
| purchase |	1617 |	806537 |
| deposit |	2671 |	1359168 |
| withdrawal |	1580 |	793003 |



**2B.What is the average total historical deposit counts and amounts for all customers?**

```SQL
SELECT ROUND(1.0*(SELECT COUNT(*) FROM customer_transactions WHERE txn_type = 'deposit') / COUNT (DISTINCT customer_id),2) AS historical_counts, ROUND(1.0*(SELECT SUM(txn_amount) FROM customer_transactions WHERE txn_type = 'deposit') / COUNT (DISTINCT customer_id),2) AS avg_degposit_amount
FROM customer_transactions
```


| historical_counts |	avg_degposit_amount |
| --- | --- |
| 5.34 |	2718.34 |



**3B.For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**

```SQL
WITH cte1 AS (SELECT customer_id, EXTRACT(month FROM txn_date) AS month_date,
CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END AS deposit,
CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END AS purchase,               
CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END AS withdrawal   
FROM customer_transactions
ORDER BY customer_id)

SELECT month_date, SUM(CASE WHEN num_deposit > 1 AND (num_purchase > 0 OR num_withdrawal > 0) THEN 1 ELSE 0 END) AS number_or_transactions
FROM (
SELECT customer_id, month_date, SUM(deposit) AS num_deposit, SUM(purchase) AS num_purchase, SUM(withdrawal) AS num_withdrawal
FROM cte1
GROUP BY customer_id, month_date
ORDER BY customer_id, month_date) cte
GROUP BY month_date
ORDER BY month_date
```


| month_date |	number_or_transactions |
| --- | --- |
| 1 |	168 |
| 2 |	181 |
| 3 |	192 |
| 4 |	70 |


**4B.What is the closing balance for each customer at the end of the month?**

```SQL
WITH cte1 AS (SELECT customer_id, EXTRACT(month FROM txn_date) AS month_date,
SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) AS deposit,
SUM(CASE WHEN txn_type = 'purchase' THEN txn_amount ELSE 0 END) AS purchase,    
SUM(CASE WHEN txn_type = 'withdrawal' THEN txn_amount ELSE 0 END) AS withdrawal   
FROM customer_transactions
GROUP BY customer_id, month_date
ORDER BY customer_id)

, cte2 AS (SELECT customer_id, month_date, LEAD(month_date) OVER (ORDER BY customer_id, month_date) AS next_month,
SUM(total_per_month) OVER (PARTITION BY customer_id ORDER BY customer_id, month_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS total
from (
SELECT customer_id, month_date, deposit - purchase - withdrawal AS total_per_month
FROM cte1
ORDER BY customer_id, month_date) cte)

SELECT customer_id, current_month, total 
FROM
(SELECT *, GENERATE_SERIES(month_date::INTEGER, CASE WHEN month_date = next_month - 1 OR month_date = 4 OR next_month ISNULL THEN month_date::INTEGER ELSE month_date::INTEGER + 1 END) AS current_month
FROM cte2)cte0
```

(Full data set not shown)

| customer_id |	current_month	total |
|--- | --- | --- |
| 1 |	1 |	312 |
| 1 |	2 |	312 |
| 1 |	3 |	-640 |
| 1 |	4 |	-640 |
| 2 |	1 |	549 |
| 2 |	2 |	549 |
| 2 |	3 |	610 |
| 2 |	4 |	610 |
| 3 |	1 |	144 |
| 3 |	2 |	-821 |
| 3 |	3 |	-1222 |
| 3 |	4 |	-729 |
| 4 |	1 |	848 |
| 4 |	2 |	848 |
| 4 |	3 |	655 |
| 4 |	4 |	655 |
| 5 |	1 |	954 |
| 5 |	2 |	954 |
| 5 |	3 |	-1923 |
| 5 |	4 |	-2413 |



**5B.What is the percentage of customers who increase their closing balance by more than 5%?**

First three tables are from previous answer

```SQL
WITH cte1 AS (SELECT customer_id, EXTRACT(month FROM txn_date) AS month_date,
SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) AS deposit,
SUM(CASE WHEN txn_type = 'purchase' THEN txn_amount ELSE 0 END) AS purchase,    
SUM(CASE WHEN txn_type = 'withdrawal' THEN txn_amount ELSE 0 END) AS withdrawal   
FROM customer_transactions
GROUP BY customer_id, month_date
ORDER BY customer_id)

, cte2 AS (SELECT customer_id, month_date, LEAD(month_date) OVER (ORDER BY customer_id, month_date) AS next_month,
SUM(total_per_month) OVER (PARTITION BY customer_id ORDER BY customer_id, month_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS total
from (
SELECT customer_id, month_date, deposit - purchase - withdrawal AS total_per_month
FROM cte1
ORDER BY customer_id, month_date) cte)

,cte3 AS (SELECT customer_id, current_month, total 
FROM
(SELECT *, GENERATE_SERIES(month_date::INTEGER, CASE WHEN month_date = next_month - 1 OR month_date = 4 OR next_month ISNULL THEN month_date::INTEGER ELSE month_date::INTEGER + 1 END) AS current_month
FROM cte2)cte0)

,cte4 AS (SELECT ct.customer_id, ct.current_month AS month_1, ct.total AS month_1_total, ctt.current_month AS month_4, ctt.total AS month_4_total
FROM cte3 ct
JOIN cte3 ctt ON 
ct.customer_id = ctt.customer_id AND ct.current_month = ctt.current_month - 3
WHERE ct.current_month = 1 OR ct.current_month = 4)

SELECT ROUND(100 * SUM (CASE WHEN month_4_total > month_1_total + (month_1_total * .05) THEN 1 ELSE 0 END) / COUNT(*),2) AS percent_total
FROM cte4
```

| percent_total |
| --- |
| 33.00 |
