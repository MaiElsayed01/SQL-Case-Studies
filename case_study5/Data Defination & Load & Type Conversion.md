#### Step 1: Prepare the Data
- The data was initially copied into a CSV file, followed by preprocessing using a Bash script to clean and standardize the format.

#### Step 2: Create Database and Table
```sql
CREATE DATABASE data_mart;

USE data_mart;

CREATE TABLE weekly_sales (
  week_date VARCHAR(12),
  region VARCHAR(20),
  platform VARCHAR(13),
  segment VARCHAR(7),
  customer_type VARCHAR(10),
  transactions VARCHAR(20),
  sales VARCHAR(30)
);

```

#### Step 3: Load Data from CSV File
```sql
BULK INSERT weekly_sales
FROM 'path_to_your_file\c_data_mart.csv'
WITH
(
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '0x0a',
    FIRSTROW = 1,
    BATCHSIZE = 1000,
    TABLOCK,
    DATAFILETYPE = 'char',
    KEEPNULLS,
    MAXERRORS = 1000
);
```

#### Step 4: Convert Column Data Types

- sales column
```sql
alter table weekly_sales alter column sales int;
```
- transactions column
```sql
alter table weekly_sales alter column transactions int;
```
- week_date column
```sql
-- create new column for date
ALTER TABLE weekly_sales 
ADD c_week_date DATE NULL;
-- add the data from week_date after conversion
UPDATE weekly_sales
SET c_week_date = TRY_CONVERT(DATE, week_date, 3);
-- remove week_date
ALTER TABLE weekly_sales DROP COLUMN week_date;
```