#### 1.What day of the week is used for each week_date value?
```sql
SELECT DISTINCT DATENAME(WEEKDAY, c_week_date) week_day
FROM weekly_sales;
```
![1](https://github.com/user-attachments/assets/9cd75bef-5acf-4891-8b20-5bdfdcac9451)

#### 2.What range of week numbers are missing from the dataset?
```sql
-- A YEAR HAVE 52 WEEKS, 1. GENERATE EACH YEAR WEEKS 2. GET WEEKS FROM TABLE 3. OUTPUT ONLY WEEKS THAT DOES NOT EXIST IN ALL WEEKS CTE

WITH all_weeks AS (
    SELECT 2018 AS year_number, 1 AS week_number
    UNION ALL
    SELECT 
        CASE WHEN week_number = 52 THEN year_number + 1 ELSE year_number END,
        CASE WHEN week_number =52 THEN 1 ELSE week_number + 1 END
    FROM all_weeks 
    WHERE year_number <= 2020 AND NOT (year_number = 2020 AND week_number = 52)
),
actual_weeks AS(
SELECT 
	year_number
	,DATEPART(WEEK, c_week_date) week_number 
FROM weekly_sales 
GROUP BY year_number, DATEPART(WEEK, c_week_date)
)
SELECT 
	a.year_number
	,a.week_number
FROM all_weeks a
LEFT JOIN actual_weeks c 
ON a.week_number = c.week_number AND a.year_number = c.year_number
WHERE c.week_number IS NULL
ORDER BY a.year_number, a.week_number
OPTION (MAXRECURSION 200);
```
![2.1](https://github.com/user-attachments/assets/01fb9705-f8d2-42bc-8d29-f5b51ac35867)

#### 3.How many total transactions were there for each year in the dataset?
```sql
SELECT year_number year, SUM(transactions) total_transactions
FROM weekly_sales
GROUP BY year_number
```
![3](https://github.com/user-attachments/assets/2aa66198-f059-4852-b12d-c64673bf215f)

#### 4.What is the total sales for each region for each month?
```sql
SELECT	
	region
	,month_number
	,SUM(CAST(sales AS BIGINT)) total_sales
FROM weekly_sales
GROUP BY region, month_number
ORDER BY region, month_number;
```
![4](https://github.com/user-attachments/assets/6c1cf850-188e-4e6d-bc34-bd71d490cd0a)

#### 5.What is the total count of transactions for each platform
```sql
SELECT platform, SUM(transactions) total_transactions
FROM weekly_sales
GROUP BY platform;
```
![5](https://github.com/user-attachments/assets/40991fe9-ce88-42a3-a1b0-239f2c0269d2)

#### 6.What is the total count of transactions for each platform
```sql
WITH TotalSales AS(
SELECT SUM(CAST(sales AS BIGINT)) total_sales FROM weekly_sales
)
SELECT 
	platform
	,DATENAME(MONTH, c_week_date) month
	,SUM(CAST(sales AS BIGINT)) platform_month_sales
	,CONCAT(CAST(ROUND(CAST(SUM(CAST(sales AS BIGINT)) AS FLOAT) / total_sales * 100, 2) AS VARCHAR), '%')  percentage
FROM weekly_sales
CROSS JOIN TotalSales
GROUP BY platform, DATENAME(MONTH, c_week_date), total_sales
ORDER BY platform;
```
![6](https://github.com/user-attachments/assets/d825dae5-7042-4f85-b468-d870f738c22c)

#### 7.What is the percentage of sales by demographic for each year in the dataset?
```sql
WITH TotalSalesByYear AS (
    SELECT 
        YEAR(c_week_date) AS year
        ,SUM(CAST(sales AS BIGINT)) AS total_sales_by_year
    FROM weekly_sales
    GROUP BY YEAR(c_week_date)
)
SELECT 
	YEAR(c_week_date) year
	,demographic
	,CONCAT(
		CAST(
			CAST(ROUND(100.0 * SUM(CAST(sales AS BIGINT)) / total_sales_by_year, 2) AS DECIMAL(5,2)) 
      AS VARCHAR), '%') AS percentage
FROM weekly_sales w
JOIN TotalSalesByYear t
ON t.year = w.year_number
GROUP BY demographic, YEAR(c_week_date), total_sales_by_year
ORDER BY year , demographic;

-- Another Solution

SELECT 
	YEAR(c_week_date) year
	,demographic
	,CONCAT(
    CAST(
        CAST(
            ROUND(100.0 * SUM(CAST(sales AS BIGINT)) / 
                SUM(SUM(CAST(sales AS BIGINT))) OVER (PARTITION BY YEAR(c_week_date))
            , 2) AS DECIMAL(5,2)
    ) AS VARCHAR), '%') AS percentage
FROM weekly_sales w
GROUP BY demographic, YEAR(c_week_date)
ORDER BY year , demographic;
```
![7](https://github.com/user-attachments/assets/35b0e6a1-ebf1-4915-840c-556f69ac1c07)

#### 8.Which age_band and demographic values contribute the most to Retail sales?
```sql
WITH DemographicSales AS(
    SELECT 
        'Top Demographic' AS category,
        demographic AS value,
        SUM(CAST(sales AS BIGINT)) AS total_sales
    FROM weekly_sales
    WHERE platform = 'Retail' 
    AND demographic != 'Unknown'
    GROUP BY demographic
),
AgeBandSales AS (
    SELECT 
        'Top Age Band' AS category,
        age_band AS value,
        SUM(CAST(sales AS BIGINT)) AS total_sales
    FROM weekly_sales
    WHERE platform = 'Retail' 
    AND demographic != 'Unknown'
    GROUP BY age_band
)
SELECT category, value, total_sales
FROM (
	SELECT category, value, total_sales, ROW_NUMBER() OVER(PARTITION BY category ORDER BY total_sales DESC) rank
	FROM(
		SELECT * FROM DemographicSales
		UNION ALL
		SELECT * FROM AgeBandSales
	) AS combined
) AS ranked
WHERE rank = 1;
```
![8](https://github.com/user-attachments/assets/a1579d3b-273b-4c92-bc7b-430dd9cca55a)

#### 9.Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
```sql
SELECT 
	YEAR(c_week_date) year
	,platform
	,SUM(CAST(sales AS BIGINT)) total_sales
	,SUM(transactions) total_transactions
	,SUM(CAST(sales AS BIGINT)) * 1.0 / SUM(transactions) avg_transactions_size
FROM weekly_sales
GROUP BY 
	YEAR(c_week_date)
	,platform
ORDER BY
	year, platform
```
![9](https://github.com/user-attachments/assets/67c10017-9063-4d2b-8748-4ed624ddcfb9)
