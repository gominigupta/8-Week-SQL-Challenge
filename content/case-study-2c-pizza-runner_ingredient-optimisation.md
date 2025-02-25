**Case Study 2: Pizza Runner**

1. **Ingredient Optimisation**

![](/images/5qR_Image_1.png)

1. What are the standard ingredients for each pizza?

SELECT

pn.pizza_name,

STRING_AGG(pt.topping_name, ', ') AS standard_ingredients

FROM pizza_recipes pr

JOIN pizza_names pn ON pr.pizza_id = pn.pizza_id

JOIN pizza_toppings pt ON CAST(pt.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(pr.toppings, ', '))

GROUP BY pn.pizza_name

ORDER BY pn.pizza_name;

![](/images/ALy_Image_2.png)

2. What was the most commonly added extra?

SELECT pt.topping_name AS extra_ingredient,

COUNT(*) AS number_of_times_added

FROM customer_orders co

JOIN pizza_toppings pt

ON CAST(pt.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(co.extras, ', '))

WHERE co.extras IS NOT NULL AND co.extras <> 'null' AND co.extras <> ''

GROUP BY pt.topping_name ORDER BY number_of_times_added DESC LIMIT 1;

![](/images/XmI_Image_3.png)

3. What was the most common exclusion?

SELECT pt.topping_name AS exclusion,

COUNT(*) AS number_of_times_excluded

FROM customer_orders co

JOIN pizza_toppings pt

ON CAST(pt.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(co.exclusions, ', '))

WHERE co.exclusions IS NOT NULL AND co.exclusions <> 'null' AND co.exclusions <> ''

GROUP BY pt.topping_name ORDER BY number_of_times_excluded DESC

LIMIT 1;

![](/images/qJI_Image_4.png)

4. Generate an order item for each record in the `customers_orders` table in the format of one of the following: `Meat Lovers / Meat Lovers - Exclude Beef / Meat Lovers - Extra Bacon / Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`

SELECT  co.order_id, pn.pizza_name ||

CASE

WHEN co.exclusions IS NOT NULL AND co.exclusions <> 'null' AND co.exclusions <> ''

THEN ' - Exclude ' || STRING_AGG(DISTINCT pt1.topping_name, ', ')

ELSE ''

END ||

CASE

WHEN co.extras IS NOT NULL AND co.extras <> 'null' AND co.extras <> ''

THEN ' - Extra ' || STRING_AGG(DISTINCT pt2.topping_name, ', ')

ELSE ''

END AS order_item

FROM customer_orders co

JOIN pizza_names pn

ON co.pizza_id = pn.pizza_id

LEFT JOIN pizza_toppings pt1

ON CAST(pt1.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(co.exclusions, ', '))

LEFT JOIN pizza_toppings pt2

ON CAST(pt2.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(co.extras, ', '))

GROUP BY co.order_id, pn.pizza_name, co.exclusions, co.extras

ORDER BY co.order_id;

![](/images/nMj_Image_5.png)

5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the `customer_orders` table and add a `2x` in front of any relevant ingredients

1. For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`

SELECT

co.order_id,

pn.pizza_name || ': ' ||

STRING_AGG(

CASE

WHEN CAST(pt.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(co.extras, ', '))

THEN '2x' || pt.topping_name

ELSE pt.topping_name

END, ', ' ORDER BY pt.topping_name

) AS ingredient_list

FROM customer_orders co

JOIN pizza_names pn

ON co.pizza_id = pn.pizza_id

JOIN pizza_recipes pr

ON co.pizza_id = pr.pizza_id

JOIN pizza_toppings pt

ON CAST(pt.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(pr.toppings, ', '))

GROUP BY co.order_id, pn.pizza_name, co.extras

ORDER BY co.order_id;

![](/images/sun_Image_6.png)

6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

SELECT  pt.topping_name,

COUNT(*)

+ COUNT(CASE

WHEN CAST(pt.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(co.extras, ', '))

THEN 1

ELSE NULL

END)

- COUNT(CASE

WHEN CAST(pt.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(co.exclusions, ', '))

THEN 1

ELSE NULL

END)

AS total_quantity

FROM customer_orders co

JOIN runner_orders ro  ON co.order_id = ro.order_id

JOIN pizza_recipes pr ON co.pizza_id = pr.pizza_id

JOIN pizza_toppings pt ON CAST(pt.topping_id AS TEXT) = ANY (STRING_TO_ARRAY(pr.toppings, ', '))

WHERE ro.cancellation IS NULL  -- Only consider delivered pizzas

GROUP BY pt.topping_name HAVING COUNT(*) > 0

ORDER BY total_quantity DESC;

![](/images/8fU_Image_7.png)
