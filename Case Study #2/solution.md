
**Schema (PostgreSQL v13)**

    CREATE SCHEMA pizza_runner;
    SET search_path = pizza_runner;
    
    DROP TABLE IF EXISTS runners;
    CREATE TABLE runners (
      "runner_id" INTEGER,
      "registration_date" DATE
    );
    INSERT INTO runners
      ("runner_id", "registration_date")
    VALUES
      (1, '2021-01-01'),
      (2, '2021-01-03'),
      (3, '2021-01-08'),
      (4, '2021-01-15');
    
    
    DROP TABLE IF EXISTS customer_orders;
    CREATE TABLE customer_orders (
      "order_id" INTEGER,
      "customer_id" INTEGER,
      "pizza_id" INTEGER,
      "exclusions" VARCHAR(4),
      "extras" VARCHAR(4),
      "order_time" TIMESTAMP
    );
    
    INSERT INTO customer_orders
      ("order_id", "customer_id", "pizza_id", "exclusions", "extras", "order_time")
    VALUES
      ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
      ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
      ('3', '102', '1', '', '', '2020-01-02 12:51:23'),
      ('3', '102', '2', '', NULL, '2020-01-02 12:51:23'),
      ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
      ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
      ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
      ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
      ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
      ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
      ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
      ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
      ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
      ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');
    
    
    DROP TABLE IF EXISTS runner_orders;
    CREATE TABLE runner_orders (
      "order_id" INTEGER,
      "runner_id" INTEGER,
      "pickup_time" VARCHAR(19),
      "distance" VARCHAR(7),
      "duration" VARCHAR(10),
      "cancellation" VARCHAR(23)
    );
    
    INSERT INTO runner_orders
      ("order_id", "runner_id", "pickup_time", "distance", "duration", "cancellation")
    VALUES
      ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
      ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
      ('3', '1', '2020-01-02 00:12:37', '13.4km', '20 mins', NULL),
      ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
      ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
      ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
      ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
      ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
      ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
      ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');
    
    
    DROP TABLE IF EXISTS pizza_names;
    CREATE TABLE pizza_names (
      "pizza_id" INTEGER,
      "pizza_name" TEXT
    );
    INSERT INTO pizza_names
      ("pizza_id", "pizza_name")
    VALUES
      (1, 'Meatlovers'),
      (2, 'Vegetarian');
    
    
    DROP TABLE IF EXISTS pizza_recipes;
    CREATE TABLE pizza_recipes (
      "pizza_id" INTEGER,
      "toppings" TEXT
    );
    INSERT INTO pizza_recipes
      ("pizza_id", "toppings")
    VALUES
      (1, '1, 2, 3, 4, 5, 6, 8, 10'),
      (2, '4, 6, 7, 9, 11, 12');
    
    
    DROP TABLE IF EXISTS pizza_toppings;
    CREATE TABLE pizza_toppings (
      "topping_id" INTEGER,
      "topping_name" TEXT
    );
    INSERT INTO pizza_toppings
      ("topping_id", "topping_name")
    VALUES
      (1, 'Bacon'),
      (2, 'BBQ Sauce'),
      (3, 'Beef'),
      (4, 'Cheese'),
      (5, 'Chicken'),
      (6, 'Mushrooms'),
      (7, 'Onions'),
      (8, 'Pepperoni'),
      (9, 'Peppers'),
      (10, 'Salami'),
      (11, 'Tomatoes'),
      (12, 'Tomato Sauce');

---
**Clear the data**

	-- Update 1
	update pizza_runner.customer_orders
	set exclusions = null
	where exclusions in ('null','');
	
	-- Update 2
	update pizza_runner.customer_orders
	set extras = null
	where extras in ('null','');

	-- Update 3
	update pizza_runner.runner_orders
	set pickup_time = null
	where pickup_time in ('null','');
	
	-- Update 4
	update pizza_runner.runner_orders
	set cancellation = null
	where cancellation in ('null','');
	
	-- Update 5
	update pizza_runner.runner_orders
	set distance = null
	where distance in ('null','');

	-- Update 6
	update pizza_runner.runner_orders
	set duration = null
	where duration in ('null','');

	-- Update 7
	alter  table pizza_runner.runner_orders add column duration_clean numeric;
	alter  table pizza_runner.runner_orders add column distance_clean numeric;

	-- Update 8
	update pizza_runner.runner_orders
	set distance_clean = NULLIF(regexp_replace(distance, '[^\.\d]','','g'), '')::numeric
	where  1=1;

	-- Update 9
	update pizza_runner.runner_orders
	set duration_clean = NULLIF(regexp_replace(duration, '\D','','g'), '')::numeric
	where  1=1;
---

