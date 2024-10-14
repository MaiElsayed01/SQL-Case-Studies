## 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT s.customer_id AS [Customer ID], SUM(m.price) AS [Total Price]
FROM sales AS s
INNER JOIN menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id;