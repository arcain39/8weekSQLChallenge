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
