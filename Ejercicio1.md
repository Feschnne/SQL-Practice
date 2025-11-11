# EJERCICIOS SQL PARA PREPARACIÓN DEL MÁSTER

## EJERCICIO 1
### ENUNCIADO

Obtener el nombre del cliente, la ciudad, y el total gastado de aquellos clientes que:
 - Hayan tenido al menos 2 pedidos
 - Y que su primer pedido (el más antiguo) haya sido mayor de 100€
Ordenar el resultado por total gastado DESC

### ANÁLISIS DE COMPRENSIÓN LECTORA

| FRASE DEL ENUNCIADO | QUÉ SIGNIFICA A NIVEL SQL | POR QUÉ |
|------------|------------|------------|
| nombre y ciudad     | columnas directas     | viene de clientes -> SELECT     |
| total gastado     | sum(total_pedidos)     | agregado     |
| al menos dos pedidos     | count(id_pedido) > 2     | condición agregada     |
| primer pedido > 100     | min(total_pedido) > 100     | primer pedido = "más antiguo" -> MIN del importe     |
| ordenar por total gastado     | order by sum(total_pedido) DESC     | orden agregado     |

CONCLUSIÓN CLAVE: Este enunciado contiene dos agregaciones -> SUM() Y COUNT() Y MIN() --- por tanto ---> necesita GROUP BY y HAVING

### SQL

```
SELECT c.nombre,
       c.ciudad, 
       SUM(p.total_pedido) AS total_gastado
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre, c.ciudad
HAVING COUNT(p.id_pedido) > 1 AND MIN(p.total_pedido) > 100
ORDER BY total_gastado DESC;
```

---

## EJERCICIO 2

### ENUNCIADO

Obtener el nombre, la ciudad y el número de pedidos de los clientes que:
 - Sean de Madrid o Sevilla
 - Y cuyo importe medio (average) de pedido sea mayor que 90€

### SQL
```
SELECT c.nombre, c.ciudad,
       COUNT(id_pedido) as num_pedidos,
       AVG(total_pedido) as importe_medio,
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre, c.ciudad
HAVING (ciudad = 'Madrid' or ciudad = 'Sevilla') AND (AVG(total_pedido) > 90);
```

### CORRECCIÓN 
```
SELECT c.nombre, 
       c.ciudad, 
       COUNT(p.id_pedido) AS num_pedidos, 
       AVG(p.total_pedido) AS importe_medio
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
WHERE c.ciudad IN ('Madrid', 'Sevilla')
GROUP BY c.nombre, c.ciudad
HAVING AVG(p.total_pedido) > 90
ORDER BY num_pedidos DESC;
```

| PARTE | MOTIVO | BLOQUE |
|------------|------------|------------|
| ciudad     | no agregada, filtra filas individuales     | where     |
| avg(total_pedido) > 90     | necesita agregación     | having     |
| count(id_pedido)     | métrica a mostrar     | select     |
| group by     | porque hay agregados y columnas no agregadas     | group by     |

---

## EJERCICIO 3
### ENUNCIADO

Obtener el nombre, la ciudad, el total gastado y el importe medio de pedido solo de los clientes cuyo importe máximo haya sido mayor que 200€ y que hayan realizado al menos 2 pedidos.
Ordenar por total gastado descendente

### SQL
```
SELECT c.nombre, 
       c.ciudad, 
       SUM(p.total_pedido) as total_gastado,
       AVG(p.total_pedido) as importe_medio
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre, c.ciudad
HAVING MAX(p.total_pedido) > 200 AND COUNT(p.id_pedido) > 1
ORDER BY total_gastado DESC;
```

### CORRECCIÓN 
Todo estaba bien

---

## EJERCICIO 4
### ENUNCIADO

Obtener el nombre, número de pedidos, el máximo total pedido y el mínimo total pedido SOLO de los clientes cuyo importe medio sea entre 80 y 150€. Además, mostrar solo los clientes que NO son de Madrid y ordenar el resultado por diferencia entre máximo y mínimo total de pedido, de mayor a menor.

