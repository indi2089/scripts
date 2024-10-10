# scripts
-- Tabla de Marcas
CREATE TABLE tienda_ropa_mujer_marca (
    id_marca INT AUTO_INCREMENT PRIMARY KEY,  -- id autoincrementable para las marcas
    nombre VARCHAR(50) NOT NULL UNIQUE        -- Nombre único de la marca
);
 
-- Tabla de Prendas
CREATE TABLE tienda_ropa_mujer_prenda (
    id_prenda INT AUTO_INCREMENT PRIMARY KEY,       -- id autoincrementable para las prendas
    nombre VARCHAR(50) NOT NULL,                    -- Nombre de la prenda
    id_marca INT,                                   -- Clave foránea que refiere a la tabla de marcas
    precio DECIMAL(10, 2) NOT NULL CHECK (precio >= 0), -- Precio de la prenda, con 2 decimales
    FOREIGN KEY (id_marca) REFERENCES tienda_ropa_mujer_marca(id_marca) -- Relación con la tabla de marcas
);
 
-- Tabla de Ventas
CREATE TABLE tienda_ropa_mujer_ventas (
    id_venta INT AUTO_INCREMENT PRIMARY KEY,        -- id autoincrementable para las ventas
    id_prenda INT,                                  -- Clave foránea que refiere a la tabla de prendas
    cantidad INT NOT NULL CHECK (cantidad > 0),     -- Cantidad vendida, debe ser mayor a 0
    fecha_venta DATE NOT NULL,                      -- Fecha de la venta
    precio DECIMAL(10, 2) NOT NULL CHECK (precio >= 0), -- Precio al que se vendió la prenda
    FOREIGN KEY (id_prenda) REFERENCES tienda_ropa_mujer_prenda(id_prenda) -- Relación con la tabla de prendas
);
INSERT INTO tienda_ropa_mujer_marca (nombre)
VALUES ('Marca A'), ('Marca B'), ('Marca C');
INSERT INTO tienda_ropa_mujer_ventas (id_prenda, cantidad, fecha_venta, precio)
VALUES (1, 2, '2024-10-01', 15000.00), (2, 1, '2024-10-02', 25000.00), (3, 3, '2024-10-03', 20000.00);
UPDATE tienda_ropa_mujer_prenda
SET precio = 18000.00
WHERE nombre = 'Blusa';
DELETE FROM tienda_ropa_mujer_ventas
WHERE id_venta = 1;

----insert----
INSERT INTO tienda_ropa_mujer_prenda (nombre, id_marca, precio) 
VALUES ('Blusa', 1, 15000.00), ('Pantalón', 2, 25000.00), ('Falda', 3, 20000.00);
 ------------------------------------------------------------------------------------
 ----select---------------
 SELECT * FROM tienda_ropa_mujer_marca;
 SELECT p.id_prenda, p.nombre AS prenda, m.nombre AS marca, p.precio
FROM tienda_ropa_mujer_prenda p
JOIN tienda_ropa_mujer_marca m ON p.id_marca = m.id_marca;
 -----------------------------------------------------------
 --------update-----
 UPDATE tienda_ropa_mujer_prenda
SET precio = 18000.00
WHERE nombre = 'Blusa';
----------------------------------------------------------------
-----delete-----
DELETE FROM tienda_ropa_mujer_ventas
WHERE id_venta = 1;
 -------------------------------------------
------------- vistas
 obtener lista de marcas que tienen menos venta----
 CREATE VIEW marcas_con_ventas AS
SELECT DISTINCT
    m.nombre AS marca
FROM
    tienda_ropa_mujer_marca m
JOIN
    tienda_ropa_mujer_prenda p ON m.id_marca = p.id_marca
JOIN
    tienda_ropa_mujer_ventas v ON p.id_prenda = v.id_prenda;

--------------------------------------------------------
obtener prendas vendidas y en stock----
CREATE VIEW prendas_stock_restante AS
SELECT
    p.nombre AS prenda,
    (s.cantidad_inicial - COALESCE(SUM(v.cantidad), 0)) AS cantidad_restante
FROM
    tienda_ropa_mujer_prenda p
JOIN
    tienda_ropa_mujer_stock s ON p.id_prenda = s.id_prenda
LEFT JOIN
    tienda_ropa_mujer_ventas v ON p.id_prenda = v.id_prenda
GROUP BY
    p.id_prenda, p.nombre, s.cantidad_inicial;
    ---------------------------------
obterner lista de prendas mas vendidas-----
CREATE VIEW top_5_marcas_mas_vendidas AS
SELECT
    m.nombre AS marca,
    SUM(v.cantidad) AS total_ventas
FROM
    tienda_ropa_mujer_marca m
