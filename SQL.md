# Índice

* **[PARCIAL 2022](#parcial-2022)**
  * [Punto 3 - Query](#punto-3---2022)
  * [Punto 4 - Procedure](#punto-4---2022)
  * [Punto 5 - Trigger](#punto-5---2022)
* **[PARCIAL 2023](#parcial-2023)**
  * [Punto 3 - Query](#punto-3---2023)
  * [Punto 4 - Procedure](#punto-4---2023)
  * [Punto 5 - Trigger](#punto-5---2023)
* **[1er CUATRIMESTRE 2025](#1er-cuatrimestre-2025)**
  * [Punto 3 - Query](#punto-3---1c2025)
  * [Punto 4 - Procedure](#punto-4---1c2025)
  * [Punto 5 - Triggers](#punto-5---1c2025)
* **[2do CUATRIMESTRE 2025](#2do-cuatrimestre-2025)**
  * [Punto 3 - Query](#punto-3---2c2025)
  * [Punto 4 - Procedure](#punto-4---2c2025)
  * [Punto 5 - Trigger](#punto-5---2c2025)

---

## PARCIAL 2022

### Punto 3 - 2022

```sql
-- PUNTO 3
-- Crear una consulta que muestre de las tres Estados que tengan la mayor cantidad de VENTAS (no compras): 
-- Nombre del Estado, monto total vendido en ese Estado, nombre del fabricante y cantidad vendida total de ese fabricante en esa provincia.
-- Solo se deberán mostrar en la consulta los fabricantes cuyas ventas totales superen el 15% de las ventas de su provincia.
-- Ordenar el resultado por el monto total vendido del Estado de mayor a menor y por monto vendido del fabricante de manera descendente.
-- Notas: Se puede utilizar SOLO UN subquery. No usar Store procedures, ni funciones de usuarios, ni tablas temporales.

-- subqueries necesarias para entender el dominio necesario
SELECT TOP 3 
    m2.state AS estado, 
    SUM(i2.quantity * i2.unit_price) AS venta_total
FROM items i2 
JOIN manufact m2 ON (i2.manu_code = m2.manu_code)
JOIN orders o2 ON (i2.order_num = o2.order_num)
GROUP BY state
ORDER BY 2 DESC;

SELECT DISTINCT
    m.state AS estado,
    m.manu_name,
    SUM(i.quantity * i.unit_price) AS venta_fabricantes
FROM manufact m
JOIN items i ON (i.manu_code = m.manu_code)
JOIN orders o ON (i.order_num = o.order_num)
GROUP BY state, manu_name
ORDER BY 1 ASC, 3 DESC;

-- RESPUESTA FINAL
SELECT 
    st.sname AS estado,
    ee.venta_total,
    m.manu_name AS nombre_fabricante,
    SUM(i.quantity * i.unit_price) AS monto_total_fabricante
FROM manufact m
JOIN state st ON (st.state = m.state)
JOIN (
    SELECT TOP 3 WITH TIES 
        m2.state AS estado, 
        SUM(i2.quantity * i2.unit_price) AS venta_total
    FROM items i2 
    JOIN manufact m2 ON (i2.manu_code = m2.manu_code)
    JOIN orders o2 ON (i2.order_num = o2.order_num)
    GROUP BY state
    ORDER BY 2 DESC
) ee ON (ee.estado = m.state)
JOIN items i ON (i.manu_code = m.manu_code)
JOIN orders o ON (o.order_num = i.order_num)
GROUP BY st.sname, m.manu_name, ee.venta_total
HAVING (ee.venta_total * 0.15) < (SUM(i.quantity * i.unit_price))
ORDER BY 2 DESC, 4 DESC;