# Revision : Estadistica Basica en SQL II 
Esta seccion será la extension de la revision pasada en la que se explicaban conceptos de estadistica basica  aplicada en SQL 
por esta razon realizaremos un resumen estadistico de lo aprendido.
``` SQL
WITH newtable AS (
SELECT 
'weight' AS measure,
ROUND(AVG(measure_value),2) AS media ,
ROUND(CAST(PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY measure_value)AS NUMERIC ) ,2) AS mediana ,
ROUND (MODE () WITHIN GROUP (ORDER BY measure_value),2) AS moda ,
ROUND(MAX(measure_value),2) AS MAXIMO,
ROUND(MIN(measure_value),2) AS MINIMO,
ROUND(VARIANCE(measure_value),2) AS VARIANZA,
ROUND(STDDEV(measure_value),2) AS D_ESTANDAR 
FROM health.user_logs
WHERE measure = 'weight')
SELECT *
FROM newtable;
```
| measure | media    | mediana | moda  | maximo      | minimo | varianza         | d_estandar |
|---------|----------|---------|-------|-------------|--------|------------------|------------|
| weight  | 28786.85 | 75.98   | 68.49 | 39642120.00 | 0.00   | 1129457862383.41 | 1062759.55 |

En este punto tenemos las bases para identificar las medidas de tendencia central y dispersion de un set de datos , en esta revision se hara la revision de los graficos de dispersion y las tecnicas de analisis de datos para identificar outliers o atipicos .

## Algoritmo para la distribucion de datos 
1 . Ordenar todos los datos de menor a mayor 
2 . Dividir los datos en 100 grupos 
```SQL
SELECT 
measure_value,NTILE(100) OVER(ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight';
```
De esta manera obtenemos una lista de datos con la columna denominada percentile que nos indica el orden ascendente de los datos
organizados en 100 grupos , la funcion NTILE asigna los percentiles a los datos dividiendolos en partes iguales , esta funcion se explicara en profundidad mas adelante. 

