# Case Study # 2 : Pizza Runner 
<p align="center">
  <img src="pizza_runner.PNG"/>
</p>

## Context of Case Study 
This is a Case of Study based on a bussiness idea about a food start-up that prepares and delivery pizza to the final customer , this start-up hire "runners" who are persons that deliver pizza and they are paid per kilometer driven , once we know this the customer hired us as data analysts to answer some questions about their bussiness through the data that they provide us .

## Dataset 
This dataset is composed for 6 tables with useful information about the bussiness and their actual sales . 
### Schema Name : pizza_runner 
* pizza_runner.**runner_orders** : This table contains all the orders that have been delivered by each runner .
```SQL 
SELECT *
FROM pizza_runner.runner_orders 
LIMIT 5 ; 
``` 
| order_id | runner_id | pickup_time         | distance | duration   | cancellation |
|----------|-----------|---------------------|----------|------------|--------------|
| 1        | 1         | 2021-01-01 18:15:34 | 20km     | 32 minutes |              |
| 2        | 1         | 2021-01-01 19:10:54 | 20km     | 27 minutes |              |
| 3        | 1         | 2021-01-03 00:12:37 | 13.4km   | 20 mins    | null         |
| 4        | 2         | 2021-01-04 13:53:03 | 23.4     | 40         | null         |
| 5        | 3         | 2021-01-08 21:10:57 | 10       | 15         | null         |

* pizza_runner.runners : This table contains the runners that are actually hired by the company .
```SQL 
SELECT *
FROM pizza_runner.runners
LIMIT 5 ; 
```
| runner_id | registration_date        |
|-----------|--------------------------|
| 1         | 2021-01-01T00:00:00.000Z |
| 2         | 2021-01-03T00:00:00.000Z |
| 3         | 2021-01-08T00:00:00.000Z |
| 4         | 2021-01-15T00:00:00.000Z |



* pizza_runner.customer_orders : This table is very important because here are all the records about sales  . 

```SQL 
SELECT *
FROM pizza_runner.customer_orders
LIMIT 5 ; 
```
| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
|----------|-------------|----------|------------|--------|--------------------------|
| 1        | 101         | 1        |            |        | 2021-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2021-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2021-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            | null   | 2021-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2021-01-04T13:23:46.000Z |

* pizza_runner.**pizza_names** : This table help us to relate the pizza names with the pizza id this last is the common column in the runner_orders and customer_orders table .

```SQL 
SELECT *
FROM pizza_runner.pizza_names
LIMIT 5 ; 

``` 
| pizza_id | pizza_name |
|----------|------------|
| 1        | Meatlovers |
| 2        | Vegetarian |


* pizza_runner.**pizza_recipes** : this table contain all the recipes of the pizzas and help us to link the id of the pizza and the toppings that are used to make it .
```SQL 
SELECT *
FROM pizza_runner.pizza_recipes
LIMIT 5 ; 
```
| pizza_id | toppings                |
|----------|-------------------------|
| 1        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 2        | 4, 6, 7, 9, 11, 12      |

* pizza_toppings : Finally we have the last table that shows up  what the topping names are .

```SQL 
SELECT *
FROM  pizza_runner.pizza_toppings 
LIMIT 5 ; 
```
| topping_id | topping_name |
|------------|--------------|
| 1          | Bacon        |
| 2          | BBQ Sauce    |
| 3          | Beef         |
| 4          | Cheese       |
| 5          | Chicken      |


## Entity Relationship Diagram 
The following ERD describes the relationship between all the tables and the primary key that link the tables that have a relationship .

<p align="center">
  <img src="ERD.PNG"/>
</p>

## Customer Questions 
This section convers the answers to the questions of our customer and are divided by three fourth sections .

A. Pizza Metrics : This section is about data exploration and apply basic methods to extract useful insights .

1. How many pizzas were ordered?
```SQL
SELECT
  COUNT(*) AS pizza_total
FROM
  pizza_runner.customer_orders;

``` 
| pizza_total |
|-------------|
| 14          |

2. How many unique customer orders were made?
```SQL
SELECT
  COUNT(DISTINCT order_id) AS unique_orders
FROM
  pizza_runner.customer_orders;
``` 
| unique_orders |
|---------------|
| 10            |

3. How many successful orders were delivered by each runner?
```SQL
WITH runner_rate AS (
    SELECT
      order_id,
      cancellation,
      runner_id,
      CASE
        WHEN cancellation isnull
        OR cancellation = ''
        OR cancellation = 'null' THEN 1 :: NUMERIC
        ELSE 0 :: NUMERIC
      END AS cancellation_binary
    FROM
      pizza_runner.runner_orders
  )
SELECT
  runner_id,
  COUNT(cancellation_binary) as successful_delivered
FROM
  runner_rate
WHERE
  cancellation_binary = 1 :: NUMERIC
GROUP BY
  runner_id
ORDER BY
  runner_id;
``` 
| runner_id | successful_delivered |
|-----------|----------------------|
| 1         | 4                    |
| 2         | 3                    |
| 3         | 1                    |

