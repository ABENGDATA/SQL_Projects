# Revision : Metodo PEAR para presentar datos en SQL 
![](pear.png)

## **Problema**
Hemos sido llamados por el area de marketing de una compañia de DVD y estamos para ayudarlos a generar la analitica tras su primer campaña de analisis de clientes , este analisis se verá representado por un Email de la compañia hacia el cliente el cual brinda los datos de su consumo y como podria sacarle mas provecho a su relacion con la compañia .

### **Plantilla del Email** 
Tambien se nos ha proveido la plantilla del Email con las metricas que debe tener este para asi poder tener una idea de los datos que necesitaremos manejar . 
![](minicase.png)

### **Insights por Categoria**
Mediante el analisis de la plantilla se infieren los siguientes requerimientos en cuanto a las categorias de los clientes . 
1. Cual era la categoria mas vista por el cliente especifico 
2.  Cuantas peliculas ha visto de esta categoria TOP y como se compara con la base de datos de clientes 
    - Cuantas peliculas ha visto el cliente en comparacion con la media de clientes para esta categoria .
     - Cual es el ranking porcentual comparado con los demas clientes en esta categoria .
3. Cual es el top 3 de peliculas recomendadas en la categoria top que el cliente no ha visto antes .
4. Cual es la segunda categoria mas vista por el cliente .
5. Cual es la proporcion ha visto  el cliente de esta categoria .
6. Cual es el top 3 de peliculas que no ha visto este cliente aun .

### **Insights por Actor**
1. Cual actor ha protagonizado mas peliculas en todo el historial de consumo del cliente .
2. Cuantas peliculas ha protagonizado el actor por cliente .
3. Cual es el top 3 de peliculas de este actor que no ha visto aun el cliente .

### **Diagrama de Relación de datos**
 ![](relation.png)

## **Exploración**
Tenemos un total de 7 tablas en nuestra base de datos para este caso en nuestro diagrama de relaciones estan resaltadas aquellas columnas que seran objeto de estudio para resolver nuestro proceso .

### **Mapeo de las tablas**
Analizamos las relaciones entre las tablas y las columnas mas importantes para determinar si existen relaciones entre los datos en cada tabla y como se dan estas relaciones (one to many , many to many , many to one) con el fin de plantear los Join necesarios .

#### **Ejemplo 1 : Analisis de Join**
![](EXAMPLE1.png)
En esta etapa identificamos que los valores de las columnas de las tablas 1 y 2 no tienen valores flotantes y tambien los datos corresponden en una relacion completa en la que podemos si bien utilizar Inner Join o Left Join . 

Como ejemplo tenemos el codigo en el que realizamos el analisis con la tabla inventory y rental 
1. Realizar una consulta para verificar si hay una interseccion completa entre las dos tablas en cuanto a la primary key .
```SQL
SELECT COUNT(DISTINCT rental.inventory_id)
FROM dvd_rentals.rental 
WHERE NOT EXISTS (
SELECT inventory_id
FROM dvd_rentals.inventory
WHERE rental.inventory_id = inventory.inventory_id
);
```
2. Revisar la tabla derecha utilizando el mismo proceso con inventory .

```SQL 
SELECT COUNT(DISTINCT inventory.inventory_id)
FROM dvd_rentals.inventory
WHERE NOT EXISTS (
SELECT inventory_id
FROM dvd_rentals.rental
WHERE rental.inventory_id = inventory.inventory_id 

)
```
Como podemos observar hay un registro que no hace parte de la tabla izquierda y que se encuentra en la derecha por lo tanto debemos inspeccionar en que consiste este registro .

3. Investigar de donde provienen los registros fuera de la interseccion . 
```SQL
SELECT *
FROM dvd_rentals.inventory
WHERE NOT EXISTS(
    SELECT inventory_id
    FROM dvd_rentals.rental
    WHERE rental.inventory_id = inventory.inventory_id
);
```
Podemos concluir que este registro no genera problemas en nuestro analisis ya que se trata de una pelicula que jamas se rentó de nuestro catalogo .