```SQL 
WITH percentiles  AS (
SELECT 
measure_value,NTILE(100) OVER(ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight';
)
SELECT percentile, MIN(measure_value) AS floorv , MAX(measure_value) as ceilv
FROM percentiles 
GROUP BY percentile 
ORDER BY percentile ; 

```
| percentile | floorv        | ceilv         |
|------------|---------------|---------------|
| 1          | 0             | 29.029888     |
| 2          | 29.48348      | 32.0689544    |
| 3          | 32.205032     | 35.380177     |
| 4          | 35.380177     | 36.74095      |
| 5          | 36.74095      | 37.194546     |
| 6          | 37.194546     | 38.101727     |
| 7          | 38.101727     | 39.00891      |
| 8          | 39.00891      | 40.36969      |
| 9          | 40.36969      | 41.27687      |
| 10         | 41.27687      | 43.54483      |
| 11         | 43.998425     | 51.709488     |
| 12         | 52.16308      | 56.3          |
| 13         | 56.4          | 57.6          |
| 14         | 57.6          | 61.688512     |
| 15         | 61.688512     | 62.142104     |
| 16         | 62.142104     | 62.595696     |
| 17         | 62.595696     | 62.595696     |
| 18         | 62.595696     | 63.049288     |
| 19         | 63.049288     | 63.50288      |
| 20         | 63.50288      | 63.50288      |
| 21         | 63.6843168    | 64.1379088    |
| 22         | 64.1379088    | 64.410064     |
| 23         | 64.410064     | 64.5007824    |
| 24         | 64.5007824    | 64.6822192    |
| 25         | 64.6822192    | 64.863656     |
| 26         | 64.863656     | 64.9543744    |
| 27         | 64.9543744    | 65.1358112    |
| 28         | 65.1358112    | 65.317248     |
| 29         | 65.317248     | 65.77084      |
| 30         | 65.77084      | 66            |
| 31         | 66.224432     | 67.585208     |
| 32         | 67.58526313   | 67.58526313   |
| 33         | 67.58526313   | 67.58526313   |
| 34         | 67.58526313   | 67.58526313   |
| 35         | 67.58526313   | 67.8          |
| 36         | 67.812059315  | 68.49244787   |
| 37         | 68.49244787   | 68.49244787   |
| 38         | 68.49244787   | 68.49244787   |
| 39         | 68.49244787   | 68.49244787   |
| 40         | 68.49244787   | 68.719244055  |
| 41         | 68.719244055  | 69.39963261   |
| 42         | 69.39963261   | 69.9          |
| 43         | 70            | 71.213944     |
| 44         | 71.213944     | 73.481904     |
| 45         | 73.481904     | 74.162292     |
| 46         | 74.162292     | 74.5705248    |
| 47         | 74.615884     | 74.84274105   |
| 48         | 74.84274105   | 75.296272     |
| 49         | 75.296272     | 75.29633342   |
| 50         | 75.3          | 76.203456     |
| 51         | 76.203456     | 76.883906715  |
| 52         | 76.883906715  | 77.337499085  |
| 53         | 77.337499085  | 77.791091455  |
| 54         | 77.791091455  | 78.017824     |
| 55         | 78.017824     | 78.698212     |
| 56         | 78.698212     | 78.698276195  |
| 57         | 78.698276195  | 79            |
| 58         | 79            | 79.151868565  |
| 59         | 79.151868565  | 79.605396     |
| 60         | 79.605396     | 79.83225712   |
| 61         | 79.83225712   | 80.013691299  |
| 62         | 80.013691299  | 80.28584949   |
| 63         | 80.28584949   | 83.91452      |
| 64         | 83.91452      | 86.18248      |
| 65         | 86.18248      | 87.3          |
| 66         | 87.543256     | 88.2690032    |
| 67         | 88.45044      | 88.904032     |
| 68         | 88.99482161   | 89.6297792    |
| 69         | 89.697006226  | 90.592002869  |
| 70         | 90.621002197  | 91.51700592   |
| 71         | 91.526000977  | 92.9          |
| 72         | 92.98636      | 94            |
| 73         | 94            | 95.980145492  |
| 74         | 95.980148261  | 96.433740631  |
| 75         | 96.433740631  | 96.887330232  |
| 76         | 96.887333001  | 97.522475529  |
| 77         | 97.522475529  | 98.784004211  |
| 78         | 98.883056     | 100.697426    |
| 79         | 100.697426    | 102.2         |
| 80         | 102.398950789 | 103.963378906 |
| 81         | 104.054092407 | 107.0930712   |
| 82         | 107.1         | 108.86208     |
| 83         | 108.9074392   | 112.490816    |
| 84         | 112.490816    | 115.0309312   |
| 85         | 115.1670088   | 117.6617648   |
| 86         | 117.6617648   | 119.1         |
| 87         | 119.1         | 120.5         |
| 88         | 120.655472    | 121.5         |
| 89         | 121.5         | 122.1         |
| 90         | 122.197685    | 122.923432    |
| 91         | 122.923432    | 123.500007629 |
| 92         | 123.500007629 | 124.6         |
| 93         | 124.6         | 127.459352    |
| 94         | 127.604003906 | 129.844       |
| 95         | 129.86485     | 130.542007446 |
| 96         | 130.54207     | 131.570999146 |
| 97         | 131.670013428 | 132.776       |
| 98         | 132.776000977 | 133.832000732 |
| 99         | 133.89095     | 136.531192    |
| 100        | 136.531192    | 39642120      |


3 . Identificar los datos atipicos 
Mediante esta asignacion en grupos por percentil tenemos una vision de que valores toman una tendencia irreal con respecto a los datos de estudio esto nos ayuda a limpiar nuestro set de datos para que nuestro analisis estadistico sea mas acertado .
En la siguiente consulta o query identificamos los datos atipicos del percentil 100 , que serian los datos mas grandes del set .