4. How many of each type of pizza was delivered?
```SQL
 DROP TABLE IF EXISTS pizza_delivered;
CREATE TEMP TABLE pizza_delivered AS
SELECT
  co.order_id as order_id,
  co.pizza_id as pizza_id,
  ro.pickup_time as pickup_time,
  ro.cancellation as cancellation,
  pa.pizza_name as pizza_name,
  co.customer_id as customer_id
FROM
  pizza_runner.customer_orders AS co FULL
  JOIN pizza_runner.runner_orders AS ro ON co.order_id = ro.order_id
  INNER JOIN pizza_runner.pizza_names AS pa ON co.pizza_id = pa.pizza_id;
SELECT
  *
FROM
  pizza_delivered
ORDER BY
  order_id;
SELECT
  pizza_name,
  count(pizza_name) as COUNTER
FROM
  pizza_delivered
WHERE
  pickup_time != 'null'
  OR null
GROUP BY
  pizza_name;
``` 
| pizza_name | counter |
|------------|---------|
| Meatlovers | 9       |
| Vegetarian | 3       |


5. How many Vegetarian and Meatlovers were ordered by each  
customer?
```SQL
 SELECT
  customer_id,
  pizza_name,
  COUNT(pizza_name) AS PIZZA
FROM
  pizza_delivered
GROUP BY
  customer_id,
  pizza_name
ORDER BY
  customer_id;
``` 
| customer_id | pizza_name | pizza |
|-------------|------------|-------|
| 101         | Vegetarian | 1     |
| 101         | Meatlovers | 2     |
| 102         | Vegetarian | 1     |
| 102         | Meatlovers | 2     |
| 103         | Meatlovers | 3     |
| 103         | Vegetarian | 1     |
| 104         | Meatlovers | 3     |
| 105         | Vegetarian | 1     |

6. What was the maximum number of pizzas delivered in a single order?
```SQL
SELECT
  order_id,
  COUNT(order_id) AS pizzas_delivered
FROM
  pizza_delivered
WHERE
  pickup_time != 'null'
  OR null
GROUP BY
  order_id
ORDER BY
  pizzas_delivered DESC
LIMIT
  1;
``` 
| order_id | pizzas_delivered |
|----------|------------------|
| 4        | 3                |

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```SQL
WITH cte_cleaned_customer_orders AS (
    SELECT
      t1.order_id,
      t1.customer_id,
      t1.pizza_id,
      CASE
        WHEN t2.cancellation ~ '[RC]' THEN 1
        ELSE NULL
      END AS cancellation,
      CASE
        WHEN exclusions IN ('null', '') THEN NULL
        ELSE exclusions
      END AS exclusions,
      CASE
        WHEN extras IN ('null', '') THEN NULL
        ELSE extras
      END AS extras
    FROM
      pizza_runner.customer_orders AS t1
      LEFT JOIN pizza_runner.runner_orders AS t2 ON t1.order_id = t2.order_id
    ORDER BY
      customer_id
  )
SELECT
  customer_id,
  SUM (
    CASE
      WHEN cancellation IS NULL
      AND exclusions IS NOT NULL
      OR extras IS NOT NULL THEN 1
      ELSE 0
    END
  ) AS one_change,
  SUM(
    CASE
      WHEN exclusions IS NULL
      AND extras IS NULL
      AND cancellation IS NULL THEN 1
      ELSE 0
    END
  ) AS no_changes
FROM
  cte_cleaned_customer_orders
GROUP BY
  customer_id
ORDER BY
  customer_id;
``` 
| customer_id | one_change | no_changes |
|-------------|------------|------------|
| 101         | 0          | 2          |
| 102         | 0          | 3          |
| 103         | 4          | 0          |
| 104         | 2          | 1          |
| 105         | 1          | 0          |

8. How many pizzas were delivered that had both exclusions and extras?
```SQL
WITH cte_cleaned_customer_orders AS (
    SELECT
      order_id,
      customer_id,
      pizza_id,
      CASE
        WHEN exclusions IN ('null', '') THEN NULL
        ELSE exclusions
      END AS exclusions,
      CASE
        WHEN extras IN ('null', '') THEN NULL
        ELSE extras
      END AS extras
    FROM
      pizza_runner.customer_orders
  )
SELECT
  COUNT(*) AS pizzas_with_extras_and_exclusions
FROM
  cte_cleaned_customer_orders
WHERE
  exclusions IS NOT NULL
  AND extras IS NOT NULL;
``` 
9. What was the total volume of pizzas ordered for each hour of the day?
```SQL
SELECT
  EXTRACT (
    HOUR
    FROM
      order_time
  ) AS order_hour,
  COUNT(*) AS orders_per_hour
FROM
  pizza_runner.customer_orders
GROUP BY
  order_hour
ORDER BY
  order_hour;
``` 
| pizzas_with_extras_and_exclusions |
|-----------------------------------|
| 2                                 |
10. What was the volume of orders for each day of the week?
```SQL
SELECT
  TO_CHAR(order_time, 'Day') AS day_of_week,
  count(*)
FROM
  pizza_runner.customer_orders
GROUP BY
  day_of_week,
  DATE_PART('dow', order_time)
ORDER BY
  DATE_PART('dow', order_time);
``` 
| day_of_week | count |
|-------------|-------|
| Sunday      | 1     |
| Monday      | 5     |
| Friday      | 5     |
| Saturday    | 3     |