4. Finalmente podemos construir nuestro Join como prefiramos con inner o left 
```SQL 
-- Ejemplo con left join 
DROP TABLE IF EXISTS left_rental_join ;
CREATE TEMP TABLE left_rental_join AS 
SELECT 
rental.customer_id ,
rental.inventory_id,
inventory.film_id 
FROM dvd_rentals.rental 
LEFT JOIN dvd_rentals.inventory 
ON rental.inventory_id = inventory.inventory_id;
```

```SQL 
-- Ejemplo con Inner Join 
DROP TABLE IF EXISTS inner_rental_join;
CREATE TEMP TABLE inner_rental_join AS
SELECT 
rental.customer_id,
rental.inventory_id,
inventory.film_id
FROM dvd_rentals.rental 
INNER JOIN dvd_rentals.inventory
ON rental.inventory_id = inventory.inventory_id;
```
```SQL 
-- Comparacion entre las outputs 
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM left_rental_join
)
UNION
(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM inner_rental_join
);
```
### **Ejemplo 2 : Analisis de Join**
Tambien necesitamos investigar las relaciones entre actor_id y film_id dentro de la tabla dvd_rentals.film_actor .

De manera intuitiva podemos suponer que a un actor le corresponden varias peliculas y que a una pelicula le corresponden varios actores , por lo tanto esta es una relacion many to many .

```SQL 
WITH actor_film_counts AS (
SELECT actor_id,COUNT(DISTINCT film_id) AS film_count
FROM dvd_rentals.film_actor
GROUP BY actor_id
)

SELECT film_count, COUNT(*) AS total_actors
FROM actor_film_counts
GROUP BY film_count
ORDER BY film_count DESC ;
```
En la siguiente consulta confirmaremos que la relacion es many to many verificando que a una pelicula le corresponden varios actores .

```SQL
WITH film_actor_counts AS (
    SELECT film_id,COUNT(DISTINCT actor_id) AS actor_count
    FROM dvd_rentals.film_actor
    GROUP BY film_id
)
SELECT actor_count, COUNT(*) AS total_films
FROM film_actor_counts
GROUP BY actor_count
ORDER BY actor_count DESC ;
```
En conclusion podemos observar que hay una relacion many to many de film_id y actor_id dentro de la tabla dvd_rentals.film_actor y tenemos que tener especial cuidado cuando hagamos el join en pasos previos .

## **Análisis**
Ahora identificaremos las columnas claves y resaltaremos aquellas cosas que debemos tener en cuenta para realizar nuestro plan de accion e implementar nuestro Join . 

Ahora listaremos una plan para solucionar nuestro proyecto .

### **Plan de solución** 
#### **Insights por categoria** :
1. Crear una base de datos y hacer el join de las tablas reelevantes : **complete_joint_dataset**
2. Calcular cuantas veces rentaron los clientes por categoria :
**category_counts**
3. Agregar todas las peliculas que el cliente ha visto : **total_counts**
4. Identificar el tip 2 de las categorias por cada cliente : **top_categories**
5. Calcular el promedio de cada categoria : **average_category_count**
6. Calcular el percentil del cliente por categoria : 
**top_category_percentile**
7. Generar nuestra primera tabla de insights con los datos anteriores :  **top_category_insights**
8. Generar los insights por la segunda categoria : **second_category_insights**

### **Recomendaciones por categoria**
1. Generar una tabla resumida donde se cuenten las peliculas por categoria y usaremos esta tabla para hacer un ranking de peliculas por popularidad : **film_counts**

2. Crear una tabla de peliculas vistas del top 2 de categorias para cada cliente : **category_film_exclusions**

3. Finalmente desarrollar un anti join para las categorias reelvantes para sugerir nuevas peliculas a cada cliente : **category_recommendations**

