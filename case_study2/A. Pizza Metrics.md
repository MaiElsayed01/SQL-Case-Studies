## 1.How many pizzas were ordered?
```sql
SELECT COUNT(pizza_id) [Ordered Pizza Count]
FROM CustomerOrder;
```
![1](https://github.com/user-attachments/assets/eef8f550-0282-4244-aca3-9474107aa91a)

## 2.How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT order_id) [Orders Count]
FROM CustomerOrder;
```
![2](https://github.com/user-attachments/assets/b5ce4409-4474-4314-8d30-76954142b015)

## 3.How many successful orders were delivered by each runner?
```sql
SELECT 
	runner_id Runner
	,COUNT(*) [Successful Orders]
FROM RunnersOrders
WHERE cancellation IS NULL
GROUP BY runner_id;
```
![3](https://github.com/user-attachments/assets/b9726a28-b7a9-452a-bf4b-f099a4a82026)

## 4.How many of each type of pizza was delivered?
```sql
SELECT 
	p.pizza_name Pizza
	,COUNT(c.order_id) [Successful Orders]
FROM RunnersOrders r
INNER JOIN CustomerOrder c
ON r.order_id = c.order_id 
INNER JOIN PizzaName p
ON  p.pizza_id = c.pizza_id
WHERE r.cancellation IS NULL
GROUP BY p.pizza_name;
```
![4](https://github.com/user-attachments/assets/75bb8a5d-b7b9-44b1-9e89-57cb51570fa4)

## 5.How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT 
	customer_id Customer
	,pizza_name Pizza
	,COUNT(c.pizza_id) [Orders]
FROM 
	CustomerOrder c
	,PizzaName p
WHERE  p.pizza_id = c.pizza_id
GROUP BY pizza_name, customer_id;
```
![5](https://github.com/user-attachments/assets/8612e143-9479-4b17-a85c-3c5820c84d07)

## 6.What was the maximum number of pizzas delivered in a single order?
```sql
SELECT  TOP 1
	c.order_id
	,COUNT(c.pizza_id) No_Pizza
FROM 
	CustomerOrder c
	,RunnersOrders r
WHERE 
	r.order_id = c.order_id
	and cancellation IS NULL
GROUP BY c.order_id
ORDER BY 2 DESC;
```
![6](https://github.com/user-attachments/assets/408c7a78-5f28-45c7-aa68-21e1eb2a2efa)

## 7.For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT  
	customer_id
	,SUM(
		CASE	
			WHEN extras IS NOT NULL OR exclusions IS NOT NULL THEN 1
			ELSE 0
			END
	) [Changed Pizza]
	,SUM(
		CASE	
			WHEN extras IS NULL AND exclusions IS NULL THEN 1
			ELSE 0
			END
	) [Not Changed Pizza]
FROM 
	CustomerOrder c
	,RunnersOrders r
WHERE 
	r.order_id = c.order_id
	AND 
	cancellation IS NULL
GROUP BY customer_id;
```
![7](https://github.com/user-attachments/assets/f66f880b-2a0b-4eca-b4e1-7a6672ab0490)

## 8.How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT 
	COUNT(*) [Pizza With Exclusions and Extras]
FROM RunnersOrders r, CustomerOrder c
WHERE 
	r.order_id = c.order_id
	AND
	exclusions IS NOT NULL
	AND
	extras IS NOT NULL;
```
![8](https://github.com/user-attachments/assets/baf76bc6-e29f-42dc-85a5-8695d1e59f52)

## 9.What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT 
		DATEPART(HOUR, order_time) hour
		,COUNT(order_id) order_count
FROM CustomerOrder
GROUP BY DATEPART(HOUR, order_time);
```
![9](https://github.com/user-attachments/assets/f0224117-a8f2-49a4-b21b-752c86ad24e6)

## 10.What was the volume of orders for each day of the week?
```sql
SELECT 
	FORMAT(order_time, 'dddd') Days
	,COUNT(order_id) Orders
FROM CustomerOrder
GROUP BY FORMAT(order_time, 'dddd');
```
![9](https://github.com/user-attachments/assets/3592385a-de69-4c5d-bc20-8e4459ca1805)