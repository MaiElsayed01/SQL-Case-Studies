#### Add a week_number as the second column for each week_date value
#### Add a month_number with the calendar month for each week_date value as the 3rd column
#### Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
```sql
ALTER TABLE weekly_sales
ADD
		week_number AS DATEPART(WEEK, c_week_date),
		month_number AS DATEPART(MONTH, c_week_date),
		year_number AS DATEPART(YEAR, c_week_date);
```

#### Add a new column called age_band after the original segment column
```sql
ALTER TABLE weekly_sales
ADD age_band AS (
					CASE WHEN RIGHT(segment, 1) = '1' THEN 'Young Adults' 
							WHEN  RIGHT(segment, 1) = '2' THEN 'Middle Aged'
							WHEN  RIGHT(segment, 1) = '3' or RIGHT(segment, 1) = '4' THEN 'Retirees'
							ELSE 'Unknown'
							END	);
```

#### Add a new demographic column
```sql
ALTER TABLE weekly_sales
ADD demographic AS (
			CASE WHEN LEFT(segment, 1) = 'F' THEN 'Families' 
				WHEN  LEFT(segment, 1) = 'C' THEN 'Couples'
				ELSE 'Unknown'
				END	
            );
```

#### Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
```sql
ALTER TABLE weekly_sales
ADD avg_transaction AS  CAST(sales * 0.1 / NULLIF(transactions, 0) AS DECIMAL(18, 2));
```

