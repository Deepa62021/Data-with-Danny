--BASIC DATA TABLES :-
--1.Join sales and menu tables
DROP TABLE IF EXISTS menu_sales;
CREATE TEMP TABLE menu_sales AS
SELECT
  sales.customer_id,
  sales.order_date,
  sales.product_id,
  menu.product_name,
  menu.price
FROM
  dannys_diner.sales
  INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id;
SELECT
  *
FROM
  menu_sales;
--2.Join sales, menu, members tables
  DROP TABLE IF EXISTS menu_sales_members;
CREATE TEMP TABLE menu_sales_members AS
SELECT
  sales.customer_id,
  sales.order_date,
  sales.product_id,
  menu.product_name,
  menu.price,
  members.join_date
FROM
  dannys_diner.sales
  INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id
  INNER JOIN dannys_diner.members ON sales.customer_id = members.customer_id;
SELECT
  *
FROM
  menu_sales_members;
--3.Join sales,menu,members tables
  DROP TABLE IF EXISTS menu_sales_members_1;
CREATE TEMP TABLE menu_sales_members_1 AS
SELECT
  sales.customer_id,
  sales.order_date,
  sales.product_id,
  menu.product_name,
  menu.price,
  members.join_date
FROM
  dannys_diner.sales
  INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id
  LEFT JOIN dannys_diner.members ON sales.customer_id = members.customer_id;
SELECT
  *
FROM
  menu_sales_members_1;
--1.What is the total amount each customer spent at the restaurant?
  --From menu_sales table we can group by customer_id to get how much each customer spent altogether in the diner.
SELECT
  customer_id,
  SUM(price) AS Total_amount
FROM
  menu_sales
GROUP BY
  customer_id
ORDER BY
  Total_amount DESC;
A has spent $ 76,
  B has spent $ 74
  and C has spent $ 36 in total.--2. How many days has each customer visited the restaurant?
  --We can get it from counting the number of times he/she ordered by counting order_date.
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS num_days
FROM
  dannys_diner.sales
GROUP BY
  customer_id
ORDER BY
  num_days DESC;
--Customer A visited the restaurant 4 times, Customer B visited 6 times and Customer Cvisited 2 times.
  --3. What was the first item from the menu purchased by each customer?
  --WE have to use sales_menu table
  --First we have to find the first item of each customer that can be found with respect to order_date.
  --We need to rank them based on the order_date and then extract the data whose rank is 1.
  WITH ranking as(
    SELECT
      customer_id,
      product_name,
      order_date,
      DENSE_RANK() OVER (
        PARTITION BY customer_id
        order by
          order_date
      ) AS dense_ranking
    FROM
      menu_sales
  )
SELECT
  customer_id,
  product_name
FROM
  ranking
WHERE
  dense_ranking = 1
GROUP BY
  customer_id,
  product_name;
--A ordered curry and sushi, B ordered curry and C ordered ramen as their first dishes.
  --4.What is the most purchased item on the menu and how many times was it purchased by all customers?
  /** This has 2 questions i.e.
  * Most purchased item by all customers
  * Number of times that this famous item purchased by all customers.
  
  This can be obtained from customer_menu table. By counting each product_id in the table, we can find the product_name as well as the number of times it was bought in total**/
SELECT
  product_name,
  COUNT(product_id) AS total
FROM
  menu_sales
GROUP BY
  product_id,
  product_name
ORDER BY
  total DESC
LIMIT
  1;
Ramen is the popular item on the menu.--5. Which item was the most popular for each customer?
  /**This can be obtained by window function.
  1. First count the no. of times each product was bought by each customer
  2. Then rank them based on each customer
  3. Extract the ones we desire.**/
  WITH ranking AS (
    SELECT
      customer_id,
      product_name,
      COUNT(product_id) as item_quantity,
      DENSE_RANK() OVER (
        PARTITION BY customer_id
        ORDER BY
          COUNT(product_id) DESC
      ) AS item_rank
    FROM
      menu_sales
    GROUP BY
      customer_id,
      product_name
)
SELECT
  customer_id,
  product_name,
  item_quantity