```SQL
-- Datos atipicos del percentil 100  
WITH percentiles  AS (
SELECT 
measure_value,NTILE(100) OVER(ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight'
)

SELECT 
measure_value,
ROW_NUMBER() OVER(ORDER BY measure_value DESC) AS row_number_order,
RANK() OVER(ORDER BY measure_value DESC ) AS rank_order ,
DENSE_RANK() OVER(ORDER BY measure_value DESC) AS dense_rank_order 
FROM percentiles 
WHERE percentile = 100 
ORDER BY measure_value DESC;
```
| measure_value | row_number_order | rank_order | dense_rank_order |
|---------------|------------------|------------|------------------|
| 39642120      | 1                | 1          | 1                |
| 39642120      | 2                | 1          | 1                |
| 576484        | 3                | 3          | 2                |
| 200.487664    | 4                | 4          | 3                |
| 190.4         | 5                | 5          | 4                |
| 188.69427     | 6                | 6          | 5                |
| 186.8799      | 7                | 7          | 6                |
| 185.51913     | 8                | 8          | 7                |
| 175.086512    | 9                | 9          | 8                |
| 173.725736    | 10               | 10         | 9                |
| 170.5506      | 11               | 11         | 10               |
| 170.5506      | 12               | 11         | 10               |
| 170.5506      | 13               | 11         | 10               |
| 164           | 14               | 14         | 11               |
| 157.5778608   | 15               | 15         | 12               |
| 149.68536     | 16               | 16         | 13               |
| 145.14944     | 17               | 17         | 14               |
| 144.242256    | 18               | 18         | 15               |
| 141.1578304   | 19               | 19         | 16               |
| 138.799152    | 20               | 20         | 17               |
| 137.438376    | 21               | 21         | 18               |
| 136.984784    | 22               | 22         | 19               |
| 136.984784    | 23               | 22         | 19               |
| 136.762008667 | 24               | 24         | 20               |
| 136.531192    | 25               | 25         | 21               |
| 136.531192    | 26               | 25         | 21               |
| 136.531192    | 27               | 25         | 21               |

``` SQL
-- Datos atipicos del percentil 1 
WITH percentiles  AS (
SELECT 
measure_value,NTILE(100) OVER(ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight'
)
SELECT 
measure_value,
ROW_NUMBER() OVER(ORDER BY measure_value ) AS row_number_order,
RANK() OVER(ORDER BY measure_value  ) AS rank_order ,
DENSE_RANK() OVER(ORDER BY measure_value ) AS dense_rank_order 
FROM percentiles 
WHERE percentile = 1 
ORDER BY measure_value ;


``` 

Las nuevas funciones que aparecen en las anteriores consultas tienen el siguiente significado 
- ROW_NUMBER : Enumera cada fila del set de datos en este caso enumera la fila del percentil 
- RANK : Enumera cada fila del set , si se trata de valores repetidos esta numeracion no es consecutiva 
- DENSE_RANK : Enumera cada fila del set , si se trata de valores repetidos esta numeracion es consecutiva repitiendo el numero 

| measure_value | row_number_order | rank_order | dense_rank_order |
|---------------|------------------|------------|------------------|
| 0             | 1                | 1          | 1                |
| 0             | 2                | 1          | 1                |
| 1.814368      | 3                | 3          | 2                |
| 2.26796       | 4                | 4          | 3                |
| 2.26796       | 5                | 4          | 3                |
| 8             | 6                | 6          | 4                |
| 10.432616     | 7                | 7          | 5                |
| 11.3398       | 8                | 8          | 6                |
| 12.700576     | 9                | 9          | 7                |
| 15.422128     | 10               | 10         | 8                |
| 18.14368      | 11               | 11         | 9                |
| 20.865232     | 12               | 12         | 10               |
| 23.586784     | 13               | 13         | 11               |
| 24.94756      | 14               | 14         | 12               |
| 25.854744     | 15               | 15         | 13               |
| 25.854744     | 16               | 15         | 13               |
| 27.21552      | 17               | 17         | 14               |
| 27.21552      | 18               | 17         | 14               |
| 27.21552      | 19               | 17         | 14               |
| 27.21552      | 20               | 17         | 14               |
| 27.669112     | 21               | 21         | 15               |
| 28.122704     | 22               | 22         | 16               |
| 28.122704     | 23               | 22         | 16               |
| 28.576296     | 24               | 24         | 17               |
| 28.576296     | 25               | 24         | 17               |
| 29.029888     | 26               | 26         | 18               |
| 29.029888     | 27               | 26         | 18               |
| 29.029888     | 28               | 26         | 18               |



