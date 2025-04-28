# Análisis Superstore 

## Tabla de contenidos

- [Preparacion de los datos](#preparacion-de-los-datos)
- [Análisis de datos](#análisis-de-datos)
- [Sugerencias](#sugerencias)
- [Power Bi](#power-bi)

### Resumen del proyecto

En este primer proyecto utilice la base de datos Superstore proveniente de Kaggle para realizar un analisis exploratorio. Utilizando herramientas como SQL y Power Bi respondí preguntas de interés para comprender mejor las fortalezas debilidades y estado general del negocio.

### Fuente de Datos

"Sample - Superstore.csv" el cual se puede obtener aqui:
- [📂 Sample - Superstore.csv](./Sample%20-%20Superstore.csv): Dataset original en formato CSV. 

### Herramientas

- SQL (para el analisis de datos y organizacion)

- Power Bi (para la visualizacion)

### Preparacion de los datos

Con el fin de enriquecer el analisis, separe la tabla de datos original en 3 tablas distintas, las cuales contenian informacion de los clientes, los productos y las ordenes.

- Tabla de clientes

```sql
CREATE TABLE customers (
    customer_id TEXT PRIMARY KEY,
    customer_name TEXT,
    segment TEXT,
    country TEXT,
    city TEXT,
    state TEXT,
    postal_code INT,
    region TEXT
);

INSERT INTO customers
SELECT DISTINCT ON (customer_id)
    customer_id,
    customer_name,
    segment,
    country,
    city,
    state_,
    postal_code,
    region
FROM superstore_orders;
```

- Tabla de productos

```sql
CREATE TABLE products (
    product_id TEXT PRIMARY KEY,
    product_name TEXT,
    category TEXT,
    sub_category TEXT
);

INSERT INTO products
SELECT DISTINCT ON (product_id)
    product_id,
    product_name,
    category,
    sub_category
FROM superstore_orders;
```
- Tabla de ordenes

```sql
CREATE TABLE orders (
    row_id INT PRIMARY KEY,
    order_id TEXT,
    order_date DATE,
    ship_date DATE,
    ship_mode TEXT,
    customer_id TEXT REFERENCES customers(customer_id),
    product_id TEXT REFERENCES products(product_id),
    sales NUMERIC,
    quantity INT,
    discount NUMERIC,
    profit NUMERIC
);

INSERT INTO orders
SELECT
    row_id,
    order_id,
    order_date,
    ship_date,
    ship_mode,
    customer_id,
    product_id,
    sales,
    quantity,
    discount,
    profit
FROM superstore_orders;

```
- Cambio la variabale state a state_, porque es una keyword

```sql
ALTER TABLE customers
RENAME column state to state_;
```
  
### Analisis exploratorio de los datos

Busco responder preguntas que puedan ser de ayuda para la mejor toma de decisiones 

- ¿Cuales son los productos con mas ventas totales?
- ¿Cuales son las ganancias por estado?
- ¿Como fue la evolucion de las ventas? ¿Que meses son los de mejores ventas?
- ¿Qué segmento de clientes (Consumer, Corporate, Home Office) deja más ganancias?
- ¿Cuál es la relación entre el porcentaje de descuento y el margen de ganancia?

### Análisis de datos

- ¿Cuales son los productos con mas ventas totales?

```sql
SELECT 
	p.product_name,
	ROUND (SUM(o.sales)) as total_sales
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_sales DESC
LIMIT 10;
```

- ¿Cuales son las ganancias por estado?
  
```sql
SELECT 
    c.state_,
    ROUND(SUM(o.profit), 2) AS total_profit
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.state_
ORDER BY total_profit DESC;
```

-¿Como fue la evolucion de las ventas? 

```sql
SELECT 
    TO_CHAR(DATE_TRUNC('month', order_date), 'FMMONTH YYYY') AS month_year,
    ROUND(SUM(sales), 2) AS total_sales
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY DATE_TRUNC('month', order_date);
```
- ¿Cuales son los meses de mejores ganancias en promedio?

```sql
SELECT 
    TO_CHAR(DATE_TRUNC('month', order_date), 'FMMONTH') AS month_year,
    ROUND(AVG(profit), 2) AS avg_profit
FROM orders
GROUP BY TO_CHAR(DATE_TRUNC('month', order_date), 'FMMONTH')
ORDER BY avg_profit DESC;
```
- ¿Qué segmento de clientes (Consumer, Corporate, Home Office) deja más ganancias?

```sql
SELECT 
    c.segment,
    ROUND(SUM(o.profit), 2) AS total_profit
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.segment
ORDER BY total_profit DESC;
```

-¿Cuál es la relación entre el porcentaje de descuento y el margen de ganancia?

```sql
SELECT 
    discount_range,
    ROUND(AVG(margin), 2) AS avg_margin
FROM (
    SELECT 
        CASE 
            WHEN discount = 0 THEN '0%'
            WHEN discount <= 0.1 THEN '0-10%'
            WHEN discount <= 0.2 THEN '10-20%'
            WHEN discount <= 0.3 THEN '20-30%'
            WHEN discount <= 0.4 THEN '30-40%'
            ELSE '>40%'
        END AS discount_range,
        CASE 
            WHEN discount = 0 THEN 0
            WHEN discount <= 0.1 THEN 1
            WHEN discount <= 0.2 THEN 2
            WHEN discount <= 0.3 THEN 3
            WHEN discount <= 0.4 THEN 4
            ELSE 5
        END AS sort_order,
        profit / NULLIF(sales, 0) AS margin
    FROM orders
) AS sub
GROUP BY discount_range, sort_order
ORDER BY sort_order;
```
### Resultados

1. El producto mas vendido de la compañia es "Canon imageCLASS 2200 Advanced Copier".
2. El estado en el cual se obtienen mas ganancias es en California con 61344.75 USD y el peor estado en cuanto a ganancias es Ohio con peridas de -5043.46.
3. El mes con mayor ganancia en promedio fue marzo, mientras que el mes con peores ganancias promedio fue abril.
4. El segmento de clientes que mas ganancias genero fue el de Consumer y el que menos genero fue el de Home Office.
5. El tramo de descuento de 10% a 20% es el ultimo en generar un margen de ganancia positivo, un mayor descuento produce ganancias negativas (perdidas).

### Sugerencias

1. Optimizar la estrategia de descuentos. No recurrir a descuentos tan altos puesto que genera perdida.
2. Aprovechar el exito del segmento Consumer. Al ser el segmento mas rentable recomiendo poner enfasis sobre el mismo.
3. Fortalecer presencia en California dada la rentabilidad en este estado.
4. Replantear la estrategia del negocio en el estado de Ohio
   
## Power Bi

Para representar gráficamente el estado del negocio, desarrollé un dashboard interactivo en Power BI. En él se pueden visualizar los productos con mayores ventas, la evolución de las ventas y ganancias a lo largo del tiempo, y las ganancias por estado. Además, se analiza el desempeño por categoría de productos y se incluyen KPIs clave como: Margen de ventas, días promedio en realizar el envío, ganancias totales y ventas totales

El dashboard también incorpora slicers que permiten filtrar fácilmente la información por región, estado y año, brindando una visión más dinámica y personalizada del negocio.

- [📄 Dashboard](./Superstore.pdf) : Archivo pdf para visualizacion.
- [📊 Superstore.pbix](./Superstore.pbix): Archivo de Power BI para edición.
   


