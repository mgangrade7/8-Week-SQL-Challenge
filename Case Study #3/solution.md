## A. Customer Journey

	Based off the 8 sample customers provided in the sample from the  `subscriptions`  table, write a brief description about each customer’s onboarding journey.

	Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

	select s.customer_id,  
	 s.plan_id,  
	 s.start_date,  
	  p.plan_name,  
	  p.price,  
	 case when s.plan_id = 4 then null  
	 when s.plan_id = 0 then s.start_date + 7  
	  when s.plan_id = 1 then s.start_date + 30  
	  when s.plan_id = 2 then s.start_date + 30  
	  else s.start_date + 365 end as end_date  
	from foodie_fi.subscriptions s  
	 left join foodie_fi.plans p  
	                   on s.plan_id = p.plan_id  
	order by 1, 3;  
	  
	Customer 1 Started Trail Plan at 2020-08-01 and upgraded to basic monthly on last day of free trial 2020-08-08  
	Customer 2 Started Trail Plan at 2020-09-20 and upgraded to pro annual on last day of free trial 2020-09-27  
	Customer 11 Started Trail Plan at 2020-11-19 and cancel the sunscroption on last day of free trial 2020-11-26  
	Customer 13 Started Trail Plan at 2020-12-15 and upgraded to basic monthly on last day of free trial 2020-12-22 and again upgraded to pro monthly on 2021-03-29  
	Customer 15 Started Trail Plan at 2020-03-17 and upgraded to pro monthly on last day of free trial 2020-03-24 then cancel subscription on 2020-04-29  
	Customer 16 Started Trail Plan at 2020-05-31 and upgraded to basic monthly on last day of free trial 2020-06-07 and again upgraded to pro monthly on 2020-10-21  
	Customer 18 Started Trail Plan at 2020-07-06 and upgraded to pro monthly on last day of free trial 2020-07-13
	Customer 19 Started Trail Plan at 2020-06-022 and upgraded to pro monthly on last day of free trial 2022-06-29 and upgraded to pro annual on 2020-08-29