4 . Limpieza del dataset 
Mediante este metodo procedemos a limpiar el dataset de datos atipicos para que el analisis estadistico no tenga valores irreales con respecto a nuestros datos .

```SQL 
-- Creacion de una tabla temporal 
DROP TABLE IF EXISTS clean_data ;
CREATE TEMP TABLE clean_data AS (
SELECT *
FROM health.user_logs
WHERE measure = 'weight' 
and measure_value > 0 
and measure_value < 201
);
-- Analisis estadistico de la tabla temporal 
SELECT
'weight' as measure,
ROUND(MIN(measure_value) , 2) AS minimum ,
ROUND(MAX(measure_value), 2 ) AS maximum,
ROUND(AVG(measure_value),2) AS mean, 
ROUND(CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC ),2) AS median,
ROUND(MODE() WITHIN GROUP (ORDER BY measure_value),2) AS mode ,
ROUND (STDDEV(measure_value),2) AS D_EST,
ROUND (VARIANCE(measure_value),2) AS variance
FROM clean_data ;

```
| measure | minimum | maximum | mean  | median | mode  | d_est | variance |
|---------|---------|---------|-------|--------|-------|-------|----------|
| weight  | 1.81    | 200.49  | 80.76 | 75.98  | 68.49 | 26.91 | 724.29   |


