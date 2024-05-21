# SQL-Sales-Analysis


## Business Problem Statement:

## Department: Sales and Marketing

The Sales and Marketing department of our organization is facing challenges in analyzing sales data effectively to identify top-selling products and top-spending customers. Without a comprehensive understanding of sales performance, the department struggles to make informed decisions regarding marketing strategies, inventory management, and customer engagement initiatives.

To address this issue, I aim to develop a SQL-based data analytics project that will analyze sales data from various sources and generate actionable insights for the Sales and Marketing department. The project will leverage  SQL techniques, including common table expressions (CTEs), window functions, and optimizations, to perform in-depth analysis of sales data.

### The key objectives of the project are as follows:

1. Identify the top-selling products for each month, considering factors such as total sales volume and revenue generated.
2. Determine the top-spending customers for each month, based on their total purchase amount and contribution to overall sales.
3. Rank products and customers within each month to prioritize marketing efforts and customer engagement strategies.
4. Provide a user-friendly interface for querying and visualizing sales data, enabling stakeholders to explore insights and trends easily.


```sql

WITH RECURSIVE monthly_sales AS (
    SELECT
        DATE_TRUNC('month', o.order_date) AS month,
        p.product_id,
        p.product_name,
        c.customer_id,
        c.customer_name,
        od.quantity,
        od.unit_price,
        od.quantity * od.unit_price AS total_sales
    FROM
        orders o
        JOIN order_details od ON o.order_id = od.order_id
        JOIN products p ON od.product_id = p.product_id
        JOIN customers c ON o.customer_id = c.customer_id
), 

product_sales_ranking AS (
    SELECT
        month,
        product_id,
        product_name,
        SUM(total_sales) AS total_sales,
        RANK() OVER (PARTITION BY month ORDER BY SUM(total_sales) DESC) AS product_sales_rank
    FROM
        monthly_sales
    GROUP BY
        month,
        product_id,
        product_name
),

customer_spending_ranking AS (
    SELECT
        month,
        customer_id,
        customer_name,
        SUM(total_sales) AS total_spent,
        RANK() OVER (PARTITION BY month ORDER BY SUM(total_sales) DESC) AS customer_spending_rank
    FROM
        monthly_sales
    GROUP BY
        month,
        customer_id,
        customer_name
),

top_products_per_month AS (
    SELECT
        month,
        product_id,
        product_name,
        total_sales,
        product_sales_rank
    FROM
        product_sales_ranking
    WHERE
        product_sales_rank <= 3
),

top_customers_per_month AS (
    SELECT
        month,
        customer_id,
        customer_name,
        total_spent,
        customer_spending_rank
    FROM
        customer_spending_ranking
    WHERE
        customer_spending_rank <= 3
),

top_product_customer_combinations AS (
    SELECT
        tppm.month,
        tppm.product_id,
        tppm.product_name,
        tccm.customer_id,
        tccm.customer_name,
        tppm.total_sales
    FROM
        top_products_per_month tppm
    CROSS JOIN
        top_customers_per_month tccm
),

final_result AS (
    SELECT
        tpc.month,
        tpc.product_id,
        tpc.product_name,
        tpc.customer_id,
        tpc.customer_name,
        tpc.total_sales,
        RANK() OVER (PARTITION BY tpc.month ORDER BY tpc.total_sales DESC) AS sales_rank,
        RANK() OVER (PARTITION BY tpc.month ORDER BY tpc.total_sales DESC) AS spending_rank
    FROM
        top_product_customer_combinations tpc
)

SELECT
    fr.month,
    fr.product_id,
    fr.product_name,
    fr.customer_id,
    fr.customer_name,
    fr.total_sales,
    fr.sales_rank,
    fr.spending_rank
FROM
    final_result fr
ORDER BY
    fr.month,
    fr.sales_rank,
    fr.spending_rank;
