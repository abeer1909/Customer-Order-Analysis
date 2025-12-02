# Project Overview

This project involves a comprehensive analysis of customer orders, sales performance, and product trends using SQL. The goal is to derive actionable insights from the gold_fact_sales, gold_dim_products, and gold_dim_customers tables. The analysis covers various aspects such as time-series sales trends, cumulative performance, product sales distribution, and detailed customer segmentation.

The SQL scripts provided here answer critical business questions regarding growth, customer retention, and inventory value distribution.

![](https://private-user-images.githubusercontent.com/87649792/521471061-3747e74c-e5bd-4be0-9e61-5bda34917995.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjQ2OTY3NzAsIm5iZiI6MTc2NDY5NjQ3MCwicGF0aCI6Ii84NzY0OTc5Mi81MjE0NzEwNjEtMzc0N2U3NGMtZTViZC00YmUwLTllNjEtNWJkYTM0OTE3OTk1LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEyMDIlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMjAyVDE3Mjc1MFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTA4NmFmZmIzNjk5NTBiOWYxMjY2NGFlOWZlOWMzYWYzN2MwMGZlOTU1ZDdiOTUxN2Y0YmFlNDQ4ZTdhNWI0ODMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.2pUnfbMCNAGhItACnyMu-gbo2o6CEwFm6fBGkdDW5_k)
# Questions and Answers

Q1. How can we analyze the monthly and yearly sales trends, including the total number of customers and quantity sold?

Answer:
```bash
SELECT 
  DATE_FORMAT(MIN(order_date), '%Y-%b') AS order_date,
  SUM(sales_amount) AS total_sales,
  COUNT(DISTINCT customer_key) AS total_customers,
  SUM(quantity) AS total_quantity
FROM gold_fact_sales
WHERE order_date IS NOT NULL
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY YEAR(order_date), MONTH(order_date);
```

Q2. How do we calculate the cumulative sales total and the moving average of prices over the years?

Answer:
```bash

SELECT
    order_date,
    total_sales,
    SUM(total_sales) OVER (ORDER BY order_date) AS running_total_sales,
    AVG(avg_price) OVER (ORDER BY order_date) AS moving_avg_price
FROM (
    SELECT
        DATE_FORMAT(order_date, '%Y') AS order_date,
        SUM(sales_amount) AS total_sales,
        AVG(price) AS avg_price
    FROM gold_fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY DATE_FORMAT(order_date, '%Y')
) t
ORDER BY order_date;
```
Q3. Can we analyze the year-over-year performance of products, comparing current sales to the average and identifying sales trends (increase/decrease)?

Answer:
```bash
WITH yearly_product_sales AS (
    SELECT
        YEAR(f.order_date) AS order_year,
        p.product_name,
        SUM(f.sales_amount) AS current_sales
    FROM CRM.gold_fact_sales f
    LEFT JOIN CRM.gold_dim_products p
        ON f.product_key = p.product_key
    WHERE f.order_date IS NOT NULL
    GROUP BY YEAR(f.order_date), p.product_name
) 
SELECT 
    order_year,
    product_name,
    current_sales,
    AVG(current_sales) OVER (PARTITION BY product_name) AS avg_sales_per_product,
    current_sales - AVG(current_sales) OVER (PARTITION BY product_name) AS sales_deviation,
    CASE WHEN current_sales - AVG(current_sales) OVER (PARTITION BY product_name) > 0 THEN 'Above Average'
        WHEN current_sales - AVG(current_sales) OVER (PARTITION BY product_name) < 0 THEN 'Below Average'
        ELSE 'Average' END avg_change,
    LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) AS previous_year_sales,
    current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) AS sales_change,
    CASE WHEN current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) > 0 THEN 'Increase'
        WHEN current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) < 0 THEN 'Decrease'
        ELSE 'No Change' END sales_trend
FROM yearly_product_sales
ORDER BY product_name, order_year;
```
![](https://private-user-images.githubusercontent.com/87649792/521471188-c78d6b4c-0b3a-41d2-954f-a305192e45d6.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjQ2OTY5NjEsIm5iZiI6MTc2NDY5NjY2MSwicGF0aCI6Ii84NzY0OTc5Mi81MjE0NzExODgtYzc4ZDZiNGMtMGIzYS00MWQyLTk1NGYtYTMwNTE5MmU0NWQ2LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEyMDIlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMjAyVDE3MzEwMVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTY4NmQyMzI1NmUyZjkyNDcwN2ExMDAwNjJmMGZkMjk5YjJhM2E0ZmZmYjVhODZkNmVmYmM0MWFjMWEzNmM2ZmUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.8LeJoZqNCJyyQb20zzW-zyTq_gGk7r-XER0qBawVpqM)
Q4. What is the total sales distribution across different product categories, and what percentage does each category contribute to the overall sales?

Answer:
```bash
WITH category_sales AS (
    SELECT
        p.category,
        SUM(f.sales_amount) AS total_sales
    FROM CRM.gold_fact_sales f
    LEFT JOIN CRM.gold_dim_products p
        ON f.product_key = p.product_key
    GROUP BY p.category
)
SELECT
    category,
    total_sales,
    SUM(total_sales) OVER () AS overall_sales,
    CONCAT(ROUND((total_sales / SUM(total_sales) OVER ()) * 100,2),'%') AS sales_percentage
FROM category_sales
ORDER BY total_sales DESC;
```

Q5. How are products segmented based on their cost (e.g., Below 100, 100-500), and how many products fall into each segment?

Answer:
```bash
WITH product_segments AS (
    SELECT
        product_key,
        product_name,
        cost,
        CASE WHEN cost < 100 THEN 'Below 100'
            WHEN cost BETWEEN 100 AND 500 THEN '100-500'
            WHEN cost BETWEEN 501 AND 1000 THEN '501-1000'
            ELSE 'Above 1000'
        END AS cost_range
    FROM CRM.gold_dim_products
)
SELECT
    cost_range,
    COUNT(product_key) AS total_products
FROM product_segments
GROUP BY cost_range    
ORDER BY total_products DESC;
```

Q6. How can we categorize customers into 'VIP', 'Regular', or 'New' segments based on their total spending and the duration of their relationship with us?

Answer:
```bash
WITH customer_spending AS (
    SELECT 
        c.customer_key,
        SUM(f.sales_amount) AS total_spending,
        MIN(f.order_date) AS first_order_date,
        MAX(f.order_date) AS last_order_date,
        CAST(DATEDIFF(MAX(f.order_date), MIN(f.order_date)) / 30 AS UNSIGNED) AS customer_lifespan
    FROM gold_fact_sales f
    LEFT JOIN gold_dim_customers c
        ON f.customer_key = c.customer_key
    GROUP BY c.customer_key
)
SELECT
    CASE 
        WHEN customer_lifespan >= 12 AND total_spending > 5000 THEN 'VIP'
        WHEN customer_lifespan >= 12 AND total_spending <= 5000 THEN 'Regular'
        ELSE 'New'
    END AS customer_segment,
    COUNT(*) AS customer_count,
    CONCAT('$',ROUND(AVG(total_spending),2)) AS avg_spending
FROM customer_spending
GROUP BY customer_segment;
```

Q7. How can we build a comprehensive customer report view that includes age groups, purchasing habits, and calculated metrics like average monthly spend?

Answer:
```bash
CREATE VIEW CRM.report_customers AS
WITH base_query AS (
    SELECT 
        f.order_number,
        f.product_key,
        f.order_date,
        f.sales_amount,
        f.quantity,
        c.customer_key,
        c.customer_number,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        ROUND(DATEDIFF(CURRENT_DATE, c.birthdate) / 365, 0) AS age
    FROM gold_fact_sales f 
    LEFT JOIN gold_dim_customers c 
        ON f.customer_key = c.customer_key
    WHERE f.order_date IS NOT NULL
), customer_aggregation AS ( 
    SELECT 
        customer_key,
        customer_number,
        customer_name,
        age,
        COUNT(DISTINCT order_number) AS total_orders,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        COUNT(DISTINCT product_key) AS total_products,
        MAX(order_date) AS last_order_date,
        CAST(DATEDIFF(MAX(order_date), MIN(order_date)) / 30 AS UNSIGNED) AS customer_lifespan
    FROM base_query
    GROUP BY customer_key, customer_number, customer_name, age
    ORDER BY total_sales DESC
)
SELECT 
    customer_key,
    customer_number,
    customer_name,
    age,
    CASE
        WHEN age < 20 THEN 'Under 20' 
        WHEN age BETWEEN 20 AND 29 THEN '20-29' 
        WHEN age BETWEEN 30 AND 39 THEN '30-39' 
        WHEN age BETWEEN 40 AND 49 THEN '40-49' 
        ELSE '50 and above'
    END AS age_group,
    CASE 
        WHEN customer_lifespan > 12 AND total_sales > 5000 THEN 'VIP' 
        WHEN customer_lifespan >= 12 AND total_sales <= 5000 THEN 'Regular'
        ELSE 'New'
    END AS customer_segment,
    total_orders,
    total_sales,
    total_quantity,
    total_products,
    last_order_date,
    ROUND(DATEDIFF(CURRENT_DATE, last_order_date) / 30, 0) AS recency_months,
    customer_lifespan,
    CASE
        WHEN total_sales = 0 THEN 0
        ELSE
            CONCAT('$',ROUND(total_sales / total_orders))
    END avg_order_value,
    CASE 
        WHEN customer_lifespan = 0 THEN total_sales
        ELSE
            CONCAT('$',ROUND(total_sales / customer_lifespan))
    END avg_monthly_spend
FROM customer_aggregation;
```
![](https://private-user-images.githubusercontent.com/87649792/521471138-a8e83d8a-5f13-4c2b-8783-3fe7a20b7f5f.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjQ2OTY5NjEsIm5iZiI6MTc2NDY5NjY2MSwicGF0aCI6Ii84NzY0OTc5Mi81MjE0NzExMzgtYThlODNkOGEtNWYxMy00YzJiLTg3ODMtM2ZlN2EyMGI3ZjVmLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEyMDIlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMjAyVDE3MzEwMVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTcwZjQ1MDZmNmQ5MTA1MWRlNTMzYzZiNThmOTZlOTFlMDRjMDQ5YzZkMzliNDI2YTdmYjY0NjllNGVmODVmZTYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.41MlsjUxF4W5fpWB7JIJeYGpjM_OOgjm1iFzxDWolbc)
