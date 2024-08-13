🍕 Case Study #2 Pizza Runner

Solution - B. Runner and Customer Experience

--1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
SELECT
	date_part('week', registration_date +3 ) AS week_nr,
	count(runner_id)
FROM runners
GROUP BY week_nr
ORDER BY week_nr ASC

--2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

SELECT
	ro.runner_id,
	AVG(pickup_time - order_time)
	from runner_orders_clean AS ro
JOIN customer_orders_clean AS co
ON ro.order_id = co.order_id
WHERE distance iS NOT NULL
GROUP BY runner_id

--3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

WITH pizza_rel AS( 

SELECT
	ro.order_id,
	(pickup_time - order_time) AS del,
	count(ro.order_id) AS pizza_count
	from runner_orders_clean AS ro
JOIN customer_orders_clean AS co
ON ro.order_id = co.order_id
WHERE distance iS NOT NULL
GROUP BY ro.order_id, pickup_time, order_time
)

SELECT
pizza_count,
AVG(del)
FROM pizza_rel
GROUP BY pizza_count

--.4  What was the average distance traveled for each customer?



SELECT 
	co.customer_id,
	(ROUND(AVG(distance)))
FROM customer_orders_clean AS co
JOIN runner_orders_clean AS ro
ON co.order_id = ro.order_id
WHERE duration IS NOT NULL
GROUP BY co.customer_id



