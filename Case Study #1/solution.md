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

**Query #1: What is the total amount each customer spent at the restaurant?**

    SELECT
    s.customer_id,
    sum(m.price) as total_amount
    FROM 
    sales s
    LEFT JOIN 
    menu m
    ON s.product_id = m.product_id
    GROUP BY 1
    ORDER BY 1
    ;

| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

---


**Query #2: How many days has each customer visited the restaurant?**

    SELECT
    s.customer_id,
    COUNT(DISTINCT order_date)
    FROM 
    sales s
    GROUP BY 1
    ORDER BY 1
    ;

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

---


**Query #4: What is the most purchased item on the menu and how many times was it purchased by all customers?**

    SELECT
    s.product_id,
    m.product_name,
    COUNT(*) as count
    FROM
    sales s
    JOIN 
    menu m
    ON s.product_id = m.product_id
    GROUP BY 1, 2
    ORDER BY 3 DESC
    LIMIT 1
    ;

| product_id | product_name | count |
| ---------- | ------------ | ----- |
| 3          | ramen        | 8     |

---


**Query #5: Which item was the most popular for each customer?**

    SELECT
    s.customer_id,
    m.product_name,
    COUNT(*) as count
    FROM
    sales s
    JOIN 
    menu m
    ON
    s.product_id = m.product_id
    GROUP BY 1,2
    ORDER BY 1,2
    ;

| customer_id | product_name | count |
| ----------- | ------------ | ----- |
| A           | curry        | 2     |
| A           | ramen        | 3     |
| A           | sushi        | 1     |
| B           | curry        | 2     |
| B           | ramen        | 2     |
| B           | sushi        | 2     |
| C           | ramen        | 3     |

---

**Query #6: Which item was purchased first by the customer after they became a member?**

    with t1 as (
    SELECT
    m.customer_id,
    m.join_date,
    s.order_date,
    s.product_id,
    me.product_name,
    ROW_NUMBER() OVER(PARTITION BY m.customer_id ORDER BY order_date) as rn
    FROM 
    members m
    JOIN
    sales s
    ON 
    m.customer_id = s.customer_id
    JOIN
    menu me
    ON
    s.product_id = me.product_id
    WHERE s.order_date > m.join_date
    GROUP BY 1,2,3,4,5
    ORDER BY 1
    )
    SELECT * FROM t1 WHERE rn = 1;

| customer_id | join_date                | order_date               | product_id | product_name | rn  |
| ----------- | ------------------------ | ------------------------ | ---------- | ------------ | --- |
| A           | 2021-01-07T00:00:00.000Z | 2021-01-10T00:00:00.000Z | 3          | ramen        | 1   |
| B           | 2021-01-09T00:00:00.000Z | 2021-01-11T00:00:00.000Z | 1          | sushi        | 1   |

---

**Query #7: Which item was purchased just before the customer became a member?**

    with t1 as (
    SELECT
    m.customer_id,
    m.join_date,
    s.order_date,
    s.product_id,
    me.product_name,
    RANK() OVER(PARTITION BY m.customer_id ORDER BY order_date DESC) as rk
    FROM 
    members m
    JOIN
    sales s
    ON 
    m.customer_id = s.customer_id
    JOIN
    menu me
    ON
    s.product_id = me.product_id
    WHERE s.order_date < m.join_date
    GROUP BY 1,2,3,4,5
    ORDER BY 1
    )
    SELECT * FROM t1 WHERE rk = 1;

| customer_id | join_date                | order_date               | product_id | product_name | rk  |
| ----------- | ------------------------ | ------------------------ | ---------- | ------------ | --- |
| A           | 2021-01-07T00:00:00.000Z | 2021-01-01T00:00:00.000Z | 1          | sushi        | 1   |
| A           | 2021-01-07T00:00:00.000Z | 2021-01-01T00:00:00.000Z | 2          | curry        | 1   |
| B           | 2021-01-09T00:00:00.000Z | 2021-01-04T00:00:00.000Z | 1          | sushi        | 1   |

---

**Query #8: What is the total items and amount spent for each member before they became a member?**

    with t1 as (
    SELECT
    m.customer_id,
    m.join_date,
    s.order_date,
    s.product_id,
    me.product_name,
    me.price,
    RANK() OVER(PARTITION BY m.customer_id ORDER BY order_date DESC) as rk
    FROM 
    members m
    JOIN
    sales s
    ON 
    m.customer_id = s.customer_id
    JOIN
    menu me
    ON
    s.product_id = me.product_id
    WHERE s.order_date < m.join_date
    GROUP BY 1,2,3,4,5,6
    ORDER BY 1
    )
    SELECT 
    customer_id,
    COUNT(*) as total_items,
    SUM(price) as amount_spent
    FROM t1 
    WHERE rk = 1
    GROUP BY 1;

| customer_id | total_items | amount_spent |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 1           | 10           |

---

**Query #9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

    SELECT
    s.customer_id,
    SUM(
      CASE
      WHEN m.product_name = 'sushi' THEN 2 * m.price
      ELSE m.price
    	END) as points
    FROM
    sales s
    JOIN 
    menu m
    ON
    s.product_id = m.product_id
    GROUP BY 1
    ORDER BY 1
    ;

| customer_id | points |
| ----------- | ------ |
| A           | 86     |
| B           | 94     |
| C           | 36     |

---

**Query #10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

    SELECT
    me.customer_id,
    SUM(
      CASE
      WHEN m.product_name = 'sushi' THEN 2 * m.price
      WHEN s.order_date BETWEEN me.join_date AND me.join_date + 6 THEN 2 * m.price
      ELSE m.price
    	END) as points
    FROM 
    members me
    JOIN
    sales s
    ON 
    me.customer_id = s.customer_id
    JOIN 
    menu m
    ON 
    s.product_id = m.product_id
    WHERE
    order_date BETWEEN '2021-01-01' AND '2021-01-31'
    GROUP BY 1
    ORDER BY 1
    ;

| customer_id | points |
| ----------- | ------ |
| A           | 137    |
| B           | 82     |
