# Revisión  : Como identificar datos duplicados  
A traves de la tabla ```health.user_logs``` se realizará el ejercicio de identificacion
de datos duplicados ,para contextualizar las habilidades de SQL a una situacion real 
La tabla antes mencionada representa el registro de mediciones que se consignan de varios 
usuarios en el dia .

## Partes de la identificacion 
### 1. Inspeccion de datos 
En este apartado debemos hacer codigo elemental de SQL con el fin de reconocer que datos estamos manejando 
en este caso tenemos columnas :
- id : Que representa al Usuario 
- log_date: Representa la fecha del registro 
- measure : Representa el tipo de medicion que se le hizo al usuario 
- measure_value : Representa el valor de la medicion 
- Systolic : Presion arterial sistolica cuando se mide presion arterial 
- Diastolic : Presion arterial diastolica cuando se mide presion arterial 
``` SQL
SELECT * 
FROM health.user_logs
LIMIT 10 ; 
```
| id                                       | log_date                 | measure        | measure_value | systolic | diastolic |
|------------------------------------------|--------------------------|----------------|---------------|----------|-----------|
| fa28f948a740320ad56b81a24744c8b81df119fa | 2020-11-15T00:00:00.000Z | weight         | 46.03959      | null     | null      |
| 1a7366eef15512d8f38133e7ce9778bce5b4a21e | 2020-10-10T00:00:00.000Z | blood_glucose  | 97            | 0        | 0         |
| bd7eece38fb4ec71b3282d60080d296c4cf6ad5e | 2020-10-18T00:00:00.000Z | blood_glucose  | 120           | 0        | 0         |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-17T00:00:00.000Z | blood_glucose  | 232           | 0        | 0         |
| d14df0c8c1a5f172476b2a1b1f53cf23c6992027 | 2020-10-15T00:00:00.000Z | blood_pressure | 140           | 140      | 113       |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-21T00:00:00.000Z | blood_glucose  | 166           | 0        | 0         |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-22T00:00:00.000Z | blood_glucose  | 142           | 0        | 0         |
| 87be2f14a5550389cb2cba03b3329c54c993f7d2 | 2020-10-12T00:00:00.000Z | weight         | 129.060012817 | 0        | 0         |
| 0efe1f378aec122877e5f24f204ea70709b1f5f8 | 2020-10-07T00:00:00.000Z | blood_glucose  | 138           | 0        | 0         |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-10-04T00:00:00.000Z | blood_glucose  | 210           | null     | null      |

### 2 . Conteo de Registros
Tambien es necesario realizar el conteo de las filas de la tabla con el fin de identificar el numero total de registros .

``` SQL
-- Record counts 
SELECT COUNT(*)
FROM health.user_logs ; 
```
| count |
|-------|
| 43891 |
### 3. Conteo de Registros unicos 
Es importante establecer en el set de datos que columna o columnas tienen categorias que son unicas , con el fin de realizar filtros mas avanzados a lo largo de la inspeccion de datos en este caso la columna ```id``` es una muestra de ello pues junto con ```measure``` es la unica en el set de datos que es una categoria ya que los id de los usuarios son unicos e intransferibles y no representan catidades solo nombres clave .
``` SQL
-- Unique Column Counts 
SELECT COUNT(DISTINCT id)
FROM health.user_logs ; 
```
| count |
|-------|
| 554   |

En este caso tenemos que ```measure``` tiene categorias como blood_glucose , weight , blood_pressure lo que realizaremos es un conteo de cual
es la frecuencia en el dataset de estos datos y el porcentaje que tienen en este .
``` SQL
-- Single Column Frequency 
SELECT measure , COUNT(*) AS Freq , ROUND(100*COUNT(*)/SUM(COUNT(*)) OVER(),2) AS PERCENTAGE
FROM health.user_logs
GROUP BY measure
ORDER BY Freq DESC;
```
| measure        | freq  | percentage |
|----------------|-------|------------|
| blood_glucose  | 38692 | 88.15      |
| weight         | 2782  | 6.34       |
| blood_pressure | 2417  | 5.51       |

