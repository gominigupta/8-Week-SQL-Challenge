**Case Study 2: Pizza Runner**

1. **Runner and Customer Experience**

![](/images/5qR_Image_1.png)

1. How many runners signed up for each 1 week period? (i.e. week starts `2021-01-01`)

SELECT

TO_CHAR( DATE_TRUNC('week', registration_date - INTERVAL '4 days') +INTERVAL '4 days',  'YYYY-MM-DD') AS week_start,

COUNT(*) AS total_runners_signed_up

FROM runners

GROUP BY week_start

ORDER BY week_start;

![](/images/PQY_Image_2.png)

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pick up the order?

SELECT  r.runner_id,  ROUND(AVG(EXTRACT(EPOCH FROM (

CAST(r.pickup_time AS TIMESTAMP) - c.order_time)) / 60), 2)

AS avg_arrival_time_minutes

FROM runner_orders AS r

JOIN customer_orders AS c ON r.order_id = c.order_id

WHERE r.pickup_time IS NOT NULL

AND (r.cancellation IS NULL OR r.cancellation = '')

GROUP BY r.runner_id

ORDER BY r.runner_id;

![](/images/jFv_Image_3.png)

3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

WITH order_pizza_count AS (

SELECT

order_id,

COUNT(pizza_id) AS num_pizzas,

MIN(order_time) AS order_time

FROM customer_orders

GROUP BY order_id

),

prep_time_calc AS (

SELECT

o.order_id,

o.num_pizzas,

r.runner_id,

(NULLIF(r.pickup_time, 'null')::TIMESTAMP - o.order_time) AS prep_time

FROM order_pizza_count o

JOIN runner_orders r

ON o.order_id = r.order_id

WHERE NULLIF(r.pickup_time, 'null') IS NOT NULL

)

SELECT

num_pizzas,

AVG(EXTRACT(EPOCH FROM prep_time)/60) AS avg_prep_time_minutes

FROM prep_time_calc

GROUP BY num_pizzas

ORDER BY num_pizzas;

![](/images/8iu_Image_4.png)

As the results suggest, there is a **<span style="text - decoration: underline;">POSITIVE CORRELATION**</span> between the number of pizzas in an order and the average preparation time.

4. What was the average distance travelled for each customer?

SELECT

co.customer_id,

ROUND(AVG(NULLIF(REGEXP_REPLACE(ro.distance, '[^0-9.]', '', 'g'), '')::NUMERIC), 2) AS avg_distance_km

FROM customer_orders co

JOIN runner_orders ro ON co.order_id = ro.order_id

WHERE ro.distance IS NOT NULL AND ro.cancellation IS NULL

GROUP BY co.customer_id

ORDER BY customer_id;

![](/images/8bz_Image_5.png)

5. What was the difference between the longest and shortest delivery times for all orders?

SELECT  MAX(NULLIF(REGEXP_REPLACE(duration, '[^0-9.]', '', 'g'), '')::NUMERIC) -

MIN(NULLIF(REGEXP_REPLACE(duration, '[^0-9.]', '', 'g'), '')::NUMERIC)

AS delivery_time_difference

FROM runner_orders

WHERE duration IS NOT NULL

AND cancellation IS NULL;

![](/images/R54_Image_6.png)

6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

SELECT runner_id, order_id,

(NULLIF(REGEXP_REPLACE(distance, '[^0-9.]', '', 'g'), '')::NUMERIC) /

(NULLIF(REGEXP_REPLACE(duration, '[^0-9.]', '', 'g'), '')::NUMERIC / 60)

AS avg_speed_kmph

FROM runner_orders

WHERE distance IS NOT NULL

AND duration IS NOT NULL

AND cancellation IS NULL

ORDER BY runner_id, order_id;

![](/images/Mtk_Image_7.png)

7. What is the successful delivery percentage for each runner?

SELECT

runner_id,

COUNT(*) AS total_orders,

SUM(CASE WHEN cancellation IS NULL OR cancellation = '' THEN 1 ELSE 0 END) AS successful_deliveries,

ROUND( (SUM(CASE WHEN cancellation IS NULL OR cancellation = '' THEN 1 ELSE 0 END) * 100.0) / COUNT(*),  2 ) AS success_percentage

FROM runner_orders

GROUP BY runner_id

ORDER BY success_percentage DESC;

![](/images/6XV_Image_8.png)
