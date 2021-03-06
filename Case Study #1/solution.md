
**Schema (PostgreSQL v13)**

    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
    VALUES
      ('A', '2021-01-01', '1'),
      ('A', '2021-01-01', '2'),
      ('A', '2021-01-07', '2'),
      ('A', '2021-01-10', '3'),
      ('A', '2021-01-11', '3'),
      ('A', '2021-01-11', '3'),
      ('B', '2021-01-01', '2'),
      ('B', '2021-01-02', '2'),
      ('B', '2021-01-04', '1'),
      ('B', '2021-01-11', '1'),
      ('B', '2021-01-16', '3'),
      ('B', '2021-02-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-07', '3');
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');

---

**Case Study Questions**

**1. What is the total amount each customer spent at the restaurant?**

	select s.customer_id,
	       sum(m.price) as total_amount_spent
	from dannys_diner.sales s
	         join dannys_diner.menu m on s.product_id = m.product_id
	group by 1
	order by 1;
| customer\_id | total\_amount\_spent |
| :--- | :--- |
| A | 76 |
| B | 74 |
| C | 36 |


**2.How many days has each customer visited the restaurant?**

	select customer_id,
	count(distinct order_date) as days_customer_visited
	from dannys_diner.sales
	group by  1
	order by  1;
| customer\_id | days\_customer\_visited |
| :--- | :--- |
| A | 4 |
| B | 6 |
| C | 2 |


**3. What was the first item from the menu purchased by each customer?**

	with t1 as (select customer_id,
	                   order_date,
	                   m.product_name,
	                   row_number() over (partition by customer_id order by order_date, s.product_id) as rn
	            from dannys_diner.sales s
	                     left join dannys_diner.menu m
	                               on s.product_id = m.product_id)
	select customer_id, product_name
	from t1
	where rn = 1;
	
| customer\_id | product\_name |
| :--- | :--- |
| A | sushi |
| B | curry |
| C | ramen |


**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

	select s.product_id,
	       m.product_name,
	       count(*) as most_perchased_item
	from dannys_diner.sales s
	         join dannys_diner.menu m on s.product_id = m.product_id
	group by 1, 2
	order by 3 desc
	limit 1;
| product\_id | product\_name | most\_perchased\_item |
| :--- | :--- | :--- |
| 3 | ramen | 8 |


**5. Which item was the most popular for each customer?**

	with t1 as (select s.customer_id,
	                   m.product_name,
	                   count(*) as purchase_time
	            from dannys_diner.sales s
	                     join dannys_diner.menu m on s.product_id = m.product_id
	            group by 1, 2
	            order by 3 desc),
	     t2 as (select *, rank() over (partition by customer_id order by purchase_time desc ) as rk
	            from t1)
	select customer_id, product_name, purchase_time
	from t2
	where rk = 1;
| customer\_id | product\_name | purchase\_time |
| :--- | :--- | :--- |
| A | ramen | 3 |
| B | ramen | 2 |
| B | sushi | 2 |
| B | curry | 2 |
| C | ramen | 3 |


**6. Which item was purchased first by the customer after they became a member?**

	with t1 as (select s.customer_id,
	                   s.order_date,
	                   s.product_id,
	                   case when s.order_date >= m.join_date then true else false end as is_customer
	            from dannys_diner.sales s
	                     join dannys_diner.members m on s.customer_id = m.customer_id),
	     t2 as (select t1.*, m.product_name, row_number() over (partition by t1.customer_id order by order_date) as rn
	            from t1
	                     join dannys_diner.menu m on t1.product_id = m.product_id
	            where is_customer is true)
	select customer_id, order_date, product_name
	from t2
	where rn = 1;
| customer\_id | order\_date | product\_name |
| :--- | :--- | :--- |
| A | 2021-01-07 | curry |
| B | 2021-01-11 | sushi |


**7.Which item was purchased just before the customer became a member?**

	with t1 as (select s.customer_id,
	                   s.order_date,
	                   s.product_id,
	                   case when s.order_date >= m.join_date then true else false end as is_customer
	            from dannys_diner.sales s
	                     join dannys_diner.members m on s.customer_id = m.customer_id),
	     t2 as (select t1.*,
	                   m.product_name,
	                   row_number()
	                   over (partition by t1.customer_id order by t1.order_date desc ,t1.product_id desc ) as rn
	            from t1
	                     join dannys_diner.menu m on t1.product_id = m.product_id
	            where is_customer is false)
	select customer_id, product_name
	from t2
	where rn = 1;
| customer\_id | product\_name |
| :--- | :--- |
| A | curry |
| B | sushi |


