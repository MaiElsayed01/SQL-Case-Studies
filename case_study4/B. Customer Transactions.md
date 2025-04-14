### 1. What is the unique count and total amount for each transaction type?

```sql
select 
	txn_type
	,sum(txn_amount) total_amout
	,count(*) total_transactions
from customer_transactions
group by txn_type;
```
![1](https://github.com/user-attachments/assets/b9956e05-35ae-45eb-be62-87d368201771)

### 2.What is the average total historical deposit counts and amounts for all customers?

```sql
select 
	'deposite' txn_type
	,sum(txn_amount) total_amout
	,count(*) total_transactions
from customer_transactions
where txn_type = 'deposit';
```

### 3.What is the average total historical deposit counts and amounts for all customers?

```sql
select
	'deposit' AS txn_type
	,avg(total_amount) avg_deposite_amount
	,avg(total_transactions) avg_total_transactions
from (
	select 
		customer_id
		,sum(txn_amount) total_amount
		,count(*) total_transactions
	from customer_transactions
	where txn_type = 'deposit'
	group by customer_id
) AvgCustomersDeposit;
```
![2](https://github.com/user-attachments/assets/09d76bc2-1072-4351-bd39-855d13b1eaf4)

### 3.For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```sql
with txn_customer_by_month as(
	select 
		customer_id
		,month(txn_date) month
		,sum(case when txn_type = 'deposit' then 1 else 0 end) deposit_transactions
		,sum(case when txn_type in ('purchase','withdrawal') then 1 else 0 end) non_deposit_transactions
	from customer_transactions
	group by month(txn_date), customer_id
) 
select 
	month
	,count(customer_id) customer_no
from txn_customer_by_month
where deposit_transactions > 1 
and non_deposit_transactions > 0
group by month
order by month;
```
![3](https://github.com/user-attachments/assets/cc602f02-1835-4a86-9e8a-aa53f7ba1968)

### 4.What is the closing balance for each customer at the end of the month?

```sql
select 
	customer_id
	,year(txn_date) year
	,month(txn_date) month
	,sum(case when txn_type = 'deposit' then txn_amount else 0 end) 
		- sum(case when txn_type in ('withdrawal','purchase') then txn_amount else 0 end) close_balance
from customer_transactions
group by year(txn_date), month(txn_date), customer_id
order by customer_id, year, month;
```
![4](https://github.com/user-attachments/assets/5849bc2a-1959-47ff-b09c-4745d67b0c70)

### 5.What is the percentage of customers who increase their closing balance by more than 5%?

```sql
with cutomer_closing_balance as (
	select 
		customer_id
		,year(txn_date) year
		,month(txn_date) month
		,sum(case when txn_type = 'deposit' then txn_amount else 0 end) 
			- sum(case when txn_type in ('withdrawal','purchase') then txn_amount else 0 end) close_balance
	from customer_transactions
	group by year(txn_date), month(txn_date), customer_id
),
cutomer_closing_balance_lead as(
	select *, lead(close_balance) over(partition by customer_id order by year, month) next_month_close_balance
	from cutomer_closing_balance
),
cutomer_increase_count as(
	select
		count(case when 1.05 * close_balance < next_month_close_balance then 1 end) customers_with_increase
		,count(*) total_customers
	from cutomer_closing_balance_lead
	where next_month_close_balance is not null
)
select *
	, cast( (customers_with_increase  * 100.0 ) /
				nullif(total_customers, 0) as decimal (10,2)) total_customers_percent
from cutomer_increase_count;
```
![5](https://github.com/user-attachments/assets/1a57d7b0-d59f-43e7-bbb7-a1830a7617f6)