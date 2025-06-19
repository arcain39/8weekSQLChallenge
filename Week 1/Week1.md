# Case Study #1 - Danny's Diner

Below is the relationship diagram and tables for this dataset:

![image](https://github.com/user-attachments/assets/7e1b254a-5d54-48d3-9e81-99c4126f1712)


**Table 1: Sales**
![Image](https://github.com/user-attachments/assets/ebf00e62-89d7-408a-92d2-6c37586fc379)


**Table 2: Menu**

![image](https://github.com/user-attachments/assets/68fda723-afd6-4c30-b9e2-f4bc58453d15)


**Table 3: Members**

![image](https://github.com/user-attachments/assets/50ea2cc0-68b1-4e8e-b6aa-d715259932a2)


**1. What is the total amount each customer spent at the restaurant?**

``` SQL 
SELECT customer_id, SUM(price) AS total_amount_spent
FROM sales s 
JOIN menu me 
ON s.product_id = me.product_id
GROUP BY customer_id
ORDER BY total_amount_spent DESC
```
- Join the two tables by product_id   
- Sum the price to find the total amount   
- Group by customer_id to find the total sum for each customer   
- Order by total desc to display totals from highest to lowest    

![image](https://github.com/user-attachments/assets/c77445ca-db23-4f00-991f-8b3a409a74fc)


**2. How many days has each customer visited the restaurant?**

```SQL
SELECT customer_id, COUNT(DISTINCT order_date) AS visit_amount
FROM sales 
GROUP BY customer_id
```



