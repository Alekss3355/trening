-- 1. What is the total amount each customer spent at the restaurant?


SELECT 
		customer_id,
		SUM(menu.price)
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY customer_id

-- 2. How many days has each customer visited the restaurant?

SELECT 
	customer_id, 
    COUNT(DISTINCT order_date) AS days_count
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY days_count DESC

-- 3. What was the first item from the menu purchased by each customer?

WITH CTE_DATE AS(
 SELECT
  sales.customer_id,
  sales.order_date,
  menu.product_name,
  DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) AS date_rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
 )
SELECT customer_id, product_name FROM CTE_DATE
WHERE date_rank = 1
GROUP BY customer_id, product_name

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT COUNT(*) as Count, menu.product_name FROM dannys_diner.sales
JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY COUNT(*) DESC
LIMIT 1

-- 5. Which item was the most popular for each customer?

WITH popular_item AS(
  SELECT
  sales.customer_id,
  menu.product_name,
  COUNT(*) as item_count,
  DENSE_RANK() OVER(PARTITION BY sales.customer_id
  ORDER BY COUNT(sales.product_id))
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
  GROUP BY sales.customer_id, menu.product_name
)
SELECT * FROM popular_item
WHERE DENSE_RANK = 1

-- 6. Which item was purchased first by the customer after they became a member?

WITH first_item AS(
  SELECT
  sales.customer_id,
  members.join_date,
  menu.product_name,
  sales.order_date,
  DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY order_date ASC)
  FROM dannys_diner.sales
  JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  AND order_date > join_date
  JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id
  )
SELECT * FROM first_item
WHERE DENSE_RANK = 1

-- 7. Which item was purchased just before the customer became a member?

WITH first_item AS(
  SELECT
  sales.customer_id,
  members.join_date,
  menu.product_name,
  sales.order_date,
  DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY order_date DESC)
  FROM dannys_diner.sales
  JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  AND sales.order_date < members.join_date
  JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id
  )
SELECT * FROM first_item
WHERE DENSE_RANK = 1


-- 8. What is the total items and amount spent for each member before they became a member?

WITH before_being_member AS (
  SELECT
  sales.customer_id,
  menu.product_name,
  sales.product_id,
  menu.price
  FROM dannys_diner.sales
  JOIN dannys_diner.members 
  ON sales.customer_id = members.customer_id
  JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
  WHERE sales.order_date < members.join_date
)
  
SELECT 
before_being_member.customer_id,
count(before_being_member.product_id),
sum(before_being_member.price)
FROM before_being_member
GROUP BY before_being_member.customer_id
ORDER BY before_being_member.customer_id ASC

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?


WITH before_being_member AS (
  SELECT
  sales.customer_id,
  menu.product_name,
  sales.product_id,
  CASE 
   WHEN sales.product_id = 1 THEN 20 * menu.price
   ELSE 10 * menu.price
   END AS points,
  menu.price
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
)
  
SELECT 
before_being_member.customer_id,
count(before_being_member.product_id),
sum(before_being_member.points)
FROM before_being_member
GROUP BY before_being_member.customer_id
ORDER BY before_being_member.customer_id ASC

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


   sales.customer_id,
    CASE
     WHEN order_date BETWEEN dates.join_date AND dates.valid_date 
      THEN 2 * 10 * menu.price
     ELSE 10 * menu.price
     END AS points
    FROM dannys_diner.sales
    JOIN dates
    ON sales.customer_id = dates.customer_id 
    JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id


    chujowe to u gory  to nadole tez

    WITH DATES AS(
  SELECT
  sales.customer_id,
  members.join_date + 6 AS valid_date,
  members.join_date
  FROM dannys_diner.sales
  JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
) 
SELECT 
 sales.customer_id,
    SUM(CASE
     WHEN order_date BETWEEN dates.join_date AND dates.valid_date 
      THEN 2 * 10 * menu.price
     ELSE 10 * menu.price
     END ) AS points
    FROM dannys_diner.sales
    JOIN dates
    ON sales.customer_id = dates.customer_id 
    JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
    GROUP BY sales.customer_id\