### **Insights por actor**
1. Crear una nueva base de datos la cual se encofara en el actor:
**actor_joint_table**
2. Identificar el actor con mayor regularidad en las peliculas de cada cliente : **top_actor_counts**

### **Recomendaciones por actor**
1. Generar una tabla de actores ordenandos por el consumo de sus peliculas : **actor_film_counts**

2. Crear una tabla de exclusion para las peliculas que se hayan viston sobre este actor :  **actor_film_exclusions**

3. Aplicar un anti join y usar funciones window para identificar 3 recomendaciones validas para nuestros clientes : **actor_recommendations**

### **Plan de solucion : Insights por categoria**
 1. **complete_joint_dataset** : Despues de analizar las relaciones entre todas las tablas tal y como se hizon en los ejemplos anteriores podemos implementar los Joins con todas la informacion requerida para nuestro caso de estudio .

```SQL
DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS 
SELECT 
rental.customer_id,
inventory.film_id,
film.title,
category.name AS category_name,
rental.rental_date
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
ON rental.inventory_id = inventory.inventory_id 
INNER JOIN dvd_rentals.film
ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
ON film_category.category_id = category.category_id ; 
```
2. **category_counts** : Esta tabla ha sido creada con nuestro dataset con el fin de realizar los calculos futuros para nuestros rankings .
```SQL 
DROP TABLE IF EXISTS category_counts ;
CREATE TEMP TABLE category_counts AS 
SELECT 
customer_id ,
category_name,
COUNT(*) AS rental_count,
MAX(rental_date) AS latest_rental_date 
FROM complete_joint_dataset
GROUP BY 
customer_id,
category_name;
```
3. **total_counts** : Una de las tabals resultantes de crear la tabla anterior .
```SQL 
DROP TABLE IF EXISTS total_counts;
CREATE TEMP TABLE total_counts AS 
SELECT customer_id, SUM(rental_count) AS total_count
FROM category_counts
GROUP BY customer_id;
```
4. **top_categories** : En esta caso usamos DENSE_RANK para generar el ranking de categorias para cada cliente .
```SQL 
DROP TABLE IF EXISTS top_categories ;
CREATE TEMP TABLE top_categories  AS 
WITH ranked_cte AS (
SELECT 
customer_id,
category_name,
rental_count,
DENSE_RANK() OVER(PARTITION BY customer_id 
ORDER BY rental_count DESC , latest_rental_date DESC,
category_name) AS category_rank
FROM category_counts
)
SELECT * FROM ranked_cte
WHERE category_rank <=2
```
5. **average_category_count**: En esta tabla necesitaremos la tabla category_counts para generar el promedio de peliculas rentadas por categoria redondiada al entero mas cercano con la funcion FLOOR. 

```SQL 
DROP TABLE IF EXISTS average_category_count;
CREATE TEMP TABLE average_category_count AS 
SELECT 
category_name,
FLOOR(AVG(rental_count)) AS category_average 
FROM category_counts
GROUP BY category_name;
```
6. **top_category_percentile** : Ahora necesitamos comparar cada cliente del top de las categorias con los otros clientes que no son top , hacemos esto usando la combinacion de LEFT JOIN y PERCENT RANK .

```SQL 
DROP TABLE IF EXISTS top_category_percentile;
CREATE TEMP TABLE top_category_percentile AS 
WITH calculated_cte AS (
SELECT 
top_categories.customer_id,
top_categories.category_name AS top_category_name,
top_categories.rental_count,
category_counts.category_name,
top_categories.category_rank ,
PERCENT_RANK() OVER (
PARTITION BY  category_counts.category_name
ORDER BY category_counts.rental_count DESC
) AS raw_percentile_value
FROM category_counts
LEFT JOIN top_categories
ON category_counts.customer_id = top_categories.customer_id
)
SELECT 
customer_id,
category_name,
rental_count,
category_rank,
CASE 
WHEN ROUND(100*raw_percentile_value) = 0 THEN 1
ELSE ROUND(100*raw_percentile_value)
END AS percentile
FROM calculated_cte 
WHERE category_rank = 1 AND top_category_name = category_name;gh

```

