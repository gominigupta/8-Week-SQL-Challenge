**Case Study 3: Foodie-Fi**

![](/images/bHC_Image_1.png)

### **A. Customer Journey**

Based off the 8 sample customers provided in the sample from the `subscriptions` table, write a brief description about each customer?s onboarding journey. Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

WITH plan_with_next AS (

SELECT s.customer_id, s.plan_id, p.plan_name, s.start_date,

LEAD(s.start_date) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS next_plan_date

FROM foodie_fi.subscriptions s

JOIN foodie_fi.plans p

ON s.plan_id = p.plan_id )

SELECT customer_id, plan_id, plan_name, start_date,

COALESCE(next_plan_date - start_date, NULL) AS duration_days,

CASE

WHEN next_plan_date IS NULL THEN 'Latest'

ELSE CONCAT(next_plan_date - start_date, ' days')

END AS status

FROM plan_with_next

WHERE  customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)

ORDER BY customer_id, start_date;

![](/images/w7n_Image_2.png)

- **Customer 1: ***Churned after trial & brief Basic attempt*

Started with a 7-day trial on 2020-08-01, immediately moved to Basic Monthly for 0 days (likely canceled right away), and has no active plan currently.

- **Customer 2: ***Churned after trial*

Started with a 7-day trial on 2020-09-20, but never transitioned to any paid plan and has no active subscription.

- **Customer 11: ***Churned after Pro Monthly*

Started with a 7-day trial on 2020-11-11, upgraded to Basic Monthly for 28 days, then switched to Pro Monthly for 83 days.

- **Customer 13: ***Active on Pro Monthly*

Started with a 7-day trial on 2020-12-15, upgraded to Basic Monthly for 96 days, and then switched to Pro Monthly on 2021-03-29, which is their current active plan.

- **Customer 15: ***Churned after Pro Monthly*

Started with a 7-day trial on 2020-10-10, transitioned to Basic Monthly for 72 days, then upgraded to Pro Monthly for 76 days, and eventually churned.

- **Customer 16: ***Active on Pro Monthly*

Started with a 7-day trial on 2020-12-01, upgraded to Basic Monthly for 86 days, and then moved to Pro Monthly on 2021-02-17, which is their current active plan.

- **Customer 18: ***Churned after Pro Monthly*

Started with a 7-day trial on 2020-12-10, upgraded to Basic Monthly for 53 days, then switched to Pro Monthly for 94 days, and eventually churned.

- **Customer 19: ***Churned after Pro Monthly*

Started with a 7-day trial on 2020-12-20, upgraded to Basic Monthly for 66 days, then switched to Pro Monthly for 61 days, and eventually churned.

### **B. Data Analysis Questions**

1. How many customers has Foodie-Fi ever had?

SELECT COUNT(DISTINCT customer_id) AS total_customers

FROM foodie_fi.subscriptions;

![](/images/xES_Image_3.png)

2. What is the monthly distribution of `trial` plan `start_date` values for our dataset - use the start of the month as the group by value

SELECT  TO_CHAR(DATE_TRUNC('month', start_date), 'YYYY-MM-DD') AS month_start,  COUNT(*)

FROM  foodie_fi.subscriptions s

JOIN  foodie_fi.plans p ON s.plan_id = p.plan_id

WHERE p.plan_name = 'trial'

GROUP BY month_start

ORDER BY month_start;

![](/images/EEb_Image_4.png)