### SQL
```
SELECT c.nombre,
       COUNT(p.id_pedido) as num_pedidos,
       MAX(total_pedido) as maximo_total_pedido,
       MIN(total_pedido) as minimo_total_pedido, 
       AVG(total_pedido) as importe_medio
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
WHERE c.ciudad <> 'Madrid'
GROUP BY c.nombre
HAVING AVG(p.total_pedido) BETWEEN 80 AND 150
ORDER BY MAX(p.total_pedido) - MIN(p.total_pedido) DESC;
```

### CORRECCIÓN
Solo se corrigió el ORDER BY.

---

## EJERCICIO 5

### ENUNCIADO

Obtener nombre, ciudad, número total de pedidos, importe máximo y mínimo, pero SOLO de clientes que:
- Se dieron de alta a partir de enero 2023
- Y cuyo total gastado sea mayor de 250€
- Ordenar el resultado por importe medio de pedido de mayor a menor

### SQL

```
SELECT c.nombre,
       c.ciudad, 
       COUNT(p.id_pedido) AS total_pedidos,
       MAX(p.total_pedido) AS maximo_pedido,
       MIN(p.total_pedido) AS minimo_pedido,
       AVG(p.total_pedido) AS importe_medio
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
WHERE c.fecha_alta > '2023-01-01'
GROUP BY c.nombre, c.ciudad
HAVING SUM(p.total_pedido) > 250
ORDER BY AVG(p.total_pedido) DESC;
```

### CORRECCIÓN
Falta incluir AVG(total_pedido) en el SELECT para claridad.

---

## EJERCICIO 6

### ENUNCIADO

Obtener nombre, ciudad, número de pedidos, y una clasificación textual:
- Si el total gastado > 300 -> "High Spender"
- Si el total gastado entre 150 y 300 -> "Medium Spender"
- Sino -> "Low Spender"
- Solo clientes con más de 1 pedido
- Ordenado por total gastado descendente

### SQL

```
SELECT c.nombre, 
       c.ciudad,
       COUNT(p.id_pedido) AS num_pedidos, 
       SUM(p.total_pedido) AS total_gastado,
       CASE
          WHEN SUM(p.total_pedido) > 300 THEN 'High Spender'
          WHEN SUM(p.total_pedido) BETWEEN 150 AND 300 THEN 'Medium Spender'
          ELSE 'Low Spender'
       END AS categoria
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre, c.ciudad
HAVING COUNT(p.id_pedido) > 1
ORDER BY total_gastado DESC;
```

### CORRECCIÓN
Alias de agregados usado directamente en CASE.

---

## EJERCICIO 7

### ENUNCIADO

Obtener nombre, ciudad, número de pedidos, total gastado, importe medio y una clasificación:
- total gastado > 300 -> super vip
- entre 200 y 300 -> vip
- entre 100 y 200 -> regular
- <100 -> low
- Además, filtrar solo clientes que tengan más de 1 pedido cuyo importe máximo de pedido sea mayor de 250€
- Ordenar por importe medio de pedido descendente

### SQL

```
SELECT c.nombre,
       c.ciudad, 
       COUNT(p.id_pedido) AS num_pedidos,
       SUM(p.total_pedido) AS total_gastado,
       MAX(p.total_pedido) AS max_pedido,
       AVG(p.total_pedido) AS importe_medio,
       CASE
          WHEN SUM(p.total_pedido) > 300 THEN 'Super Vip'
          WHEN SUM(p.total_pedido) BETWEEN 200 AND 300 THEN 'Vip'
          WHEN SUM(p.total_pedido) BETWEEN 100 AND 200 THEN 'Regular'
          ELSE 'Low'
       END AS categoria
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre, c.ciudad
HAVING COUNT(p.id_pedido) > 1 AND MAX(p.total_pedido) > 250
ORDER BY AVG(p.total_pedido) DESC;
```

