# 8-Week SQL Challenge: Case Study #1 - Danny's Diner

This is Case Study #1 from the 8-Week SQL Challenge of [Danny Ma's Serious SQL course](https://www.datawithdanny.com/ "Data With Danny"). My SQL [exploration](#data-exploration) and [solutions](#sql-query-solutions) can be found after the [background](#background) section below.

## Table of Contents
1. [Background](#background)
2. [Data Exploration](#data-exploration)
3. [SQL Query Solutions](#sql-query-solutions)

## [Background](#table-of-contents)
### Problem Statement
Paraphrashed from Danny's course:
> Danny wants to use the data to answer questions about his customers, their visiting patterns, spending habits, and favorite menu items. This will improve their experience as well as the existing customer loyalty program.

### Example Datasets
> Danny has shared three key datasets that are enough to write fully functioning SQL queries to help him answer his questions:
1. `sales`
2. `menu`
3. `members`

### Entity Relationship Diagram


## [Data Exploration](#table-of-contents)
```sql
-- Check data types for `members` table

SELECT
  table_schema,
  table_name,
  column_name,
  data_type
FROM information_schema.columns
WHERE
  table_schema = 'dannys_diner'
  AND table_name = 'members';
```

| table_schema | table_name | column_name | data_type         |
|--------------|------------|-------------|-------------------|
| dannys_diner | members    | customer_id | character varying |
| dannys_diner | members    | join_date   | date              |

```sql
-- Explore the `members` table

SELECT *
FROM dannys_diner.members;
```
| customer_id | join_date                |
|-------------|--------------------------|
| A           | 2021-01-07T00:00:00.000Z |
| B           | 2021-01-09T00:00:00.000Z |

```sql
-- Check data types for `menu` table

SELECT
  table_schema,
  table_name,
  column_name,
  data_type
FROM information_schema.columns
WHERE
  table_schema = 'dannys_diner'
  AND table_name = 'menu';
  ```
| table_schema | table_name | column_name  | data_type         |
|--------------|------------|--------------|-------------------|
| dannys_diner | menu       | product_id   | integer           |
| dannys_diner | menu       | product_name | character varying |
| dannys_diner | menu       | price        | integer           |

```sql
-- Explore the `menu` table

SELECT *
FROM dannys_diner.menu;
```
| product_id | product_name | price |
|------------|--------------|-------|
| 1          | sushi        | 10    |
| 2          | curry        | 15    |
| 3          | ramen        | 12    |

```sql
-- Check data types for `price` table

SELECT
  table_schema,
  table_name,
  column_name,
  data_type
FROM information_schema.columns
WHERE
  table_schema = 'dannys_diner'
  AND table_name = 'sales';
  ```
| table_schema | table_name | column_name | data_type         |
|--------------|------------|-------------|-------------------|
| dannys_diner | sales      | customer_id | character varying |
| dannys_diner | sales      | order_date  | date              |
| dannys_diner | sales      | product_id  | integer           |

```sql
-- Explore the `sales` table

SELECT *
FROM dannys_diner.sales
LIMIT 5;
```
| customer_id | order_date               | product_id |
|-------------|--------------------------|------------|
| A           | 2021-01-01T00:00:00.000Z | 1          |
| A           | 2021-01-01T00:00:00.000Z | 2          |
| A           | 2021-01-07T00:00:00.000Z | 2          |
| A           | 2021-01-10T00:00:00.000Z | 3          |
| A           | 2021-01-11T00:00:00.000Z | 3          |

```sql
-- Check number of rows in `sales` table

SELECT COUNT(*)
FROM dannys_diner.sales;
```
| count |
|-------|
| 15    |

```sql
-- Make sure the unique values match the previous tables

SELECT
  COUNT(DISTINCT customer_id) AS customer_id_unique,
  COUNT(DISTINCT product_id) AS product_count_unique
FROM dannys_diner.sales;
```
| customer_id_unique | product_count_unique |
|--------------------|----------------------|
| 3                  | 3                    |

```sql
-- Check unique customer_id values in `sales` table

SELECT
  customer_id
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id |
|-------------|
| A           |
| B           |
| C           |

```sql
-- Confirm many-to-one relationship between this table and `customer_id` field.

WITH customer_id_cte AS (
  SELECT
    customer_id,
    COUNT(*) AS row_count
  FROM dannys_diner.sales
  GROUP BY customer_id
)
SELECT
  row_count,
  COUNT(DISTINCT customer_id) AS customer_count
FROM customer_id_cte
GROUP BY row_count
ORDER BY row_count DESC;
```
| row_count | customer_count |
|-----------|----------------|
| 6         | 2              |
| 3         | 1              |

```sql
-- Confirm many-to-one relationship between this table and `product_id` field.

WITH product_id_cte AS (
  SELECT
    product_id,
    COUNT(*) AS row_count
  FROM dannys_diner.sales
  GROUP BY product_id
)
SELECT 
  row_count,
  COUNT(DISTINCT product_id) AS product_count
FROM product_id_cte
GROUP BY row_count
ORDER BY row_count DESC;
```
| row_count | product_count |
|-----------|---------------|
| 8         | 1             |
| 4         | 1             |
| 3         | 1             |


## [SQL Query Solutions](#table-of-contents)
> 1. What is the total amount each customer spent at the restaurant?
```sql
SELECT
  sales.customer_id,
  SUM(menu.price) AS total_customer_expense
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY customer_id;
```
| customer_id | total_customer_expense |
|-------------|------------------------|
| A           | 76                     |
| B           | 74                     |
| C           | 36                     |

> 2. How many days has each customer visited the restaurant?
```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS day_count
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | day_count |
|-------------|-----------|
| A           | 4         |
| B           | 6         |
| C           | 2         |

> 3. What was the first item from the menu purchased by each customer?
```sql
WITH rank_cte AS (
  SELECT
    customer_id,
    order_date,
    product_id,
    RANK() OVER (
      PARTITION BY customer_id
      ORDER BY order_date
    ) AS _rank
  FROM dannys_diner.sales
)
SELECT DISTINCT
  rank_cte.customer_id,
  menu.product_name
FROM rank_cte
INNER JOIN dannys_diner.menu
  ON rank_cte.product_id = menu.product_id
WHERE _rank = 1
ORDER BY customer_id;
```
| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

> 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT
  menu.product_name,
  COUNT(*) AS product_count
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY product_count DESC
LIMIT 1;
```
| product_name | product_count |
|--------------|---------------|
| ramen        | 8             |

> 5. Which item was the most popular for each customer?
```sql
WITH rank_cte AS (
  SELECT
    sales.customer_id,
    menu.product_name,
    COUNT(menu.product_name) AS most_pop_count,
    RANK() OVER (
      PARTITION BY sales.customer_id
      ORDER BY COUNT(menu.product_name) DESC 
    ) AS _rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  GROUP BY sales.customer_id, menu.product_name
)
SELECT
  customer_id,
  product_name,
  most_pop_count
FROM rank_cte
WHERE _rank = 1
ORDER BY customer_id;
```
| customer_id | product_name | most_pop_count |
|-------------|--------------|----------------|
| A           | ramen        | 3              |
| B           | sushi        | 2              |
| B           | curry        | 2              |
| B           | ramen        | 2              |
| C           | ramen        | 3              |

> 6. Which item was purchased first by the customer after they became a member?
```sql
WITH member_cte AS (
  SELECT
    sales.customer_id,
    sales.order_date,
    sales.product_id,
    menu.product_name,
    RANK() OVER (
      PARTITION BY sales.customer_id
      ORDER BY order_date
    ) AS _rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
-- This anti-join isn't strictly necessary here.
  WHERE EXISTS (
    SELECT 1
    FROM dannys_diner.members
    WHERE sales.customer_id = members.customer_id
  )
  AND sales.order_date >= members.join_date
)
SELECT
  customer_id,
  order_date,
  product_name
FROM member_cte
WHERE _rank = 1
ORDER BY customer_id;
```
| customer_id | order_date               | product_name |
|-------------|--------------------------|--------------|
| A           | 2021-01-07T00:00:00.000Z | curry        |
| B           | 2021-01-11T00:00:00.000Z | sushi        |

> 7. Which item was purchased just before the customer became a member?
```sql
WITH member_cte AS (
  SELECT
    sales.customer_id,
    sales.order_date,
    sales.product_id,
    menu.product_name,
    RANK() OVER (
      PARTITION BY sales.customer_id
      ORDER BY order_date DESC
    ) AS _rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
-- This anti-join isn't strictly necessary here.
  WHERE EXISTS (
    SELECT 1
    FROM dannys_diner.members
    WHERE sales.customer_id = members.customer_id
  )
  AND sales.order_date < members.join_date
)
SELECT
  customer_id,
  order_date,
  product_name
FROM member_cte
WHERE _rank = 1
ORDER BY customer_id;
```
| customer_id | order_date               | product_name |
|-------------|--------------------------|--------------|
| A           | 2021-01-01T00:00:00.000Z | sushi        |
| A           | 2021-01-01T00:00:00.000Z | curry        |
| B           | 2021-01-04T00:00:00.000Z | sushi        |

> 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT
  sales.customer_id,
  COUNT(DISTINCT sales.product_id) unique_product_count,
  SUM(menu.price) total_expense
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
WHERE EXISTS (
  SELECT 1
  FROM dannys_diner.members
  WHERE sales.customer_id = members.customer_id
)
AND sales.order_date < members.join_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
| customer_id | unique_product_count | total_expense |
|-------------|----------------------|---------------|
| A           | 2                    | 25            |
| B           | 2                    | 40            |

> 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT
  sales.customer_id,
  SUM(
    CASE
      WHEN sales.product_id = 1 THEN menu.price * 20
      ELSE menu.price * 10
    END
  ) total_points
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | total_points |
|-------------|--------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |

> 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT
  sales.customer_id,
  SUM(
    CASE
      WHEN sales.order_date BETWEEN members.join_date AND (members.join_date + INTERVAL '6 DAYS') THEN menu.price * 20
      WHEN sales.order_date NOT BETWEEN members.join_date AND (members.join_date + INTERVAL '6 DAYS') AND sales.product_id = 1 THEN menu.price * 20
      ELSE menu.price * 10
    END
  ) AS total_points
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
WHERE order_date < '2021-02-01'
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
| customer_id | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 820          |

> 11. BONUS Q: Recreate the following table output using the available data:
```sql
SELECT
  sales.customer_id,
  sales.order_date,
  menu.product_name,
  menu.price,
  CASE
    WHEN sales.order_date >= members.join_date THEN 'Y' 
    ELSE 'N' 
  END AS member 
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
LEFT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
ORDER BY customer_id, order_date, product_name;
```
| customer_id | order_date               | product_name | price | member |
|-------------|--------------------------|--------------|-------|--------|
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

> 12. BONUS Q: Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
```sql
-- Create `member` field
WITH member_cte AS (
  SELECT
    sales.customer_id,
    sales.order_date,
    menu.product_name,
    menu.price,
    CASE
      WHEN sales.order_date >= members.join_date THEN 'Y' 
      ELSE 'N' 
    END AS member
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  LEFT JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  ORDER BY customer_id, order_date, product_name
)
-- Rank and use `member` in partition
SELECT 
  customer_id,
  order_date,
  product_name,
  price,
  member,
  CASE
    WHEN member = 'N' THEN NULL
    ELSE RANK() OVER (
      PARTITION BY customer_id, member
      ORDER BY order_date
      )
    END AS ranking
FROM member_cte;
```
| customer_id | order_date               | product_name | price | member | ranking |
|-------------|--------------------------|--------------|-------|--------|---------|
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      | NULL    |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      | NULL    |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1       |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      | NULL    |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      | NULL    |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      | NULL    |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      | NULL    |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      | NULL    |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      | NULL    |