**A. Pizza Metrics**
---

	-- 1. How many pizzas were ordered?
	select count(*) as pizza_orders  
	from pizza_runner.customer_orders;  
	
	  
	-- 2. How many unique customer orders were made?  
	select count(distinct customer_id) as unique_customer_orders  
	from pizza_runner.customer_orders;  
	  
	-- 3. How many successful orders were delivered by each runner?  
	select runner_id, count(*) as number_order_deliver  
	from pizza_runner.runner_orders  
	where cancellation is null  
	group by 1  
	order by 1;  
	  
	-- 4. How many of each type of pizza was delivered?  
	with t1 as (select co.order_id, pizza_id, cancellation  
	  from pizza_runner.customer_orders co  
	                     left join pizza_runner.runner_orders ro on co.order_id = ro.order_id)  
	select pizza_id, count(*) as number_of_order  
	from t1  
	where cancellation is null  
	group by 1  
	order by 1;  
	  
	-- 5. How many Vegetarian and Meatlovers were ordered by each customer?  
	with t1 as (select co.order_id, co.customer_id, co.pizza_id, pn.pizza_name  
	  from pizza_runner.customer_orders co  
	                     join pizza_runner.pizza_names pn on co.pizza_id = pn.pizza_id)  
	select customer_id, pizza_name, count(*) as number_of_order  
	from t1  
	group by 1, 2  
	order by 1, 2;  
	  
	-- 6. What was the maximum number of pizzas delivered in a single order?  
	with t1 as (select co.order_id, pizza_id, cancellation  
	  from pizza_runner.customer_orders co  
	                     left join pizza_runner.runner_orders ro on co.order_id = ro.order_id)  
	select order_id, count(*) as number_of_pizza_order  
	from t1  
	where cancellation is null  
	group by 1  
	order by count(*) desc  
	limit 1 ;  
	  
	-- 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?  
	with t1 as (select co.order_id, co.customer_id, pizza_id, co.exclusions, co.extras, cancellation  
	  from pizza_runner.customer_orders co  
	                     left join pizza_runner.runner_orders ro on co.order_id = ro.order_id)  
	select customer_id,  
	  count(*)                                                                        as total_order,  
	  sum(case when (exclusions is not null or extras is not null) then 1 else 0 end) as atleast_one_change,  
	  sum(case when (exclusions is null and extras is null) then 1 else 0 end)        as no_change  
	from t1  
	where cancellation is null  
	group by 1  
	order by 1;  
	  
	-- 8. How many pizzas were delivered that had both exclusions and extras?  
	with t1 as (select co.order_id, co.customer_id, pizza_id, co.exclusions, co.extras, cancellation  
	  from pizza_runner.customer_orders co  
	                     left join pizza_runner.runner_orders ro on co.order_id = ro.order_id)  
	select sum(case when (exclusions is not null and extras is not null) then 1 else 0 end) as have_both_exclusions_and_extras  
	from t1  
	where cancellation is null  
	order by 1;  
	  
	-- 9. What was the total volume of pizzas ordered for each hour of the day?  
	select order_time::date              as date,  
	  extract(hour from order_time) as hour,  
	  count(*)                      as pizza_orders  
	from pizza_runner.customer_orders  
	group by 1, 2  
	order by 1, 2;  
	  
	-- 10. What was the volume of orders for each day of the week?  
	select order_time::date as date,  
	  count(*)         as pizza_orders  
	from pizza_runner.customer_orders  
	group by 1  
	order by 1;
---

**B. Runner and Customer Experience**
---
	-- 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)  
	select date_trunc('week', registration_date)::date as week_start_date,  
	  EXTRACT('week' FROM registration_date)         week_number,  
	  count(*)                                    as number_of_runners_signup  
	from pizza_runner.runners  
	group by 1, 2  
	order by 1;  
	  
	-- 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?  
	select avg(ro.pickup_time::timestamp - co.order_time) as average_time_to_pickup  
	from pizza_runner.customer_orders co  
	         join pizza_runner.runner_orders ro  
	              on co.order_id = ro.order_id  
	where cancellation is null;  
	  
	-- 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?  
	select co.order_id,  
	  count(pizza_id)                                as number_of_pizza,  
	  avg(ro.pickup_time::timestamp - co.order_time) as avg_time_to_prepare  
	from pizza_runner.customer_orders co  
	         join pizza_runner.runner_orders ro  
	              on co.order_id = ro.order_id  
	where cancellation is null  
	group by 1;  
	  
	-- Yes..usually one pizza takes average 10 minutes to prepare  
	  
	-- 4. What was the average distance travelled for each customer?  
	select avg(distance_clean) as avg_distance_travelled  
	from pizza_runner.runner_orders ro;  
	  
	-- 5. What was the difference between the longest and shortest delivery times for all orders?  
	select max(duration_clean) - min(duration_clean) as difference  
	from pizza_runner.runner_orders ro;  
	  
	-- 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?  
	select sum(distance_clean) / sum(duration_clean) as avg_speed_in_km_per_min  
	from pizza_runner.runner_orders ro  
	where cancellation is null;  
	  
	-- 7. What is the successful delivery percentage for each runner?  
	with t1 as (select runner_id, count(*) as number_of_order_per_user  
	            from pizza_runner.runner_orders ro  
	            where cancellation is null  
	 group by 1),  
	  t2 as (select count(*) as total_order  
	            from pizza_runner.runner_orders ro  
	            where cancellation is null)  
	select runner_id,  
	  total_order,  
	  number_of_order_per_user,  
	  (number_of_order_per_user::numeric / total_order::numeric) * 100 as pct  
	from t1,  
	  t2  
	order by 1;