### CORRECCIÓN
No se puede usar alias total_gastado en CASE, usar SUM(p.total_pedido).

---

## EJERCICIO 8

### ENUNCIADO

Obtener nombre, ciudad, número de pedidos, total gastado, importe medio, importe máximo y una clasificación avanzada:
- total gastado > 300 e importe máximo > 150 -> super vip
- total gastado entre 200 y 300 o importe máximo entre 120-150 -> vip
- total gastado entre 100-200 y número de pedidos > 1 -> regular
- En cualquier otro caso -> low
- Clientes con pedido mínimo > 50
- Clientes con fecha_alta >= 2023-01-01
- Ordenar por diferencia entre máximo y mínimo total del pedido descendente

### SQL

```
SELECT c.nombre,
       c.ciudad,
       COUNT(p.id_pedido) AS num_pedidos,
       SUM(p.total_pedido) AS total_gastado,
       AVG(p.total_pedido) AS importe_medio,
       MAX(p.total_pedido) AS importe_maximo,
       CASE
          WHEN SUM(p.total_pedido) > 300 AND MAX(p.total_pedido) > 150 THEN 'Super Vip'
          WHEN SUM(p.total_pedido) BETWEEN 200 AND 300 OR MAX(p.total_pedido) BETWEEN 120 AND 150 THEN 'Vip'
          WHEN SUM(p.total_pedido) BETWEEN 100 AND 200 AND COUNT(p.id_pedido) > 1 THEN 'Regular'
          ELSE 'Low'
       END AS categoria
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
WHERE c.fecha_alta >= '2023-01-01'
GROUP BY c.nombre, c.ciudad
HAVING MIN(p.total_pedido) > 50
ORDER BY MAX(p.total_pedido) - MIN(p.total_pedido) DESC;
```

### CORRECCIÓN
Fecha de alta movida al WHERE, HAVING solo para agregados.

---

## EJERCICIO 9

### ENUNCIADO

Obtener el nombre, ciudad, número de pedidos y total gastado de los clientes que:
- sean de Madrid o Valencia
- tengan más de 1 pedido
- Ordenar por total gastado descendente

### SQL

```
SELECT c.nombre, 
       c.ciudad, 
       COUNT(p.id_pedido) AS num_pedidos,
       SUM(p.total_pedido) AS total_gastado
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
WHERE c.ciudad IN ('Madrid','Valencia')
GROUP BY c.nombre, c.ciudad
HAVING COUNT(p.id_pedido) > 1
ORDER BY total_gastado DESC;
```

---

## EJERCICIO 10

### ENUNCIADO

Obtener nombre, ciudad, número de pedidos, total gastado e importe medio de los clientes que:
- su importe máximo de pedido sea mayor de 200
- hayan realizado al menos 2 pedidos
- Clasificar a los clientes según total gastado:
  - >300 → super vip
  - 200–300 → vip
  - 100–200 → regular
  - <100 → low
- Ordenar por importe medio descendente

### SQL

```
SELECT c.nombre, 
       c.ciudad,
       COUNT(p.id_pedido) AS num_pedidos,
       SUM(p.total_pedido) AS total_gastado,
       AVG(p.total_pedido) AS importe_medio,
       MAX(p.total_pedido) AS max_pedido,
       CASE
          WHEN SUM(p.total_pedido) > 300 THEN 'Super Vip'
          WHEN SUM(p.total_pedido) BETWEEN 200 AND 300 THEN 'Vip'
          WHEN SUM(p.total_pedido) BETWEEN 100 AND 200 THEN 'Regular'
          ELSE 'Low'
       END AS categoria
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre, c.ciudad
HAVING MAX(p.total_pedido) > 200 AND COUNT(p.id_pedido) >= 2
ORDER BY AVG(p.total_pedido) DESC;
```

### CORRECCIÓN
CASE correctamente terminado con END AS alias.

---

## EJERCICIO 11