B. Runner and Customer Experience : In this section we developed an analysis of the deliveries in terms of speed and time .

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```SQL 
SELECT
  DATE_TRUNC('week', registration_date) :: DATE + 4 AS registration_week,
  COUNT(*) AS runners
FROM
  pizza_runner.runners
GROUP BY
  registration_week
ORDER BY
  registration_week;
```
| registration_week        | runners |
|--------------------------|---------|
| 2021-01-01T00:00:00.000Z | 2       |
| 2021-01-08T00:00:00.000Z | 1       |
| 2021-01-15T00:00:00.000Z | 1       |

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```SQL 
WITH time_per_order AS (
SELECT
  a.runner_id AS runner_id,
  a.pickup_time AS  pickup_time,
  b.order_time AS order_time
FROM
  pizza_runner.runner_orders AS a 
  FULL JOIN pizza_runner.customer_orders AS b 
  ON a.order_id = b.order_id
  WHERE pickup_time !='null'),
filter_time_per_order AS (
SELECT DISTINCT *
FROM time_per_order 
ORDER BY runner_id),
minutes_per_order AS (
SELECT runner_id,((EXTRACT(EPOCH FROM  pickup_time::TIMESTAMP)  - EXTRACT (EPOCH FROM  order_time::TIMESTAMP))/60::NUMERIC) AS time_in_minutes
FROM filter_time_per_order
)
SELECT AVG(time_in_minutes) as avg_time
FROM minutes_per_order;

```
| avg_time           |
|--------------------|
| 15.977083333333333 |


3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```SQL 
WITH time_to_prepare AS (
SELECT 
co.order_id AS order_id,
ro.pickup_time AS pickup_time,
co.order_time AS order_time
FROM pizza_runner.customer_orders AS co 
FULL JOIN pizza_runner.runner_orders AS ro 
ON co.order_id = ro.order_id 
WHERE pickup_time != 'null'
)
SELECT order_id , COUNT(*) AS number_of_pizzas  , 
ROUND(((EXTRACT(EPOCH FROM pickup_time::TIMESTAMP) - EXTRACT(EPOCH FROM order_time::TIMESTAMP))/60)::NUMERIC,2) AS time_spending 
FROM time_to_prepare
GROUP BY order_id,pickup_time,order_time
ORDER BY number_of_pizzas DESC; 
```
| order_id | number_of_pizzas | time_spending |
|----------|------------------|---------------|
| 4        | 3                | 29.28         |
| 10       | 2                | 15.52         |
| 3        | 2                | 21.23         |
| 2        | 1                | 10.03         |
| 8        | 1                | 20.48         |
| 7        | 1                | 10.27         |
| 5        | 1                | 10.47         |
| 1        | 1                | 10.53         |

4. What was the average distance travelled for each customer?
```SQL 
WITH average_distance AS (
SELECT 
co.customer_id AS customer_id,
ro.order_id AS order_id, 
ro.distance AS distance
FROM pizza_runner.runner_orders AS ro
LEFT JOIN pizza_runner.customer_orders AS co
ON ro.order_id = co.order_id
WHERE distance != 'null'
)
SELECT customer_id,ROUND(AVG((REGEXP_MATCH(distance,'(^[0-9.,]+)'))[1]::NUMERIC),2) AS AVERAGE
FROM average_distance
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | average |
|-------------|---------|
| 101         | 20.00   |
| 102         | 16.73   |
| 103         | 23.40   |
| 104         | 10.00   |
| 105         | 25.00   |

5. What was the difference between the longest and shortest delivery times for all orders?
```SQL 
WITH delivery_times AS (
SELECT order_id,(REGEXP_MATCH(duration,'(^[0-9]+)'))[1]::NUMERIC AS duration_in_minutes
FROM pizza_runner.runner_orders
WHERE duration != 'null'
)
SELECT MAX(duration_in_minutes) - MIN(duration_in_minutes) AS difference_in_minutes
FROM delivery_times ; 
```
| difference_in_minutes |
|-----------------------|
| 30                    |
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```SQL 
WITH average_speed AS (
SELECT 
runner_id,
(REGEXP_MATCH(distance,'(^[0-9.,]+)'))[1]::NUMERIC AS distance_in_km,
(REGEXP_MATCH(duration,'(^[0-9]+)'))[1]::NUMERIC AS duration_in_min
FROM pizza_runner.runner_orders
WHERE distance != 'null' AND duration != 'null'

)
SELECT ROUND(distance_in_km/(duration_in_min/60),2) AS speed_per_runner_in_meters_per_second
FROM average_speed ; 
```
| speed_per_runner_in_meters_per_second |
|---------------------------------------|
| 37.50                                 |
| 44.44                                 |
| 40.20                                 |
| 35.10                                 |
| 40.00                                 |
| 60.00                                 |
| 93.60                                 |
| 60.00                                 |

7. What is the successful delivery percentage for each runner?
```SQL 
WITH total_counts AS (
SELECT runner_id,COUNT(runner_id) AS orders 
FROM pizza_runner.runner_orders
GROUP BY runner_id ) ,
success_counts AS (
SELECT runner_id,COUNT(runner_id) AS orders_check 
FROM pizza_runner.runner_orders
WHERE pickup_time != 'null'
GROUP BY runner_id
)

