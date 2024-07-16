**Case Study 1: Danny?s Diner**

![](/images/98j_Image_1.png)

1. What is the total amount each customer spent at the restaurant?

SELECT s.customer_id, sum(m.price)

FROM dannys_diner.sales AS s JOIN dannys_diner.menu AS m

ON s.product_id = m.product_id GROUP BY s.customer_id

![](/images/4Qu_Image_2.png)

2. How many days has each customer visited the restaurant?

SELECT s.customer_id, count(distinct s.order_date)

FROM dannys_diner.sales AS s GROUP BY s.customer_id

![](/images/8DS_Image_3.png)

3. What was the first item from the menu purchased by each customer?

WITH ordered_sales AS ( SELECT   sales.customer_id,  sales.order_date, menu.product_name,

DENSE_RANK() OVER( PARTITION BY sales.customer_id

ORDER BY sales.order_date) AS rank

FROM dannys_diner.sales

JOIN dannys_diner.menu

ON sales.product_id = menu.product_id

)

SELECT customer_id,  product_name FROM ordered_sales

WHERE rank = 1 GROUP BY customer_id, product_name

![](/images/cwb_Image_4.png)

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT m.product_name, COUNT(s.product_id) AS most_purchased_item

FROM dannys_diner.sales AS s

JOIN dannys_diner.menu AS m  ON s.product_id = m.product_id

GROUP BY m.product_name

ORDER BY most_purchased_item DESC

LIMIT 1

![](/images/N8v_Image_5.png)

5. Which item was the most popular for each customer?

WITH most_popular AS (

SELECT  sales.customer_id, menu.product_name,    COUNT(menu.product_id) AS order_count,

DENSE_RANK() OVER( PARTITION BY sales.customer_id

ORDER BY COUNT(sales.customer_id) DESC) AS rank

FROM dannys_diner.menu

JOIN dannys_diner.sales

ON menu.product_id = sales.product_id

GROUP BY sales.customer_id, menu.product_name

)

SELECT customer_id, product_name, order_count

FROM most_popular

WHERE rank = 1;

![](/images/06w_Image_6.png)

6. Which item was purchased first by the customer after they became a member?

WITH joined_as_member AS (

SELECT members.customer_id, sales.product_id,

ROW_NUMBER() OVER( PARTITION BY members.customer_id

ORDER BY sales.order_date) AS row_num

FROM dannys_diner.members

JOIN dannys_diner.sales

ON members.customer_id = sales.customer_id

AND sales.order_date > members.join_date

)

SELECT

customer_id,  product_name

FROM joined_as_member

JOIN dannys_diner.menu

ON joined_as_member.product_id = menu.product_id

WHERE row_num = 1

ORDER BY customer_id ASC;

![](/images/BuT_Image_7.png)

7. Which item was purchased just before the customer became a member?

WITH purchased_prior_member AS

(

SELECT  S.customer_id, M.product_name, S.Order_date, 		  DENSE_RANK() OVER (PARTITION BY S.Customer_id ORDER BY S.Order_date) AS Rank

FROM dannys_diner.Sales S

JOIN dannys_diner.Menu M

ON m.product_id = s.product_id

JOIN dannys_diner.Members Mem

ON Mem.Customer_id = S.customer_id

WHERE S.order_date < Mem.join_date

)

SELECT customer_ID, Product_name

FROM purchased_prior_member WHERE Rank = 1;

![](/images/3uO_Image_8.png)

8. What is the total items and amount spent for each member before they became a member?

SELECT sales.customer_id, COUNT(sales.product_id) AS total_items,

SUM(menu.price) AS total_sales

FROM dannys_diner.sales

JOIN dannys_diner.members

ON sales.customer_id = members.customer_id

JOIN dannys_diner.menu

ON sales.product_id = menu.product_id

WHERE sales.order_date < members.join_date

GROUP BY sales.customer_id

ORDER BY sales.customer_id;

![](/images/RmU_Image_9.png)

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

WITH points_cte AS (

SELECT

menu.product_id, CASE  WHEN product_id = 1 THEN price * 20

ELSE price * 10

END AS points

FROM dannys_diner.menu

)

SELECT

sales.customer_id, SUM(points_cte.points) AS total_points

FROM dannys_diner.sales

JOIN points_cte

ON sales.product_id = points_cte.product_id

GROUP BY sales.customer_id

ORDER BY sales.customer_id;

![](/images/Huc_Image_10.png)

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

WITH dates_cte AS (

SELECT customer_id, join_date,  join_date + 6 AS valid_date,

DATE_TRUNC( 'month', '2021-01-31'::DATE)

+ interval '1 month'  -  interval '1 day' AS last_date

FROM dannys_diner.members

)

SELECT

sales.customer_id,   SUM(CASE

WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price

WHEN sales.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 * menu.price

ELSE 10 * menu.price END) AS points

FROM dannys_diner.sales

JOIN dates_cte AS dates

ON sales.customer_id = dates.customer_id

AND sales.order_date <= dates.last_date

JOIN dannys_diner.menu

ON sales.product_id = menu.product_id

GROUP BY sales.customer_id;

![](/images/HRp_Image_11.png)
