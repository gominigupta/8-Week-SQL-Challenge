**Case Study 3: Foodie-Fi**

![](/images/Xja_Image_1.png)

### **A. Customer Nodes Exploration**

1. How many unique nodes are there on the Data Bank system?

SELECT COUNT(DISTINCT node_id)

FROM customer_nodes;

![](/images/cs2_Image_2.png)

2. What is the number of nodes per region?

SELECT r.region_id, r.region_name, COUNT(DISTINCT cn.node_id) AS number_of_nodes

FROM regions AS r

JOIN customer_nodes AS cn

ON r.region_id = cn.region_id

GROUP BY r.region_id, r.region_name

ORDER BY r.region_id;

![](/images/xYK_Image_3.png)

3. How many customers are allocated to each region?

SELECT r.region_id, r.region_name, COUNT(DISTINCT cn.customer_id) AS number_of_customers

FROM regions AS r

JOIN customer_nodes AS cn

ON r.region_id = cn.region_id

GROUP BY r.region_id, r.region_name

ORDER BY r.region_id;

![](/images/AxH_Image_4.png)

4. How many days on average are customers reallocated to a different node?

SELECT ROUND(AVG(end_date - start_date), 2) AS avg_days_per_allocation

FROM customer_nodes

WHERE end_date != '9999-12-31'; *-- excluding active allocations & only considering completed reallocations.*

![](/images/Iaj_Image_5.png)

5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

SELECT

r.region_id, r.region_name,

PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY (cn.end_date - cn.start_date)) AS median,

PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY (cn.end_date - cn.start_date)) AS percentile_80,

PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY (cn.end_date - cn.start_date)) AS percentile_95

FROM regions AS r

JOIN customer_nodes AS cn

ON r.region_id = cn.region_id

WHERE cn.end_date != '9999-12-31'*  -- Excluding active allocations*

GROUP BY r.region_id, r.region_name

ORDER BY r.region_id;

![](/images/V77_Image_6.png)

### **B. Customer Transactions**

1. What is the unique count and total amount for each transaction type?

SELECT txn_type, COUNT(*) AS unique_txn_count, SUM(txn_amount) AS total_amount

FROM customer_transactions

GROUP BY txn_type

ORDER BY txn_type;

![](/images/fbr_Image_7.png)

2. What is the average total historical deposit counts and amounts for all customers?

SELECT

ROUND(AVG(deposit_txn_count),2) AS avg_deposit_txn_count,

ROUND(AVG(total_deposit_amount),2) AS avg_deposit_amount

FROM ( SELECT customer_id, COUNT(*) AS deposit_txn_count, SUM(txn_amount)

AS total_deposit_amount

FROM customer_transactions

WHERE txn_type = 'deposit'

GROUP BY customer_id

) AS customer_deposits;

![](/images/KMp_Image_8.png)

3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

WITH monthly_txn AS ( SELECT

DATE_PART('month', txn_date) AS month,

customer_id,

COUNT(CASE WHEN txn_type = 'deposit' THEN 1 END) AS deposit_count,

COUNT(CASE WHEN txn_type = 'purchase' THEN 1 END) AS purchase_count,

COUNT(CASE WHEN txn_type = 'withdrawal' THEN 1 END) AS withdrawal_count

FROM customer_transactions

GROUP BY month, customer_id

)

SELECT

month,

COUNT(customer_id) AS qualifying_customers

FROM monthly_txn

WHERE deposit_count > 1 AND (purchase_count = 1 OR withdrawal_count = 1)

GROUP BY month

ORDER BY month;

![](/images/0q6_Image_9.png)

4. What is the closing balance for each customer at the end of the month?

WITH Amount AS(

SELECT

customer_id,

DATE_PART('month', txn_date) AS month,

SUM(CASE WHEN txn_type = 'deposit'

THEN txn_amount ELSE -txn_amount END) AS amount

FROM customer_transactions

GROUP BY customer_id, month

)

SELECT  customer_id, month,

SUM(amount) OVER (PARTITION BY customer_id

ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING

AND CURRENT ROW) AS closing_balance

FROM Amount

ORDER BY customer_id;

![](/images/mjc_Image_10.png)

5. What is the percentage of customers who increase their closing balance by more than 5%?

WITH

closing_balance_cte AS (

SELECT customer_id, month, SUM(amount) OVER( PARTITION BY

customer_id  ORDER BY month ROWS BETWEEN UNBOUNDED

PRECEDING AND CURRENT ROW) AS closing_balance

FROM ( SELECT  customer_id, DATE_PART('month', txn_date) AS month,

SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount

ELSE -txn_amount END) AS amount

FROM customer_transactions

GROUP BY customer_id, month

) AS monthly_amount

),

balance_change AS (

SELECT customer_id,  month, closing_balance, LAG(closing_balance)

OVER(PARTITION BY customer_id ORDER BY month) AS prev_balance,

CASE  WHEN

LAG(closing_balance) OVER(PARTITION BY customer_id

ORDER BY month) IS NOT NULL

AND

LAG(closing_balance) OVER(PARTITION BY customer_id

ORDER BY month) != 0

THEN (closing_balance - LAG(closing_balance) OVER(PARTITION BY

customer_id ORDER BY month)) * 100.0 / LAG(closing_balance)

OVER(PARTITION BY customer_id ORDER BY month)

ELSE NULL

END AS pct_change

FROM closing_balance_cte

),

total_customers AS (

SELECT COUNT(DISTINCT customer_id) AS total_count FROM customer_transactions),

qualified_customers AS (

SELECT COUNT(DISTINCT customer_id) AS qualified_count FROM balance_change WHERE pct_change > 5

)

SELECT

CASE WHEN total_count = 0 THEN 0

ELSE ROUND((qualified_count * 100.0 / total_count),2)

END AS pct_customers_with_increase

FROM total_customers, qualified_customers;

![](/images/ryl_Image_11.png)