SELECT 
tc.runner_id,
tc.orders,
sc.orders_check,
CONCAT((CEILING((sc.orders_check::NUMERIC/tc.orders::NUMERIC)*100)::varchar),'%'::varchar) AS success_percentage
FROM total_counts AS tc
FULL JOIN success_counts AS sc
ON tc.runner_id = sc.runner_id ;
```
| runner_id | orders | orders_check | success_percentage |
|-----------|--------|--------------|--------------------|
| 3         | 2      | 1            | 50%                |
| 2         | 4      | 3            | 75%                |
| 1         | 4      | 4            | 100%               |

C. Ingredient Optimisation : In this section we generated queries that help us to understand the customer's preferences and what was the impact in the product.

1. What are the standard ingredients for each pizza?
```SQL 
SELECT cs.pizza_id ,STRING_AGG(pt.topping_name,',') AS ingredients
FROM split_pizza_names AS cs
FULL JOIN pizza_runner.pizza_toppings AS pt
ON cs.topping_id = pt.topping_id
GROUP BY pizza_id
ORDER BY pizza_id
```
| pizza_id | ingredients                                                    |
|----------|----------------------------------------------------------------|
| 1        | Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 2        | Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce          |

2. What was the most commonly added extra?
```SQL 
WITH common_extras AS (
SELECT REGEXP_SPLIT_TO_TABLE(extras,'[,\s]+')::INTEGER AS topping_id
FROM pizza_runner.customer_orders 
WHERE extras != 'null' AND extras != ''
)
SELECT ce.topping_id , tp.topping_name,count(*) AS number_of_extras
FROM pizza_runner.pizza_toppings AS tp
INNER JOIN common_extras AS ce 
ON ce.topping_id = tp.topping_id
GROUP BY ce.topping_id , tp.topping_name
ORDER BY number_of_extras DESC; 

```
| topping_id | topping_name | number_of_extras |
|------------|--------------|------------------|
| 1          | Bacon        | 4                |
| 4          | Cheese       | 1                |
| 5          | Chicken      | 1                |

3. What was the most common exclusion?
```SQL 
WITH common_extras AS (
SELECT REGEXP_SPLIT_TO_TABLE(exclusions,'[,\s]+')::INTEGER AS topping_id
FROM pizza_runner.customer_orders 
WHERE exclusions != 'null' AND exclusions != '' 
)
SELECT ce.topping_id , tp.topping_name,count(*) AS number_of_exclusions
FROM pizza_runner.pizza_toppings AS tp
INNER JOIN common_extras AS ce 
ON ce.topping_id = tp.topping_id
GROUP BY ce.topping_id , tp.topping_name
ORDER BY number_of_exclusions DESC; 

```
| topping_id | topping_name | number_of_exclusions |
|------------|--------------|----------------------|
| 4          | Cheese       | 4                    |
| 6          | Mushrooms    | 1                    |
| 2          | BBQ Sauce    | 1                    |

4. Generate an order item for each record in the customers_orders table in the format of one of the following:

Meat Lovers 

Meat Lovers - Exclude Beef 

Meat Lovers - Extra Bacon

Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```SQL 
DROP TABLE IF EXISTS pizzas_custom ;
CREATE TEMP TABLE  pizzas_custom AS 
SELECT 
ROW_NUMBER() OVER () AS pizza_order,
co.order_id AS order_id,
co.customer_id AS customer_id,
co.pizza_id AS pizza_id,
CASE 
WHEN co.exclusions = '' THEN 'null' ELSE co.exclusions END AS exclusions,
CASE 
WHEN co.extras= '' OR co.extras ISNULL THEN 'null' ELSE co.extras END  AS extras,
co.order_time AS order_time ,
pn.pizza_name AS pizza_name
FROM pizza_runner.customer_orders AS co 
FULL JOIN pizza_runner.pizza_names AS pn
ON co.pizza_id = pn.pizza_id;