### ENUNCIADO

Obtener nombre, ciudad, número de pedidos, total gastado, importe máximo, importe mínimo y una clasificación avanzada:
- total gastado > 300 y número de pedidos > 3 → “SUPER VIP”
- total gastado 200–300 o importe máximo 150–200 → “VIP”
- total gastado 100–200 y pedido mínimo > 50 → “REGULAR”
- resto → “LOW”
Filtros:
- Clientes con fecha_alta >= '2023-01-01'
- Clientes con más de 1 pedido o cuyo importe máximo > 100
- Ordenar por diferencia entre máximo y mínimo pedido descendente y luego por total gastado descendente

### SQL

```
SELECT c.nombre,
       c.ciudad,
       COUNT(p.id_pedido) AS num_pedidos,
       SUM(p.total_pedido) AS total_gastado,
       MAX(p.total_pedido) AS max_pedido,
       MIN(p.total_pedido) AS min_pedido,
       CASE  
          WHEN SUM(p.total_pedido) > 300 AND COUNT(p.id_pedido) > 3 THEN 'SUPER VIP'
          WHEN SUM(p.total_pedido) BETWEEN 200 AND 300 OR MAX(p.total_pedido) BETWEEN 150 AND 200 THEN 'VIP'
          WHEN SUM(p.total_pedido) BETWEEN 100 AND 200 AND MIN(p.total_pedido) > 50 THEN 'REGULAR' 
          ELSE 'LOW'
       END AS categoria
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
WHERE c.fecha_alta >= '2023-01-01'
GROUP BY c.nombre, c.ciudad
HAVING COUNT(p.id_pedido) > 1 OR MAX(p.total_pedido) > 100
ORDER BY MAX(p.total_pedido) - MIN(p.total_pedido) DESC, total_gastado DESC;
```

### CORRECCIÓN
- Corregido COUNT(p.total_pedidos) → COUNT(p.id_pedido)
- Orden correcto con alias calculados.

---

## EJERCICIO 12

### ENUNCIADO

Obtener nombre, ciudad, número de pedidos, total gastado, importe medio, importe máximo, importe mínimo, y una clasificación avanzada de clientes según total gastado y número de pedidos:
- total gastado > 400 y número de pedidos > 5 → elite
- total gastado 300–400 o importe máximo 200–300 → super vip
- total gastado 150–300 y número de pedidos > 2 → vip
- resto → regular
Filtros:
- Clientes con fecha_alta >= 2023-01-01
- Clientes con pedido mínimo > 50 e importe medio > 80
Ordenar por:
1. Diferencia máximo-mínimo descendente
2. Total gastado descendente
3. Número de pedidos descendente

### SQL

```
SELECT c.nombre,
       c.ciudad,
       COUNT(p.id_pedido) AS num_pedidos,
       SUM(p.total_pedido) AS total_gastado,
       AVG(p.total_pedido) AS importe_medio, 
       MAX(p.total_pedido) AS importe_maximo,
       MIN(p.total_pedido) AS importe_minimo,
       CASE
          WHEN SUM(p.total_pedido) > 400 AND COUNT(p.id_pedido) > 5 THEN 'ELITE'
          WHEN SUM(p.total_pedido) BETWEEN 300 AND 400 OR MAX(p.total_pedido) BETWEEN 200 AND 300 THEN 'SUPER VIP'
          WHEN SUM(p.total_pedido) BETWEEN 150 AND 300 AND COUNT(p.id_pedido) > 2 THEN 'VIP'
          ELSE 'REGULAR'
       END AS categoria
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
WHERE c.fecha_alta >= '2023-01-01'
GROUP BY c.nombre, c.ciudad
HAVING MIN(p.total_pedido) > 50 AND AVG(p.total_pedido) > 80
ORDER BY MAX(p.total_pedido) - MIN(p.total_pedido) DESC, total_gastado DESC, num_pedidos DESC;
```