FROM
  ranking
WHERE
  item_rank = 1;
-- Customers A & C like ramen . Customer B ordered all the items inthe menu same number of times.

  --6.Which item was purchased first by the customer after they became a member and what date was it? (including the date they joined)
  /** we use menu_sales_members table.
  We will use window function to get the result
  First we have to compare joining date with order dates and extract all the dates that fall after the joining date of each member of loyalty program.
  Then rank them according to their ordering date and extract the first item each customer purchased.**/
  
  WITH first_product AS (
SELECT
  customer_id,
  order_date,
  join_date,
  product_name,
  DENSE_RANK() OVER (PARTITION BY customer_id
                     ORDER BY order_date ASC) AS order_rank
FROM
  menu_sales_members
WHERE
  order_date >= join_date
  )
  
SELECT
  customer_id,
  product_name,
  order_date
FROM first_product
WHERE order_rank = 1;

/** Customer A purchased "curry" on "2021-01-07" and 
   Customer B purchased "sushi" on "2021-01-11" after they became bembers.**/
   
/** 7. Which menu item(s) was purchased just before the customer became a member and when?
we use the table menu_sales_members.
This is very similar to question 6 previously but now the record orders should be reversed using the window functions.**/

  WITH before_join AS (
SELECT
  customer_id,
  order_date,
  join_date,
  product_name,
  DENSE_RANK() OVER (PARTITION BY customer_id
                     ORDER BY order_date DESC) AS order_rank
FROM
  menu_sales_members
WHERE
  order_date < join_date
  )
  
SELECT
  customer_id,
  product_name,
  order_date
FROM before_join
WHERE order_rank = 1;

--A ordered sushi and curry where as B ordered sushi before becoming members.

/**8. What is the number of unique menu items and total amount spent for each member before they became a member?

From menu_sales_members table we can group the customers who ordered before they became members.
Filter the data who have order_date < join_date
Group By customers
Count unique items
Total the price.
**/

SELECT 
  customer_id,
  COUNT(DISTINCT product_name) AS num_unique_menu,
  SUM(price) AS total_amount
FROM menu_sales_members
WHERE order_date < join_date
GROUP BY customer_id;

--A has spent $25  on 2 unique items and B has spent $40 on 2 unique items on the menu from danny’s diner.

/**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
Here we use menu_sales_members_1 table.
For each customer , points are caluculated by 10 * price, if they ordered Sushi Points become 20 * price **/

SELECT 
  customer_id,
  SUM(CASE WHEN product_name = 'sushi' THEN 2*10*price
      ELSE 10*price
      END) AS total_points
FROM menu_sales_members_1
GROUP BY customer_id
ORDER BY total_points DESC;

--Customers A, B ,C would have points 860, 940, 360 respectively.

/** In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
  
  To calculate the points each product's price was multiplied by 10 and times 20 if the product was sushi. Additionally, if the customer became a member then the product's price would be multiplied by 20 for a week.**/
  
SELECT 
  customer_id,
  SUM(
  CASE WHEN order_date BETWEEN join_date AND  (join_date + 6) THEN 2 * 10 * price
       WHEN product_name = 'sushi' THEN 2 * 10 * price
       ELSE 10 * price
  END ) AS total_points
FROM menu_sales_members
WHERE order_date < '2021-02-01'
GROUP BY customer_id
ORDER BY customer_id;

--A has 1370 points and B has 820 points by ens of january.

/** 12.Recreate the following table using the available data:
  


| customer_id | order_date | product_name | price | member |
|-------------|------------|--------------|-------|--------|
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

**/

SELECT 
  customer_id, 
  order_date, 
  product_name, 
  price,
  (CASE 
    WHEN order_date >= join_date THEN 'Y'
    ELSE 'N'
  END) AS member
FROM menu_sales_members_1
ORDER BY customer_id, order_date;

/** 

  
  
  
  
