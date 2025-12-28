# CASE STUDY 4 Data Mart

All data and questions can be found at [Dannys Website.](https://8weeksqlchallenge.com/case-study-1/)
All solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) using POSTGRE SQL.


<img width="1080" height="1080" alt="image" src="https://github.com/user-attachments/assets/744861ca-ef81-492a-a74b-dede64fd47c6" />




Table 1: Data Mart Weekly Sales (only 10 rows shown)



| week_date |	region |	platform |	segment |	customer_type |	transactions |	sales |
| --- | --- | --- | --- | --- | --- | --- |
| 9/9/20 |	OCEANIA |	Shopify |	C3 |	New |	610 |	110033.89 |
| 29/7/20 |	AFRICA |	Retail |	C1 |	New |	110692 |	3053771.19 |
| 22/7/20 |	EUROPE |	Shopify |	C4 |	Existing |	24 |	8101.54 |
| 13/5/20 |	AFRICA |	Shopify |	null |	Guest |	5287 |	1003301.37 |
| 24/7/19 |	ASIA |	Retail |	C1 |	New |	127342 |	3151780.41 |
| 10/7/19 |	CANADA |	Shopify |	F3 |	New |	51 |	8844.93 |
| 26/6/19 |	OCEANIA |	Retail |	C3 |	New |	152921 |	5551385.36 |
| 29/5/19 |	SOUTH AMERICA |	Shopify |	null |	New |	53 |	10056.2 |
| 22/8/18 |	AFRICA |	Retail |	null |	Existing |	31721 |	1718863.58 |
| 25/7/18 |	SOUTH AMERICA |	Retail |	null |	New |	2136 |	81757.91 |



##Part 1 - Also creation of CTE

```SQL
SELECT TO_DATE(week_date, 'DD-MM-YY') AS new_week_date, 
EXTRACT(week FROM TO_DATE(week_date, 'DD-MM-YY')) AS week_number,
EXTRACT(month FROM TO_DATE(week_date, 'DD-MM-YY')) AS month_number,
EXTRACT(year  FROM TO_DATE(week_date, 'DD-MM-YY')) AS year_number,
region, platform, segment,
CASE WHEN segment LIKE '%1%' THEN 'Young Adults'
WHEN segment LIKE '%2%' THEN 'Middle Aged'
WHEN segment LIKE '%3%' THEN 'Retirees'
WHEN segment LIKE '%4%' THEN 'Retirees' 
ELSE 'unknown' END AS age_brand,
CASE WHEN segment LIKE 'F%' THEN 'Families'
WHEN segment LIKE 'C%' THEN 'Couples' 
ELSE 'unknown' END AS demographic,
transactions, sales, ROUND(1.0*sales/transactions, 2) AS avg_transactions
FROM data_mart.weekly_sales
LIMIT 5;
```

| new_week_date |	week_number |	month_number |	year_number |	region |	platform |	segment |	age_brand |	demographic |	transactions |	sales |	avg_transactions |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2020-08-31 |	36 |	8 |	2020 |	ASIA |	Retail |	C3 |	Retirees |	Couples |	120631 |	3656163 |	30.31 |
| 2020-08-31 |	36 |	8 |	2020 |	ASIA |	Retail |	F1 |	Young Adults |	Families |	31574 |	996575 |	31.56 |
| 2020-08-31 |	36 |	8 |	2020 |	USA |	Retail |	null |	unknown |	unknown |	529151 |	16509610 |	31.20 |
| 2020-08-31 |	36 |	8 |	2020 |	EUROPE |	Retail |	C1 |	Young Adults |	Couples |	4517 |	141942 |	31.42 |
| 2020-08-31 |	36 |	8 |	2020 |	AFRICA |	Retail |	C2 |	Middle Aged |	Couples |	58046 |	1758388 |	30.29 |


**1B.What day of the week is used for each week_date value?**

```SQL
SELECT DISTINCT TO_CHAR(new_week_date, 'Day') AS day_of_week
FROM cte1
```

| day_of_week |
| --- |
| Monday |



**2B.What range of week numbers are missing from the dataset?**