Asimismo una buena practica con nuestros datos es observar como se distribuyen las columnas en este caso se realizó una 
tabla de frecuencias con la columna ```Measure_value``` ,```Systolic ``` y ```Diastolic``` .
``` SQL
-- Individual Column distributions 
-- MEASURE VALUE 
SELECT measure_value ,COUNT(*) AS frequency 
FROM health.user_logs
GROUP BY measure_value
ORDER BY 2 DESC 
LIMIT 10 ; 
```
#### Measure_value
| measure_value | frequency |
|---------------|-----------|
| 0             | 572       |
| 401           | 433       |
| 117           | 390       |
| 118           | 346       |
| 123           | 342       |
| 122           | 331       |
| 126           | 326       |
| 120           | 323       |
| 128           | 319       |
| 115           | 319       |

#### Systolic
``` SQL
-- Systolic 
SELECT systolic ,COUNT(*) AS frequency 
FROM health.user_logs
GROUP BY systolic
ORDER BY 2 DESC 
LIMIT 10 ; 
```
| systolic | frequency |
|----------|-----------|
| null     | 26023     |
| 0        | 15451     |
| 120      | 71        |
| 123      | 70        |
| 128      | 66        |
| 127      | 64        |
| 130      | 60        |
| 119      | 60        |
| 135      | 57        |
| 124      | 55        |

#### Diastolic
``` SQL
-- Distolic 
SELECT diastolic ,COUNT(*) AS frequency 
FROM health.user_logs
GROUP BY diastolic
ORDER BY 2 DESC 
LIMIT 10 ;
```
| diastolic | frequency |
|-----------|-----------|
| null      | 26023     |
| 0         | 15449     |
| 80        | 156       |
| 79        | 124       |
| 81        | 119       |
| 78        | 110       |
| 77        | 109       |
| 73        | 109       |
| 83        | 106       |
| 76        | 102       |

### Buscar valores de 0 y NULL 
#### Valores de Cero 
Tambien es indispensable aplicar la logica con nuestros datos , por lo que en este caso tenemos como ejemplo la columna measure
las mediciones de los usuarios logicamente deben ser valores superiores a 0 , por lo que un valor de 0 es inadmisible .
Por lo tanto se realizará la busqueda de las categorias que tienen valores de medida en la columna measure_value = 0 .

``` SQL
-- Deep Look 
SELECT measure , COUNT(*)
FROM health.user_logs
WHERE measure_value = 0 
GROUP BY 1 ;

```
| measure        | count |
|----------------|-------|
| blood_glucose  | 8     |
| blood_pressure | 562   |
| weight         | 2     |

En este caso las tres categorias presentan valores de cero , pero nos llama la atencion el de blood_pressure ya que son demasiados registros con
este valor , por lo que se estdiará la razon por la que este valor es asi de elevado . 

``` SQL
SELECT *
FROM health.user_logs
WHERE measure_value != 0 AND measure = 'blood_pressure'
LIMIT 10 ;
```
| id                                       | log_date                 | measure        | measure_value | systolic | diastolic |
|------------------------------------------|--------------------------|----------------|---------------|----------|-----------|
| ee653a96022cc3878e76d196b1667d95beca2db6 | 2020-03-18T00:00:00.000Z | blood_pressure | 0             | 115      | 76        |
| ee653a96022cc3878e76d196b1667d95beca2db6 | 2020-03-15T00:00:00.000Z | blood_pressure | 0             | 115      | 76        |
| ee653a96022cc3878e76d196b1667d95beca2db6 | 2020-02-03T00:00:00.000Z | blood_pressure | 0             | 105      | 70        |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-02-24T00:00:00.000Z | blood_pressure | 0             | 136      | 87        |
| c7af488f4c8efc0ecdfd6d0c427e7c133bf2f2d9 | 2020-02-06T00:00:00.000Z | blood_pressure | 0             | 164      | 84        |
| c7af488f4c8efc0ecdfd6d0c427e7c133bf2f2d9 | 2020-02-10T00:00:00.000Z | blood_pressure | 0             | 190      | 94        |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-02-07T00:00:00.000Z | blood_pressure | 0             | 125      | 79        |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-02-19T00:00:00.000Z | blood_pressure | 0             | 136      | 84        |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-02-15T00:00:00.000Z | blood_pressure | 0             | 135      | 89        |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-02-27T00:00:00.000Z | blood_pressure | 0             | 138      | 85        |

Como lo muestra la tabla la razon principal de que este registro sea cero es que la presion sistolica y distolica como columnas adicionales 
juegan un papel representativo mas completo que measure_value ya que la presion arterial esta sujeta a estas dos variables .