WITH convert_exclusions AS (
SELECT pizza_order,order_id ,pizza_name, REGEXP_SPLIT_TO_TABLE(exclusions,'[,\s]+')::INTEGER AS topping_id
FROM pizzas_custom
WHERE exclusions != 'null'),
convert_exclusions_names AS(
SELECT 
ce.pizza_order,
ce.order_id ,
ce.pizza_name,
pt.topping_name
FROM convert_exclusions AS ce
LEFT JOIN pizza_runner.pizza_toppings AS pt
ON ce.topping_id = pt.topping_id
),
string_list_exclusions AS (
SELECT  pizza_order , order_id, pizza_name,CONCAT('Exclude : '::TEXT,STRING_AGG(topping_name,',')) AS pizza_exclusions
FROM convert_exclusions_names 
GROUP BY pizza_order, order_id, pizza_name),
convert_extras AS (
SELECT pizza_order,order_id ,pizza_name, REGEXP_SPLIT_TO_TABLE(extras,'[,\s]+')::INTEGER AS topping_id
FROM pizzas_custom
WHERE extras != 'null'
),
convert_extras_names AS(
SELECT 
ce.pizza_order,
ce.order_id ,
ce.pizza_name,
pt.topping_name
FROM convert_extras AS ce
LEFT JOIN pizza_runner.pizza_toppings AS pt
ON ce.topping_id = pt.topping_id
),
string_list_extras AS (
SELECT  pizza_order , order_id, pizza_name,CONCAT('Extra : '::TEXT,STRING_AGG(topping_name,',')) AS pizza_extras
FROM convert_extras_names 
GROUP BY pizza_order, order_id, pizza_name
),
combinations_query AS (
SELECT 
pc.pizza_order, 
pc.order_id,
pc.customer_id,
pc.order_time,
pc.pizza_name,
CASE 
WHEN  sl.pizza_extras ISNULL THEN '' ELSE sl.pizza_extras END AS pizza_extras ,
CASE
WHEN  sle.pizza_exclusions ISNULL THEN '' ELSE sle.pizza_exclusions END AS pizza_exclusions
FROM pizzas_custom AS pc
FULL JOIN string_list_extras as sl 
ON pc.pizza_order = sl.pizza_order
FULL JOIN  string_list_exclusions AS sle
ON pc.pizza_order = sle.pizza_order )

SELECT 
order_id,
customer_id,
order_time,
CONCAT(pizza_name,' -',pizza_extras,' -',pizza_exclusions) AS order_item
FROM combinations_query 
ORDER BY order_id;

```
| order_id | customer_id | order_time               | order_item                                                      |
|----------|-------------|--------------------------|-----------------------------------------------------------------|
| 1        | 101         | 2021-01-01T18:05:02.000Z | Meatlovers - -                                                  |
| 2        | 101         | 2021-01-01T19:00:52.000Z | Meatlovers - -                                                  |
| 3        | 102         | 2021-01-02T23:51:23.000Z | Meatlovers - -                                                  |
| 3        | 102         | 2021-01-02T23:51:23.000Z | Vegetarian - -                                                  |
| 4        | 103         | 2021-01-04T13:23:46.000Z | Vegetarian - -Exclude : Cheese                                  |
| 4        | 103         | 2021-01-04T13:23:46.000Z | Meatlovers - -Exclude : Cheese                                  |
| 4        | 103         | 2021-01-04T13:23:46.000Z | Meatlovers - -Exclude : Cheese                                  |
| 5        | 104         | 2021-01-08T21:00:29.000Z | Meatlovers -Extra : Bacon -                                     |
| 6        | 101         | 2021-01-08T21:03:13.000Z | Vegetarian - -                                                  |
| 7        | 105         | 2021-01-08T21:20:29.000Z | Vegetarian -Extra : Bacon -                                     |
| 8        | 102         | 2021-01-09T23:54:33.000Z | Meatlovers - -                                                  |
| 9        | 103         | 2021-01-10T11:22:59.000Z | Meatlovers -Extra : Bacon,Chicken -Exclude : Cheese             |
| 10       | 104         | 2021-01-11T18:34:49.000Z | Meatlovers -Extra : Bacon,Cheese -Exclude : BBQ Sauce,Mushrooms |
| 10       | 104         | 2021-01-11T18:34:49.000Z | Meatlovers - -                                                  |

5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients

For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
```SQL 
DROP TABLE IF EXISTS pizzas_custom ;
CREATE TEMP TABLE  pizzas_custom AS 
SELECT 
ROW_NUMBER() OVER () AS pizza_order,
co.order_id AS order_id,
co.customer_id AS customer_id,
co.pizza_id AS pizza_id,
CASE 
WHEN co.exclusions  = 'null' THEN '' ELSE co.exclusions END AS exclusions,
CASE 
WHEN co.extras ISNULL OR co.extras = 'null' THEN '' ELSE co.extras END  AS extras,
co.order_time AS order_time ,
pn.pizza_name AS pizza_name,
pr.toppings AS pizza_toppings
FROM pizza_runner.customer_orders AS co 
FULL JOIN pizza_runner.pizza_names AS pn
ON co.pizza_id = pn.pizza_id
FULL JOIN pizza_runner.pizza_recipes AS pr 
ON pn.pizza_id = pr.pizza_id ;

