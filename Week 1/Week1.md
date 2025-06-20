# Case Study #1 - Danny's Diner

All data and questions can be found at [Dannys Website.](https://8weeksqlchallenge.com/case-study-1/)
All solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) using POSTGRE SQL.

Below is the relationship diagram and tables for this dataset:

![image](https://github.com/user-attachments/assets/7e1b254a-5d54-48d3-9e81-99c4126f1712)


**Table 1: Sales**
![Image](https://github.com/user-attachments/assets/ebf00e62-89d7-408a-92d2-6c37586fc379)


**Table 2: Menu**

![image](https://github.com/user-attachments/assets/68fda723-afd6-4c30-b9e2-f4bc58453d15)


**Table 3: Members**

![image](https://github.com/user-attachments/assets/50ea2cc0-68b1-4e8e-b6aa-d715259932a2)


Below is the CTE table created from which all answers from this practice are generated:

``` SQL
WITH CTE1 AS (SELECT s.*, me.product_name, me.price, mem.join_date
FROM sales s
FULL OUTER JOIN menu me ON 
s.product_id = me.product_id
FULL OUTER JOIN members mem ON 
s.customer_id = mem.customer_id)
```

| customer_id	| order_date	| product_id	| product_name	| price	| join_date
| ---- | ---- | ---- | ---- | ---- | ---- | 
| A |	2021-01-07 | 2 |	curry |	15	| 2021-01-07 |
| A |	2021-01-11 | 3 | ramen	| 12	| 2021-01-07 |
| A |	2021-01-11 | 3 |	ramen |	12	| 2021-01-07 |
| A |	2021-01-10 | 3 | ramen	 | 12	| 2021-01-07 |
| A | 2021-01-01 |	1 | sushi |	10	| 2021-01-07 |
| A | 2021-01-01 |	2 |	curry |	15	| 2021-01-07 |
| B	| 2021-01-04 |	1 |	sushi |	10	| 2021-01-09 |
| B	| 2021-01-11 |	1 |	sushi	| 10	| 2021-01-09 |
| B	| 2021-01-01 |	2	| curry |	15 |	2021-01-09 |
| B	| 2021-01-02 |	2	| curry |	15	| 2021-01-09 |
| B	| 2021-01-16 |	3	| ramen |	12 |	2021-01-09 |
| B	| 2021-02-01 |	3	| ramen |	12	| 2021-01-09 |
| C	| 2021-01-01 |	3	| ramen	| 12	| null |
| C	| 2021-01-01 |	3	| ramen	| 12 |	null |
| C	| 2021-01-07 |	3	| ramen |	12 |	null |



**1. What is the total amount each customer spent at the restaurant?**

``` SQL 
SELECT customer_id, SUM(price) AS total_amount_spent
FROM cte1
GROUP BY customer_id
ORDER BY total_amount_spent DESC
``` 
- Sum the price to find the total amount   
- Group by customer_id to find the total sum for each customer   
- Order by total desc to display totals from highest to lowest    

| customer_id |	total_amount_spent |
| --- | --- |
| A |	76 |
| B |	74 |
| C |	36 |


**2. How many days has each customer visited the restaurant?**

```SQL
SELECT customer_id, COUNT(DISTINCT order_date) AS visit_amount
FROM sales 
GROUP BY customer_id
```

- Counted distinct dates so each day is only counted once
- Group by customer_id to find total day count for each customer

| customer_id |	visit_amount |
| --- | ---|
| A |	4 |
| B |	6 |
| C |	2 |


**3. What was the first item from the menu purchased by each customer?**

```SQL
SELECT DISTINCT customer_id, product_name
FROM
(SELECT DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank1,
customer_id, order_date, product_name
FROM cte1
ORDER BY order_date) cte2
WHERE rank1 = 1
```

- Cte2 table was made to assign a rank to each order based on customer_id and when the order was placed
- Used dense rank to assign a consecutive ranks without gaps, this ranked the first order placed by each customer as 1
-  Partition by was used to group the order dates by customer_id
-  Selected distinct data where ranking = 1

| customer_id |	product_name |
| --- | --- |
| A |	curry |
| A |	sushi |
| B |	curry |
| C |	ramen |


**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```SQL
SELECT COUNT(*) AS times_ordered, product_name
FROM cte1
GROUP BY product_name 
ORDER BY times_ordered DESC
LIMIT 1
```
- Counted how many times each prodcut was ordered  
- Ordered by desc so highest order was first  
- Limit 1 to only grab top answer  

| times_ordered |	product_name |
| --- | --- |
| 8 |	ramen |



**5. Which item was the most popular for each customer?**

```SQL
, top_orders AS
(SELECT DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(*) DESC) AS top_rank,
customer_id, COUNT(*) AS times_ordered, product_name
FROM cte1
GROUP BY customer_id, product_name 
ORDER BY times_ordered DESC)

SELECT customer_id, product_name, times_ordered
FROM top_orders
WHERE top_rank = 1
ORDER BY customer_id
```

- Created top_orders table to add an extra column counting how many times each order was made
- Used dense rank to rank orders by grouping data by customer_id and ordered the data based on how many rows were counted, used desc so top orders would be ranked first
- Selected top ranking orders from rop_orders table

| customer_id |	product_name |	times_ordered |
| --- | --- | ---|
| A |	ramen |	3 |
| B |	curry |	2 |
| B |	ramen |	2 |
| B |	sushi |	2 |
| C |	ramen |	3 |


**6. Which item was purchased first by the customer after they became a member?**

```SQL
, orders_table AS (SELECT *, DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) as order_rank
FROM cte1
WHERE join_date < order_date)

SELECT customer_id, product_name
FROM orders_table
WHERE order_rank = 1
```

- Created orders_table to create a ranking system
- Dense_rank to rank each row, using parition by to group data by customer_id and ranking them by earliest order date
- used join_date less than order_date to only select data that was ordered after the member join date
- Only selected rows with ranking of 1 (which was the earliest date placed before the join date)


| customer_id |	product_name |
| --- | --- |
| A	 | ramen |
| B |	sushi |



**7. Which item was purchased just before the customer became a member?**

```SQL
, orders_table AS (SELECT customer_id, product_name,
DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date desc) as order_rank
FROM cte1
WHERE join_date > order_date)

SELECT customer_id, product_name
from orders_table
WHERE order_rank = 1
```

- Similar code to question 6, only changes include ordering the dates by descending order (most recent date is ranked first) and selecting rows where the join date is after the ordered date
- By selecting only the rows where the join date is after the order date, ranking the dates by descending we will have the most recent order date before the customer became a memeber



**8. What is the total items and amount spent for each member before they became a member?**

```SQL
SELECT customer_id, COUNT(*) AS total_items, SUM(price) AS amount_spent
FROM cte1
WHERE join_date > order_date
GROUP BY customer_id
ORDER BY customer_id
```