#### Valores NULL 
Los valores NULL son aquellos que no existen en la base de datos por lo tanto se deben identificar de manera cuidadosa para que no afecten nuestros analisis .
Primeramente identificamos que valores NULL hay en las diferentes categorias de mediciones .
``` SQL
-- Looking NULL values 
SELECT measure , COUNT(*) 
FROM health.user_logs
WHERE systolic IS NULL 
GROUP BY 1;
```
``` SQL
SELECT
  measure,
  COUNT(*)
FROM health.user_logs
WHERE diastolic IS NULL
GROUP BY 1;
```
| measure       | count |
|---------------|-------|
| weight        | 443   |
| blood_glucose | 25580 |

En la pasada operacion identificamos que registros estan diligenciados que tuvieran datos NULL en la columna Systolic y Diastolic ya que 
esto nos indica que registros son aquellos que estan en weight y blood_glucose ademas de que es util para identificar si hay filas de blood_pressure irregulares .

### Identificacion de duplicados 
#### 1. Metodo de Subquery 
En este caso creamos una subquery o subconsulta de datos que nos permita identificar cuales son aquellas filas duplicadas para esto contamos las filas de todas aquellas filas con valores distintos en la tabla health.user_logs , nuestra cuenta nos arrojará un valor de 31004 y su significado es que son aquellas filas que no estan duplicadas .

``` SQL
SELECT COUNT(*)
FROM( 
SELECT DISTINCT *
FROM health.user_logs
)AS subquery;
```
| count |
|-------|
| 31004 |

#### 2. Metodo de CTE o (Common Table Expression)
Este metodo utiliza un query o consulta que manipula datos existentes de un set de datos , a traves de la creacion de una referencia , esto 
nos ayuda a extraer datos necesarios y filtrar aquellos con los que no estamos interesados en trabajar .
``` SQL
WITH deduped_logs AS (
SELECT DISTINCT *
FROM health.user_logs
)
SELECT COUNT(*)
FROM deduped_logs;
```
| count |
|-------|
| 31004 |

En este caso el metodo alternativo nos arroja el mismo resultado que el de subquery .

### 3. Metodo de TT o (Temporary Table)
El metodo de temporary table nos asegura la creacion de una tabla con los datos que seran objetivo de trabajo sin necesidad de realizar operaciones de filtrado de datos posteriores ya que la tabla creada cumplirá nuestras condiciones asignadas con antelacion . 
Consta de 3 pasos : 
- Eliminar la tabla si ha existido anteriormente mediante ``` DROP TABLE IF EXISTS deduplicated_user_logs ```
- Crear la tabla con su respectivo nombre mediante ```CREATE TEMP TABLE deduplicated_user_logs AS  ```
- Escribir las condiciones necesarias para crear la tabla 
``` SQL
-- Create Temporary table 
DROP TABLE IF EXISTS deduplicated_user_logs;

CREATE TEMP TABLE deduplicated_user_logs AS 
SELECT DISTINCT *
FROM health.user_logs;
```
``` SQL
SELECT *
FROM deduplicated_user_logs
LIMIT 10;
```
| id                                       | log_date                 | measure        | measure_value | systolic | diastolic |
|------------------------------------------|--------------------------|----------------|---------------|----------|-----------|
| 576fdb528e5004f733912fae3020e7d322dbc31a | 2019-12-15T00:00:00.000Z | blood_pressure | 0             | 124      | 72        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-04-15T00:00:00.000Z | blood_glucose  | 267           | null     | null      |
| 8b130a2836a80239b4d1e3677302709cea70a911 | 2019-12-31T00:00:00.000Z | blood_glucose  | 109.799995    | null     | null      |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-05-07T00:00:00.000Z | blood_glucose  | 189           | null     | null      |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-08-18T00:00:00.000Z | weight         | 68.49244787   | 0        | 0         |
| 46d921f1111a1d1ad5dd6eb6e4d0533ab61907c9 | 2019-12-24T00:00:00.000Z | blood_pressure | 0             | 183      | 161       |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-01-04T00:00:00.000Z | blood_glucose  | 113           | null     | null      |
| d41d5b2fa4d9a2caf3a3e839432da76450bc2041 | 2019-11-11T00:00:00.000Z | blood_glucose  | 176           | 0        | 0         |
| 6c2f9a8372dac248192c50219c97f9087ab778ba | 2019-12-30T00:00:00.000Z | blood_glucose  | 207           | 0        | 0         |
| ef84a9ac5319687e5fba6174cd1db3d7f1cfca8f | 2020-08-20T00:00:00.000Z | blood_glucose  | 206           | 0        | 0         |