7. **top_category_insights** : 
```SQL
DROP TABLE IF EXISTS first_category_insights;
CREATE TEMP TABLE first_category_insights AS 
SELECT 
base.customer_id
base.category_name,
base.rental_count,
base.rental_count - average.category_average AS average_comparison,
base.percentile
FROM top_category_percentile AS base
LEFT JOIN average_category_count AS average
ON base.category_name = average.category_name;
```

8. **second_category_insights**
```SQL 
DROP TABLE IF EXISTS second_category_insights;
CREATE TEMP TABLE second_category_insights AS
SELECT
  top_categories.customer_id,
  top_categories.category_name,
  top_categories.rental_count,
  -- need to cast as NUMERIC to avoid INTEGER floor division!
  ROUND(
    100 * top_categories.rental_count::NUMERIC / total_counts.total_count
  ) AS total_percentage
FROM top_categories
LEFT JOIN total_counts
  ON top_categories.customer_id = total_counts.customer_id
WHERE category_rank = 2;
```
### **Plan de solucion : Recomendaciones por categoria**
1. **film_counts** : 
```SQL
DROP TABLE IF EXISTS film_counts;
CREATE TEMP TABLE film_counts AS 
SELECT DISTINCT 
film_id,
title,
category_name,
COUNT(*) OVER(
    PARTITION BY film_id
)  AS  rental_count
FROM complete_joint_dataset;
```
2. **category_film_exclusions** :
```SQL 
DROP TABLE IF EXISTS category_film_exclusions;
CREATE TEMP TABLE category_film_exclusions AS
SELECT DISTINCT 
customer_id,
film_id
FROM complete_joint_dataset;
```
3. **category_recommendations**  : 
```SQL 
DROP TABLE IF EXISTS category_recommendations;
CREATE TEMP TABLE category_recommendations AS 
WITH ranked_films_cte AS (
    SELECT 
    top_categories.customer_id,
    top_categories.category_name,
    top_categories.category_rank,
    film_counts.film_id,
    film_counts.title,
    film_counts.rental_count,
    DENSE_RANK() 
    OVER(PARTITION BY 
    top_categories.customer_id,
    top_categories.category_rank
    ORDER BY
    film_counts.rental_count DESC
    film_counts.title) AS reco_rank
    FROM top_categories
    INNER JOIN film_counts
    ON top_categories.category_name = film_counts.category_name
    WHERE NOT EXISTS (
        SELECT 1
        FROM category_film_exclusions
        WHERE 
        category_film_exclusions.customer_id = top_categories.customer_id AND    category_film_exclusions.film_id = film_counts.film_id))
SELECT *
FROM ranked_films_cte
WHERE reco_rank <=3;

SELECT *
FROM category_recommendations
WHERE customer_id = 1
ORDER BY category_rank, reco_rank;
```
### **Plan de solucion : Insights por actor**
1. **actor_joint_table** :
```SQL 
DROP TABLE IF EXISTS actor_joint_dataset;
CREATE TEMP TABLE actor_joint_dataset AS 
SELECT 
rental.customer_id,
rental.rental_id,
rental.rental_date,
film.film_id
film.title,
actor.actor_id,
actor.first_name,
actor.last_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_actor
ON film.film_id = film_actor.film_id
INNER JOIN dvd_rentals.actor
ON film_actor.actor_id = actor.actor_id 
```