JOIN
    tienda_ropa_mujer_prenda p ON m.id_marca = p.id_marca
JOIN
    tienda_ropa_mujer_ventas v ON p.id_prenda = v.id_prenda
GROUP BY
    m.id_marca, m.nombre
ORDER BY
    total_ventas DESC
LIMIT 5;
--------------------------------------------------------


-----obtener la cantidad de prendas mas vendidas----
SELECT
    fecha_venta,
    SUM(cantidad) AS total_prendas_vendidas
FROM
    tienda_ropa_mujer_ventas
WHERE
    fecha_venta = '2024-10-01'  -- Cambia la fecha según lo que necesites
GROUP BY
    fecha_venta;
-----------------------------------------------------------------------------------

1. Tabla `tienda_ropa_mujer_marca` (Entidad: Marca)
- Esta tabla representa las marcas de las prendas de ropa. Cada marca es única y se identifica mediante un **ID**.
- Atributos:
  - `id_marca` (clave primaria): Es un identificador único para cada marca.
  - `nombre`: El nombre de la marca (como Nike, Adidas, etc.).
- Relación:
  - Una marca puede tener varias prendas asociadas (relación uno a muchos con la tabla `tienda_ropa_mujer_prenda`).
 
 2. Tabla `tienda_ropa_mujer_prenda` (Entidad: Prenda)
- Esta tabla representa las **prendas** disponibles en la tienda. Cada prenda tiene su propio nombre, un precio y está asociada a una marca.
- Atributos:
  - `id_prenda` (clave primaria): Es un identificador único para cada prenda.
  - `nombre`: El nombre de la prenda (por ejemplo, "Blusa", "Pantalón").
  - `id_marca` (clave foránea): Relaciona cada prenda con una marca específica (relación con `tienda_ropa_mujer_marca`).
  - `precio`: Precio de la prenda.
- Relación:
  - Cada prenda pertenece a una marca. Es decir, hay una **relación uno a muchos** entre **marca** y **prenda**: una marca puede tener varias prendas, pero una prenda solo puede pertenecer a una marca.
  - Cada prenda puede aparecer en muchas ventas (relación uno a muchos con la tabla `tienda_ropa_mujer_ventas`).
 
3. Tabla `tienda_ropa_mujer_ventas` (Entidad: Venta)
- Esta tabla registra las **ventas** realizadas en la tienda. Cada registro de venta contiene información sobre qué prenda se vendió, la cantidad, la fecha de la venta y el precio de venta.
- Atributos:
  - `id_venta` (clave primaria): Identificador único para cada venta.
  - `id_prenda` (clave foránea): Relaciona la venta con la prenda que se vendió (relación con `tienda_ropa_mujer_prenda`).
  - `cantidad`: Cantidad de prendas vendidas en esta venta.
  - `fecha_venta`: Fecha en la que se realizó la venta.
  - `precio`: Precio al que se vendió la prenda.
- Relación:
  - Cada venta está asociada a una prenda específica, lo que significa que hay una relación uno a muchos entre prenda y ventas: una prenda puede aparecer en varias ventas, pero cada venta registra una prenda específica.
 
 Relaciones entre las tablas:
 
- Relación entre Marca y Prenda:
  - Una marca puede tener varias prendas, pero una prenda solo puede pertenecer a una marca. En el modelo E-R, esta es una **relación uno a muchos**. En la base de datos, se implementa mediante la clave foránea `id_marca` en la tabla `tienda_ropa_mujer_prenda`.
 
- Relación entre Prenda y Ventas:
  - Una prenda puede aparecer en muchas ventas, pero cada venta hace referencia a una única prenda. Esta es otra **relación uno a muchos**. En la base de datos, se implementa mediante la clave foránea `id_prenda` en la tabla `tienda_ropa_mujer_ventas`.
 
 Diagrama básico del modelo E-R:
 
```
tienda_ropa_mujer_marca             tienda_ropa_mujer_prenda           tienda_ropa_mujer_ventas
  +-------------------+             +-----------------------+           +--------------------+
  | id_marca (PK)      |<-----------| id_prenda (PK)         |<----------| id_venta (PK)       |
  | nombre             |             | nombre                |           | id_prenda (FK)      |
  +-------------------+             | id_marca (FK)          |           | cantidad            |
                                     | precio                 |           | fecha_venta         |
                                     +-----------------------+           | precio              |
                                                                          +--------------------+
```
 
- La tabla `tienda_ropa_mujer_prenda` está relacionada con la tabla `tienda_ropa_mujer_marca` a través del campo `id_marca`, formando una relación uno a muchos.
- La tabla `tienda_ropa_mujer_ventas` está relacionada con la tabla `tienda_ropa_mujer_prenda` a través del campo `id_prenda`, también con una relación uno a muchos.
