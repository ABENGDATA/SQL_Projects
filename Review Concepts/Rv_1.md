# Review :  Record Counts and Distinct values 

## How Many Records
#### How many rows are there in the film_list table?
First of all we need to explore the dataset and how are the names of the columns in order to answer future questions about the data this should be possible with the follow lines of code . 
The ``` SELECT * ``` line help us to select all the columns of our data set , but if we need to be more specific with the columns we can type the column name without problem .
```SQL
SELECT *
FROM dvd_rentals.film_list LIMIT 10;
```
| fid | title            | description                                                                                                           | category    | price | length | rating | actors                                                                                                                                         |
|-----|------------------|-----------------------------------------------------------------------------------------------------------------------|-------------|-------|--------|--------|------------------------------------------------------------------------------------------------------------------------------------------------|
| 1   | ACADEMY DINOSAUR | A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher in The Canadian Rockies                      | Documentary | 0.99  | 86     | PG     | ROCK DUKAKIS, MARY KEITEL, JOHNNY CAGE, PENELOPE GUINESS, SANDRA PECK, CHRISTIAN GABLE, OPRAH KILMER, WARREN NOLTE, LUCILLE TRACY, MENA TEMPLE |
| 2   | ACE GOLDFINGER   | A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China                  | Horror      | 4.99  | 48     | G      | MINNIE ZELLWEGER, CHRIS DEPP, BOB FAWCETT, SEAN GUINESS                                                                                        |
| 3   | ADAPTATION HOLES | A Astounding Reflection of a Lumberjack And a Car who must Sink a Lumberjack in A Baloon Factory                      | Documentary | 2.99  | 50     | NC-17  | CAMERON STREEP, BOB FAWCETT, NICK WAHLBERG, RAY JOHANSSON, JULIANNE DENCH                                                                      |
| 4   | AFFAIR PREJUDICE | A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a Monkey in A Shark Tank                          | Horror      | 2.99  | 117    | G      | JODIE DEGENERES, KENNETH PESCI, FAY WINSLET, OPRAH KILMER, SCARLETT DAMON                                                                      |
| 5   | AFRICAN EGG      | A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a Forensic Psychologist in The Gulf of Mexico | Family      | 2.99  | 130    | G      | DUSTIN TAUTOU, MATTHEW LEIGH, GARY PHOENIX, MATTHEW CARREY, THORA TEMPLE                                                                       |
| 6   | AGENT TRUMAN     | A Intrepid Panorama of a Robot And a Boy who must Escape a Sumo Wrestler in Ancient China                             | Foreign     | 2.99  | 169    | PG     | WARREN NOLTE, SANDRA KILMER, JAYNE NEESON, MORGAN WILLIAMS, KIRSTEN PALTROW, KENNETH HOFFMAN, REESE WEST                                       |
| 7   | AIRPLANE SIERRA  | A Touching Saga of a Hunter And a Butler who must Discover a Butler in A Jet Boat                                     | Comedy      | 4.99  | 62     | PG-13  | MENA HOPPER, JIM MOSTEL, MICHAEL BOLGER, OPRAH KILMER, RICHARD PENN                                                                            |
| 8   | AIRPORT POLLOCK  | A Epic Tale of a Moose And a Girl who must Confront a Monkey in Ancient India                                         | Horror      | 4.99  | 54     | R      | LUCILLE DEE, SUSAN DAVIS, FAY KILMER, GENE WILLIS                                                                                              |
| 9   | ALABAMA DEVIL    | A Thoughtful Panorama of a Database Administrator And a Mad Scientist who must Outgun a Mad Scientist in A Jet Boat   | Horror      | 2.99  | 114    | PG-13  | WILLIAM HACKMAN, RIP CRAWFORD, RIP WINSLET, GRETA KEITEL, CHRISTIAN GABLE, MENA TEMPLE, MERYL ALLEN, WARREN NOLTE, ELVIS MARX                  |
| 10  | ALADDIN CALENDAR | A Action-Packed Tale of a Man And a Lumberjack who must Reach a Feminist in Ancient China                             | Sports      | 4.99  | 63     | NC-17  | GRETA MALDEN, ROCK DUKAKIS, RAY JOHANSSON, RENEE TRACY, VAL BOLGER, JUDY DEAN, JADA RYDER, ALEC WAYNE                                          |
##  Column Aliases
Once we explored our data set we could learn how do we put names or aliases to new columns , we can do this with the statement  "AS" and the name of the column without spaces .

The function COUNT is useful when we need to make a count of rows or data in a specific column , in this example we used ``` COUNT(*) ``` to count all the rows in our data set.
```SQL
SELECT COUNT(*) AS NUMBER_OF_ROWS
FROM dvd_rentals.film_list;
```
| number_of_rows |
|----------------|
| 997            |

