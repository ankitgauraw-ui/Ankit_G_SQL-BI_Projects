# Ankit_G_SQL-BI_Projects


/* SKILLS USED 
1. Sub-Query
2. Joins
3. CTE
4. Cast 
5. Window functions
6. case when
*/

-- 1- write a query to print top 5 cities with highest spends and their percentage contribution of total credit card spends 

with CTE1 as (select city, SUM(amount) as T_Amount from credit_card_transcations

group by city),
CTE2 as (select SUM(cast(amount as bigint)) as T_spend from credit_card_transcations)
select top 5 CTE1.*,round(CTE1.T_Amount*1.0/CTE2.T_spend*100,2)  from CTE1, CTE2
order by T_Amount desc

-- 2- write a query to print highest spend month and amount spent in that month for each card type

-- Variation 1 - when overeall month is referred during all these years

select * from credit_card_transcations


Select top 4 A.*, B.card_type ,B.amount_spent from
(select DATEPART(month, transaction_date) as month_spent,  SUM(amount) as amount_spent_in_a_month from credit_card_transcations
group by DATEPART(month, transaction_date)
) A inner join 


(select DATEPART(month, transaction_date) as month_spent, card_type, SUM(amount) as amount_spent from credit_card_transcations
group by DATEPART(month, transaction_date), card_type
) B
on A.month_spent = B.month_spent
order by A.amount_spent_in_a_month desc

-- Variation 2 - when overeall card type is primary category

select * from credit_card_transcations;

with CTE as (select card_type, DATEPART(year, transaction_date) as year_spent, DATEPART(month,transaction_date) as month_spent, 
SUM(amount) as total_spent from credit_card_transcations
group by card_type, DATEPART(year, transaction_date), DATEPART(month,transaction_date)),
CTE2 as (select *, RANK() over (partition by card_type order by total_spent desc) as rnk from CTE)
select * from CTE2
where rnk = 1

/* 
3- write a query to print the transaction details(all columns from the table) for each card type when
it reaches a cumulative of 1000000 total spends(We should have 4 rows in the o/p one for each card type)
*/

select * from credit_card_transcations;


with CTE1 as (select *, SUM(amount) over (partition by card_type order by transaction_date,transaction_id) as running_spend from credit_card_transcations),
CTE2 as (select *, RANK() over (partition by card_type order by running_spend) as rnk from CTE1
where running_spend >1000000)
select * from CTE2
where rnk = 1



-- 4- write a query to find city which had lowest percentage spend for gold card type

select * from credit_card_transcations;

with CTE1 as (select card_type,city, SUM(amount) as T_Spend from credit_card_transcations
group by card_type,city),
CTE2 as (select SUM(cast(amount as bigint)) as F_spend from credit_card_transcations)
select top 1 CTE1.*, CTE2.*, T_spend*1.0/ F_spend*100 as per_spend from CTE1, CTE2
where card_type = 'gold'
order by per_spend

-- 5- write a query to print 3 columns:  city, highest_expense_type , lowest_expense_type (example format : Delhi , bills, Fuel)

select * from credit_card_transcations;

with CTE1 as (select city,exp_type,SUM(amount) as expense from credit_card_transcations
group by city,exp_type),
CTE2 as (select *, RANK() over (partition by city order by expense desc) as rnk_desc,RANK() over (partition by city order by expense asc) 
as rnk_asc from CTE1)
select city, max(case when rnk_asc =1  then exp_type  end) as lowest_expense_type, max(case when rnk_desc =1  then exp_type  end) as 
highest_expense_type from CTE2

group by city

-- 6- write a query to find percentage contribution of spends by females for each expense type

select * from credit_card_transcations;

select exp_type, SUM(case when gender = 'F' then amount else 0 end)*1.0 / SUM(amount) as
percentage_female_spend_cont from credit_card_transcations
group by exp_type
order by percentage_female_spend_cont desc

-- 7- which card and expense type combination saw highest month over month growth in Jan-2014


select * from credit_card_transcations;

with CTE1 as(select card_type, exp_type, DATEPART(year, transaction_date) as year_spent,DATEPART(month, transaction_date) as 
month_spent, SUM(amount) as t_spent from credit_card_transcations
group by card_type, exp_type, DATEPART(year, transaction_date),DATEPART(month, transaction_date)),

CTE2 as (select *, Lag(t_spent) over (partition by card_type,exp_type order by year_spent asc,month_spent asc) as 
prev_month_spent from CTE1)

select top 1 *, (t_spent-prev_month_spent)*1.0/ prev_month_spent as mom_growth from CTE2 

where year_spent = 2014 and month_spent = 1
order by mom_growth desc

-- 8- during weekends which city has highest total spend to total no of transcations ratio 

select top 1 city,SUM(amount)*1.0 / COUNT(1) as ratio from credit_card_transcations
where DATEPART(weekday, transaction_date) in (1,7)
group by city
order by ratio desc

-- 9- which city took least number of days to reach its 500th transaction after the first transaction in that city


select * from credit_card_transcations

with CTE1 as (select *, ROW_NUMBER() over (partition by city order by transaction_date) as tran_count from credit_card_transcations)
select top 1 city, DATEDIFF(day,min(transaction_date), Max(transaction_date)) as days_contri from CTE1
where tran_count in(1,500)
group by city
having COUNT(1) = 2
order by days_contri