```SQL
-- Mostrar la distibucion acumulativa con los datos tratados 
WITH acumulative AS (
SELECT measure_value , NTILE(100) OVER(ORDER BY measure_value) AS percentile
FROM clean_data)

SELECT  percentile , min(measure_value),max(measure_value),COUNT(*) AS perc_count
FROM acumulative 
GROUP BY percentile
ORDER BY percentile ;
```
| percentile | min           | max           | perc_count |
|------------|---------------|---------------|------------|
| 1          | 1.814368      | 29.48348      | 28         |
| 2          | 29.48348      | 32.4771872    | 28         |
| 3          | 32.658623     | 35.380177     | 28         |
| 4          | 35.380177     | 36.74095      | 28         |
| 5          | 36.74095      | 37.194546     | 28         |
| 6          | 37.648136     | 38.101727     | 28         |
| 7          | 38.101728     | 39.00891      | 28         |
| 8          | 39.00891      | 40.36969      | 28         |
| 9          | 40.36969      | 41.730464     | 28         |
| 10         | 41.730465     | 43.998425     | 28         |
| 11         | 44.452015     | 52.16308      | 28         |
| 12         | 52.163082     | 56.5          | 28         |
| 13         | 56.517609302  | 57.6          | 28         |
| 14         | 57.6          | 61.688512     | 28         |
| 15         | 61.688512     | 62.142104     | 28         |
| 16         | 62.142104     | 62.595696     | 28         |
| 17         | 62.595696     | 62.595696     | 28         |
| 18         | 62.595696     | 63.049288     | 28         |
| 19         | 63.049288     | 63.50288      | 28         |
| 20         | 63.50288      | 63.7750352    | 28         |
| 21         | 63.7750352    | 64.1379088    | 28         |
| 22         | 64.1379088    | 64.410064     | 28         |
| 23         | 64.410064     | 64.5915008    | 28         |
| 24         | 64.5915008    | 64.6822192    | 28         |
| 25         | 64.6822192    | 64.863656     | 28         |
| 26         | 64.863656     | 64.9543744    | 28         |
| 27         | 64.9543744    | 65.1358112    | 28         |
| 28         | 65.1358112    | 65.317248     | 28         |
| 29         | 65.317248     | 65.77084      | 28         |
| 30         | 65.77084      | 66.224432     | 28         |
| 31         | 66.224432     | 67.58526313   | 28         |
| 32         | 67.58526313   | 67.58526313   | 28         |
| 33         | 67.58526313   | 67.58526313   | 28         |
| 34         | 67.58526313   | 67.58526313   | 28         |
| 35         | 67.58526313   | 67.812059315  | 28         |
| 36         | 67.812059315  | 68.49244787   | 28         |
| 37         | 68.49244787   | 68.49244787   | 28         |
| 38         | 68.49244787   | 68.49244787   | 28         |
| 39         | 68.49244787   | 68.49244787   | 28         |
| 40         | 68.49244787   | 68.719244055  | 28         |
| 41         | 68.719244055  | 69.39963261   | 28         |
| 42         | 69.39963261   | 70            | 28         |
| 43         | 70            | 71.213944     | 28         |
| 44         | 71.213944     | 73.481904     | 28         |
| 45         | 73.48196394   | 74.162292     | 28         |
| 46         | 74.162352495  | 74.615884     | 28         |
| 47         | 74.615884     | 75            | 28         |
| 48         | 75            | 75.296272     | 28         |
| 49         | 75.296272     | 75.5          | 28         |
| 50         | 75.5          | 76.203456     | 28         |
| 51         | 76.203456     | 76.883906715  | 28         |
| 52         | 76.883906715  | 77.337499085  | 28         |
| 53         | 77.337499085  | 77.791091455  | 28         |
| 54         | 77.791091455  | 78.01788764   | 28         |
| 55         | 78.01788764   | 78.698212     | 28         |
| 56         | 78.698276195  | 78.698276195  | 28         |
| 57         | 78.698276195  | 79.151804     | 28         |
| 58         | 79.151804     | 79.151868565  | 28         |
| 59         | 79.151868565  | 79.605460935  | 28         |
| 60         | 79.605460935  | 79.83225712   | 28         |
| 61         | 79.83225712   | 80.013691299  | 28         |
| 62         | 80.058988     | 80.739376     | 28         |
| 63         | 80.739376     | 83.91452      | 28         |
| 64         | 84            | 86.18248      | 28         |
| 65         | 86.18248      | 87.54332741   | 28         |
| 66         | 87.54332741   | 88.45044      | 28         |
| 67         | 88.5          | 88.994822994  | 28         |
| 68         | 88.999811846  | 89.75         | 28         |
| 69         | 89.759002686  | 90.7184       | 28         |
| 70         | 90.7184       | 91.55         | 28         |
| 71         | 91.625584     | 92.98636      | 28         |
| 72         | 92.98636      | 94            | 28         |
| 73         | 94.347136     | 96.05         | 28         |
| 74         | 96.05         | 96.433740631  | 28         |
| 75         | 96.524456336  | 96.887333001  | 28         |
| 76         | 96.9          | 97.703796498  | 28         |
| 77         | 97.73400116   | 98.88313666   | 28         |
| 78         | 99            | 100.697426    | 27         |
| 79         | 100.697426    | 102.2         | 27         |
| 80         | 102.398950789 | 103.872568    | 27         |
| 81         | 103.963378906 | 107.047712    | 27         |
| 82         | 107.047714    | 108.86208     | 27         |
| 83         | 108.86208     | 112.037224    | 27         |
| 84         | 112.037224    | 114.9402128   | 27         |
| 85         | 114.985572    | 117.5         | 27         |
| 86         | 117.5710464   | 119.1         | 27         |
| 87         | 119.1         | 120.4         | 27         |
| 88         | 120.4         | 121.4719376   | 27         |
| 89         | 121.4719376   | 122           | 27         |
| 90         | 122.050003052 | 122.9         | 27         |
| 91         | 122.9         | 123.450004578 | 27         |
| 92         | 123.5         | 124.5         | 27         |
| 93         | 124.5         | 127.4         | 27         |
| 94         | 127.43800354  | 129.762008667 | 27         |
| 95         | 129.77278     | 130.52802     | 27         |
| 96         | 130.5389      | 131.54168     | 27         |
| 97         | 131.54169     | 132.6599      | 27         |
| 98         | 132.736       | 133.765       | 27         |
| 99         | 133.80965     | 136.0776      | 27         |
| 100        | 136.0776      | 200.487664    | 27         |