## Show Unique Column Values
#### What are the unique values for the rating column in the film table?
through the keyword  ```DISTINCT``` we can get unique values in a column to observe wich are the most common records inside the column , this work in columns with categories such as countries or scores and we have the same values repeated along the this .
```SQL
SELECT COUNT(DISTINCT category) AS unique_category_count
FROM  dvd_rentals.film_list;
```
| unique_category_count |
|-----------------------|
| 16                    |

##  Plus : Use of WITH 
The ```WITH``` clause was introduced by oracle and allows us to give  a subquery a name in order to create reference of a part of an original table , just like the follow example .
It's important to remember that WITH is not avaiable in some databases systems .
``` SQL
WITH example_table AS (
SELECT 
fid,
title,
category,
rating,
price 
FROM dvd_rentals.film_list
LIMIT 10
)
```
## Group and Order by 
These statements are quite important as far as learning SQL is concerned because we can do a lot of tasks of grouping and ordering in order to display results with operations like ``` SUM , COUNT ``` .....

We have to learn an aspect of this notation an this is Group by is before ORDER BY because the Syntax of the language have that rule , and in a logical aspect we could not order data without group this because we could have a dimentional error in our calculations .
```SQL
SELECT rating , COUNT(*) as record_count
FROM example_table
GROUP BY rating
ORDER BY record_count DESC ;
```
| rating | record_count |
|--------|--------------|
| PG-13  | 4            |
| G      | 2            |
| NC-17  | 2            |
| PG     | 1            |
| R      | 1            |

In this case we used the rating column and counted the rows in order to group this values as frequencies of rating and finally we made a calculation of the percentage of this frequencies .

To keep in mind the statement ``` ROUND(100*COUNT(*) ::NUMERIC/ SUM(COUNT(*)) OVER () ,2 )AS percentage  ``` first use the ```ROUND``` function to get two decimal positions in the result then have the numerator ```100*COUNT(*) ::NUMERIC``` that describes the count of all rows multiplied by 100 and the keyword ```::NUMERIC``` convert the result in float type then we have the denominator ```SUM(COUNT(*))``` that sums all all of rows and finally ```OVER()``` and 2 are arguments of the ```ROUND``` function . 

This can be confusing but these calculations are linked with the group and order by that follow the code .

```SQL
SELECT rating, COUNT(*) AS frequency, ROUND(100*COUNT(*) ::NUMERIC/ SUM(COUNT(*)) OVER () ,2 )AS percentage 
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC ;
```
| rating | frequency | percentage |
|--------|-----------|------------|
| PG-13  | 223       | 22.37      |
| NC-17  | 210       | 21.06      |
| PG     | 194       | 19.46      |
| R      | 193       | 19.36      |
| G      | 177       | 17.75      |

This is a similar example , with rating,category and frequency of the movies in the dataset .
``` SQL
SELECT rating,category,COUNT(*) AS FREQUENCY 
FROM dvd_rentals.film_list
GROUP BY rating,category
ORDER BY frequency DESC ;
```
| rating | category    | frequency |
|--------|-------------|-----------|
| PG-13  | Drama       | 22        |
| NC-17  | Music       | 20        |
| PG-13  | Foreign     | 19        |
| PG-13  | Animation   | 19        |
| PG     | Family      | 18        |
| G      | Action      | 18        |
| NC-17  | New         | 18        |
| NC-17  | Sports      | 17        |
| R      | Sci-Fi      | 17        |
| NC-17  | Games       | 16        |
| PG     | Sports      | 16        |
| R      | Family      | 16        |
| PG     | Comedy      | 16        |
| PG     | Documentary | 16        |
| PG-13  | New         | 15        |
| R      | Games       | 15        |
| R      | Foreign     | 15        |
| NC-17  | Animation   | 15        |
| NC-17  | Drama       | 15        |
| PG     | Children    | 15        |
| R      | Sports      | 15        |
| R      | Classics    | 14        |
| PG-13  | Horror      | 14        |
| R      | Horror      | 14        |
| NC-17  | Family      | 14        |
| PG-13  | Sports      | 14        |
| G      | Documentary | 14        |
| R      | Action      | 14        |
| PG-13  | Games       | 14        |
| PG-13  | Children    | 14        |
| PG     | Foreign     | 14        |
| PG     | Travel      | 14        |
| G      | Foreign     | 13        |
| PG-13  | Classics    | 13        |
| G      | Animation   | 13        |
| PG-13  | Sci-Fi      | 13        |
| NC-17  | Documentary | 13        |
| R      | Documentary | 13        |
| G      | Drama       | 12        |
| NC-17  | Children    | 12        |
| PG     | Sci-Fi      | 12        |
| G      | New         | 12        |
| NC-17  | Foreign     | 12        |
| PG-13  | Travel      | 12        |
| NC-17  | Action      | 12        |
| PG-13  | Comedy      | 12        |
| PG     | Horror      | 12        |
| PG-13  | Documentary | 12        |
| G      | Games       | 11        |
| PG-13  | Family      | 11        |
| PG-13  | Action      | 11        |
| G      | Comedy      | 11        |
| NC-17  | Comedy      | 11        |
| G      | Sports      | 11        |
| G      | Classics    | 11        |
| PG     | Animation   | 11        |
| R      | Music       | 11        |
| G      | Children    | 10        |
| R      | Travel      | 10        |
| G      | Family      | 10        |
| G      | Travel      | 10        |
| NC-17  | Travel      | 10        |
| PG     | Music       | 10        |
| G      | Sci-Fi      | 10        |
| PG     | Classics    | 10        |
| G      | Horror      | 9         |
| NC-17  | Sci-Fi      | 9         |
| PG     | New         | 9         |
| R      | Children    | 9         |
| PG     | Action      | 9         |
| NC-17  | Classics    | 9         |
| R      | New         | 9         |
| R      | Comedy      | 8         |
| R      | Animation   | 8         |
| PG-13  | Music       | 8         |
| NC-17  | Horror      | 7         |
| PG     | Drama       | 7         |
| PG     | Games       | 5         |
| R      | Drama       | 5         |
| G      | Music       | 2         |