``` SQL
SELECT COUNT(*)
FROM deduplicated_user_logs;
```
| count |
|-------|
| 31004 |

Una vez mas obtenemos el mismo resultado por lo tanto tenemos 3 formas diferentes de filtrar los datos unicos y descartar los duplicados .
``` SQL
-- Identify the duplicate records as frequency table 

SELECT id ,log_date,measure,measure_value,systolic,diastolic,COUNT(*) AS FREQ
FROM health.user_logs
GROUP BY id ,log_date,measure,measure_value,systolic,diastolic 
HAVING COUNT(*)>1
ORDER BY FREQ DESC ;
```
| id                                       | log_date                 | measure       | measure_value | systolic | diastolic | freq |
|------------------------------------------|--------------------------|---------------|---------------|----------|-----------|------|
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-06T00:00:00.000Z | blood_glucose | 401           | null     | null      | 104  |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-05T00:00:00.000Z | blood_glucose | 401           | null     | null      | 77   |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-04T00:00:00.000Z | blood_glucose | 401           | null     | null      | 72   |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-07T00:00:00.000Z | blood_glucose | 401           | null     | null      | 70   |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-09-30T00:00:00.000Z | blood_glucose | 401           | null     | null      | 39   |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-09-29T00:00:00.000Z | blood_glucose | 401           | null     | null      | 24   |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-10-02T00:00:00.000Z | blood_glucose | 401           | null     | null      | 18   |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-10T00:00:00.000Z | blood_glucose | 140           | null     | null      | 12   |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-11T00:00:00.000Z | blood_glucose | 220           | null     | null      | 12   |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-04-15T00:00:00.000Z | blood_glucose | 236           | null     | null      | 12   |

### 4. Identificar los datos duplicados 
``` SQL
-- Identify as temporary table
DROP TABLE IF EXISTS unique_duplicate_records;

CREATE TEMPORARY TABLE unique_duplicate_records AS
SELECT *
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
HAVING COUNT(*) > 1;
```

``` SQL
-- Finally let's inspect the top 10 rows of our temp table
SELECT *
FROM unique_duplicate_records
LIMIT 10;
```
| id                                       | log_date                 | measure       | measure_value | systolic | diastolic |
|------------------------------------------|--------------------------|---------------|---------------|----------|-----------|
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-04-15T00:00:00.000Z | blood_glucose | 267           | null     | null      |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-08-18T00:00:00.000Z | weight        | 68.49244787   | 0        | 0         |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-01-04T00:00:00.000Z | blood_glucose | 113           | null     | null      |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-01-12T00:00:00.000Z | blood_glucose | 121           | null     | null      |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-05-11T00:00:00.000Z | blood_glucose | 76            | null     | null      |
| ee653a96022cc3878e76d196b1667d95beca2db6 | 2020-05-31T00:00:00.000Z | blood_glucose | 99            | 0        | 0         |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-04-27T00:00:00.000Z | blood_glucose | 106           | null     | null      |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-01-07T00:00:00.000Z | blood_glucose | 243           | null     | null      |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-05-03T00:00:00.000Z | blood_glucose | 379           | null     | null      |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-21T00:00:00.000Z | blood_glucose | 237           | null     | null      |

### Recomendaciones importantes 
A lo largo de esta revision se ha hecho la explicacion de tres diferentes metodos para obtener o mapear los valores duplicados de un dataset
pero ¿Cual de estos debemos utilizar y en que caso ? para responder esta incognita es necesario preguntarnos mas a fondo si necesitaremos en el futuro utilizar los datos filtrados ? si nuestra respuesta es si debemos optar por usar las temporary tables , en caso contrario utilizaremos las Common Table Expressions .


## Exercises 

Cual id tiene el mayor numero de datos duplicados en la tabla 
health.user_logs . 
``` SQL 
WITH NEW_TABLE AS (
SELECT id,log_date,measure,measure_value,systolic,diastolic,COUNT(*) AS FREQ 
FROM health.user_logs
GROUP BY id,log_date,measure,measure_value,systolic,diastolic
ORDER BY FREQ DESC 
)
SELECT id , sum(FREQ) AS total_dup
FROM NEW_TABLE 
WHERE FREQ >1
GROUP BY id 
ORDER BY total_dup DESC
LIMIT 10 ; 
```
| id                                       | total_dup |
|------------------------------------------|-----------|
| 054250c692e07a9fa9e62e345231df4b54ff435d | 17279     |
| ee653a96022cc3878e76d196b1667d95beca2db6 | 758       |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 485       |
| 6c2f9a8372dac248192c50219c97f9087ab778ba | 106       |
| 981197b530b9ec5032abb0ffe4b69dba3649f467 | 77        |
| 907d1231aed4a3540b30f91b336130746042ce47 | 69        |
| 316f2f73322d8edcea74d4f22b6749bf2dc3dd9a | 58        |
| df0da942ab95c538f96d70dead8353e348845efe | 45        |
| abc634a555bbba7d6d6584171fdfa206ebf6c9a0 | 42        |
| 576fdb528e5004f733912fae3020e7d322dbc31a | 41        |