WITH extras_concat AS (
SELECT 
pizza_order ,
REGEXP_SPLIT_TO_TABLE (CASE 
WHEN extras = '' THEN pizza_toppings ELSE (CONCAT(extras ,',',pizza_toppings)) END ,'[,\s]+')::INTEGER AS topping_id
FROM pizzas_custom

),
topping_counts AS(
SELECT 
ec.pizza_order  AS pizza_order, 
ec.topping_id AS pizza_toppings_table,
pt.topping_name AS topping_name,
COUNT(topping_name) AS counts
FROM extras_concat AS ec 
FULL JOIN pizza_runner.pizza_toppings AS pt 
ON ec.topping_id = pt.topping_id
GROUP BY pizza_order,pizza_toppings_table,topping_name
),
table_extra_quantities AS(
SELECT
pizza_order,
CASE 
WHEN counts >= 2 THEN CONCAT(counts::TEXT ,'X ',topping_name) ELSE topping_name END AS topping_name
FROM topping_counts
GROUP BY pizza_order,counts,topping_name
),
exclusions_extraction AS (
SELECT 
pizza_order ,
REGEXP_SPLIT_TO_TABLE(exclusions,'[,\s]+')::INTEGER AS topping_id 
FROM pizzas_custom
WHERE exclusions != ''
),
topping_exclusions AS (
SELECT 
ex.pizza_order ,
pt.topping_name AS topping_name
FROM exclusions_extraction  AS ex
INNER JOIN pizza_runner.pizza_toppings AS  pt
ON ex.topping_id = pt.topping_id 
),
pizza_overall AS (
SELECT 
teq.pizza_order ,
teq.topping_name 
FROM table_extra_quantities as teq 
WHERE NOT EXISTS(
SELECT ex.pizza_order,ex.topping_name
FROM topping_exclusions AS ex
WHERE teq.pizza_order = ex.pizza_order AND teq.topping_name =ex.topping_name )
ORDER BY pizza_order),
string_unions AS (
SELECT 
pizza_order ,STRING_AGG(topping_name,',')AS pizza_ingredients 
FROM pizza_overall
GROUP BY pizza_order 
)
SELECT
pc.order_id,
pc.pizza_order,
su.pizza_ingredients,
pc.customer_id 
FROM string_unions AS su 
FULL JOIN pizzas_custom AS pc 
ON pc.pizza_order = su.pizza_order
ORDER BY customer_id,pizza_order;

```
| order_id | pizza_order | pizza_ingredients                                                 | customer_id |
|----------|-------------|-------------------------------------------------------------------|-------------|
| 2        | 2           | Pepperoni,BBQ Sauce,Beef,Bacon,Salami,Cheese,Chicken,Mushrooms    | 101         |
| 1        | 7           | Bacon,Mushrooms,Salami,BBQ Sauce,Cheese,Beef,Chicken,Pepperoni    | 101         |
| 6        | 13          | Cheese,Peppers,Mushrooms,Tomatoes,Tomato Sauce,Onions             | 101         |
| 3        | 3           | Chicken,Mushrooms,Pepperoni,Beef,Cheese,BBQ Sauce,Bacon,Salami    | 102         |
| 8        | 4           | Pepperoni,Mushrooms,Chicken,BBQ Sauce,Salami,Cheese,Beef,Bacon    | 102         |
| 3        | 11          | Cheese,Onions,Tomato Sauce,Peppers,Mushrooms,Tomatoes             | 102         |
| 9        | 5           | Beef,2X Bacon,Salami,Mushrooms,BBQ Sauce,Pepperoni,2X Chicken     | 103         |
| 4        | 8           | Beef,Chicken,Salami,Pepperoni,Bacon,Mushrooms,BBQ Sauce           | 103         |
| 4        | 9           | Bacon,BBQ Sauce,Salami,Pepperoni,Chicken,Beef,Mushrooms           | 103         |
| 4        | 14          | Onions,Tomato Sauce,Peppers,Mushrooms,Tomatoes                    | 103         |
| 10       | 1           | 2X Cheese,2X Bacon,Chicken,Beef,Salami,Pepperoni                  | 104         |
| 10       | 6           | Pepperoni,Beef,Mushrooms,Cheese,BBQ Sauce,Bacon,Chicken,Salami    | 104         |
| 5        | 10          | BBQ Sauce,Cheese,2X Bacon,Mushrooms,Pepperoni,Beef,Chicken,Salami | 104         |
| 7        | 12          | Mushrooms,Tomato Sauce,Peppers,Tomatoes,Bacon,Cheese,Onions       | 105         |

6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
```SQL 
DROP TABLE IF EXISTS pizzas_custom ;
CREATE TEMP TABLE  pizzas_custom AS 
SELECT 
ROW_NUMBER() OVER () AS pizza_order,
co.order_id AS order_id,
co.customer_id AS customer_id,
co.pizza_id AS pizza_id,
CASE 
WHEN co.exclusions  = 'null' THEN '' ELSE co.exclusions END AS exclusions,
CASE 
WHEN co.extras ISNULL OR co.extras = 'null' THEN '' ELSE co.extras END  AS extras,
co.order_time AS order_time ,
pn.pizza_name AS pizza_name,
pr.toppings AS pizza_toppings
FROM pizza_runner.customer_orders AS co 
FULL JOIN pizza_runner.pizza_names AS pn
ON co.pizza_id = pn.pizza_id
FULL JOIN pizza_runner.pizza_recipes AS pr 
ON pn.pizza_id = pr.pizza_id ;

