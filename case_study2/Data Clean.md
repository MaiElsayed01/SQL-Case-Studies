### RunnersOrders Table Clean

```sql
UPDATE RunnersOrders
SET cancellation = null
WHERE cancellation = '' or cancellation = 'null';

UPDATE RunnersOrders
SET duration =
	CASE
		WHEN duration = 'null' THEN null
		WHEN PATINDEX('%[^0-9]%' ,duration) = 0 THEN CAST(duration AS INT)
		ELSE CAST(SUBSTRING(duration, 1, PATINDEX('%[^0-9]%', duration) - 1) AS INT)
		END;

UPDATE RunnersOrders
SET distance =
	CASE
		WHEN distance = 'null' THEN null
		WHEN PATINDEX('%[^0-9.]%' ,distance) = 0 THEN CAST(distance AS FLOAT)
		ELSE CAST(SUBSTRING(distance, 1, PATINDEX('%[^0-9]%', distance) - 1) AS FLOAT)
		END;

UPDATE RunnersOrders
SET pickup_time = null
WHERE pickup_time = 'null';

ALTER TABLE RunnersOrders
ALTER COLUMN pickup_time DATETIME;

ALTER TABLE RunnersOrders
ALTER COLUMN distance FLOAT;

ALTER TABLE RunnersOrders
ALTER COLUMN duration INT;
```
| before | after |
|--------|-------|
| ![1](https://github.com/user-attachments/assets/16f57dc7-ab83-40ee-b496-3475c61d3c6f) | ![2](https://github.com/user-attachments/assets/8c1a44c5-d934-4c75-8c87-646c60b375a3) |

### CustomerOrder Table Clean

```sql
UPDATE CustomerOrder
SET exclusions = null
WHERE exclusions = '' or exclusions = 'null';

UPDATE CustomerOrder
SET extras = null
WHERE extras = '' or extras = 'null';

ALTER TABLE CustomerOrder
ALTER COLUMN order_time DATETIME;

ALTER TABLE CustomerOrder
ADD record_id INT IDENTITY(1,1) PRIMARY KEY;
```
| before | after |
|--------|-------|
| ![3](https://github.com/user-attachments/assets/a760a26d-7fa1-496e-9648-bcf43901a61a) | ![4](https://github.com/user-attachments/assets/edb092ae-9fb1-4d7a-bf93-0a15a1118da0) |

### PizzaName Table

```sql
ALTER TABLE PizzaName
ALTER COLUMN pizza_name VARCHAR(max);
```

### PizzaRecieps Table

```sql
ALTER TABLE PizzaRecieps
ALTER COLUMN toppings VARCHAR(max);
```

### Toppings Table 

```sql
ALTER TABLE Toppings
ALTER COLUMN topping_name VARCHAR(max);
```