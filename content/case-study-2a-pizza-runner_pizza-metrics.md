**Case Study 2: Pizza Runner**

1. **Pizza Metrics**

![](/images/5qR_Image_1.png)

1. How many pizzas were ordered?

SELECT COUNT(*) AS pizzas_ordered

FROM pizza_runner.customer_orders;

![](/images/tuA_Image_2.png)

2. How many unique customer orders were made?

SELECT COUNT(DISTINCT order_id) AS unique_customer_orders

FROM pizza_runner.customer_orders;

![](/images/7za_Image_3.png)

3. How many successful orders were delivered by each runner?

SELECT runner_id, COUNT(DISTINCT order_id) AS successful_orders

FROM pizza_runner.runner_orders

WHERE pickup_time <> 'null'

GROUP BY runner_id;

![](/images/97i_Image_4.png)

4. How many of each type of pizza was delivered?

SELECT  c.pizza_id, COUNT(*) AS total_delivered

FROM customer_orders AS c

JOIN runner_orders AS r

ON c.order_id = r.order_id

WHERE r.cancellation IS NULL OR r.cancellation = 'NaN'

GROUP BY c.pizza_id

ORDER BY total_delivered DESC;

![](/images/DRn_Image_5.png)

5. How many Vegetarian and Meatlovers were ordered by each customer?

SELECT  c.customer_id,

SUM(CASE WHEN c.pizza_id = 1 THEN 1 ELSE 0 END) AS meat_lovers_count,

SUM(CASE WHEN c.pizza_id = 2 THEN 1 ELSE 0 END) AS vegetarian_count

FROM customer_orders AS c

GROUP BY c.customer_id

ORDER BY c.customer_id;

![](/images/iGN_Image_6.png)

6. What was the maximum number of pizzas delivered in a single order?

SELECT MAX(pizza_count) AS max_pizzas_in_single_order

FROM (  SELECT c.order_id, COUNT(c.pizza_id) AS pizza_count

FROM customer_orders AS c

JOIN runner_orders AS r

ON c.order_id = r.order_id

WHERE r.cancellation IS NULL

AND c.pizza_id IS NOT NULL

GROUP BY c.order_id ) AS delivered_orders;

![](/images/env_Image_7.png)

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

SELECT  c.customer_id,

SUM( CASE  WHEN (c.exclusions IS NOT NULL AND c.exclusions <> '')

OR (c.extras IS NOT NULL AND c.extras <> '')

THEN 1  ELSE 0

END) AS changed_pizzas,

SUM( CASE  WHEN (c.exclusions IS NULL OR c.exclusions = '')

AND (c.extras IS NULL OR c.extras = '')

THEN 1 ELSE 0

END) AS unchanged_pizzas

FROM customer_orders AS c

JOIN runner_orders AS r

ON c.order_id = r.order_id

WHERE r.cancellation IS NULL  -- only delivered orders

GROUP BY c.customer_id

ORDER BY c.customer_id;

![](/images/fxp_Image_8.png)

8. How many pizzas were delivered that had both exclusions and extras?

SELECT

COUNT(*) AS total_pizzas_with_both_exclusions_and_extras

FROM customer_orders AS c

JOIN runner_orders AS r

ON c.order_id = r.order_id

WHERE r.cancellation IS NULL  -- Only delivered orders

AND c.exclusions IS NOT NULL AND c.exclusions <> ''

AND c.extras IS NOT NULL AND c.extras <> '';

![](/images/nFK_Image_9.png)

9. What was the total volume of pizzas ordered for each hour of the day?

SELECT  EXTRACT(HOUR FROM order_time) AS order_hour,

COUNT(*) AS total_pizzas_ordered

FROM customer_orders

GROUP BY order_hour ORDER BY order_hour;

![](/images/wd0_Image_10.png)

10. What was the volume of orders for each day of the week?

SELECT  TO_CHAR(order_time, 'Day') AS order_day, COUNT(DISTINCT order_id) AS total_orders

FROM customer_orders

GROUP BY order_day

ORDER BY MIN(order_time);

![](/images/7e8_Image_11.png)