```SQL
SELECT week_numbers AS missing_weeks
FROM cte1 ct
RIGHT JOIN (
SELECT *
FROM GENERATE_SERIES(1,52) AS week_numbers)ctt
ON ct.week_number = ctt.week_numbers
WHERE week_number ISNULL
ORDER BY week_numbers
```
Only first 10 rows shown

| missing_weeks |
| --- |
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |
| 6 |
| 7 |
| 8 |
| 9 |
| 10 |


**3B.How many total transactions were there for each year in the dataset?**

```SQL
SELECT year_number, SUM(transactions) AS trnsactions_per_year
FROM cte1
GROUP BY year_number
ORDER BY year_number
```


| year_number |	trnsactions_per_year |
| --- | --- |
| 2018 | 346406460 |
| 2019 | 365639285 |
| 2020 | 375813651 |



**4B.What is the total sales for each region for each month?**

```SQL
SELECT region, month_number, SUM(sales) AS total_sales
FROM cte1
GROUP BY region, month_number 
ORDER BY region, month_number
```

Only showing results for Africa

| region |	month_number |	total_sales |
| --- | --- | --- |
| AFRICA |	3 |	567767480 |
| AFRICA |	4 |	1911783504 |
| AFRICA |	5 |	1647244738 |
| AFRICA |	6 |	1767559760 |
| AFRICA |	7 |	1960219710 |
| AFRICA |	8 |	1809596890 |
| AFRICA |	9 |	276320987 |


*5B.What is the total count of transactions for each platform**

```SQL
SELECT platform, SUM(transactions) AS total_transactions
FROM cte1
GROUP BY platform
```

| platform |	total_transactions |
| --- | --- |
| Shopify |	5925169 |
| Retail |	1081934227 |


**6B.What is the percentage of sales for Retail vs Shopify for each month?**

```SQL
, cte2 AS (SELECT platform, month_number, SUM(sales) AS total_sales,
RANK() OVER(PARTITION BY month_number ORDER BY month_number, platform) AS rank_order
FROM cte1
GROUP BY platform, month_number
ORDER BY month_number)

SELECT ct.month_number, ROUND(100.0*ct.total_sales / (ct.total_sales + ctt.total_sales),2) AS percent_sales_retail,
ROUND(100.0*ctt.total_sales/ (ct.total_sales + ctt.total_sales),2) AS percent_sales_shopify
FROM cte2 ct
JOIN cte2 ctt ON 
ct.month_number = ctt.month_number AND ct.rank_order = ctt.rank_order - 1
```

| month_number |	percent_sales_retail |	percent_sales_shopify |
| --- | --- | --- |
| 3 |	97.54 |	2.46 |
| 4 |	97.59 |	2.41 |
| 5 |	97.30 |	2.70 |
| 6 |	97.27 |	2.73 |
| 7 |	97.29 |	2.71 |
| 8 |	97.08 |	2.92 |
| 9 |	97.38 |	2.62 |


**7B.What is the percentage of sales by demographic for each year in the dataset?**

```SQL
, cte2 AS (SELECT year_number, SUM(CASE WHEN demographic = 'Couples' THEN sales ELSE 0 END) AS Couples,
SUM(CASE WHEN demographic = 'Families' THEN sales ELSE 0 END) AS Families,
SUM(CASE WHEN demographic = 'unknown' THEN sales ELSE 0 END) AS unknown
FROM cte1 ct
GROUP BY year_number)

SELECT year_number, SUM(ROUND(100.0*couples / (couples + families + unknown),2)) AS couples_sales, 
SUM(ROUND(100.0*families / (couples + families + unknown),2)) AS families_sales,
SUM(ROUND(100.0*unknown / (couples + families + unknown),2)) AS unknown_sales
FROM cte2
GROUP BY year_number
```


| year_number |	couples_sales |	families_sales |	unknown_sales |
| --- | --- | --- | --- |
| 2018 |	26.38 |	31.99 |	41.63 |
| 2019 |	27.28 |	32.47 |	40.25 |
| 2020 |	28.72 |	32.73 |	38.55 |
