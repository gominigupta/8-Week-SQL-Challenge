**Case Study 2: Pizza Runner**

1. **Pricing And Ratings**

![](/images/5qR_Image_1.png)

1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

SELECT  SUM( CASE WHEN pn.pizza_name = 'Meatlovers' THEN 12

WHEN pn.pizza_name = 'Vegetarian' THEN 10

ELSE 0

END) AS "Total Revenue ($)"

FROM customer_orders co

JOIN pizza_names pn ON co.pizza_id = pn.pizza_id

JOIN runner_orders ro  ON co.order_id = ro.order_id

WHERE ro.cancellation IS NULL;

![](/images/5mh_Image_2.png)

2. What if there was an additional $1 charge for any pizza extras?

1. Add cheese is $1 extra

SELECT  SUM(CASE WHEN pn.pizza_name = 'Meatlovers' THEN 12

WHEN pn.pizza_name = 'Vegetarian' THEN 10

ELSE 0

END

+

(CASE WHEN co.extras IS NOT NULL AND co.extras <> 'null' AND co.extras <> ''

THEN array_length(string_to_array(co.extras, ','), 1)

ELSE 0

END)

+

(CASE  WHEN co.extras ILIKE '%4%' THEN 1

ELSE 0

END)

) AS "Total Revenue ($)"

FROM customer_orders co

JOIN pizza_names pn ON co.pizza_id = pn.pizza_id

JOIN runner_orders ro ON co.order_id = ro.order_id

WHERE ro.cancellation IS NULL;

![](/images/6wV_Image_3.png)

3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner. How would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

CREATE TABLE runner_ratings (

order_id INTEGER PRIMARY KEY,  -- Each order gets only one rating

customer_id INTEGER,           -- Customer who provided the rating

runner_id INTEGER,             -- Runner who is being rated

rating INTEGER CHECK (rating >= 1 AND rating <= 5),  -- Rating between 1 and 5

rating_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP  -- Timestamp of when rating was submitted

);

INSERT INTO runner_ratings (order_id, customer_id, runner_id, rating)

VALUES

(1, 101, 1, 5),  -- Customer 101 gave runner 1 a 5-star rating

(2, 101, 1, 4),

(3, 102, 1, 5),

(4, 103, 2, 3),

(5, 104, 3, 4),

(7, 105, 2, 5),

(8, 102, 2, 4),

(10, 104, 1, 3);

4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?

4. 
2. ```
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time btwn order & pickup
Delivery duration
Average speed
Total #of pizzas
```
WITH delivered_orders AS ( SELECT co.customer_id, co.order_id, ro.runner_id,

rr.rating, co.order_time, ro.pickup_time::timestamp AS pickup_time,

ro.duration, ro.distance,

(ro.pickup_time::timestamp - co.order_time) AS time_to_pickup,

COUNT(co.pizza_id) AS total_pizzas

FROM customer_orders co

JOIN runner_orders ro ON co.order_id = ro.order_id

LEFT JOIN runner_ratings rr ON co.order_id = rr.order_id

WHERE ro.cancellation IS NULL  -- only successful deliveries

GROUP BY  co.customer_id, co.order_id, ro.runner_id, rr.rating, co.order_time, ro.pickup_time, ro.duration, ro.distance

)

SELECT  customer_id, order_id, runner_id,  \
COALESCE(CAST(rating AS TEXT), 'No Rating') AS rating,  \
order_time, pickup_time,  \
ROUND((EXTRACT(EPOCH FROM time_to_pickup) / 60)::numeric, 2) AS minutes_to_pickup, \
ROUND(CASE WHEN duration ~ '^\d+$' THEN duration::numeric \
WHEN duration ~ '(\d+)' THEN substring(duration FROM '(\d+)')::numeric \
ELSE NULL \
END, 2) AS delivery_duration_minutes, \
 \
 \
 \
ROUND(

CASE

WHEN distance ~ '^\d+(\.\d+)?$' THEN distance::numeric

WHEN distance ~ '(\d+(\.\d+)?)' THEN substring(distance FROM '(\d+(\.\d+)?)')::numeric

ELSE NULL

END, 2

) AS delivery_distance_km,

CASE

WHEN

-- Ensure no division by zero

ROUND(

CASE

WHEN duration ~ '^\d+$' THEN duration::numeric

WHEN duration ~ '(\d+)' THEN substring(duration FROM '(\d+)')::numeric

ELSE NULL

END, 2

) > 0

THEN

ROUND(

(

CASE

WHEN distance ~ '^\d+(\.\d+)?$' THEN distance::numeric

WHEN distance ~ '(\d+(\.\d+)?)' THEN substring(distance FROM '(\d+(\.\d+)?)')::numeric

ELSE NULL

END

) /

(

ROUND(

CASE

WHEN duration ~ '^\d+$' THEN duration::numeric

WHEN duration ~ '(\d+)' THEN substring(duration FROM '(\d+)')::numeric

ELSE NULL

END, 2

) / 60.0

)::numeric,

2

)

ELSE NULL

END AS average_speed_kph,

total_pizzas

FROM delivered_orders

ORDER BY order_id;

![](/images/uBO_Image_4.png)

5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

WITH pizza_prices AS (

SELECT

order_id,

SUM(

CASE

WHEN pizza_id = 1 THEN 12  -- Meat Lovers

WHEN pizza_id = 2 THEN 10  -- Vegetarian

END

) AS total_revenue

FROM customer_orders

GROUP BY order_id

),

delivery_costs AS (

SELECT

order_id,

SUM(

CASE

WHEN distance ~ '^\d+(\.\d+)?$' THEN distance::numeric

WHEN distance ~ '(\d+(\.\d+)?)' THEN substring(distance FROM '(\d+(\.\d+)?)')::numeric

ELSE 0

END

) AS total_distance_km

FROM runner_orders

WHERE cancellation IS NULL

GROUP BY order_id

)

SELECT

SUM(pp.total_revenue) AS total_revenue,

SUM(dc.total_distance_km) AS total_distance,

ROUND(SUM(pp.total_revenue) - (SUM(dc.total_distance_km) * 0.30), 2) AS profit

FROM pizza_prices pp

JOIN delivery_costs dc

ON pp.order_id = dc.order_id;

![](/images/gRs_Image_5.png)