Cual log_date tiene el mayor numero de datos duplicados cuando removemos el id del primer ejercicio ?
``` SQL
WITH NEW_TABLE2 AS (
SELECT id,log_date,measure,measure_value,systolic,diastolic,COUNT(*) AS FREQ 
FROM health.user_logs
WHERE id != '054250c692e07a9fa9e62e345231df4b54ff435d'
GROUP BY id,log_date,measure,measure_value,systolic,diastolic
ORDER BY FREQ DESC 
)
SELECT log_date , sum(FREQ) AS total_dup
FROM NEW_TABLE2 
WHERE FREQ >1
GROUP BY log_date 
ORDER BY total_dup DESC
LIMIT 10 ; 
```
| log_date                 | total_dup |
|--------------------------|-----------|
| 2019-12-11T00:00:00.000Z | 55        |
| 2019-12-10T00:00:00.000Z | 22        |
| 2020-03-08T00:00:00.000Z | 20        |
| 2020-04-11T00:00:00.000Z | 20        |
| 2020-06-12T00:00:00.000Z | 19        |
| 2020-07-30T00:00:00.000Z | 18        |
| 2020-07-31T00:00:00.000Z | 17        |
| 2020-06-11T00:00:00.000Z | 16        |
| 2020-07-20T00:00:00.000Z | 16        |
| 2020-06-14T00:00:00.000Z | 16        |

Cual Measure_value tiene la mayor cantidad de ocurrencias en la tabla health.user_logs cuando el measure = 'weight' 
``` SQL
SELECT measure_value,COUNT(*) AS FREQ 
FROM health.user_logs
WHERE measure = 'weight'
GROUP BY measure_value
ORDER BY FREQ DESC 
LIMIT 5 ; 
```
| measure_value | freq |
|---------------|------|
| 68.49244787   | 109  |
| 67.58526313   | 107  |
| 62.595696     | 44   |
| 63.50288      | 44   |
| 62.142104     | 39   |


How many single duplicated rows exist when measure = 'blood_pressure' in the health.user_logs? How about the total number of duplicate records in the same table?

``` SQL
WITH NEW_TABLE3 AS (
SELECT id,log_date,measure,measure_value,systolic,diastolic,COUNT(*) AS FREQ 
FROM health.user_logs
WHERE measure = 'blood_pressure'
GROUP BY id,log_date,measure,measure_value,systolic,diastolic
)
SELECT COUNT(*) AS dup , SUM(FREQ) AS tot
FROM NEW_TABLE3
WHERE FREQ > 1 ;
```
| dup | tot |
|-----|-----|
| 147 | 301 |

What percentage of records measure_value = 0 when measure = 'blood_pressure' in the health.user_logs table? How many records are there also for this same condition?

``` SQL
WITH NT4 AS (
SELECT measure_value , COUNT(*) AS FREQ ,  SUM(COUNT(*)) OVER () AS PERCENT
FROM health.user_logs
WHERE measure = 'blood_pressure'
GROUP BY measure_value)
SELECT measure_value,FREQ,PERCENT,ROUND(100 *FREQ::NUMERIC / PERCENT, 2) AS percentage
FROM NT4
WHERE measure_value = 0;
```
| measure_value | freq | percent | percentage |
|---------------|------|---------|------------|
| 0             | 562  | 2417    | 23.25      |


What percentage of records are duplicates in the health.user_logs table?
``` SQL
WITH NT5 AS (
SELECT id,log_date,measure,measure_value,systolic,diastolic,COUNT(*) AS FREQ 
FROM health.user_logs
GROUP BY id,log_date,measure,measure_value,systolic,diastolic
)
SELECT ROUND(100*SUM(CASE WHEN FREQ >1 THEN FREQ - 1 ELSE 0 END)::NUMERIC/SUM(FREQ),2) AS dup
FROM NT5;
```
| dup   |
|-------|
| 29.36 |