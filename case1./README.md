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