**8. What is the total items and amount spent for each member before they became a member?**

	with t1 as (select s.customer_id,
	                   s.order_date,
	                   s.product_id,
	                   case when s.order_date >= m.join_date then true else false end as is_customer
	            from dannys_diner.sales s
	                     join dannys_diner.members m on s.customer_id = m.customer_id),
	     t2 as (select t1.*,
	                   m.product_name,
	                   m.price,
	                   row_number()
	                   over (partition by t1.customer_id order by t1.order_date desc ,t1.product_id desc ) as rn
	            from t1
	                     join dannys_diner.menu m on t1.product_id = m.product_id
	            where is_customer is false)
	select customer_id, sum(price) as amount_spent, count(*) as total_item
	from t2
	group by 1
	order by 1;
| customer\_id | amount\_spent | total\_item |
| :--- | :--- | :--- |
| A | 25 | 2 |
| B | 40 | 3 |


**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

	with t1 as (select s.customer_id,
	                   s.order_date,
	                   s.product_id,
	                   case when s.order_date >= m.join_date then true else false end as is_customer
	            from dannys_diner.sales s
	                     join dannys_diner.members m on s.customer_id = m.customer_id),
	     t2 as (select t1.*,
	                   m.product_name,
	                   m.price
	            from t1
	                     join dannys_diner.menu m on t1.product_id = m.product_id
	            where is_customer is true)
	select customer_id, sum(case when product_name = 'sushi' then 2*price else price end ) as total_points
	from t2
	group by 1
	order by 1;
| customer\_id | total\_points |
| :--- | :--- |
| A | 510 |
| B | 440 |


**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

	select s.customer_id,
	       sum(case when s.order_date between join_date and join_date + 7 then 20*price else 10*price end) as total_points
	from dannys_diner.sales s
	join dannys_diner.menu m on s.product_id = m.product_id
	join dannys_diner.members m2 on s.customer_id = m2.customer_id
	where s.order_date >= m2.join_date
	group by 1
	order by 1;
| customer\_id | total\_points |
| :--- | :--- |
| A | 1020 |
| B | 440 |


**Bonus Questions**
**Join All The Things**

		select s.customer_id,
		       order_date,
		       product_name,
		       price,
		       case when s.order_date >= m2.join_date then true else false end as is_customer
		from dannys_diner.sales s
		         left join dannys_diner.menu m on s.product_id = m.product_id
		         left join dannys_diner.members m2 on s.customer_id = m2.customer_id
		order by customer_id, order_date, product_name;
| customer\_id | order\_date | product\_name | price | is\_customer |
| :--- | :--- | :--- | :--- | :--- |
| A | 2021-01-01 | curry | 15 | false |
| A | 2021-01-01 | sushi | 10 | false |
| A | 2021-01-07 | curry | 15 | true |
| A | 2021-01-10 | ramen | 12 | true |
| A | 2021-01-11 | ramen | 12 | true |
| A | 2021-01-11 | ramen | 12 | true |
| B | 2021-01-01 | curry | 15 | false |
| B | 2021-01-02 | curry | 15 | false |
| B | 2021-01-04 | sushi | 10 | false |
| B | 2021-01-11 | sushi | 10 | true |
| B | 2021-01-16 | ramen | 12 | true |
| B | 2021-02-01 | ramen | 12 | true |
| C | 2021-01-01 | ramen | 12 | false |
| C | 2021-01-01 | ramen | 12 | false |
| C | 2021-01-07 | ramen | 12 | false |


**Rank All The Things**

	with t1 as (select s.customer_id,
	                   order_date,
	                   product_name,
	                   price,
	                   case when s.order_date >= m2.join_date then true else false end as is_customer
	            from dannys_diner.sales s
	                     left join dannys_diner.menu m on s.product_id = m.product_id
	                     left join dannys_diner.members m2 on s.customer_id = m2.customer_id
	            order by customer_id, order_date, product_name)
	select *,
	       case
	           when is_customer is true then rank() over (partition by customer_id, is_customer order by order_date)
	           else null end as ranking
	from t1;
| customer\_id | order\_date | product\_name | price | is\_customer | ranking |
| :--- | :--- | :--- | :--- | :--- | :--- |
| A | 2021-01-01 | curry | 15 | false | NULL |
| A | 2021-01-01 | sushi | 10 | false | NULL |
| A | 2021-01-07 | curry | 15 | true | 1 |
| A | 2021-01-10 | ramen | 12 | true | 2 |
| A | 2021-01-11 | ramen | 12 | true | 3 |
| A | 2021-01-11 | ramen | 12 | true | 3 |
| B | 2021-01-01 | curry | 15 | false | NULL |
| B | 2021-01-02 | curry | 15 | false | NULL |
| B | 2021-01-04 | sushi | 10 | false | NULL |
| B | 2021-01-11 | sushi | 10 | true | 1 |
| B | 2021-01-16 | ramen | 12 | true | 2 |
| B | 2021-02-01 | ramen | 12 | true | 3 |
| C | 2021-01-01 | ramen | 12 | false | NULL |
| C | 2021-01-01 | ramen | 12 | false | NULL |
| C | 2021-01-07 | ramen | 12 | false | NULL |


DB Fiddle: https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138
