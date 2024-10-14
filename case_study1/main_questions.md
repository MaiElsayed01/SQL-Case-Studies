## 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT s.customer_id AS [Customer ID], SUM(m.price) AS [Total Price]
FROM sales AS s
INNER JOIN menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```
+-------------+-------------+
| Customer ID | Total Price |
+-------------+-------------+
| A           | 76          |
| B           | 74          |
| C           | 36          |
+-------------+-------------+

## 2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id AS [Customer ID], COUNT(DISTINCT order_date) AS [Days]
FROM sales 
GROUP BY customer_id;
```
+-------------+------+
| Customer ID | Days |
+-------------+------+
| A           | 4    |
| B           | 6    |
| C           | 2    |
+-------------+------+

## 3.What was the first item from the menu purchased by each customer?

```sql
with FirstOrder as(
	select 
		s.customer_id,
		s.product_id,
		ROW_NUMBER() over (partition by s.customer_id order by s.order_date) as rank
	from sales s
)
select 
	d.customer_id as [Customer ID],
	m.product_name as [First Purchased Product]
from FirstOrder as d
inner join menu as m
on d.product_id = m.product_id
where d.rank = 1;
```
+-------------+------------------------+
| Customer ID | First Purchased Product |
+-------------+------------------------+
| A           | sushi                  |
| B           | curry                  |
| C           | ramen                  |
+-------------+------------------------+

## 4.What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
select 
	top 1
	m.product_name Product,
	count(s.product_id) as Purchased
from sales s
inner join menu m
on m.product_id = s.product_id
group by m.product_name
order by Purchased desc;
```

+------------------------+----------+
| Product                | Purchased|
+------------------------+----------+
| ramen                  | 8        |
+------------------------+----------+

## 5. Which item was the most popular for each customer?

```sql
WITH product_rank AS (
    SELECT 
        customer_id,
        product_id,
        COUNT(*) AS product_count,
        RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(*) DESC) AS rank
    FROM sales
    GROUP BY customer_id, product_id
)
SELECT 
    p.customer_id AS Customer, 
    m.product_name AS Product,
    p.product_count AS Count
FROM product_rank p
INNER JOIN menu m ON m.product_id = p.product_id
WHERE rank = 1;
```
+-------------+------------------------+-------+
| Customer    | Product                | Count |
+-------------+------------------------+-------+
| A           | ramen                  | 3     |
| B           | sushi                  | 2     |
| B           | curry                  | 2     |
| B           | ramen                  | 2     |
| C           | ramen                  | 3     |
+-------------+------------------------+-------+

## 6. Which item was purchased first by the customer after they became a member?

```sql
WITH ranked_orders AS (
    SELECT 
        m.customer_id,
        s.product_id,
        s.order_date,
        m.join_date,
        ROW_NUMBER() OVER (PARTITION BY m.customer_id ORDER BY s.order_date) AS rank
    FROM member m
    INNER JOIN sales s ON s.customer_id = m.customer_id
    WHERE s.order_date >= m.join_date
)
SELECT 
    r.customer_id,
    m.product_name,
    r.order_date,
    r.join_date
FROM ranked_orders r, menu m
WHERE r.product_id = m.product_id AND r.rank = 1;
```
+-------------+------------------------+------------+------------+
| Customer ID | Product                | Order Date | Join Date  |
+-------------+------------------------+------------+------------+
| A           | curry                  | 2021-01-07 | 2021-01-07 |
| B           | sushi                  | 2021-01-11 | 2021-01-09 |
+-------------+------------------------+------------+------------+

## 7. Which item was purchased just before the customer became a member?

```sql
WITH ranked_orders AS (
    SELECT 
        s.customer_id,
        product_id,
        order_date,
        join_date,
        ROW_NUMBER() OVER (PARTITION BY m.customer_id ORDER BY s.order_date DESC) AS rank
    FROM member m, sales s
    WHERE s.customer_id = m.customer_id
    AND s.order_date < m.join_date
)
SELECT 
    r.customer_id,
    m.product_name,
    r.order_date,
    r.join_date
FROM ranked_orders r, menu m
WHERE r.product_id = m.product_id 
AND r.rank = 1;
```
+-------------+------------------------+------------+------------+
| Customer ID | Product                | Order Date | Join Date  |
+-------------+------------------------+------------+------------+
| A           | sushi                  | 2021-01-01 | 2021-01-07 |
| B           | sushi                  | 2021-01-04 | 2021-01-09 |
| C           | ramen                  | 2021-01-07 | 2021-01-25 |
+-------------+------------------------+------------+------------+

## 8. What is the total items and amount spent for each member before they became a member?

```sql
WITH before_join_orders AS (
    SELECT 
        m.customer_id,
        s.product_id,
        s.order_date,
        m.join_date
    FROM member m
    INNER JOIN sales s ON s.customer_id = m.customer_id
    WHERE s.order_date < m.join_date
)
SELECT 
    b.customer_id AS Customer,
    COUNT(*) AS Products,
    SUM(m.price) AS [Total Price]