WITH extras_concat AS (
SELECT 
pizza_order ,
REGEXP_SPLIT_TO_TABLE (CASE 
WHEN extras = '' THEN pizza_toppings ELSE (CONCAT(extras ,',',pizza_toppings)) END ,'[,\s]+')::INTEGER AS topping_id
FROM pizzas_custom
),
topping_counts AS(
SELECT 
pt.topping_name AS topping_name,
COUNT(*) AS topping_counts
FROM extras_concat AS ec 
FULL JOIN pizza_runner.pizza_toppings AS pt 
ON ec.topping_id = pt.topping_id
GROUP BY topping_name 
),
exclusions_extraction AS (
SELECT 
REGEXP_SPLIT_TO_TABLE(exclusions,'[,\s]+')::INTEGER AS topping_id 
FROM pizzas_custom
WHERE exclusions != ''
),
topping_exclusions AS (
SELECT 
pt.topping_name AS topping_name,COUNT(*) AS total_count
FROM exclusions_extraction  AS ex
INNER JOIN pizza_runner.pizza_toppings AS  pt
ON ex.topping_id = pt.topping_id 
GROUP BY topping_name 
),
pizza_overall AS (
SELECT 
teq.topping_name,
teq.topping_counts - (CASE WHEN te.total_count ISNULL THEN 0 ELSE te.total_count END ) AS total_count
FROM topping_counts as teq
FULL JOIN topping_exclusions AS te 
ON teq.topping_name  = te.topping_name)

SELECT *
FROM pizza_overall
ORDER BY total_count DESC;

```
| topping_name | total_count |
|--------------|-------------|
| Bacon        | 14          |
| Mushrooms    | 13          |
| Chicken      | 11          |
| Cheese       | 11          |
| Pepperoni    | 10          |
| Salami       | 10          |
| Beef         | 10          |
| BBQ Sauce    | 9           |
| Tomato Sauce | 4           |
| Onions       | 4           |
| Tomatoes     | 4           |
| Peppers      | 4           |

D. Pricing and Ratings : This section covers the pricing of the products , the payment of the runners and how we can improve the customer service . 

1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```SQL 
WITH runner_customer_join AS (
SELECT co.*,
ro.pickup_time 
FROM pizza_runner.customer_orders AS co 
FULL JOIN pizza_runner.runner_orders AS ro 
ON co.order_id = ro.order_id 
)
SELECT 
SUM(CASE 
WHEN pizza_id = 1 THEN 12 ELSE 10 END )AS total_money_pizza
FROM runner_customer_join
```
| total_money_pizza |
|-------------------|
| 160               |

2. What if there was an additional $1 charge for any pizza extras?

    Add cheese is $1 extra

```SQL 
WITH runner_customer_join AS (
SELECT co.*,
ro.pickup_time 
FROM pizza_runner.customer_orders AS co 
FULL JOIN pizza_runner.runner_orders AS ro 
ON co.order_id = ro.order_id 
),
extras_division AS (
SELECT 
ROW_NUMBER() OVER() AS pizza_order,
order_id,
REGEXP_SPLIT_TO_TABLE (extras,'[,\s]+') AS extras 
FROM runner_customer_join
WHERE extras != 'null' AND extras != '' AND pickup_time != 'null'
),
extras_money AS(
SELECT SUM(extras::INTEGER) AS extras_usd
FROM extras_division 
),
extras_sum AS (
SELECT *
FROM extras_money 
UNION ALL 
SELECT 
SUM(CASE 
WHEN pizza_id = 1 THEN 12 ELSE 10 END )AS total_money_pizza
FROM pizza_runner.customer_orders )

SELECT SUM(extras_usd) AS total_with_extras 
FROM extras_sum ; 

```
| total_with_extras |
|-------------------|
| 167               |

3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```SQL 
DROP SCHEMA IF EXISTS pizza_runner_two CASCADE;
CREATE SCHEMA pizza_runner_two;

