# SQL Exploratory Data Analysis (EDA) Project

This project demonstrates a step-by-step Exploratory Data Analysis (EDA) on a Data Warehouse that I recently built as part of a separate project using SQL Server. By leveraging foundational SQL techniques, we explore a multi-table database to uncover valuable business insights. The analysis is organized into six logical steps.

🔍 Want to see the SQL Data Warehouse project this EDA is based on? Check it out here: [sql-dwh-project](https://github.com/devoodian/sql-dwh-project)

➔ Next step with advanced analytics and business reports: [sql-data-analytics-project](https://github.com/devoodian/sql-data-analytics-project)

## 1. Database Exploration

**Goal**: Understand the structure and scale of the database.

- **Query 1**: Get list of all tables

```sql
SELECT TABLE_CATALOG, TABLE_SCHEMA, TABLE_NAME, TABLE_TYPE
FROM INFORMATION_SCHEMA.TABLES;
```

➡️ Found 15 tables.

- **Query 2**: Inspect columns in `dim_customers`

```sql
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'dim_customers';
```

➡️ Found 9 columns.

## 2. Dimension Exploration

**Goal**: Explore key dimension values and their granularity.

- **Query 1**: Unique customer countries

```sql
SELECT DISTINCT country FROM gold.dim_customers ORDER BY country;
```

➡️ Countries: Germany, United States, Australia, United Kingdom, Canada, France

- **Query 2**: Unique categories, subcategories, products

```sql
SELECT DISTINCT category, subcategory, product_name
FROM gold.dim_products
ORDER BY category, subcategory, product_name;
```

➡️

- Categories: Accessories, Bikes, Clothing, Components
- Subcategories (example for Bikes): Mountain Bikes, Road Bikes, Touring Bikes
- Products: \~295 unique names

📌 **Insight**: Category-level aggregations yield 4 rows vs. 295+ for product\_name, showing granularity differences.

## 3. Date Exploration

**Goal**: Discover date ranges to understand the business time span.

- **Query 1**: Order date boundaries and range

```sql
SELECT 
    MIN(order_date) AS first_order_date,
    MAX(order_date) AS last_order_date,
    DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS order_range_months
FROM gold.fact_sales;
```

➡️ First: 2010-12-29, Last: 2014-01-28, Range: 37 months

- **Query 2**: Age range of customers

```sql
SELECT
    MIN(birthdate) AS oldest_birthdate,
    DATEDIFF(YEAR, MIN(birthdate), GETDATE()) AS oldest_age,
    MAX(birthdate) AS youngest_birthdate,
    DATEDIFF(YEAR, MAX(birthdate), GETDATE()) AS youngest_age
FROM gold.dim_customers;
```

➡️ Age Range: 39 to 109 years

📌 **Insight**: There is a wide age range among customers. This can be valuable when analyzing customer segmentation or age-based buying patterns.

## 4. Measures Exploration

**Goal**: Get quick insights using aggregated metrics, Identify overall trends or anomalies.

- **Query**: Aggregate key metrics

```sql
SELECT 'Total Sales' AS measure_name, SUM(sales_amount) FROM gold.fact_sales
UNION ALL
SELECT 'Total Quantity', SUM(quantity) FROM gold.fact_sales
UNION ALL
SELECT 'Average Price', AVG(price) FROM gold.fact_sales
UNION ALL
SELECT 'Total Orders', COUNT(DISTINCT order_number) FROM gold.fact_sales
UNION ALL
SELECT 'Total Products', COUNT(DISTINCT product_name) FROM gold.dim_products
UNION ALL
SELECT 'Total Customers', COUNT(customer_key) FROM gold.dim_customers;
```

➡️

- Total Sales: 29,356,250
- Total Quantity: 60,423
- Average Price: 486
- Total Orders: 27,659
- Products: 295
- Customers: 18,484

📌 **Insight**: Full snapshot of business operations achieved.

✅ With this view, we now have a clear understanding of the dataset and business structure, which sets the foundation for deeper insights in the next steps.

## 5. Magnitude Analysis

**Goal**: To quantify data and group results by specific dimensions For understanding data distribution across categories.

- **Customers by Country**

```sql
SELECT country, COUNT(customer_key) AS total_customers
FROM gold.dim_customers
GROUP BY country ORDER BY total_customers DESC;
```

➡️ Top: United States (7482), Australia (3591)

- **Customers by Gender**

```sql
SELECT gender, COUNT(customer_key) AS total_customers
FROM gold.dim_customers
GROUP BY gender ORDER BY total_customers DESC;
```

➡️ Male (9341), Female (9128)

- **Products by Category**

```sql
SELECT category, COUNT(product_key) AS total_products
FROM gold.dim_products
GROUP BY category ORDER BY total_products DESC;
```

➡️ Top: Components (127), Bikes (97)

- **Average Cost per Category**

```sql
SELECT category, AVG(cost) AS avg_cost
FROM gold.dim_products
GROUP BY category ORDER BY avg_cost DESC;
```

➡️ Top: Bikes (949), Components (264)

- **Revenue by Category**

```sql
SELECT p.category, SUM(f.sales_amount) AS total_revenue
FROM gold.fact_sales f
LEFT JOIN gold.dim_products p ON p.product_key = f.product_key
GROUP BY p.category ORDER BY total_revenue DESC;
```

➡️ Bikes: 28M+, Accessories & Clothing: <1M

📌 **Insight**: High revenue concentration in Bikes category.

## 6. Ranking Analysis

**Goal**: Rank items based on performance or other metrics To identify top performers or laggards.

- **Top 5 Products by Revenue**

```sql
SELECT * FROM (
    SELECT p.product_name, SUM(f.sales_amount) AS total_revenue,
           RANK() OVER (ORDER BY SUM(f.sales_amount) DESC) AS rank_products
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p ON p.product_key = f.product_key
    GROUP BY p.product_name
) ranked_products
WHERE rank_products <= 5;
```

➡️ All top 5 are Mountain-200 bike variations

- **Bottom 5 Products by Revenue**

```sql
SELECT TOP 5 p.product_name, SUM(f.sales_amount) AS total_revenue
FROM gold.fact_sales f
LEFT JOIN gold.dim_products p ON p.product_key = f.product_key
GROUP BY p.product_name ORDER BY total_revenue;
```

➡️ Includes Socks, Patch Kits, and Bike Wash

- **3 Customers with Fewest Orders**

```sql
SELECT TOP 3 c.customer_key, c.first_name, c.last_name,
              COUNT(DISTINCT order_number) AS total_orders
FROM gold.fact_sales f
LEFT JOIN gold.dim_customers c ON c.customer_key = f.customer_key
GROUP BY c.customer_key, c.first_name, c.last_name
ORDER BY total_orders;
```

➡️ Jordan King, Wyatt Hill, Destiny Wilson

📌 **Insight**: Useful for identifying VIP customers and potential churn risk.


##  Final Summary
With these steps, our Exploratory Data Analysis (EDA) is complete. We now have a comprehensive and in-depth understanding of the data structure, sales performance, and customer behavior — providing a solid foundation for advanced business analytics, reporting, or dashboard development.