## Using positional numbers instead of column names 
In this case the Group and Order by statements have a positional number that identifies what column is being used to present the results .

Which actor_id has the most number of unique film_id records in the dvd_rentals.film_actor table?
``` SQL
SELECT actor_id, COUNT(DISTINCT film_id ) 
FROM dvd_rentals.film_actor
GROUP BY 1
ORDER BY 2 DESC LIMIT 5; 
```
| actor_id | count |
|----------|-------|
| 107      | 42    |
| 102      | 41    |
| 198      | 40    |
| 181      | 39    |
| 23       | 37    |

How many distinct fid values are there for the 3rd most common price value in the dvd_rentals.nicer_but_slower_film_list table?
```SQL
SELECT
  price,
  COUNT(DISTINCT fid)
FROM dvd_rentals.nicer_but_slower_film_list
GROUP BY 1
ORDER BY 2 DESC;
```
| price | count |
|-------|-------|
| 0.99  | 340   |
| 4.99  | 334   |
| 2.99  | 323   |

How many unique country_id values exist in the dvd_rentals.city table?
```SQL 
SELECT COUNT(DISTINCT country_id)
FROM dvd_rentals.city;
```
| count |
|-------|
| 109   |

What percentage of overall total_sales does the Sports category make up in the dvd_rentals.sales_by_film_category table?
```SQL
SELECT category,  ROUND(
    100 * total_sales::NUMERIC / SUM(total_sales) OVER (),
    2
  ) as percentage
FROM dvd_rentals.sales_by_film_category;
```
| category    | percentage |
|-------------|------------|
| Sports      | 7.88       |
| Sci-Fi      | 7.06       |
| Animation   | 6.91       |
| Drama       | 6.80       |
| Comedy      | 6.50       |
| Action      | 6.49       |
| New         | 6.46       |
| Games       | 6.35       |
| Foreign     | 6.33       |
| Family      | 6.28       |
| Documentary | 6.26       |
| Horror      | 5.52       |
| Children    | 5.42       |
| Classics    | 5.40       |
| Travel      | 5.27       |
| Music       | 5.07       |


What percentage of unique fid values are in the Children category in the dvd_rentals.film_list table?
```SQL
SELECT category, ROUND(
    100 * COUNT(DISTINCT fid)::NUMERIC / SUM(COUNT(DISTINCT fid)) OVER (),
    2
  ) as percentage
FROM dvd_rentals.film_list
GROUP BY category
ORDER BY category;
```
| category    | percentage |
|-------------|------------|
| Action      | 6.42       |
| Animation   | 6.62       |
| Children    | 6.02       |
| Classics    | 5.72       |
| Comedy      | 5.82       |
| Documentary | 6.82       |
| Drama       | 6.12       |
| Family      | 6.92       |
| Foreign     | 7.32       |
| Games       | 6.12       |
| Horror      | 5.62       |
| Music       | 5.12       |
| New         | 6.32       |
| Sci-Fi      | 6.12       |
| Sports      | 7.32       |
| Travel      | 5.62       |