### CORRECCIÓN
Alias consistentes, CASE correctamente terminado, HAVING y ORDER BY correctamente ajustados.

## EJERCICIO 13

###  ENUNCIADO
Obtener los nombres y ciudades de los clientes cuyo total gastado sea mayor que el total medio de todos los clientes.

### SQL
```
SELECT c.nombre,
       c.ciudad,
       sum(p.total_pedido) as total_gastado,
       avg(p.total_pedido) as total_medio
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre, c.ciudad
HAVING sum(p.total_gastado) > avg(p.total_pedido);
```

### CORRECIÓN

- HAVING SUM(p.total_gastado) > AVG(p.total_gastado) -> p.total_gastado no existe, esa columna no está en la tabla pedidos.
- Además, la comparación que pide el enunciado no es con AVG del mismo grupo, sino con la media de todos los clientes -> necesitamos una subconsulta.

- Agrupamos por cliente -> SUM(p.total_pedido) -> total gastado por cliente
- Comparamos ese total con la media de todos los clientes -> subconsulta:

### SQL 
```
(SELECT AVG(total_clientes)
  FROM(
        SELECT SUM(p2.total_pedido AS total_cliente
        FROM clientes c2
        JOIN pedidos p2 ON c2.id_cliente = p2.id_cliente
        GROUP BY c2.id_cliente
        ) c2.id_cliente
)
```
### SQL FINAL
```
SELECT c.nombre,
       c.ciudad,
       SUM(p.total_pedido) AS total_gastado
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre, c.ciudad
HAVING SUM(p.total_pedido) > (
    SELECT AVG(total_cliente)
    FROM (
        SELECT SUM(p2.total_pedido) AS total_cliente
        FROM clientes c2
        JOIN pedidos p2 ON c2.id_cliente = p2.id_cliente
        GROUP BY c2.id_cliente
    ) AS sub
)
ORDER BY total_gastado DESC;
```

### EXPLICACIÓN LÍNEA POR LÍNEA
1. SELECT c.nombre, c.ciudad, SUM(p.total_pedido) AS total_gastado -> oobtenemos total gastado por cliente
2. FROM clientes c JOIN pedidos p -> unimos pedidos con clientes
3. GROUP BY c.nombre, c.ciudad -> agregamos por cliente
4. HAVING SUM(p.total_pedido) > (...) -> filtramos solo los clientes cuyo total individual supere la m edia de todos
5. Subconsulta -> calcula la media de totales por cliente
6. ORDER BY total_gastado DESC -> ordenamos los clientes que más gastaron

### QUÉ ES UNA SUBSONCONSULTA
Una subconsulta (o subquery) es una consulta dentro de otrta consulta, que devuelve un valor (o conjunto de valores) que se 
usa para filtrar, comparar o agregar en la consulta principal.

### CUANDO NO NECESITAS SUBCONSULTA
Si toda la información que necesitas ya está disponible en la misma tabla o puedes calcularla directamente con JOIN y GROUP BY,
no necesitas subconsulta.

Ejemplo: Obtener el total gastado por cada cliente
```
SELECT c.nombre, SUM(p.total_pedido) as total_gastado
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre;
```
No necesitamos subconsulta porque todos los datos están en clientes y pedidos.

### CUANDO SÍ NECESITAS SUBCONSULTA
Necesitas subconsulta cuando el criterio de filtrado o comparación requiere un cálculo que depende de todos los registros de la tabla y no solo del grupo 
que estas procesando.

Ejemplo: Obtener los clientes cuyo total gastado sea mayor que la media de todos los clientes.

1. Primero necesitas saber la media de todos los clientes -> no es un valor que esté en la tabla, tienes que calcularlo:
```
SELECT AVG(total_cliente) AS promedio_total
FROM (
    SELECT SUM(p.total_pedido) AS total_cliente
    FROM clientes c
    JOIN pedidos p ON c.id_cliente = p.id_cliente
    GROUP BY c.id_cliente
) AS sub;
```
Output sería: promedio_total -> 256