3. What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`

SELECT  p.plan_name, COUNT(*) AS event_count

FROM foodie_fi.subscriptions s

JOIN foodie_fi.plans p

ON s.plan_id = p.plan_id

WHERE s.start_date >= '2021-01-01'

GROUP BY p.plan_name

ORDER BY event_count DESC;

![](/images/ZYe_Image_5.png)

4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

WITH latest_plan AS ( SELECT customer_id, plan_id, ROW_NUMBER() OVER

(PARTITION BY customer_id ORDER BY start_date DESC) AS rn

FROM foodie_fi.subscriptions),

churned_customers AS (SELECT COUNT(*) AS churned_count

FROM  latest_plan lp

JOIN foodie_fi.plans p

ON lp.plan_id = p.plan_id

WHERE lp.rn = 1 AND p.plan_name = 'churn' ),

total_customers AS ( SELECT COUNT(DISTINCT customer_id) AS total_count

FROM foodie_fi.subscriptions)

SELECT  churned_count,

ROUND(100.0 * churned_count / total_count, 1) AS churned_percentage

FROM churned_customers, total_customers;

![](/images/qGK_Image_6.png)

5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

WITH customer_plans AS (SELECT customer_id, plan_id,

ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS plan_order

FROM foodie_fi.subscriptions),

churned_after_trial AS (SELECT cp.customer_id

FROM customer_plans cp

JOIN foodie_fi.plans p1

ON cp.plan_id = p1.plan_id AND cp.plan_order = 1

JOIN customer_plans cp2

ON cp.customer_id = cp2.customer_id AND cp2.plan_order = 2

JOIN foodie_fi.plans p2

ON cp2.plan_id = p2.plan_id

WHERE p1.plan_name = 'trial' AND p2.plan_name = 'churn'),

total_customers AS (SELECT COUNT(DISTINCT customer_id) AS total_cust

FROM foodie_fi.subscriptions)

SELECT COUNT(*) AS churned_after_trial_count,

ROUND(100.0 * COUNT(DISTINCT cat.customer_id) / MAX(tc.total_cust)) AS churned_after_trial_percentage

FROM churned_after_trial cat

CROSS JOIN total_customers tc;

![](/images/tg9_Image_7.png)

6. What is the number and percentage of customer plans after their initial free trial?

WITH customer_plans AS (SELECT customer_id, plan_id,

ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS plan_order

FROM foodie_fi.subscriptions),

converted_after_trial AS (SELECT cp.customer_id

FROM customer_plans cp

JOIN foodie_fi.plans p1

ON cp.plan_id = p1.plan_id AND cp.plan_order = 1

JOIN customer_plans cp2

ON cp.customer_id = cp2.customer_id AND cp2.plan_order = 2

JOIN foodie_fi.plans p2

ON cp2.plan_id = p2.plan_id

WHERE p1.plan_name = 'trial' AND p2.plan_name != 'churn'),

total_customers AS (SELECT COUNT(DISTINCT customer_id) AS total_cust

FROM foodie_fi.subscriptions)

SELECT COUNT(*) AS converted_after_trial_count,

ROUND(100.0 * COUNT(DISTINCT cat.customer_id) / MAX(tc.total_cust)) AS converted_after_trail_percentage

FROM converted_after_trial cat

CROSS JOIN total_customers tc;

![](/images/zYo_Image_8.png)

7. What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`?

WITH customer_latest_plan AS ( SELECT customer_id, plan_id, start_date

FROM ( SELECT customer_id, plan_id, start_date,

ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date

DESC) AS plan_order

FROM foodie_fi.subscriptions

WHERE start_date <= '2020-12-31' ) ranked_plans

WHERE plan_order = 1),

total_customers AS (SELECT COUNT(*) AS total_cust

FROM customer_latest_plan),

plan_breakdown AS ( SELECT p.plan_name,

COUNT(clp.customer_id) AS customer_count

FROM customer_latest_plan clp

JOIN foodie_fi.plans p ON clp.plan_id = p.plan_id

GROUP BY p.plan_id, p.plan_name

ORDER BY p.plan_id)

SELECT pb.plan_name, pb.customer_count,

ROUND(100.0 * pb.customer_count / tc.total_cust, 1) AS customer_percentage

FROM plan_breakdown pb CROSS JOIN total_customers tc;

![](/images/jvI_Image_9.png)

8. How many customers have upgraded to an annual plan in 2020?

WITH customer_plans AS (SELECT customer_id, plan_id, start_date,

ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS plan_order

FROM foodie_fi.subscriptions),

annual_in_2020 AS (SELECT customer_id

FROM customer_plans cp

JOIN foodie_fi.plans p ON cp.plan_id = p.plan_id

WHERE p.plan_name = 'pro annual'

AND EXTRACT(YEAR FROM cp.start_date) = 2020),