2. **top_actor_counts** : 
```SQL 
DROP TABLE IF EXISTS top_actor_counts;
CREATE TEMP TABLE top_actor_counts AS 
WITH actor_counts AS (
SELECT 
customer_id,
actor_id,
first_name,
last_name,
COUNT(*) AS rental_count,
MAX(rental_date) AS latest_rental_date
FROM actor_joint_dataset
GROUP BY 
customer_id,
actor_id,
first_name,
last_name
),
ranked_actor_counts AS (
    SELECT 
    actor_counts.*,
    DENSE_RANK() OVER(
        PARTITION BY customer_id
        ORDER BY 
        rental_count DESC,
        latest_rental_date DESC,
        first_name,
        last_name
    ) AS actor_rank 
FROM actor_counts
)
SELECT 
customer_id,
actor_id,
first_name,
last_name,
rental_count
FROM ranked_actor_counts
WHERE actor_rank = 1 ;
```
### **Plan de Solucion : Recomendaciones por actor**
1. **actor_film_counts** :
```SQL 
DROP TABLE IF EXISTS actor_film_counts;
CREATE TEMP TABLE actor_film_counts AS
WITH film_counts AS (
  SELECT
    film_id,
    COUNT(DISTINCT rental_id) AS rental_count
  FROM actor_joint_dataset
  GROUP BY film_id
)
SELECT DISTINCT
  actor_joint_dataset.film_id,
  actor_joint_dataset.actor_id,
  -- why do we keep the title here? can you figure out why?
  actor_joint_dataset.title,
  film_counts.rental_count
FROM actor_joint_dataset
LEFT JOIN film_counts
  ON actor_joint_dataset.film_id = film_counts.film_id;

SELECT *
FROM actor_film_counts
LIMIT 10;
```
2. **actor_film_exclusions** :
```SQL 
DROP TABLE IF EXISTS actor_film_exclusions;
CREATE TEMP TABLE actor_film_exclusions AS
-- repeat the first steps as per the category exclusions
-- we'll use our original complete_joint_dataset as the base here
-- can you figure out why???
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM complete_joint_dataset
)
-- we use a UNION to combine the previously watched and the recommended films!
UNION
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM category_recommendations
);
```
3. **actor_recommendations** : 
```SQL 
DROP TABLE IF EXISTS actor_recommendations;
CREATE TEMP TABLE actor_recommendations AS
WITH ranked_actor_films_cte AS (
  SELECT
    top_actor_counts.customer_id,
    top_actor_counts.first_name,
    top_actor_counts.last_name,
    top_actor_counts.rental_count,
    actor_film_counts.title,
    actor_film_counts.film_id,
    actor_film_counts.actor_id,
    DENSE_RANK() OVER (
      PARTITION BY
        top_actor_counts.customer_id
      ORDER BY
        actor_film_counts.rental_count DESC,
        actor_film_counts.title
    ) AS reco_rank
  FROM top_actor_counts
  INNER JOIN actor_film_counts
    -- join on actor_id instead of category_name!
    ON top_actor_counts.actor_id = actor_film_counts.actor_id
  -- This is a tricky anti-join where we need to "join" on 2 different tables!
  WHERE NOT EXISTS (
    SELECT 1
    FROM actor_film_exclusions
    WHERE
      actor_film_exclusions.customer_id = top_actor_counts.customer_id AND
      actor_film_exclusions.film_id = actor_film_counts.film_id
  )
)
SELECT * FROM ranked_actor_films_cte
WHERE reco_rank <= 3;

SELECT *
FROM actor_recommendations
ORDER BY customer_id, reco_rank
LIMIT 15;
```
## **Reporte : Transformaciones finales**
Una vez realizadas todas las consultas y convertidos todos lo datos que necesitamos para el area de marketing es necesario entregarlos de una manera limpia y facil de leer , por lo tanto realizaremos unas transformaciones adicionales con el fin de simplificar la lectura de nuestro caso de estudio . 
```SQL 
DROP TABLE IF EXISTS final_data_asset;
CREATE TEMP TABLE final_data_asset AS
WITH first_category AS (
  SELECT
    customer_id,
    category_name,
    CONCAT(
      'You''ve watched ', rental_count, ' ', category_name,
      ' films, that''s ', average_comparison,
      ' more than the DVD Rental Co average and puts you in the top ',
      percentile, '% of ', category_name, ' gurus!'
    ) AS insight
  FROM first_category_insights
),
second_category AS (
  SELECT
    customer_id,
    category_name,
    CONCAT(
      'You''ve watched ', rental_count, ' ', category_name,
      ' films making up ', total_percentage,
      '% of your entire viewing history!'
    ) AS insight
  FROM second_category_insights
),
top_actor AS (
  SELECT
    customer_id,
    -- use INITCAP to transform names into Title case
    CONCAT(INITCAP(first_name), ' ', INITCAP(last_name)) AS actor_name,
    CONCAT(
      'You''ve watched ', rental_count, ' films featuring ',
      INITCAP(first_name), ' ', INITCAP(last_name),
      '! Here are some other films ', INITCAP(first_name),
      ' stars in that might interest you!'
    ) AS insight
  FROM top_actor_counts
),
adjusted_title_case_category_recommendations AS (
  SELECT
    customer_id,
    INITCAP(title) AS title,
    category_rank,
    reco_rank
  FROM category_recommendations
),
wide_category_recommendations AS (
  SELECT
    customer_id,
    MAX(CASE WHEN category_rank = 1  AND reco_rank = 1
      THEN title END) AS cat_1_reco_1,
    MAX(CASE WHEN category_rank = 1  AND reco_rank = 2
      THEN title END) AS cat_1_reco_2,
    MAX(CASE WHEN category_rank = 1  AND reco_rank = 3
      THEN title END) AS cat_1_reco_3,
    MAX(CASE WHEN category_rank = 2  AND reco_rank = 1
      THEN title END) AS cat_2_reco_1,
    MAX(CASE WHEN category_rank = 2  AND reco_rank = 2
      THEN title END) AS cat_2_reco_2,
    MAX(CASE WHEN category_rank = 2  AND reco_rank = 3
      THEN title END) AS cat_2_reco_3
  FROM adjusted_title_case_category_recommendations
  GROUP BY customer_id
),
adjusted_title_case_actor_recommendations AS (
  SELECT
    customer_id,
    INITCAP(title) AS title,
    reco_rank
  FROM actor_recommendations
),
wide_actor_recommendations AS (
  SELECT
    customer_id,
    MAX(CASE WHEN reco_rank = 1 THEN title END) AS actor_reco_1,
    MAX(CASE WHEN reco_rank = 2 THEN title END) AS actor_reco_2,
    MAX(CASE WHEN reco_rank = 3 THEN title END) AS actor_reco_3
  FROM adjusted_title_case_actor_recommendations
  GROUP BY customer_id
),
final_output AS (
  SELECT
    t1.customer_id,
    t1.category_name AS cat_1,
    t4.cat_1_reco_1,
    t4.cat_1_reco_2,
    t4.cat_1_reco_3,
    t2.category_name AS cat_2,
    t4.cat_2_reco_1,
    t4.cat_2_reco_2,
    t4.cat_2_reco_3,
    t3.actor_name AS actor,
    t5.actor_reco_1,
    t5.actor_reco_2,
    t5.actor_reco_3,
    t1.insight AS insight_cat_1,
    t2.insight AS insight_cat_2,
    t3.insight AS insight_actor
FROM first_category AS t1
INNER JOIN second_category AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN top_actor t3
  ON t1.customer_id = t3.customer_id
INNER JOIN wide_category_recommendations AS t4
  ON t1.customer_id = t4.customer_id
INNER JOIN wide_actor_recommendations AS t5
  ON t1.customer_id = t5.customer_id
)
SELECT * FROM final_output;

SELECT * FROM final_data_asset
LIMIT 5;
```