### Pre-Requirments 
##### Insert PK to customer_transactions table

```sql
alter table customer_transactions
add txn_id int identity(1,1) not null;

alter table customer_transactions
add constraint pk_customer_transactions primary key (txn_id);
```

##### Calc Balance for each Customer after each Transaction

```sql
create view CustomerRunningBalance as
with CustomerTransactionsWithRank as (
	select 
		customer_id
		,txn_date
		,txn_type
		,txn_amount
		,ROW_NUMBER() over(partition by customer_id order by txn_date, txn_id) seq
	from customer_transactions
)
select
	customer_id
	,txn_date
	,txn_type
	,sum( case when txn_type = 'deposit' then txn_amount else -txn_amount end)
		over(partition by customer_id order by seq) balance
from CustomerTransactionsWithRank;
```

##### Customer Balance at the end of each month

```sql
select 
	customer_id
	,format(txn_date,'yyyy-MM') year_month
	,sum(balance) closing_balance
from CustomerRunningBalance
group by customer_id, format(txn_date,'yyyy-MM')
order by customer_id, year_month;
```

##### Min, Avg, Max Values of Running Balance for each Customer

```sql
select 
	customer_id
	,sum(balance) sum_balance
	,avg(balance) avg_balance
	,min(balance) min_balance
	,max(balance) max_balance
from CustomerRunningBalance
group by customer_id
order by sum_balance desc;
```

---
### To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

#### Option 1: data is allocated based off the amount of money at the end of the previous month

Each Customer Monthly Balance
```sql
select 
	customer_id
	,format(txn_date,'yyyy-MM') year_month
	,sum(txn_amount) closing_balance
from customer_transactions
group by customer_id, format(txn_date,'yyyy-MM')
order by customer_id, year_month;
```
![1.1](https://github.com/user-attachments/assets/3f5e6169-07a1-4d6e-a1fe-0c290edb773e)

Total Balance each Month
```sql
select 
    format(txn_date, 'yyyy-mm') as year_month,
    sum(txn_amount) as monthly_total_balance
from customer_transactions
group by format(txn_date, 'yyyy-mm')
order by year_month;
```
![1.2](https://github.com/user-attachments/assets/018c3b45-491a-4ffc-a08c-8e0547cdbf63)

#### Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days

```sql
	select 
		customer_id
		,format(txn_date,'yyyy-MM') year_month
		,avg(txn_amount) avg_monthy_balance
	from customer_transactions
	group by customer_id, format(txn_date,'yyyy-MM')
```
![2.1](https://github.com/user-attachments/assets/3b9126da-e17d-49cf-86bb-180fd9c37c89)

```sql
with CustomerStorage as(
	select 
		customer_id
		,format(txn_date,'yyyy-MM') year_month
		,case when avg(balance) > 0 then avg(balance) else 0 end storage_needed
	from CustomerRunningBalance
	group by customer_id, format(txn_date,'yyyy-MM')
)
select sum(storage_needed) total_storage
from CustomerStorage;
```

#### Option 3: data is updated real-time

```sql
select 
	customer_id
	,txn_date date
	,case when balance > 0 then balance else 0 end storage_needed
from CustomerRunningBalance;
```

```sql
with CustomerStorage as(
	select 
		customer_id
		,txn_date date
		,case when balance > 0 then balance else 0 end storage_needed
	from CustomerRunningBalance
)
select 
	sum(storage_needed) total_storage
from CustomerStorage;
```