upgraded_to_annual AS (SELECT cp1.customer_id

FROM customer_plans cp1

JOIN customer_plans cp2

ON cp1.customer_id = cp2.customer_id

AND cp1.plan_order = cp2.plan_order - 1

JOIN foodie_fi.plans p1 ON cp1.plan_id = p1.plan_id

JOIN foodie_fi.plans p2 ON cp2.plan_id = p2.plan_id

WHERE p2.plan_name = 'pro annual'

AND EXTRACT(YEAR FROM cp2.start_date) = 2020

AND p1.plan_name IN ('trial', 'basic monthly', 'pro monthly'))

SELECT COUNT(DISTINCT customer_id) AS upgraded_to_annual_2020_count

FROM upgraded_to_annual;

![](/images/k1o_Image_10.png)

9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

WITH trial_start AS (SELECT customer_id, start_date AS trial_start_date

FROM foodie_fi.subscriptions s

JOIN foodie_fi.plans p ON s.plan_id = p.plan_id

WHERE p.plan_name = 'trial'),

first_pro_annual AS (SELECT customer_id, start_date AS pro_annual_date

FROM (SELECT s.customer_id, s.start_date,

ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS rn

FROM foodie_fi.subscriptions s

JOIN foodie_fi.plans p ON s.plan_id = p.plan_id

WHERE p.plan_name = 'pro annual') pa

WHERE rn = 1)

SELECT ROUND(AVG(fpa.pro_annual_date - ts.trial_start_date), 0) AS avg_days_to_annual

FROM trial_start ts JOIN first_pro_annual fpa ON ts.customer_id = fpa.customer_id;

![](/images/5nX_Image_11.png)

10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

WITH trial_start AS (SELECT customer_id, start_date AS trial_start_date

FROM foodie_fi.subscriptions s JOIN foodie_fi.plans p ON s.plan_id = p.plan_id

WHERE p.plan_name = 'trial'),

first_pro_annual AS (SELECT customer_id, start_date AS pro_annual_date

FROM (SELECT s.customer_id, s.start_date,

ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS rn

FROM foodie_fi.subscriptions s JOIN foodie_fi.plans p ON s.plan_id = p.plan_id

WHERE p.plan_name = 'pro annual') pa

WHERE rn = 1),

bucket_30_days AS (SELECT ts.customer_id,

(fpa.pro_annual_date - ts.trial_start_date) AS days_to_annual,

CASE WHEN (fpa.pro_annual_date - ts.trial_start_date) <= 30 THEN '0-30 days'

WHEN (fpa.pro_annual_date - ts.trial_start_date) <= 60 THEN '31-60 days'

WHEN (fpa.pro_annual_date - ts.trial_start_date) <= 90 THEN '61-90 days'

WHEN (fpa.pro_annual_date - ts.trial_start_date) <= 120 THEN '91-120 days'

WHEN (fpa.pro_annual_date - ts.trial_start_date) <= 150 THEN '121-150 days'

ELSE '150+ days'

END AS days_to_annual_bucket

FROM trial_start ts JOIN first_pro_annual fpa ON ts.customer_id = fpa.customer_id)

SELECT days_to_annual_bucket, COUNT(*) AS customer_count,

ROUND(100.0 * COUNT(*) / (SELECT COUNT(*) FROM bucket_30_days), 1) AS percentage

FROM bucket_30_days

GROUP BY days_to_annual_bucket ORDER BY MIN(days_to_annual);

![](/images/tip_Image_12.png)

11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

WITH customer_plans AS (SELECT customer_id, plan_id, start_date, ROW_NUMBER()

OVER (PARTITION BY customer_id ORDER BY start_date) AS plan_order

FROM foodie_fi.subscriptions),

pro_monthly_to_basic_monthly AS (SELECT cp1.customer_id

FROM customer_plans cp1 JOIN  customer_plans cp2

ON cp1.customer_id = cp2.customer_id  AND cp1.plan_order = cp2.plan_order - 1

JOIN foodie_fi.plans p1 ON cp1.plan_id = p1.plan_id

JOIN foodie_fi.plans p2 ON cp2.plan_id = p2.plan_id

WHERE p1.plan_name = 'pro monthly' AND p2.plan_name = 'basic monthly'

AND EXTRACT(YEAR FROM cp2.start_date) = 2020)

SELECT COUNT(DISTINCT customer_id) AS downgraded_to_basic_monthly_2020_count

FROM  pro_monthly_to_basic_monthly;

![](/images/3Me_Image_13.png)