2. Luego quieres filtrar solo los clientes que superan esa media:
```
SELECT c.nombre, SUM(p.total_pedido) AS total_gastado
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nombre
HAVING SUM(p.total_pedido) > (
    SELECT AVG(total_cliente)
    FROM (
        SELECT SUM(p2.total_pedido) AS total_cliente
        FROM clientes c2
        JOIN pedidos p2 ON c2.id_cliente = p2.id_cliente
        GROUP BY c2.id_cliente
    ) AS sub
);
```
Subconsulta necesaria porque AVG(total_cliente) depende de todos los clientes y no puede calcularse dentro del GROUP BY de un cliente individual.

### RESUMEN PRÁCTICO

| Situación                                                                     | Subconsulta necesaria? | Ejemplo                                                        |
| ----------------------------------------------------------------------------- | ---------------------- | -------------------------------------------------------------- |
| Calcular totales o promedios **por grupo** que ya tienes en la misma consulta | ❌ No                   | SUM(p.total_pedido) GROUP BY c.nombre                          |
| Comparar con un **valor agregado global** (como media de todos los clientes)  | ✅ Sí                   | HAVING SUM(p.total_pedido) > (SELECT AVG(...))                 |
| Filtrar registros basados en otra tabla o conjunto de resultados              | ✅ Sí                   | Clientes que han hecho pedidos mayores al máximo de otra tabla |
| Solo necesitas filtrar por condiciones simples de la misma fila o tabla       | ❌ No                   | WHERE ciudad = 'Madrid'                                        |


Si haces: 
```
SELECT AVG(total_cliente) AS promedio_total
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.id_cliente;
```

Esto no te da un solo valor, sino un conjunto de medias por cliente:
| id_cliente | total_cliente |
| ---------- | ------------- |
| 1          | 320           |
| 2          | 180           |
| 3          | 270           |

 - Cada fila es la suma de un cliente
 - AVG(...) sobre esta consulta con GROUP BY seguiría devolviendo varias filas, no un único número global.

Para obterner un solo valor global (la media de todos los clientes) necesitamos una subconsulta que agregue primero por cliente y luego tome la media de esos resultados:
```
SELECT AVG(total_cliente) AS promedio_total
FROM (
    SELECT SUM(p.total_pedido) AS total_cliente
    FROM clientes c
    JOIN pedidos p ON c.id_cliente = p.id_cliente
    GROUP BY c.id_cliente
) AS sub;
```
Ahora la subconsulta devuelve
| promedio_total |
| -------------- |
| 256            |

 - Este es un solo valor, que ya puedes usar para comparar con cada cliente en HAVING

### TIPOS DE ENUNCIADOS QUE REQUIEREN SUBCONSULTAS
En general, los subqueries aparecen cuando el enunciado pide comparar, filtrar o relacionar con un valor que no existe directamente en la tabla. Algunos patrones comunes:
| Patrón de enunciado                                                     | Qué hacer / Subconsulta necesaria?                                 |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Clientes cuyo total sea **mayor que el promedio de todos los clientes** | Subconsulta para calcular la media global                          |
| Clientes cuyo total sea **mayor que el máximo de otra tabla**           | Subconsulta para obtener el máximo                                 |
| Productos que se vendieron **más que la media de ventas por categoría** | Subconsulta agrupando por categoría                                |
| Clientes que han hecho pedidos **mayores que su promedio personal**     | Subconsulta para calcular AVG por cliente dentro de la misma tabla |
| Filtrar según resultados de otra consulta compleja                      | Subconsulta en WHERE o FROM                                        |

### CÓMO DETECTARLO EN UN ENUNCIADO
1. "Mayor que la media de todos...", "menor que el máximo de...", "más que X"
2. "Comparar con un valor calculado sobre todos los registros"
3. "Filtrar según un resultado agregado de otra tabla"

Si ves alguna de estas frases, piensa Subconsulta.

















