---
## B. Data Analysis Questions
---
	-- How many customers has Foodie-Fi ever had?  
	
	select count(distinct s.customer_id)  
	from foodie_fi.subscriptions s;  
	  
	-- What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value  
	
	select date_part('mon', s.start_date) as month, count(*)  
	from foodie_fi.subscriptions s  
	where s.plan_id = 0  
	group by 1  
	order by 1;  
	  
	-- What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name  
	
	select date_part('y', s.start_date) as year, s.plan_id, p.plan_name, count(*)  
	from foodie_fi.subscriptions s  
	 left join foodie_fi.plans p on s.plan_id = p.plan_id  
	where date_part('y', s.start_date) = '2021'  
	group by 1, 2, 3  
	order by 2;  
	  
	-- What is the customer count and percentage of customers who have churned rounded to 1 decimal place?  
	
	select count(distinct s.customer_id)                                                                                 as customer_count,  
	  sum(case when s.plan_id = 4 then 1 else 0 end)                                                                as churn_count,  
	  100 * (sum(case when s.plan_id = 4 then 1 else 0 end) /  
	              cast(count(distinct s.customer_id) as decimal(7, 2)))                                                  as churn_percentage  
	from foodie_fi.subscriptions s;  
	  
	-- How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?  
	
	with t1 as (select customer_id,  
	  plan_id,  
	  start_date,  
	  row_number() over (partition by customer_id order by start_date) as rn  
	            from foodie_fi.subscriptions s  
	 order by 1),  
	  t2 as (select count(*) as churn_count  
	            from t1  
	            where plan_id = 4  
	  and rn = 2)  
	select ceil(cast(churn_count as decimal(7, 2)) / count(distinct s.customer_id))  
	from foodie_fi.subscriptions s,  
	  t2  
	group by churn_count  
	  
	-- What is the number and percentage of customer plans after their initial free trial?  
	
	with t1 as (select customer_id,  
	  plan_id,  
	  start_date,  
	  row_number() over (partition by customer_id order by start_date) as rn  
	            from foodie_fi.subscriptions s  
	 order by 1),  
	  t2 as (select count(*) as non_churn_count  
	            from t1  
	            where plan_id != 4  
	  and rn = 2)  
	select non_churn_count, (cast(non_churn_count as decimal(7, 2)) / count(distinct s.customer_id)) * 100 as percentage  
	from foodie_fi.subscriptions s,  
	  t2  
	group by non_churn_count;  
	  
	  
	-- What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?  
	
	with t1 as (select customer_id,  
	  plan_id,  
	  start_date,  
	  row_number() over (partition by customer_id order by start_date desc ) as rn  
	            from foodie_fi.subscriptions s  
	 where s.start_date <= '2020-12-31'),  
	  t2 as (select count(*) as total  
	            from t1  
	            where rn = 1)  
	select plan_id, count(*), (cast(count(*) as decimal(7, 2)) / total) * 100 as percentage  
	from t1,  
	  t2  
	where t1.rn = 1  
	group by 1, t2.total;  
	  
	-- How many customers have upgraded to an annual plan in 2020?  
	select count(distinct customer_id)  
	from foodie_fi.subscriptions  
	where plan_id = 3  
	  and start_date BETWEEN '2020-01-01' and '2020-12-31';  
	  
	-- How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?  
	
		with t1 as (select s.customer_id,  
	 s.start_date as start_date_trial_plan  
	            from foodie_fi.subscriptions s  
	 where plan_id = 0),  
	  t2 as (select s.customer_id,  
	 s.start_date as start_date_annual_plan  
	            from foodie_fi.subscriptions s  
	 where plan_id = 3  
	  and s.customer_id in (select distinct customer_id from t1))  
	select round(avg(start_date_annual_plan - start_date_trial_plan))  
	from t2  
	         left join t1 on t2.customer_id = t1.customer_id;
	         
	-- Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)  
	
	with t1 as (select s.customer_id,  
	 s.start_date as start_date_trial_plan  
	            from foodie_fi.subscriptions s  
	 where plan_id = 0),  
	  t2 as (select s.customer_id,  
	 s.start_date as start_date_annual_plan  
	            from foodie_fi.subscriptions s  
	 where plan_id = 3  
	  and s.customer_id in (select distinct customer_id from t1))  
	select case  
	 when (start_date_annual_plan - start_date_trial_plan) <= 50 then '1-50'  
	  when (start_date_annual_plan - start_date_trial_plan) BETWEEN 51 and 100 then '51-100'  
	  when (start_date_annual_plan - start_date_trial_plan) BETWEEN 101 and 200 then '100-200'  
	  when (start_date_annual_plan - start_date_trial_plan) BETWEEN 201 and 400 then '200-400'  
	  else 'More than 401'  
	  end  as range,  
	  sum(case  
	 when (start_date_annual_plan - start_date_trial_plan) <= 50 then 1  
	  when (start_date_annual_plan - start_date_trial_plan) BETWEEN 51 and 100 then 1  
	  when (start_date_annual_plan - start_date_trial_plan) BETWEEN 100 and 200 then 1  
	  when (start_date_annual_plan - start_date_trial_plan) BETWEEN 200 and 400 then 1  
	  else 1  
	  end) as days  
	from t2  
	         left join t1 on t2.customer_id = t1.customer_id  
	group by 1;
	
	-- How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
	
		with t1 as (select customer_id, start_date pro_start_date  
	            from foodie_fi.subscriptions  
	            where plan_id = 2),  
	t2 as (select customer_id, start_date basic_start_date  
	            from foodie_fi.subscriptions  
	            where plan_id = 1)  
	select count(*) from t1 left join t2 on t1.customer_id = t2.customer_id  
	where basic_start_date > pro_start_date;