FROM before_join_orders b, menu m
WHERE b.product_id = m.product_id
GROUP BY b.customer_id;
```
+-------------+----------+------------+
| Customer    | Products | Total Price|
+-------------+----------+------------+
| A           | 2        | 25         |
| B           | 3        | 40         |
| C           | 3        | 36         |
+-------------+----------+------------+

## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT 
    customer_id,
    SUM( 
        CASE
            WHEN (s.product_id = 1) THEN (m.price * 20)  -- Sushi has a 2x points multiplier
            ELSE (m.price * 10)                          -- Standard points calculation
        END
    ) AS Points
FROM sales s, menu m
WHERE s.product_id = m.product_id
GROUP BY customer_id;
```
+-------------+--------+
| Customer ID | Points |
+-------------+--------+
| A           | 860    |
| B           | 940    |
| C           | 360    |
+-------------+--------+

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items - how many points do customer A and B have at the end of January?

```sql
WITH valid_dates AS (
    SELECT *,
        DATEADD(DAY, 6, m.join_date) AS valid_date,
        EOMONTH('2021-01-01') AS last_date
    FROM member m
)
SELECT 
    d.customer_id,
    SUM( 
        CASE
            WHEN (s.product_id = 1) THEN (m.price * 20)                    -- Sushi has a 2x points multiplier
            WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN (m.price * 20)  -- All items have a 2x points multiplier in the first week
            ELSE (m.price * 10)                                            -- Standard points calculation
        END
    ) AS Points
FROM valid_dates d, menu m, sales s
WHERE d.customer_id = s.customer_id AND s.product_id = m.product_id 
GROUP BY d.customer_id;
```
+-------------+--------+
| Customer ID | Points |
+-------------+--------+
| A           | 1370   |
| B           | 940    |
| C           | 360    |
+-------------+--------+

## Bonus: Join All The Things

```sql
WITH join_all AS (
    SELECT 
        mm.customer_id AS Customer,
        s.order_date AS [Order Date],
        m.product_name AS Product,
        m.price AS Price,
        CASE
            WHEN s.order_date < mm.join_date THEN 'N'  -- Not a member
            ELSE 'Y'                                    -- Member
        END AS Member
    FROM sales s, menu m, member mm
    WHERE s.customer_id = mm.customer_id 
    AND s.product_id = m.product_id
)

SELECT * FROM join_all;
```
+-------------+------------+----------------+-------+--------+
| Customer ID | Order Date | Product        | Price | Member |
+-------------+------------+----------------+-------+--------+
| A           | 2021-01-01 | sushi          | 10    | N      |
| A           | 2021-01-01 | curry          | 15    | N      |
| A           | 2021-01-07 | curry          | 15    | Y      |
| A           | 2021-01-10 | ramen          | 12    | Y      |
| A           | 2021-01-11 | ramen          | 12    | Y      |
| A           | 2021-01-11 | ramen          | 12    | Y      |
| B           | 2021-01-01 | curry          | 15    | N      |
| B           | 2021-01-02 | curry          | 15    | N      |
| B           | 2021-01-04 | sushi          | 10    | N      |
| B           | 2021-01-11 | sushi          | 10    | Y      |
| B           | 2021-01-16 | ramen          | 12    | Y      |
| B           | 2021-02-01 | ramen          | 12    | Y      |
| C           | 2021-01-01 | ramen          | 12    | N      |
| C           | 2021-01-01 | ramen          | 12    | N      |
| C           | 2021-01-07 | ramen          | 12    | N      |
+-------------+------------+----------------+-------+--------+

## Bonus: Rank All The Things

```sql
WITH join_all AS (
    SELECT 
        mm.customer_id, 
        s.order_date, 
        m.product_name, 
        m.price, 
        CASE
            WHEN s.order_date < mm.join_date THEN 'N'  -- Not a member
            ELSE 'Y'                                    -- Member
        END AS member
    FROM sales s, menu m, member mm
    WHERE s.customer_id = mm.customer_id 
    AND s.product_id = m.product_id
)

SELECT *,
    CASE
        WHEN a.member = 'N' THEN NULL
        ELSE DENSE_RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date)
    END AS Ranking
FROM join_all a;
```
+-------------+------------+----------------+-------+--------+--------+
| Customer ID | Order Date | Product        | Price | Member | Ranking|
+-------------+------------+----------------+-------+--------+--------+
| A           | 2021-01-01 | sushi          | 10    | N      | NULL   |
| A           | 2021-01-01 | curry          | 15    | N      | NULL   |
| A           | 2021-01-07 | curry          | 15    | Y      | 1      |
| A           | 2021-01-10 | ramen          | 12    | Y      | 2      |
| A           | 2021-01-11 | ramen          | 12    | Y      | 3      |
| A           | 2021-01-11 | ramen          | 12    | Y      | 3      |
| B           | 2021-01-01 | curry          | 15    | N      | NULL   |
| B           | 2021-01-02 | curry          | 15    | N      | NULL   |
| B           | 2021-01-04 | sushi          | 10    | N      | NULL   |
| B           | 2021-01-11 | sushi          | 10    | Y      | 1      |
| B           | 2021-01-16 | ramen          | 12    | Y      | 2      |
| B           | 2021-02-01 | ramen          | 12    | Y      | 3      |
| C           | 2021-01-01 | ramen          | 12    | N      | NULL   |
| C           | 2021-01-01 | ramen          | 12    | N      | NULL   |
| C           | 2021-01-07 | ramen          | 12    | N      | NULL   |
+-------------+------------+----------------+-------+--------+--------+