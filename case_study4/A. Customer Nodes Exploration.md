### 1.How many unique nodes are there on the Data Bank system?

```sql
select sum(unique_nodes) sum_unique_nodes
from(
	select  
		count(distinct node_id) unique_nodes
	from customer_nodes
	group by region_id
) as RegoinsUniqueNodes;
```
![1](https://github.com/user-attachments/assets/8a3f92cd-e046-4dcc-b989-0633a68ecdd0)

### 2.What is the number of nodes per region?

```sql
select  
	region_id
	,count(distinct node_id) unique_nodes
from customer_nodes
group by region_id
order by region_id;
```
![2](https://github.com/user-attachments/assets/3503fcfa-dc85-4b6f-93ca-d246c2be8a2c)

### 3.How many customers are allocated to each region?

```sql
select  
	region_id
	,count(distinct customer_id) unique_customers
from customer_nodes
group by region_id
order by region_id;
```
![3](https://github.com/user-attachments/assets/f0d0a57e-eb38-4f0f-94a7-f2a2b0c1b329)

### 4.How many days on average are customers reallocated to a different node?

```sql
select  
	avg(datediff(day, start_date, end_date)) avg_reallocation_days
from customer_nodes c;
```
![4](https://github.com/user-attachments/assets/b1384876-2a8e-4db7-a012-2dc01492ca60)

### 5.What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

```sql
select  distinct
	region_id
	,percentile_cont(0.5) within group (order by datediff(day, start_date, end_date)) 
		over(partition by region_id) median_reallocation_days
	,percentile_cont(0.8) within group (order by datediff(day, start_date, end_date)) 
		over(partition by region_id) _80th_percentile_reallocation_days80
	,percentile_cont(0.95) within group (order by datediff(day, start_date, end_date)) 
		over(partition by region_id) _95th_percentile_reallocation_days80
from customer_nodes;
```
![5](https://github.com/user-attachments/assets/77d3b4e4-0aa8-4125-b845-b14356d04bba)