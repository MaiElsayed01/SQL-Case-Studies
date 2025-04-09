## 1.If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```sql
SELECT  
	SUM(
		CASE
			WHEN pizza_id = 1 THEN 12
			ELSE 10
			END
		) Profit
FROM CustomerOrder c, RunnersOrders r
WHERE 
	c.order_id = r.order_id
	AND
	cancellation IS NULL;
```
![1](https://github.com/user-attachments/assets/9b8593b0-6e6f-4322-a2a2-fca407351ad9)

## 2.What if there was an additional $1 charge for any pizza extras?Add cheese is $1 extra

```sql
SELECT  
	SUM(
		CASE
			WHEN pizza_id = 1 THEN 12
			ELSE 10
			END
		 + 
		CASE 
			WHEN extras IS NOT NULL THEN LEN(REPLACE(extras,',',''))
			ELSE 0
			END
		) Revenue
FROM CustomerOrder c, RunnersOrders r
WHERE 
	c.order_id = r.order_id
	AND
	cancellation IS NULL;
```
![2](https://github.com/user-attachments/assets/e3c4a3a7-d3b1-4df1-90bc-1e0a8bf31929)

## 3.The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```sql
CREATE TABLE Rating(
	rate_id INT PRIMARY KEY IDENTITY(1,1)
	,customer_id INT
	,runner_id INT
	,order_id INT
	,rate_date DATE DEFAULT GETDATE()
	,rate INT CHECK (rate BETWEEN 1 AND 5)
);
```

## 5.If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```sql
SELECT  
	SUM(
		CASE
			WHEN pizza_id = 1 THEN 12
			ELSE 10
			END
		 + 
		  (distance * 0.3)
		) Revenue
FROM CustomerOrder c, RunnersOrders r
WHERE 
	c.order_id = r.order_id
	AND
	cancellation IS NULL;
```
![5](https://github.com/user-attachments/assets/a0546836-fd36-40c5-84f5-c986e77adb4c)
