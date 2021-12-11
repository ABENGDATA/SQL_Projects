# Revision : Solucion de problemas en SQL 
En la pasada revision se implementaron los joins con los cuales podiamos investigar la categoria y las peliculas que el cliente suele ver , hasta este punto tenemos el 80% de nuestro trabajo como analistas SQL , el siguiente 20 % se desarrolla realizando una consulta que nos permita modificar la informacion en nuestra consulta y presentarla como nos la pide el area de marketing . 

Por esta razon nuestro punto de inicio es la consulta anterior con sus respectivos Joins para asi poder unir toda la informacion .

``` SQL 
DROP TABLE IF EXISTS complete_joint_dataset_with_rental_date;
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
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
  ON film_category.category_id = category.category_id

SELECT * FROM complete_joint_dataset_with_rental_date limit 10;
``` 
Calculo de propiedades 
- **category_name** : El nombre de las 2 categorias mas consumidas por el cliente 
- **rental_count** : Cuantas peliculas en total ha visto el cliente en esta categoria .
- **average_comparison**  : Cuantas peliculas el cliente ha visto en esta categoria comparado con la media de clientes .
- **percentile**: Como esta el cliente en el ranking en terminos de top
- **category_percentage** : Que proporcion de peliculas en total ha visto en esta categoria .

## Tabla 1 : Category Rental Counts 
La creacion de esta tabla nos va a permitir relacionar las columnas customer_id y category_name dando como resultado una tabal en donde se agrupará la el id del cliente y las categorias que ha consumido , tambien cuantas peliculas ha visto y la fecha de su ultimo consumo . 

``` SQL 
DROP TABLE IF EXISTS category_rental_counts;
CREATE TEMP TABLE category_rental_counts AS 
SELECT 
customer_id,
category_name ,
COUNT(*) AS rental_count,
MAX(rental_date) AS latest_rental_date 
FROM complete_joint_dataset_with_rental_date 
GROUP BY 
customer_id,
category_name;

SELECT * 
FROM category_rental_counts
LIMIT 5 ; 
``` 

## Tabla 2 : Total customer rentals (category percentage)
Esta tabla tiene como finalidad hacer el conteo del total de peliculas rentadas o vistas por un usuario en todo el uso del producto . 

``` SQL 
DROP TABLE IF EXISTS customer_total_rentals;
CREATE TEMP TABLE customer_total_rentals AS 
SELECT 
customer_id,
SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;
```
## Tabla 3 : Average category rental counts 
La creacion de esta tabla nos ayuda a observar el promedio de peliculas que se rentan por genero en el negocio .

``` SQL 
DROP TABLE IF EXISTS average_category_rental_counts;
CREATE TEMP TABLE average_category_rental_counts AS 
SELECT 
category_name,
FLOOR(AVG(rental_count)) AS avg_rental_count
FROM category_rental_counts
GROUP BY 
category_name;
```
## Tabla 4 : customer_category_percentiles
La creacion de esta tabla establece un percentil a un usuario sobre el numero o ranking que ocupa en su categoria segun las peliculas que ha adquirido .
``` SQL 
DROP TABLE IF EXISTS customer_category_percentiles;
CREATE TEMP TABLE customer_category_percentiles AS
SELECT 
customer_id,
category_name,
CEILING(100*PERCENT_RANK() OVER(PARTITION BY category_name ORDER BY rental_count DESC))AS percentile
FROM category_rental_counts; 
SELECT * 
FROM customer_category_percentiles
LIMIT 5  ;
```

## Realizar el Join de las tablas con las columnas necesarias 
```SQL 
DROP TABLE IF EXISTS customer_category_joint_table ;
CREATE TEMP TABLE customer_category_joint_table AS 
SELECT 
t1.customer_id,
t1.category_name,
t1.rental_count,
t1.latest_rental_date,
t2.total_rental_count,
t3.avg_rental_count,
t4.percentile,
t1.rental_count - t3.avg_rental_count AS average_comparison,
ROUND(100 * t1.rental_count / t2.total_rental_count) AS category_percentage
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3
ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4
ON t1.customer_id = t4.customer_id
AND t1.category_name = t4.category_name;
  
SELECT *
FROM customer_category_joint_table
WHERE customer_id = 1
ORDER BY percentile
LIMIT 5 ; 
```
## Presentar los resultados obtenidos con PARTITION 
```SQL 
DROP TABLE IF EXISTS top_categories_information;

CREATE TEMP TABLE top_categories_information AS (
WITH ordered_customer_category_joint_table AS (
  SELECT
    customer_id,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY rental_count DESC, latest_rental_date DESC
    ) AS category_ranking,
    category_name,
    rental_count,
    average_comparison,
    percentile,
    category_percentage
  FROM customer_category_joint_table
)
SELECT *
FROM ordered_customer_category_joint_table
WHERE category_ranking <= 3
);

SELECT *
FROM top_categories_information
WHERE customer_id in (1, 2, 3)
ORDER BY customer_id, category_ranking;
```