DROP TABLE IF EXISTS pizza_runner_two.customer_orders;
CREATE TABLE pizza_runner_two.customer_orders AS
SELECT * FROM pizza_runner.customer_orders;

DROP TABLE IF EXISTS pizza_runner_two.runner_orders;
CREATE TABLE pizza_runner_two.runner_orders AS
SELECT * FROM pizza_runner.runner_orders;

DROP TABLE IF EXISTS pizza_runner_two.runners;
CREATE TABLE pizza_runner_two.runners AS
SELECT * FROM pizza_runner.runners;

DROP TABLE IF EXISTS pizza_runner_two.runners;
CREATE TABLE pizza_runner_two.runners AS
SELECT * FROM pizza_runner.runners;


DROP TABLE IF EXISTS pizza_runner_two.pizza_toppings;
CREATE TABLE pizza_runner_two.pizza_toppings AS
SELECT * FROM pizza_runner.pizza_toppings;


DROP TABLE IF EXISTS pizza_runner_two.pizza_recipes;
CREATE TABLE pizza_runner_two.pizza_recipes AS
SELECT * FROM pizza_runner.pizza_recipes;

DROP TABLE IF EXISTS pizza_runner_two.pizza_names;
CREATE TABLE pizza_runner_two.pizza_names AS
SELECT * FROM pizza_runner.pizza_names ;

DROP TABLE IF EXISTS pizza_runner_two.runner_score;
CREATE TABLE pizza_runner_two.runner_score (
order_id int ,
runner_id int ,
duration_in_min int,
distance_in_km int,
ratio DECIMAL)
;

INSERT INTO pizza_runner_two.runner_score
SELECT 
order_id,
runner_id,
(REGEXP_MATCH (duration,'[0-9]+'))[1]::INTEGER AS duration_in_min,
(REGEXP_MATCH (distance,'[0-9]+'))[1]::INTEGER AS distance_in_km,
ROUND((REGEXP_MATCH (distance,'[0-9]+'))[1]::DECIMAL /((REGEXP_MATCH (duration,'[0-9]+'))[1]::DECIMAL/60) , 2) AS ratio
FROM pizza_runner.runner_orders;

DROP TABLE IF EXISTS runner_scores ;
CREATE TEMP TABLE runner_scores AS
SELECT *, 
CASE 
WHEN ratio <= 0.5 
THEN  1
WHEN ratio <= 0.6
THEN 2
WHEN ratio <=0.7
THEN 3
WHEN ratio <= 0.8
THEN 4
WHEN ratio >= 1
THEN 5 END AS score 
FROM pizza_runner_two.runner_score;

```
4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?

    customer_id                 
    order_id  
    runner_id      
rating     
order_time      
pickup_time     
Time between order and pickup     
Delivery duration    
Average speed   
Total number of pizzas

```SQL 
DROP TABLE IF EXISTS info_pizza ;
CREATE TEMP TABLE info_pizza AS 
SELECT 
co.customer_id AS customer_id , 
co.order_id AS order_ids ,
ro.runner_id AS runner_ids ,
re.score AS  rating ,
co.order_time  AS order_time , 
ro.pickup_time AS pickup_time,
re.ratio AS average_speed,
COUNT(co.*) AS total_number_of_pizzas 
FROM pizza_runner_two.customer_orders AS co 
FULL JOIN pizza_runner_two.runner_orders AS ro 
ON co.order_id = ro.order_id 
FULL JOIN runner_scores AS re 
ON ro.order_id  = re.order_id 
GROUP BY order_ids,customer_id,runner_ids,rating,order_time,pickup_time,average_speed;


SELECT *
FROM info_pizza
ORDER BY customer_id;
```
5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```SQL 
WITH cte_adjusted_runner_orders AS (
  SELECT
    t1.order_id,
    t1.runner_id,
    t2.order_time,
    t1.pickup_time::TIMESTAMP AS pickup_time,
    UNNEST(REGEXP_MATCH(t1.duration, '(^[0-9]+)'))::NUMERIC AS duration,
    UNNEST(REGEXP_MATCH(t1.distance, '(^[0-9,.]+)'))::NUMERIC AS distance,
    SUM(CASE WHEN t2.pizza_id = 1 THEN 1 ELSE 0 END) AS meatlovers_count,
    SUM(CASE WHEN t2.pizza_id = 2 THEN 1 ELSE 0 END) AS vegetarian_count
  FROM pizza_runner.runner_orders AS t1
  INNER JOIN pizza_runner.customer_orders AS t2
    ON t1.order_id = t2.order_id
  WHERE t1.pickup_time != 'null'
  GROUP BY
    t1.order_id,
    t1.runner_id,
    t2.order_time,
    t1.pickup_time,
    t1.duration,
    t1.distance
)
SELECT
  SUM(
    (12 * meatlovers_count + 10*vegetarian_count )- 0.3 * distance
  ) AS leftover_revenue
FROM cte_adjusted_runner_orders;
```
