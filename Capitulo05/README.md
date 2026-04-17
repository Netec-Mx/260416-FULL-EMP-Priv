# Laboratorio 5. Proyecto Integrador — Capa de Persistencia Completa

## Metadatos

| Propiedad | Valor |
|-----------|-------|
| **Duración** | 120 minutos |
| **Complejidad** | Intermedio |
| **Nivel Bloom** | Crear |
| **Tecnologías** | PostgreSQL 16.x, MongoDB 7.0, Spring Boot 3.2.x, Spring Data JPA, Spring Data MongoDB |

## Descripción General

En este laboratorio implementarás la capa de persistencia completa del proyecto integrador empresarial, trabajando simultáneamente con dos paradigmas de almacenamiento: el modelo relacional en PostgreSQL y el modelo documental en MongoDB. Diseñarás un esquema relacional normalizado con mínimo 5 tablas relacionadas, ejecutarás consultas SQL de complejidad creciente incluyendo JOINs, subconsultas y window functions, e implementarás transacciones explícitas con control ACID. En la parte NoSQL, modelarás colecciones MongoDB para datos no estructurados, ejecutarás operaciones CRUD con operadores avanzados y construirás pipelines de agregación. Al finalizar, integrarás ambas bases de datos en el proyecto Spring Boot demostrando la coexistencia de Spring Data JPA y Spring Data MongoDB en una misma aplicación.

Este laboratorio refleja un escenario real de arquitectura empresarial donde los sistemas modernos combinan bases de datos relacionales para transacciones críticas con bases de datos documentales para datos flexibles y logs de actividad.

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Diseñar e implementar un esquema relacional normalizado en PostgreSQL con claves primarias, claves foráneas, índices y constraints de integridad
- [ ] Escribir consultas SQL avanzadas incluyendo JOINs múltiples, subconsultas correlacionadas, funciones de agregación y window functions
- [ ] Implementar transacciones explícitas en PostgreSQL aplicando los principios ACID para operaciones críticas del negocio
- [ ] Modelar y crear colecciones MongoDB con documentos embebidos y referencias para datos no estructurados
- [ ] Ejecutar operaciones CRUD completas en MongoDB usando operadores de consulta, actualización y agregación avanzada
- [ ] Integrar Spring Data JPA y Spring Data MongoDB en un mismo proyecto Spring Boot con configuración dual de fuentes de datos

## Prerrequisitos

### Conocimientos Requeridos

- Comprensión de tablas, filas, columnas, claves primarias y claves foráneas (Lección 5.1)
- Conocimientos básicos de SQL: SELECT, INSERT, UPDATE, DELETE
- Familiaridad con el modelo de documentos JSON y colecciones MongoDB
- Experiencia con Spring Boot y configuración de dependencias Maven (Labs anteriores)
- Comprensión de inyección de dependencias y anotaciones de Spring

### Acceso Requerido

- PostgreSQL 16.x instalado y en ejecución (local o via Docker)
- MongoDB 7.0 Community instalado y en ejecución (local o via Docker)
- pgAdmin 4 o DBeaver para administración visual de PostgreSQL
- MongoDB Compass para administración visual de MongoDB
- IntelliJ IDEA con el proyecto Spring Boot del Laboratorio 4 como base
- Acceso a terminal/consola con permisos de administración

## Entorno de Laboratorio

### Requisitos de Hardware

| Componente | Especificación |
|------------|----------------|
| Procesador | Intel Core i5 8va gen o superior / AMD Ryzen 5 |
| Memoria RAM | Mínimo 16 GB DDR4 (recomendado 32 GB) |
| Almacenamiento | Mínimo 20 GB libres en SSD |
| Pantalla | Resolución mínima 1920x1080 |

### Requisitos de Software

| Software | Versión | Propósito |
|----------|---------|-----------|
| PostgreSQL | 16.x | Base de datos relacional principal |
| pgAdmin 4 | 8.x | Administración visual de PostgreSQL |
| MongoDB | 7.0 Community | Base de datos documental |
| MongoDB Compass | 1.42+ | Administración visual de MongoDB |
| Java JDK | 17 LTS | Entorno de ejecución Java |
| Spring Boot | 3.2.x | Framework backend |
| IntelliJ IDEA | 2024.1 | IDE principal |
| Maven | 3.9.x | Gestión de dependencias |
| Docker Desktop | 4.29+ | Alternativa para infraestructura |

### Configuración Inicial

Si prefieres usar Docker para los servicios de infraestructura (opción recomendada para evitar conflictos de instalación), ejecuta los siguientes comandos antes de comenzar:

```bash
# Crear red Docker para el proyecto integrador
docker network create proyecto-integrador-net

# Iniciar PostgreSQL 16 con Docker
docker run -d \
  --name postgres-integrador \
  --network proyecto-integrador-net \
  -e POSTGRES_DB=tienda_db \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=admin123 \
  -p 5432:5432 \
  postgres:16-alpine

# Iniciar MongoDB 7.0 con Docker
docker run -d \
  --name mongo-integrador \
  --network proyecto-integrador-net \
  -e MONGO_INITDB_DATABASE=tienda_logs \
  -p 27017:27017 \
  mongo:7.0

# Verificar que ambos contenedores están en ejecución
docker ps --filter "name=postgres-integrador" --filter "name=mongo-integrador"
```

**Para Windows (PowerShell):**
```powershell
# Crear red Docker
docker network create proyecto-integrador-net

# Iniciar PostgreSQL
docker run -d `
  --name postgres-integrador `
  --network proyecto-integrador-net `
  -e POSTGRES_DB=tienda_db `
  -e POSTGRES_USER=admin `
  -e POSTGRES_PASSWORD=admin123 `
  -p 5432:5432 `
  postgres:16-alpine

# Iniciar MongoDB
docker run -d `
  --name mongo-integrador `
  --network proyecto-integrador-net `
  -e MONGO_INITDB_DATABASE=tienda_logs `
  -p 27017:27017 `
  mongo:7.0
```

> ⚠️ **Nota:** Si ya tienes PostgreSQL y MongoDB instalados localmente, asegúrate de que los puertos 5432 y 27017 estén disponibles. Ajusta las credenciales según tu configuración local en los pasos siguientes.

## Instrucciones Paso a Paso

### Paso 1: Diseño e Implementación del Esquema Relacional en PostgreSQL

**Objetivo:** Crear el esquema completo de la base de datos relacional con 5 tablas normalizadas, aplicando claves primarias, claves foráneas, índices y constraints de integridad para el dominio de una tienda en línea.

**Instrucciones:**

1. Abre pgAdmin 4 y conéctate a tu instancia de PostgreSQL. Si usas Docker, crea una nueva conexión con los siguientes datos:
   - Host: `localhost`
   - Puerto: `5432`
   - Base de datos: `tienda_db`
   - Usuario: `admin`
   - Contraseña: `admin123`

2. Abre el Query Tool en pgAdmin (clic derecho sobre `tienda_db` → Query Tool) y ejecuta el siguiente script DDL completo. Este script crea el esquema normalizado aplicando los principios de la Lección 5.1:

```sql
-- ============================================================
-- ESQUEMA RELACIONAL: TIENDA EN LÍNEA
-- Aplicando 3FN: sin redundancias, sin dependencias transitivas
-- ============================================================

-- Eliminar tablas si existen (para reintentos limpios)
DROP TABLE IF EXISTS detalle_pedidos CASCADE;
DROP TABLE IF EXISTS pedidos CASCADE;
DROP TABLE IF EXISTS productos CASCADE;
DROP TABLE IF EXISTS categorias CASCADE;
DROP TABLE IF EXISTS clientes CASCADE;

-- ============================================================
-- TABLA 1: clientes
-- Clave primaria: cliente_id (SERIAL = autoincremental)
-- Constraints: email único, nombre obligatorio
-- ============================================================
CREATE TABLE clientes (
    cliente_id      SERIAL          PRIMARY KEY,
    nombre          VARCHAR(100)    NOT NULL,
    email           VARCHAR(150)    NOT NULL UNIQUE,
    telefono        VARCHAR(20),
    ciudad          VARCHAR(80),
    fecha_registro  DATE            NOT NULL DEFAULT CURRENT_DATE,
    activo          BOOLEAN         NOT NULL DEFAULT TRUE,
    CONSTRAINT chk_email_formato CHECK (email LIKE '%@%.%')
);

-- Índice para búsquedas frecuentes por ciudad
CREATE INDEX idx_clientes_ciudad ON clientes(ciudad);
CREATE INDEX idx_clientes_activo ON clientes(activo);

-- ============================================================
-- TABLA 2: categorias
-- Clave primaria: categoria_id
-- Permite jerarquía: categoria_padre_id referencia a sí misma
-- ============================================================
CREATE TABLE categorias (
    categoria_id    SERIAL          PRIMARY KEY,
    nombre          VARCHAR(80)     NOT NULL UNIQUE,
    descripcion     TEXT,
    categoria_padre_id INT,
    FOREIGN KEY (categoria_padre_id) REFERENCES categorias(categoria_id)
);

-- ============================================================
-- TABLA 3: productos
-- Clave primaria: producto_id
-- Clave foránea: categoria_id → categorias
-- Constraints: precio > 0, stock >= 0
-- ============================================================
CREATE TABLE productos (
    producto_id     SERIAL          PRIMARY KEY,
    nombre          VARCHAR(150)    NOT NULL,
    descripcion     TEXT,
    precio          DECIMAL(10,2)   NOT NULL,
    stock           INT             NOT NULL DEFAULT 0,
    categoria_id    INT             NOT NULL,
    sku             VARCHAR(50)     UNIQUE,
    fecha_creacion  TIMESTAMP       NOT NULL DEFAULT NOW(),
    CONSTRAINT chk_precio_positivo CHECK (precio > 0),
    CONSTRAINT chk_stock_no_negativo CHECK (stock >= 0),
    FOREIGN KEY (categoria_id) REFERENCES categorias(categoria_id)
);

-- Índices para búsquedas frecuentes
CREATE INDEX idx_productos_categoria ON productos(categoria_id);
CREATE INDEX idx_productos_precio ON productos(precio);
CREATE INDEX idx_productos_nombre ON productos USING gin(to_tsvector('spanish', nombre));

-- ============================================================
-- TABLA 4: pedidos
-- Clave primaria: pedido_id
-- Clave foránea: cliente_id → clientes
-- Estado controlado por CHECK constraint
-- ============================================================
CREATE TABLE pedidos (
    pedido_id       SERIAL          PRIMARY KEY,
    cliente_id      INT             NOT NULL,
    fecha_pedido    TIMESTAMP       NOT NULL DEFAULT NOW(),
    fecha_entrega   DATE,
    estado          VARCHAR(20)     NOT NULL DEFAULT 'PENDIENTE',
    total           DECIMAL(10,2),
    direccion_envio VARCHAR(250),
    CONSTRAINT chk_estado_valido CHECK (
        estado IN ('PENDIENTE', 'CONFIRMADO', 'ENVIADO', 'ENTREGADO', 'CANCELADO')
    ),
    CONSTRAINT chk_fecha_entrega CHECK (
        fecha_entrega IS NULL OR fecha_entrega >= fecha_pedido::DATE
    ),
    FOREIGN KEY (cliente_id) REFERENCES clientes(cliente_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);

-- Índices para reportes frecuentes
CREATE INDEX idx_pedidos_cliente ON pedidos(cliente_id);
CREATE INDEX idx_pedidos_estado ON pedidos(estado);
CREATE INDEX idx_pedidos_fecha ON pedidos(fecha_pedido DESC);

-- ============================================================
-- TABLA 5: detalle_pedidos
-- Clave primaria compuesta: (pedido_id, producto_id)
-- Claves foráneas: pedido_id → pedidos, producto_id → productos
-- Representa relación N:M entre pedidos y productos
-- ============================================================
CREATE TABLE detalle_pedidos (
    pedido_id       INT             NOT NULL,
    producto_id     INT             NOT NULL,
    cantidad        INT             NOT NULL,
    precio_unitario DECIMAL(10,2)   NOT NULL,
    descuento       DECIMAL(5,2)    NOT NULL DEFAULT 0,
    PRIMARY KEY (pedido_id, producto_id),
    CONSTRAINT chk_cantidad_positiva CHECK (cantidad > 0),
    CONSTRAINT chk_precio_unitario_positivo CHECK (precio_unitario > 0),
    CONSTRAINT chk_descuento_valido CHECK (descuento >= 0 AND descuento <= 100),
    FOREIGN KEY (pedido_id) REFERENCES pedidos(pedido_id)
        ON DELETE CASCADE,
    FOREIGN KEY (producto_id) REFERENCES productos(producto_id)
        ON DELETE RESTRICT
);

-- ============================================================
-- COMENTARIOS DE DOCUMENTACIÓN EN EL ESQUEMA
-- ============================================================
COMMENT ON TABLE clientes IS 'Almacena información de clientes registrados en la tienda';
COMMENT ON TABLE categorias IS 'Categorías jerárquicas de productos';
COMMENT ON TABLE productos IS 'Catálogo de productos disponibles en la tienda';
COMMENT ON TABLE pedidos IS 'Órdenes de compra realizadas por los clientes';
COMMENT ON TABLE detalle_pedidos IS 'Líneas de detalle de cada pedido (relación N:M pedidos-productos)';
```

3. Verifica que las 5 tablas se crearon correctamente ejecutando:

```sql
-- Verificar tablas creadas
SELECT 
    table_name,
    (SELECT COUNT(*) FROM information_schema.columns 
     WHERE table_name = t.table_name 
     AND table_schema = 'public') AS num_columnas
FROM information_schema.tables t
WHERE table_schema = 'public'
ORDER BY table_name;

-- Verificar constraints de integridad referencial
SELECT
    tc.table_name AS tabla_origen,
    kcu.column_name AS columna_fk,
    ccu.table_name AS tabla_referenciada,
    ccu.column_name AS columna_pk_referenciada
FROM information_schema.table_constraints AS tc
JOIN information_schema.key_column_usage AS kcu
    ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage AS ccu
    ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
ORDER BY tc.table_name;
```

**Salida Esperada:**

```
 table_name       | num_columnas
------------------+--------------
 categorias       | 4
 clientes         | 7
 detalle_pedidos  | 5
 pedidos          | 8
 productos        | 8
(5 filas)

 tabla_origen    | columna_fk        | tabla_referenciada | columna_pk_referenciada
-----------------+-------------------+--------------------+------------------------
 categorias      | categoria_padre_id| categorias         | categoria_id
 detalle_pedidos | pedido_id         | pedidos            | pedido_id
 detalle_pedidos | producto_id       | productos          | producto_id
 pedidos         | cliente_id        | clientes           | cliente_id
 productos       | categoria_id      | categorias         | categoria_id
(5 filas)
```

**Verificación:**

- Confirma que aparecen exactamente 5 tablas en el resultado
- Verifica que las 5 relaciones de clave foránea están correctamente definidas
- En pgAdmin, expande el nodo `Tables` en el panel izquierdo y confirma que todas las tablas tienen el ícono de llave junto a sus columnas PK

---

### Paso 2: Poblar la Base de Datos con Datos de Prueba

**Objetivo:** Insertar un conjunto representativo de datos de prueba en todas las tablas para poder ejecutar consultas significativas en los pasos siguientes.

**Instrucciones:**

1. En el Query Tool de pgAdmin, ejecuta el siguiente script DML para insertar datos de prueba:

```sql
-- ============================================================
-- DATOS DE PRUEBA: CATEGORÍAS
-- ============================================================
INSERT INTO categorias (nombre, descripcion, categoria_padre_id) VALUES
('Electrónica',      'Dispositivos y equipos electrónicos', NULL),
('Computación',      'Computadores, laptops y accesorios',  1),
('Telefonía',        'Smartphones y accesorios móviles',    1),
('Periféricos',      'Teclados, ratones, monitores',        2),
('Ropa',             'Prendas de vestir para todas las edades', NULL),
('Ropa Masculina',   'Prendas para hombres',                5),
('Ropa Femenina',    'Prendas para mujeres',                5),
('Hogar',            'Artículos para el hogar',             NULL),
('Cocina',           'Utensilios y electrodomésticos de cocina', 8);

-- ============================================================
-- DATOS DE PRUEBA: CLIENTES
-- ============================================================
INSERT INTO clientes (nombre, email, telefono, ciudad, fecha_registro) VALUES
('Ana Torres',        'ana.torres@email.com',    '3001234567', 'Bogotá',     '2023-01-15'),
('Carlos Mendoza',    'carlos.m@email.com',      '3109876543', 'Medellín',   '2023-02-20'),
('Laura Jiménez',     'laura.j@email.com',       '3205551234', 'Cali',       '2023-03-10'),
('Pedro Ramírez',     'pedro.r@email.com',       '3154443322', 'Bogotá',     '2023-04-05'),
('María González',    'maria.g@email.com',       '3006667788', 'Barranquilla','2023-05-18'),
('Luis Hernández',    'luis.h@email.com',        '3178889900', 'Medellín',   '2023-06-22'),
('Sofía Castro',      'sofia.c@email.com',       '3021112233', 'Bogotá',     '2023-07-30'),
('Andrés Morales',    'andres.m@email.com',       '3134445566', 'Cartagena',  '2023-08-14'),
('Valentina Ruiz',    'valentina.r@email.com',   '3067778899', 'Cali',       '2023-09-25'),
('Diego Vargas',      'diego.v@email.com',       '3193334455', 'Bogotá',     '2023-10-11');

-- ============================================================
-- DATOS DE PRUEBA: PRODUCTOS
-- ============================================================
INSERT INTO productos (nombre, descripcion, precio, stock, categoria_id, sku) VALUES
('Laptop ProBook 450',    'Laptop empresarial 15.6", Intel i7, 16GB RAM', 2850000.00, 25, 2, 'LAP-001'),
('Smartphone Galaxy S24', 'Teléfono Android 5G, 256GB, cámara 200MP',    3200000.00, 40, 3, 'PHO-001'),
('Teclado Mecánico RGB',  'Teclado gaming con switches Cherry MX',        450000.00,  60, 4, 'KEY-001'),
('Monitor UltraWide 34"', 'Monitor curvo 34", 3440x1440, 144Hz',         1800000.00, 15, 4, 'MON-001'),
('Mouse Inalámbrico Pro', 'Mouse ergonómico, 4000 DPI, batería 90 días',  180000.00, 100, 4, 'MOU-001'),
('Camisa Oxford Azul',    'Camisa formal algodón 100%, talla M',           89000.00, 200, 6, 'CAM-001'),
('Vestido Floral Verano', 'Vestido casual flores, talla S/M/L',            75000.00, 150, 7, 'VES-001'),
('Sartén Antiadherente',  'Sartén 28cm, recubrimiento cerámico',           65000.00,  80, 9, 'SAR-001'),
('Tablet Android 11"',    'Tablet 11", 128GB, WiFi + 4G LTE',            950000.00,  35, 2, 'TAB-001'),
('Auriculares Bluetooth', 'Auriculares over-ear, cancelación de ruido',   320000.00,  55, 1, 'AUR-001'),
('Cargador USB-C 65W',    'Cargador rápido universal USB-C',               45000.00, 200, 3, 'CAR-001'),
('Pantalón Chino Beige',  'Pantalón casual slim fit, talla 32',            95000.00, 120, 6, 'PAN-001');

-- ============================================================
-- DATOS DE PRUEBA: PEDIDOS
-- ============================================================
INSERT INTO pedidos (cliente_id, fecha_pedido, fecha_entrega, estado, total, direccion_envio) VALUES
(1,  '2024-01-10 09:30:00', '2024-01-15', 'ENTREGADO',  3300000.00, 'Cra 7 #45-12, Bogotá'),
(2,  '2024-01-12 14:20:00', '2024-01-18', 'ENTREGADO',  2850000.00, 'Av El Poblado #12-34, Medellín'),
(1,  '2024-02-05 11:00:00', '2024-02-10', 'ENTREGADO',   630000.00, 'Cra 7 #45-12, Bogotá'),
(3,  '2024-02-14 16:45:00', '2024-02-20', 'ENVIADO',    1800000.00, 'Cll 10 #22-45, Cali'),
(4,  '2024-02-20 10:15:00', '2024-02-26', 'CONFIRMADO',  500000.00, 'Tv 42 #67-89, Bogotá'),
(5,  '2024-03-01 08:00:00', '2024-03-07', 'ENTREGADO',  3520000.00, 'Cll 72 #43-21, Barranquilla'),
(6,  '2024-03-10 13:30:00', '2024-03-16', 'ENTREGADO',   225000.00, 'Cll 33 #76-54, Medellín'),
(7,  '2024-03-15 15:00:00', '2024-03-21', 'CANCELADO',  4650000.00, 'Cra 15 #88-12, Bogotá'),
(2,  '2024-03-20 09:45:00', '2024-03-26', 'ENVIADO',     950000.00, 'Av El Poblado #12-34, Medellín'),
(8,  '2024-04-02 11:20:00', '2024-04-08', 'PENDIENTE',   320000.00, 'Cll 30 #15-67, Cartagena'),
(9,  '2024-04-05 14:00:00', '2024-04-11', 'CONFIRMADO', 1270000.00, 'Av Roosevelt #44-22, Cali'),
(10, '2024-04-10 10:30:00', '2024-04-16', 'PENDIENTE',  2895000.00, 'Cra 11 #55-33, Bogotá');

-- ============================================================
-- DATOS DE PRUEBA: DETALLE_PEDIDOS
-- ============================================================
INSERT INTO detalle_pedidos (pedido_id, producto_id, cantidad, precio_unitario, descuento) VALUES
-- Pedido 1: Smartphone + Auriculares
(1,  2,  1, 3200000.00, 0),
(1,  10, 1,  320000.00, 10),
-- Pedido 2: Laptop
(2,  1,  1, 2850000.00, 0),
-- Pedido 3: Teclado + Mouse
(3,  3,  1,  450000.00, 0),
(3,  5,  1,  180000.00, 0),
-- Pedido 4: Monitor
(4,  4,  1, 1800000.00, 0),
-- Pedido 5: Teclado + Camisa
(5,  3,  1,  450000.00, 5),
(5,  6,  1,   89000.00, 0),
-- Pedido 6: Smartphone + Laptop
(6,  2,  1, 3200000.00, 5),
(6,  1,  1, 2850000.00, 10),
-- Pedido 7: Mouse + Auriculares
(7,  5,  2,  180000.00, 0),
(7,  10, 1,  320000.00, 0),
-- Pedido 8: Laptop + Monitor (CANCELADO)
(8,  1,  1, 2850000.00, 0),
(8,  4,  1, 1800000.00, 0),
-- Pedido 9: Tablet
(9,  9,  1,  950000.00, 0),
-- Pedido 10: Auriculares
(10, 10, 1,  320000.00, 0),
-- Pedido 11: Tablet + Cargador + Vestido
(11, 9,  1,  950000.00, 0),
(11, 11, 2,   45000.00, 0),
(11, 7,  3,   75000.00, 10),
-- Pedido 12: Laptop + Mouse
(12, 1,  1, 2850000.00, 0),
(12, 5,  1,  180000.00, 0);
```

2. Verifica la inserción con conteos rápidos:

```sql
SELECT 'clientes'       AS tabla, COUNT(*) AS registros FROM clientes
UNION ALL
SELECT 'categorias',    COUNT(*) FROM categorias
UNION ALL
SELECT 'productos',     COUNT(*) FROM productos
UNION ALL
SELECT 'pedidos',       COUNT(*) FROM pedidos
UNION ALL
SELECT 'detalle_pedidos', COUNT(*) FROM detalle_pedidos
ORDER BY tabla;
```

**Salida Esperada:**

```
 tabla            | registros
------------------+-----------
 categorias       | 9
 clientes         | 10
 detalle_pedidos  | 21
 pedidos          | 12
 productos        | 12
(5 filas)
```

**Verificación:**

- Todos los conteos deben coincidir con los valores de la tabla anterior
- En pgAdmin, haz clic derecho sobre cualquier tabla → "View/Edit Data" → "All Rows" para confirmar visualmente los datos

---

### Paso 3: Consultas SQL de Complejidad Creciente

**Objetivo:** Ejecutar 10 consultas SQL de complejidad creciente que demuestren el poder del modelo relacional: desde SELECTs básicos hasta window functions y reportes complejos.

**Instrucciones:**

1. Ejecuta las siguientes consultas en orden. Cada una introduce un concepto nuevo:

```sql
-- ============================================================
-- CONSULTA 1: SELECT básico con filtro
-- Objetivo: Listar clientes activos de Bogotá
-- ============================================================
SELECT 
    cliente_id,
    nombre,
    email,
    fecha_registro
FROM clientes
WHERE ciudad = 'Bogotá' 
  AND activo = TRUE
ORDER BY fecha_registro DESC;
```

```sql
-- ============================================================
-- CONSULTA 2: INNER JOIN - Pedidos con datos del cliente
-- Objetivo: Ver pedidos con nombre del cliente (no solo el ID)
-- ============================================================
SELECT 
    p.pedido_id,
    c.nombre        AS cliente,
    c.ciudad,
    p.fecha_pedido,
    p.estado,
    p.total
FROM pedidos p
INNER JOIN clientes c ON p.cliente_id = c.cliente_id
ORDER BY p.fecha_pedido DESC;
```

```sql
-- ============================================================
-- CONSULTA 3: LEFT JOIN - Clientes con o sin pedidos
-- Objetivo: Identificar clientes que nunca han comprado
-- ============================================================
SELECT 
    c.cliente_id,
    c.nombre,
    c.email,
    COUNT(p.pedido_id) AS total_pedidos
FROM clientes c
LEFT JOIN pedidos p ON c.cliente_id = p.cliente_id
GROUP BY c.cliente_id, c.nombre, c.email
ORDER BY total_pedidos ASC;
```

```sql
-- ============================================================
-- CONSULTA 4: JOIN múltiple (3 tablas) con agregación
-- Objetivo: Productos más vendidos con su categoría
-- ============================================================
SELECT 
    pr.nombre           AS producto,
    cat.nombre          AS categoria,
    SUM(dp.cantidad)    AS unidades_vendidas,
    SUM(dp.cantidad * dp.precio_unitario * (1 - dp.descuento/100)) AS ingresos_netos
FROM detalle_pedidos dp
INNER JOIN productos pr  ON dp.producto_id = pr.producto_id
INNER JOIN categorias cat ON pr.categoria_id = cat.categoria_id
INNER JOIN pedidos ped   ON dp.pedido_id = ped.pedido_id
WHERE ped.estado != 'CANCELADO'
GROUP BY pr.producto_id, pr.nombre, cat.nombre
ORDER BY unidades_vendidas DESC, ingresos_netos DESC;
```

```sql
-- ============================================================
-- CONSULTA 5: GROUP BY con HAVING
-- Objetivo: Clientes con más de 1 pedido y gasto total > 1,000,000
-- ============================================================
SELECT 
    c.nombre,
    c.ciudad,
    COUNT(p.pedido_id)          AS num_pedidos,
    SUM(p.total)                AS gasto_total,
    AVG(p.total)                AS ticket_promedio,
    MAX(p.fecha_pedido)         AS ultimo_pedido
FROM clientes c
INNER JOIN pedidos p ON c.cliente_id = p.cliente_id
WHERE p.estado != 'CANCELADO'
GROUP BY c.cliente_id, c.nombre, c.ciudad
HAVING COUNT(p.pedido_id) > 1 
   AND SUM(p.total) > 1000000
ORDER BY gasto_total DESC;
```

```sql
-- ============================================================
-- CONSULTA 6: Subconsulta en WHERE (correlacionada)
-- Objetivo: Productos con precio mayor al promedio de su categoría
-- ============================================================
SELECT 
    p.nombre            AS producto,
    p.precio,
    cat.nombre          AS categoria,
    (SELECT AVG(p2.precio) 
     FROM productos p2 
     WHERE p2.categoria_id = p.categoria_id) AS precio_promedio_categoria
FROM productos p
INNER JOIN categorias cat ON p.categoria_id = cat.categoria_id
WHERE p.precio > (
    SELECT AVG(p3.precio) 
    FROM productos p3 
    WHERE p3.categoria_id = p.categoria_id
)
ORDER BY cat.nombre, p.precio DESC;
```

```sql
-- ============================================================
-- CONSULTA 7: Subconsulta en FROM (tabla derivada)
-- Objetivo: Resumen de ventas por ciudad usando subconsulta
-- ============================================================
SELECT 
    resumen.ciudad,
    resumen.total_pedidos,
    resumen.ingresos_totales,
    ROUND(resumen.ingresos_totales / resumen.total_pedidos, 0) AS ticket_promedio
FROM (
    SELECT 
        c.ciudad,
        COUNT(p.pedido_id)  AS total_pedidos,
        SUM(p.total)        AS ingresos_totales
    FROM clientes c
    INNER JOIN pedidos p ON c.cliente_id = p.cliente_id
    WHERE p.estado NOT IN ('CANCELADO', 'PENDIENTE')
    GROUP BY c.ciudad
) AS resumen
WHERE resumen.total_pedidos >= 1
ORDER BY resumen.ingresos_totales DESC;
```

```sql
-- ============================================================
-- CONSULTA 8: Window Function - ROW_NUMBER y RANK
-- Objetivo: Ranking de clientes por gasto total usando RANK()
-- ============================================================
SELECT 
    c.nombre,
    c.ciudad,
    SUM(p.total)    AS gasto_total,
    RANK() OVER (ORDER BY SUM(p.total) DESC)                           AS ranking_global,
    RANK() OVER (PARTITION BY c.ciudad ORDER BY SUM(p.total) DESC)    AS ranking_por_ciudad
FROM clientes c
INNER JOIN pedidos p ON c.cliente_id = p.cliente_id
WHERE p.estado != 'CANCELADO'
GROUP BY c.cliente_id, c.nombre, c.ciudad
ORDER BY gasto_total DESC;
```

```sql
-- ============================================================
-- CONSULTA 9: Window Function - SUM acumulado con OVER PARTITION
-- Objetivo: Ventas acumuladas por mes con total corriente
-- ============================================================
SELECT 
    TO_CHAR(fecha_pedido, 'YYYY-MM')    AS mes,
    COUNT(*)                            AS pedidos_del_mes,
    SUM(total)                          AS ventas_del_mes,
    SUM(SUM(total)) OVER (
        ORDER BY TO_CHAR(fecha_pedido, 'YYYY-MM')
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )                                   AS ventas_acumuladas
FROM pedidos
WHERE estado NOT IN ('CANCELADO', 'PENDIENTE')
GROUP BY TO_CHAR(fecha_pedido, 'YYYY-MM')
ORDER BY mes;
```

```sql
-- ============================================================
-- CONSULTA 10: Reporte complejo - Dashboard ejecutivo
-- Objetivo: Reporte completo con múltiples métricas de negocio
-- ============================================================
WITH metricas_productos AS (
    SELECT 
        pr.producto_id,
        pr.nombre                           AS producto,
        cat.nombre                          AS categoria,
        pr.precio                           AS precio_catalogo,
        pr.stock                            AS stock_actual,
        COALESCE(SUM(dp.cantidad), 0)       AS unidades_vendidas,
        COALESCE(SUM(dp.cantidad * dp.precio_unitario), 0) AS ingresos_brutos
    FROM productos pr
    LEFT JOIN categorias cat     ON pr.categoria_id = cat.categoria_id
    LEFT JOIN detalle_pedidos dp ON pr.producto_id = dp.producto_id
    LEFT JOIN pedidos ped        ON dp.pedido_id = ped.pedido_id 
                                AND ped.estado != 'CANCELADO'
    GROUP BY pr.producto_id, pr.nombre, cat.nombre, pr.precio, pr.stock
)
SELECT 
    producto,
    categoria,
    precio_catalogo,
    stock_actual,
    unidades_vendidas,
    ROUND(ingresos_brutos, 0)   AS ingresos_brutos,
    CASE 
        WHEN unidades_vendidas = 0 THEN 'SIN VENTAS'
        WHEN unidades_vendidas < 3 THEN 'BAJA ROTACIÓN'
        WHEN unidades_vendidas < 6 THEN 'ROTACIÓN MEDIA'
        ELSE 'ALTA ROTACIÓN'
    END                         AS clasificacion_rotacion,
    ROUND(100.0 * ingresos_brutos / NULLIF(SUM(ingresos_brutos) OVER(), 0), 2) AS pct_ingresos_totales
FROM metricas_productos
ORDER BY ingresos_brutos DESC;
```

**Salida Esperada (Consulta 10 - fragmento):**

```
 producto              | categoria  | precio_catalogo | stock_actual | unidades_vendidas | ingresos_brutos | clasificacion_rotacion | pct_ingresos_totales
-----------------------+------------+-----------------+--------------+-------------------+-----------------+------------------------+---------------------
 Laptop ProBook 450    | Computación| 2850000.00      | 25           | 4                 | 11400000        | ROTACIÓN MEDIA         | 38.52
 Smartphone Galaxy S24 | Telefonía  | 3200000.00      | 40           | 2                 | 6400000         | BAJA ROTACIÓN          | 21.61
 Monitor UltraWide 34" | Periféricos| 1800000.00      | 15           | 1                 | 1800000         | BAJA ROTACIÓN          | 6.08
 ...
```

**Verificación:**

- Cada consulta debe retornar resultados sin errores
- La Consulta 3 debe mostrar clientes con 0 pedidos (Sofía Castro, Valentina Ruiz no tienen pedidos activos)
- La Consulta 9 debe mostrar ventas acumuladas crecientes mes a mes
- La Consulta 10 debe mostrar exactamente 12 productos con su clasificación

---

### Paso 4: Implementación de Transacciones Explícitas ACID

**Objetivo:** Implementar 3 transacciones explícitas que demuestren los principios ACID (Atomicidad, Consistencia, Aislamiento, Durabilidad) para operaciones críticas de negocio.

**Instrucciones:**

1. **Transacción 1:** Crear un nuevo pedido completo (operación atómica: insertar pedido + detalles + actualizar stock):

```sql
-- ============================================================
-- TRANSACCIÓN 1: Crear pedido completo
-- Demuestra ATOMICIDAD: todo se confirma o nada se confirma
-- ============================================================
BEGIN;

-- Paso 1: Verificar stock disponible antes de proceder
DO $$
DECLARE
    stock_laptop    INT;
    stock_mouse     INT;
BEGIN
    SELECT stock INTO stock_laptop FROM productos WHERE producto_id = 1;
    SELECT stock INTO stock_mouse  FROM productos WHERE producto_id = 5;
    
    IF stock_laptop < 1 THEN
        RAISE EXCEPTION 'Stock insuficiente para Laptop ProBook 450';
    END IF;
    IF stock_mouse < 2 THEN
        RAISE EXCEPTION 'Stock insuficiente para Mouse Inalámbrico Pro';
    END IF;
END $$;

-- Paso 2: Insertar el pedido
INSERT INTO pedidos (cliente_id, fecha_pedido, estado, total, direccion_envio)
VALUES (3, NOW(), 'CONFIRMADO', 3210000.00, 'Cll 15 #30-45, Cali');

-- Paso 3: Insertar los detalles del pedido (usando el ID recién creado)
INSERT INTO detalle_pedidos (pedido_id, producto_id, cantidad, precio_unitario, descuento)
VALUES 
    (LASTVAL(), 1, 1, 2850000.00, 0),
    (LASTVAL(), 5, 2,  180000.00, 0);

-- Paso 4: Actualizar el stock (descontar unidades vendidas)
UPDATE productos SET stock = stock - 1 WHERE producto_id = 1;
UPDATE productos SET stock = stock - 2 WHERE producto_id = 5;

-- Confirmar todos los cambios
COMMIT;

-- Verificar resultado
SELECT 'Pedido creado exitosamente' AS resultado;
SELECT p.pedido_id, c.nombre AS cliente, p.total, p.estado
FROM pedidos p JOIN clientes c ON p.cliente_id = c.cliente_id
ORDER BY p.pedido_id DESC LIMIT 1;
```

2. **Transacción 2:** Cancelar un pedido con rollback de stock (operación que restaura el estado):

```sql
-- ============================================================
-- TRANSACCIÓN 2: Cancelar pedido y restaurar stock
-- Demuestra CONSISTENCIA: el sistema vuelve a un estado válido
-- ============================================================
BEGIN;

-- Registrar el estado actual del stock antes de cancelar
SELECT producto_id, nombre, stock 
FROM productos 
WHERE producto_id IN (
    SELECT producto_id FROM detalle_pedidos WHERE pedido_id = 5
);

-- Cambiar estado del pedido a CANCELADO
UPDATE pedidos 
SET estado = 'CANCELADO'
WHERE pedido_id = 5 AND estado = 'CONFIRMADO';

-- Verificar que la actualización afectó exactamente 1 fila
DO $$
DECLARE
    filas_afectadas INT;
BEGIN
    GET DIAGNOSTICS filas_afectadas = ROW_COUNT;
    -- Nota: En PostgreSQL, GET DIAGNOSTICS va después del UPDATE
    -- Esta verificación es conceptual; en producción usar FOUND
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Pedido 5 no encontrado o ya no está en estado CONFIRMADO';
    END IF;
END $$;

-- Restaurar el stock de los productos del pedido cancelado
UPDATE productos p
SET stock = p.stock + dp.cantidad
FROM detalle_pedidos dp
WHERE dp.pedido_id = 5 
  AND dp.producto_id = p.producto_id;

COMMIT;

-- Verificar que el stock fue restaurado
SELECT producto_id, nombre, stock 
FROM productos 
WHERE producto_id IN (
    SELECT producto_id FROM detalle_pedidos WHERE pedido_id = 5
);
```

3. **Transacción 3:** Demostrar ROLLBACK explícito cuando ocurre un error de integridad:

```sql
-- ============================================================
-- TRANSACCIÓN 3: Demostración de ROLLBACK
-- Demuestra que una operación inválida revierte TODO
-- ============================================================

-- Primero, veamos el estado actual del stock del producto 1
SELECT producto_id, nombre, stock FROM productos WHERE producto_id = 1;

BEGIN;

-- Operación válida: reducir stock de Laptop
UPDATE productos SET stock = stock - 1 WHERE producto_id = 1;
SELECT 'Stock reducido temporalmente' AS estado_intermedio;
SELECT producto_id, nombre, stock FROM productos WHERE producto_id = 1;

-- Operación INVÁLIDA: intentar insertar un pedido con cliente inexistente
-- Esto violará la integridad referencial y causará un error
INSERT INTO pedidos (cliente_id, fecha_pedido, estado, total)
VALUES (9999, NOW(), 'CONFIRMADO', 100000.00);

-- Esta línea NUNCA se ejecutará porque el INSERT anterior fallará
COMMIT;
```

Después del error, ejecuta el ROLLBACK y verifica:

```sql
-- Después del error, PostgreSQL requiere ROLLBACK explícito
ROLLBACK;

-- Verificar que el stock NO fue modificado (ROLLBACK revirtió todo)
SELECT producto_id, nombre, stock 
FROM productos 
WHERE producto_id = 1;

SELECT 'ROLLBACK exitoso: el stock fue restaurado automáticamente' AS verificacion;
```

**Salida Esperada (Transacción 3):**

```
ERROR:  insert or update on table "pedidos" violates foreign key constraint "pedidos_cliente_id_fkey"
DETAIL:  Key (cliente_id)=(9999) is not present in table "clientes".

-- Después del ROLLBACK:
 producto_id | nombre           | stock
-------------+------------------+-------
 1           | Laptop ProBook 450| 23    ← mismo valor que antes, ROLLBACK exitoso
```

**Verificación:**

- La Transacción 1 debe crear el pedido 13 y reducir el stock de los productos involucrados
- La Transacción 2 debe cambiar el estado del pedido 5 a CANCELADO y restaurar el stock
- La Transacción 3 debe mostrar el error de integridad referencial y confirmar que el stock no cambió después del ROLLBACK

---

### Paso 5: Diseño e Implementación del Modelo MongoDB

**Objetivo:** Diseñar y crear colecciones MongoDB con validación de esquema JSON Schema para almacenar datos no estructurados del proyecto: logs de actividad, configuraciones dinámicas y datos de sesión.

**Instrucciones:**

1. Abre MongoDB Compass y conéctate a tu instancia:
   - Connection String: `mongodb://localhost:27017`
   - Si usas Docker: `mongodb://localhost:27017`

2. Crea la base de datos `tienda_logs` haciendo clic en "+" en el panel izquierdo.

3. Abre el shell de MongoDB Compass (pestaña "MONGOSH" en la parte inferior) y ejecuta los siguientes comandos para crear las colecciones con validación de esquema:

```javascript
// ============================================================
// SELECCIONAR BASE DE DATOS
// ============================================================
use tienda_logs

// ============================================================
// COLECCIÓN 1: logs_actividad
// Registra cada acción del usuario en el sistema
// Usa documentos embebidos para metadatos adicionales
// ============================================================
db.createCollection("logs_actividad", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["usuario_id", "accion", "timestamp", "modulo"],
      properties: {
        usuario_id: {
          bsonType: "int",
          description: "ID del usuario que realizó la acción (requerido)"
        },
        accion: {
          bsonType: "string",
          enum: ["LOGIN", "LOGOUT", "CREAR_PEDIDO", "CANCELAR_PEDIDO", 
                 "VER_PRODUCTO", "BUSCAR", "ACTUALIZAR_PERFIL", "PAGO"],
          description: "Tipo de acción realizada"
        },
        timestamp: {
          bsonType: "date",
          description: "Fecha y hora de la acción (requerido)"
        },
        modulo: {
          bsonType: "string",
          description: "Módulo del sistema donde ocurrió la acción"
        },
        detalles: {
          bsonType: "object",
          description: "Metadatos adicionales específicos de la acción"
        },
        ip_address: {
          bsonType: "string",
          description: "Dirección IP del cliente"
        },
        exitoso: {
          bsonType: "bool",
          description: "Si la acción fue exitosa"
        }
      }
    }
  },
  validationAction: "warn"
})

// ============================================================
// COLECCIÓN 2: configuraciones_dinamicas
// Almacena configuraciones que cambian frecuentemente
// Patrón: un documento por módulo de configuración
// ============================================================
db.createCollection("configuraciones_dinamicas", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["modulo", "version", "configuracion", "ultima_actualizacion"],
      properties: {
        modulo: {
          bsonType: "string",
          description: "Nombre del módulo de configuración"
        },
        version: {
          bsonType: "int",
          minimum: 1,
          description: "Versión de la configuración"
        },
        configuracion: {
          bsonType: "object",
          description: "Objeto de configuración flexible"
        },
        ultima_actualizacion: {
          bsonType: "date"
        },
        activo: {
          bsonType: "bool"
        }
      }
    }
  }
})

// ============================================================
// COLECCIÓN 3: sesiones_usuario
// Gestión de sesiones activas con datos de carrito
// ============================================================
db.createCollection("sesiones_usuario")

// Crear índice TTL para expiración automática de sesiones (24 horas)
db.sesiones_usuario.createIndex(
  { "creada_en": 1 }, 
  { expireAfterSeconds: 86400, name: "idx_ttl_sesion" }
)

// Crear índice único por token de sesión
db.sesiones_usuario.createIndex(
  { "token": 1 }, 
  { unique: true, name: "idx_token_unico" }
)

// Índice para búsquedas por usuario
db.sesiones_usuario.createIndex(
  { "usuario_id": 1 }, 
  { name: "idx_usuario_sesion" }
)
```

4. Verifica la creación de las colecciones:

```javascript
// Listar todas las colecciones
db.getCollectionNames()

// Ver la configuración de validación de logs_actividad
db.getCollectionInfos({ name: "logs_actividad" })
```

**Salida Esperada:**

```javascript
[ 'configuraciones_dinamicas', 'logs_actividad', 'sesiones_usuario' ]
```

**Verificación:**

- Las 3 colecciones deben aparecer en MongoDB Compass en el panel izquierdo bajo `tienda_logs`
- En la colección `sesiones_usuario`, verifica que los índices se crearon en la pestaña "Indexes" de Compass

---

### Paso 6: Operaciones CRUD en MongoDB con Operadores Avanzados

**Objetivo:** Ejecutar operaciones CRUD completas en MongoDB utilizando operadores de consulta y actualización avanzados para demostrar el poder del modelo documental.

**Instrucciones:**

1. Inserta documentos de prueba en las colecciones:

```javascript
// ============================================================
// INSERT MANY: Poblar logs_actividad con datos de prueba
// ============================================================
db.logs_actividad.insertMany([
  {
    usuario_id: 1,
    accion: "LOGIN",
    timestamp: new Date("2024-04-01T08:30:00Z"),
    modulo: "autenticacion",
    ip_address: "192.168.1.10",
    exitoso: true,
    detalles: {
      navegador: "Chrome 123",
      dispositivo: "Desktop",
      sistema_operativo: "Windows 11"
    }
  },
  {
    usuario_id: 1,
    accion: "VER_PRODUCTO",
    timestamp: new Date("2024-04-01T08:35:00Z"),
    modulo: "catalogo",
    ip_address: "192.168.1.10",
    exitoso: true,
    detalles: {
      producto_id: 1,
      producto_nombre: "Laptop ProBook 450",
      tiempo_vista_segundos: 45
    }
  },
  {
    usuario_id: 1,
    accion: "CREAR_PEDIDO",
    timestamp: new Date("2024-04-01T09:00:00Z"),
    modulo: "pedidos",
    ip_address: "192.168.1.10",
    exitoso: true,
    detalles: {
      pedido_id: 13,
      total: 3210000,
      productos: [
        { producto_id: 1, cantidad: 1 },
        { producto_id: 5, cantidad: 2 }
      ]
    }
  },
  {
    usuario_id: 2,
    accion: "LOGIN",
    timestamp: new Date("2024-04-01T10:00:00Z"),
    modulo: "autenticacion",
    ip_address: "10.0.0.25",
    exitoso: true,
    detalles: {
      navegador: "Firefox 124",
      dispositivo: "Laptop",
      sistema_operativo: "macOS 14"
    }
  },
  {
    usuario_id: 2,
    accion: "BUSCAR",
    timestamp: new Date("2024-04-01T10:05:00Z"),
    modulo: "catalogo",
    ip_address: "10.0.0.25",
    exitoso: true,
    detalles: {
      termino_busqueda: "laptop gaming",
      resultados_encontrados: 3,
      tiempo_respuesta_ms: 45
    }
  },
  {
    usuario_id: 3,
    accion: "LOGIN",
    timestamp: new Date("2024-04-02T14:00:00Z"),
    modulo: "autenticacion",
    ip_address: "172.16.0.5",
    exitoso: false,
    detalles: {
      razon_fallo: "CONTRASEÑA_INCORRECTA",
      intentos: 2
    }
  },
  {
    usuario_id: 3,
    accion: "LOGIN",
    timestamp: new Date("2024-04-02T14:02:00Z"),
    modulo: "autenticacion",
    ip_address: "172.16.0.5",
    exitoso: true,
    detalles: {
      navegador: "Safari 17",
      dispositivo: "iPhone",
      sistema_operativo: "iOS 17"
    }
  },
  {
    usuario_id: 4,
    accion: "PAGO",
    timestamp: new Date("2024-04-02T16:30:00Z"),
    modulo: "pagos",
    ip_address: "192.168.2.100",
    exitoso: true,
    detalles: {
      pedido_id: 11,
      monto: 1270000,
      metodo_pago: "TARJETA_CREDITO",
      banco: "Bancolombia",
      ultimos_4_digitos: "4521"
    }
  },
  {
    usuario_id: 5,
    accion: "CANCELAR_PEDIDO",
    timestamp: new Date("2024-04-03T09:15:00Z"),
    modulo: "pedidos",
    ip_address: "10.10.0.50",
    exitoso: true,
    detalles: {
      pedido_id: 8,
      motivo: "CAMBIO_DE_OPINION",
      reembolso_generado: true,
      monto_reembolso: 4650000
    }
  },
  {
    usuario_id: 1,
    accion: "LOGOUT",
    timestamp: new Date("2024-04-01T17:00:00Z"),
    modulo: "autenticacion",
    ip_address: "192.168.1.10",
    exitoso: true,
    detalles: {
      duracion_sesion_minutos: 510
    }
  }
])

// ============================================================
// INSERT: Configuraciones dinámicas
// ============================================================
db.configuraciones_dinamicas.insertMany([
  {
    modulo: "catalogo",
    version: 3,
    activo: true,
    ultima_actualizacion: new Date("2024-03-15"),
    configuracion: {
      productos_por_pagina: 20,
      ordenamiento_default: "precio_asc",
      mostrar_stock: true,
      descuento_maximo_permitido: 50,
      categorias_destacadas: [1, 2, 3],
      filtros_habilitados: ["precio", "categoria", "marca", "rating"]
    }
  },
  {
    modulo: "pagos",
    version: 2,
    activo: true,
    ultima_actualizacion: new Date("2024-04-01"),
    configuracion: {
      metodos_habilitados: ["TARJETA_CREDITO", "PSE", "EFECTIVO", "NEQUI"],
      monto_minimo: 10000,
      monto_maximo: 50000000,
      reintentos_maximos: 3,
      timeout_segundos: 30,
      comision_porcentaje: {
        TARJETA_CREDITO: 2.5,
        PSE: 1.2,
        EFECTIVO: 0,
        NEQUI: 0.8
      }
    }
  },
  {
    modulo: "notificaciones",
    version: 1,
    activo: true,
    ultima_actualizacion: new Date("2024-02-20"),
    configuracion: {
      email_habilitado: true,
      sms_habilitado: false,
      push_habilitado: true,
      eventos: {
        pedido_confirmado: ["email", "push"],
        pedido_enviado: ["email", "sms", "push"],
        pago_exitoso: ["email"],
        pago_fallido: ["email", "push"]
      }
    }
  }
])

// ============================================================
// INSERT: Sesiones de usuario activas
// ============================================================
db.sesiones_usuario.insertMany([
  {
    token: "eyJhbGciOiJIUzI1NiJ9.user1.abc123",
    usuario_id: 1,
    creada_en: new Date(),
    ultima_actividad: new Date(),
    carrito: {
      items: [
        { producto_id: 2, nombre: "Smartphone Galaxy S24", cantidad: 1, precio: 3200000 },
        { producto_id: 11, nombre: "Cargador USB-C 65W", cantidad: 2, precio: 45000 }
      ],
      total_items: 3,
      subtotal: 3290000
    },
    preferencias: {
      idioma: "es",
      moneda: "COP",
      notificaciones_push: true
    }
  },
  {
    token: "eyJhbGciOiJIUzI1NiJ9.user2.def456",
    usuario_id: 2,
    creada_en: new Date(Date.now() - 3600000),
    ultima_actividad: new Date(),
    carrito: {
      items: [],
      total_items: 0,
      subtotal: 0
    },
    preferencias: {
      idioma: "es",
      moneda: "COP",
      notificaciones_push: false
    }
  }
])
```

2. Ejecuta operaciones de consulta con operadores avanzados:

```javascript
// ============================================================
// CONSULTAS CON OPERADORES AVANZADOS
// ============================================================

// Consulta 1: Operadores $gt, $lt - Logs de usuarios específicos en un rango de fechas
db.logs_actividad.find({
  timestamp: {
    $gte: new Date("2024-04-01T00:00:00Z"),
    $lte: new Date("2024-04-01T23:59:59Z")
  },
  exitoso: true
}).sort({ timestamp: 1 }).pretty()

// Consulta 2: Operador $in - Logs de acciones críticas
db.logs_actividad.find({
  accion: { $in: ["CREAR_PEDIDO", "CANCELAR_PEDIDO", "PAGO"] }
}, {
  usuario_id: 1,
  accion: 1,
  timestamp: 1,
  "detalles.pedido_id": 1,
  _id: 0
}).sort({ timestamp: -1 })

// Consulta 3: Operador $regex - Buscar en detalles de navegador
db.logs_actividad.find({
  "detalles.navegador": { $regex: /Chrome|Firefox/i }
}, {
  usuario_id: 1,
  accion: 1,
  "detalles.navegador": 1,
  _id: 0
})

// Consulta 4: Operador $exists - Logs que tienen información de pago
db.logs_actividad.find({
  "detalles.metodo_pago": { $exists: true }
})

// Consulta 5: Consulta en documentos embebidos anidados
db.configuraciones_dinamicas.find({
  "configuracion.metodos_habilitados": { $in: ["NEQUI"] }
}, {
  modulo: 1,
  "configuracion.metodos_habilitados": 1,
  "configuracion.comision_porcentaje": 1,
  _id: 0
})
```

3. Ejecuta operaciones de actualización con operadores avanzados:

```javascript
// ============================================================
// OPERACIONES DE ACTUALIZACIÓN
// ============================================================

// UPDATE 1: $set - Actualizar configuración de catálogo
db.configuraciones_dinamicas.updateOne(
  { modulo: "catalogo" },
  {
    $set: {
      "configuracion.productos_por_pagina": 24,
      "configuracion.mostrar_stock": false,
      ultima_actualizacion: new Date()
    },
    $inc: { version: 1 }
  }
)

// Verificar la actualización
db.configuraciones_dinamicas.findOne({ modulo: "catalogo" })

// UPDATE 2: $push - Agregar un método de pago a la lista
db.configuraciones_dinamicas.updateOne(
  { modulo: "pagos" },
  {
    $push: {
      "configuracion.metodos_habilitados": "DAVIPLATA"
    },
    $set: {
      "configuracion.comision_porcentaje.DAVIPLATA": 0.5,
      ultima_actualizacion: new Date()
    }
  }
)

// UPDATE 3: $pull - Remover un elemento de un array
db.configuraciones_dinamicas.updateOne(
  { modulo: "catalogo" },
  {
    $pull: {
      "configuracion.categorias_destacadas": 3
    }
  }
)

// UPDATE 4: Actualizar carrito de sesión de usuario (upsert)
db.sesiones_usuario.updateOne(
  { usuario_id: 1 },
  {
    $set: {
      ultima_actividad: new Date(),
      "carrito.subtotal": 3335000
    },
    $push: {
      "carrito.items": {
        producto_id: 10,
        nombre: "Auriculares Bluetooth",
        cantidad: 1,
        precio: 320000
      }
    },
    $inc: { "carrito.total_items": 1 }
  }
)

// Verificar el carrito actualizado
db.sesiones_usuario.findOne(
  { usuario_id: 1 },
  { "carrito": 1, "ultima_actividad": 1, _id: 0 }
)
```

**Salida Esperada (verificación del carrito):**

```javascript
{
  ultima_actividad: ISODate("2024-..."),
  carrito: {
    items: [
      { producto_id: 2, nombre: 'Smartphone Galaxy S24', cantidad: 1, precio: 3200000 },
      { producto_id: 11, nombre: 'Cargador USB-C 65W', cantidad: 2, precio: 45000 },
      { producto_id: 10, nombre: 'Auriculares Bluetooth', cantidad: 1, precio: 320000 }
    ],
    total_items: 4,
    subtotal: 3335000
  }
}
```

**Verificación:**

- Confirma que los 10 logs de actividad fueron insertados (usa `db.logs_actividad.countDocuments()`)
- Verifica que la configuración de catálogo tiene `version: 4` después de la actualización
- El carrito del usuario 1 debe tener 4 items después del `$push`

---

### Paso 7: Pipeline de Agregación MongoDB

**Objetivo:** Construir un pipeline de agregación MongoDB con las etapas `$match`, `$group`, `$sort` y `$project` para generar un reporte de actividad de usuarios.

**Instrucciones:**

1. Ejecuta el pipeline de agregación completo:

```javascript
// ============================================================
// PIPELINE DE AGREGACIÓN 1: Reporte de actividad por usuario
// Etapas: $match → $group → $sort → $project
// ============================================================
db.logs_actividad.aggregate([
  // Etapa 1: $match - Filtrar solo logs exitosos del mes de abril
  {
    $match: {
      exitoso: true,
      timestamp: {
        $gte: new Date("2024-04-01T00:00:00Z"),
        $lte: new Date("2024-04-30T23:59:59Z")
      }
    }
  },
  // Etapa 2: $group - Agrupar por usuario y contar acciones
  {
    $group: {
      _id: "$usuario_id",
      total_acciones: { $sum: 1 },
      acciones_unicas: { $addToSet: "$accion" },
      primera_actividad: { $min: "$timestamp" },
      ultima_actividad: { $max: "$timestamp" },
      modulos_visitados: { $addToSet: "$modulo" }
    }
  },
  // Etapa 3: $sort - Ordenar por total de acciones descendente
  {
    $sort: { total_acciones: -1 }
  },
  // Etapa 4: $project - Dar formato al resultado final
  {
    $project: {
      _id: 0,
      usuario_id: "$_id",
      total_acciones: 1,
      num_acciones_distintas: { $size: "$acciones_unicas" },
      acciones_realizadas: "$acciones_unicas",
      num_modulos_visitados: { $size: "$modulos_visitados" },
      primera_actividad: 1,
      ultima_actividad: 1
    }
  }
])

// ============================================================
// PIPELINE DE AGREGACIÓN 2: Estadísticas por módulo y acción
// ============================================================
db.logs_actividad.aggregate([
  // Etapa 1: $group - Agrupar por módulo
  {
    $group: {
      _id: {
        modulo: "$modulo",
        exitoso: "$exitoso"
      },
      count: { $sum: 1 },
      usuarios_distintos: { $addToSet: "$usuario_id" }
    }
  },
  // Etapa 2: $group - Reestructurar para mostrar éxitos vs fallos
  {
    $group: {
      _id: "$_id.modulo",
      total_operaciones: { $sum: "$count" },
      detalle: {
        $push: {
          exitoso: "$_id.exitoso",
          cantidad: "$count",
          usuarios: { $size: "$usuarios_distintos" }
        }
      }
    }
  },
  // Etapa 3: $sort
  {
    $sort: { total_operaciones: -1 }
  },
  // Etapa 4: $project - Formato limpio
  {
    $project: {
      _id: 0,
      modulo: "$_id",
      total_operaciones: 1,
      detalle: 1
    }
  }
])

// ============================================================
// PIPELINE DE AGREGACIÓN 3: Análisis temporal - actividad por hora
// ============================================================
db.logs_actividad.aggregate([
  {
    $match: { exitoso: true }
  },
  {
    $group: {
      _id: {
        hora: { $hour: "$timestamp" },
        accion: "$accion"
      },
      frecuencia: { $sum: 1 }
    }
  },
  {
    $sort: { "_id.hora": 1, "frecuencia": -1 }
  },
  {
    $project: {
      _id: 0,
      hora_del_dia: "$_id.hora",
      accion: "$_id.accion",
      frecuencia: 1
    }
  }
])
```

2. Verifica el conteo total de documentos en cada colección:

```javascript
// Resumen de todas las colecciones
print("=== RESUMEN DE COLECCIONES ===")
print("logs_actividad:           " + db.logs_actividad.countDocuments())
print("configuraciones_dinamicas: " + db.configuraciones_dinamicas.countDocuments())
print("sesiones_usuario:          " + db.sesiones_usuario.countDocuments())
```

**Salida Esperada (Pipeline 1):**

```javascript
[
  {
    usuario_id: 1,
    total_acciones: 3,
    num_acciones_distintas: 3,
    acciones_realizadas: [ 'CREAR_PEDIDO', 'VER_PRODUCTO', 'LOGIN' ],
    num_modulos_visitados: 3,
    primera_actividad: ISODate("2024-04-01T08:30:00.000Z"),
    ultima_actividad: ISODate("2024-04-01T09:00:00.000Z")
  },
  ...
]
```

**Verificación:**

- El Pipeline 1 debe retornar resultados agrupados por usuario con sus métricas
- El Pipeline 2 debe mostrar estadísticas por módulo con desglose de éxitos/fallos
- El conteo de `logs_actividad` debe ser 10

---

### Paso 8: Integración Spring Boot — Configuración Dual de Fuentes de Datos

**Objetivo:** Configurar el proyecto Spring Boot para conectarse simultáneamente a PostgreSQL (via Spring Data JPA) y MongoDB (via Spring Data MongoDB), demostrando la coexistencia de ambas tecnologías.

**Instrucciones:**

1. Abre el proyecto Spring Boot en IntelliJ IDEA. Actualiza el archivo `pom.xml` para incluir las dependencias necesarias:

```xml
<!-- Agregar dentro de <dependencies> en pom.xml -->

<!-- Spring Data JPA para PostgreSQL -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Driver PostgreSQL -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Spring Data MongoDB -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>

<!-- Lombok para reducir boilerplate -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<!-- Spring Boot Starter Web (si no está ya) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

2. Crea el archivo de configuración `src/main/resources/application.yml`:

```yaml
# ============================================================
# CONFIGURACIÓN DUAL: PostgreSQL + MongoDB
# ============================================================
spring:
  application:
    name: tienda-integrador

  # ============================================================
  # CONFIGURACIÓN POSTGRESQL / JPA
  # ============================================================
  datasource:
    url: jdbc:postgresql://localhost:5432/tienda_db
    username: admin
    password: admin123
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 600000

  jpa:
    hibernate:
      ddl-auto: validate   # No recrear tablas (ya existen del Paso 1)
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
    open-in-view: false

  # ============================================================
  # CONFIGURACIÓN MONGODB
  # ============================================================
  data:
    mongodb:
      uri: mongodb://localhost:27017/tienda_logs
      auto-index-creation: true

# ============================================================
# CONFIGURACIÓN DE LOGGING
# ============================================================
logging:
  level:
    org.springframework.data.mongodb: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql: TRACE
  pattern:
    console: "%d{HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"

# ============================================================
# CONFIGURACIÓN DEL SERVIDOR
# ============================================================
server:
  port: 8080
```

3. Crea la entidad JPA para la tabla `clientes`. Crea el archivo `src/main/java/com/tienda/entity/Cliente.java`:

```java
package com.tienda.entity;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDate;

@Entity
@Table(name = "clientes")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cliente_id")
    private Integer clienteId;

    @Column(name = "nombre", nullable = false, length = 100)
    private String nombre;

    @Column(name = "email", nullable = false, unique = true, length = 150)
    private String email;

    @Column(name = "telefono", length = 20)
    private String telefono;

    @Column(name = "ciudad", length = 80)
    private String ciudad;

    @Column(name = "fecha_registro", nullable = false)
    private LocalDate fechaRegistro;

    @Column(name = "activo", nullable = false)
    private Boolean activo;
}
```

4. Crea la entidad JPA para `Pedido`. Crea `src/main/java/com/tienda/entity/Pedido.java`:

```java
package com.tienda.entity;

import jakarta.persistence.*;
import lombok.*;
import java.math.BigDecimal;
import java.time.LocalDate;
import java.time.LocalDateTime;

@Entity
@Table(name = "pedidos")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Pedido {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "pedido_id")
    private Integer pedidoId;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cliente_id", nullable = false)
    private Cliente cliente;

    @Column(name = "fecha_pedido", nullable = false)
    private LocalDateTime fechaPedido;

    @Column(name = "fecha_entrega")
    private LocalDate fechaEntrega;

    @Column(name = "estado", nullable = false, length = 20)
    private String estado;

    @Column(name = "total", precision = 10, scale = 2)
    private BigDecimal total;

    @Column(name = "direccion_envio", length = 250)
    private String direccionEnvio;
}
```

5. Crea el documento MongoDB para `LogActividad`. Crea `src/main/java/com/tienda/document/LogActividad.java`:

```java
package com.tienda.document;

import lombok.*;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;
import org.springframework.data.mongodb.core.index.Indexed;
import java.time.Instant;
import java.util.Map;

@Document(collection = "logs_actividad")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class LogActividad {

    @Id
    private String id;

    @Field("usuario_id")
    @Indexed
    private Integer usuarioId;

    @Field("accion")
    private String accion;

    @Field("timestamp")
    @Indexed
    private Instant timestamp;

    @Field("modulo")
    private String modulo;

    @Field("ip_address")
    private String ipAddress;

    @Field("exitoso")
    private Boolean exitoso;

    @Field("detalles")
    private Map<String, Object> detalles;
}
```

6. Crea los repositorios. Primero `src/main/java/com/tienda/repository/ClienteRepository.java`:

```java
package com.tienda.repository;

import com.tienda.entity.Cliente;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface ClienteRepository extends JpaRepository<Cliente, Integer> {

    // Método derivado: buscar por ciudad
    List<Cliente> findByCiudadAndActivoTrue(String ciudad);

    // Query JPQL: clientes con pedidos
    @Query("SELECT DISTINCT c FROM Cliente c JOIN c.pedidos p WHERE p.estado = :estado")
    List<Cliente> findClientesConPedidosPorEstado(@Param("estado") String estado);

    // Query nativa SQL para usar funcionalidades específicas de PostgreSQL
    @Query(value = """
        SELECT c.* FROM clientes c
        WHERE c.activo = true
        ORDER BY c.fecha_registro DESC
        LIMIT :limite
        """, nativeQuery = true)
    List<Cliente> findClientesRecientes(@Param("limite") int limite);
}
```

Luego `src/main/java/com/tienda/repository/LogActividadRepository.java`:

```java
package com.tienda.repository;

import com.tienda.document.LogActividad;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Aggregation;
import org.springframework.stereotype.Repository;
import java.time.Instant;
import java.util.List;

@Repository
public interface LogActividadRepository extends MongoRepository<LogActividad, String> {

    // Método derivado: logs por usuario ordenados por timestamp
    List<LogActividad> findByUsuarioIdOrderByTimestampDesc(Integer usuarioId);

    // Método derivado: logs por acción y estado de éxito
    List<LogActividad> findByAccionAndExitoso(String accion, Boolean exitoso);

    // Método derivado: logs en rango de fechas
    List<LogActividad> findByTimestampBetween(Instant inicio, Instant fin);

    // Conteo por módulo
    long countByModuloAndExitosoTrue(String modulo);

    // Agregación: contar acciones por usuario
    @Aggregation(pipeline = {
        "{ $match: { exitoso: true } }",
        "{ $group: { _id: '$usuario_id', total: { $sum: 1 } } }",
        "{ $sort: { total: -1 } }",
        "{ $limit: 10 }"
    })
    List<org.bson.Document> findTopUsuariosPorActividad();
}
```

7. Crea un servicio de integración que use ambas bases de datos. Crea `src/main/java/com/tienda/service/DashboardService.java`:

```java
package com.tienda.service;

import com.tienda.document.LogActividad;
import com.tienda.entity.Cliente;
import com.tienda.repository.ClienteRepository;
import com.tienda.repository.LogActividadRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Service
@RequiredArgsConstructor
@Slf4j
public class DashboardService {

    private final ClienteRepository clienteRepository;
    private final LogActividadRepository logActividadRepository;

    /**
     * Obtiene un dashboard combinando datos de PostgreSQL y MongoDB.
     * Demuestra la integración dual de fuentes de datos.
     */
    @Transactional(readOnly = true)
    public Map<String, Object> obtenerDashboard() {
        Map<String, Object> dashboard = new HashMap<>();

        // Datos desde PostgreSQL
        List<Cliente> clientesRecientes = clienteRepository.findClientesRecientes(5);
        long totalClientes = clienteRepository.count();

        // Datos desde MongoDB
        Instant hace24Horas = Instant.now().minus(24, ChronoUnit.HOURS);
        List<LogActividad> logsRecientes = logActividadRepository
            .findByTimestampBetween(hace24Horas, Instant.now());
        long totalLogs = logActividadRepository.count();

        // Combinar resultados
        dashboard.put("total_clientes_postgresql", totalClientes);
        dashboard.put("clientes_recientes", clientesRecientes.stream()
            .map(c -> Map.of(
                "id", c.getClienteId(),
                "nombre", c.getNombre(),
                "ciudad", c.getCiudad()
            )).toList());
        dashboard.put("total_logs_mongodb", totalLogs);
        dashboard.put("logs_ultimas_24h", logsRecientes.size());
        dashboard.put("top_usuarios_activos",
            logActividadRepository.findTopUsuariosPorActividad());

        log.info("Dashboard generado: {} clientes, {} logs recientes",
            totalClientes, logsRecientes.size());

        return dashboard;
    }

    /**
     * Registra un log de actividad en MongoDB cuando se crea un pedido en PostgreSQL.
     * Demuestra el patrón de escritura dual.
     */
    public void registrarAccion(Integer usuarioId, String accion,
                                 String modulo, Map<String, Object> detalles) {
        LogActividad log = LogActividad.builder()
            .usuarioId(usuarioId)
            .accion(accion)
            .timestamp(Instant.now())
            .modulo(modulo)
            .exitoso(true)
            .detalles(detalles)
            .build();

        logActividadRepository.save(log);
    }
}
```

8. Crea un controlador REST básico para probar la integración. Crea `src/main/java/com/tienda/controller/DashboardController.java`:

```java
package com.tienda.controller;

import com.tienda.service.DashboardService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.Map;

@RestController
@RequestMapping("/api/dashboard")
@RequiredArgsConstructor
@CrossOrigin(origins = "*") // Solo para desarrollo local
public class DashboardController {

    private final DashboardService dashboardService;

    @GetMapping
    public ResponseEntity<Map<String, Object>> obtenerDashboard() {
        return ResponseEntity.ok(dashboardService.obtenerDashboard());
    }

    @PostMapping("/log")
    public ResponseEntity<String> registrarLog(
            @RequestParam Integer usuarioId,
            @RequestParam String accion,
            @RequestParam String modulo) {

        dashboardService.registrarAccion(
            usuarioId, accion, modulo,
            Map.of("origen", "api_manual", "timestamp_request", System.currentTimeMillis())
        );
        return ResponseEntity.ok("Log registrado exitosamente");
    }
}
```

9. Compila y ejecuta la aplicación:

```bash
# Navegar al directorio del proyecto
cd /ruta/a/tu/proyecto

# Compilar y ejecutar
mvn clean compile

# Ejecutar la aplicación
mvn spring-boot:run
```

**Para Windows (PowerShell):**
```powershell
cd C:\ruta\a\tu\proyecto
mvn clean compile
mvn spring-boot:run
```

10. Prueba los endpoints con curl:

```bash
# Probar el endpoint del dashboard
curl -X GET http://localhost:8080/api/dashboard

# Registrar un log de actividad
curl -X POST "http://localhost:8080/api/dashboard/log?usuarioId=1&accion=VER_PRODUCTO&modulo=catalogo"
```

**Salida Esperada (fragmento del dashboard):**

```json
{
  "total_clientes_postgresql": 10,
  "clientes_recientes": [
    {"id": 10, "nombre": "Diego Vargas", "ciudad": "Bogotá"},
    {"id": 9, "nombre": "Valentina Ruiz", "ciudad": "Cali"},
    ...
  ],
  "total_logs_mongodb": 10,
  "logs_ultimas_24h": 0,
  "top_usuarios_activos": [
    {"_id": 1, "total": 3},
    {"_id": 2, "total": 2},
    ...
  ]
}
```

**Verificación:**

- La aplicación debe iniciar sin errores en la consola
- El endpoint `GET /api/dashboard` debe retornar HTTP 200 con datos combinados de ambas bases de datos
- Los logs de Hibernate deben mostrar las queries SQL a PostgreSQL
- Los logs de MongoDB deben mostrar las operaciones a la colección `logs_actividad`

---

### Paso 9: Análisis Comparativo — Modelo Relacional vs. Documental

**Objetivo:** Documentar y analizar las diferencias prácticas observadas durante el laboratorio entre el modelo relacional (PostgreSQL) y el modelo documental (MongoDB) aplicadas al dominio del proyecto.

**Instrucciones:**

1. Ejecuta las siguientes consultas comparativas para evidenciar las diferencias de diseño:

```sql
-- ============================================================
-- POSTGRESQL: Consulta que requiere 4 JOINs para obtener
-- el detalle completo de un pedido
-- ============================================================
SELECT 
    p.pedido_id,
    c.nombre        AS cliente,
    c.email,
    p.fecha_pedido,
    p.estado,
    pr.nombre       AS producto,
    cat.nombre      AS categoria,
    dp.cantidad,
    dp.precio_unitario,
    dp.descuento,
    ROUND(dp.cantidad * dp.precio_unitario * (1 - dp.descuento/100), 2) AS subtotal_linea
FROM pedidos p
JOIN clientes c         ON p.cliente_id = c.cliente_id
JOIN detalle_pedidos dp ON p.pedido_id = dp.pedido_id
JOIN productos pr       ON dp.producto_id = pr.producto_id
JOIN categorias cat     ON pr.categoria_id = cat.categoria_id
WHERE p.pedido_id = 1
ORDER BY pr.nombre;
```

```javascript
// ============================================================
// MONGODB: El mismo pedido como documento único (sin JOINs)
// En MongoDB, el pedido y sus detalles están en un solo documento
// ============================================================
// Simulación de cómo se vería un pedido en MongoDB
// (modelo desnormalizado / embebido)
db.logs_actividad.findOne(
  { accion: "CREAR_PEDIDO" },
  {
    usuario_id: 1,
    accion: 1,
    timestamp: 1,
    "detalles.pedido_id": 1,
    "detalles.productos": 1,
    "detalles.total": 1,
    _id: 0
  }
)
```

2. Crea un archivo de análisis en el proyecto. Crea `src/main/resources/analisis_comparativo.md`:

```markdown
# Análisis Comparativo: PostgreSQL vs MongoDB
## Proyecto Integrador - Tienda en Línea

### 1. Estructura de Datos

| Aspecto | PostgreSQL (Relacional) | MongoDB (Documental) |
|---------|------------------------|----------------------|
| Organización | Tablas con esquema fijo | Colecciones con documentos flexibles |
| Relaciones | Claves foráneas explícitas | Referencias o documentos embebidos |
| Esquema | Rígido (DDL define estructura) | Flexible (JSON Schema opcional) |
| Redundancia | Mínima (normalización) | Controlada (desnormalización estratégica) |

### 2. Casos de Uso Identificados en el Proyecto

**Datos en PostgreSQL (relacionales):**
- Clientes, productos, categorías: datos maestros con integridad referencial
- Pedidos y detalle_pedidos: transacciones que requieren ACID
- Datos financieros que requieren consistencia garantizada

**Datos en MongoDB (documentales):**
- Logs de actividad: alta velocidad de escritura, esquema variable por tipo de acción
- Configuraciones dinámicas: estructura flexible que cambia por módulo
- Sesiones de usuario: datos temporales con TTL automático, estructura anidada (carrito)

### 3. Consultas: Diferencias Observadas

**PostgreSQL:**
- Requiere múltiples JOINs para reconstituir entidades relacionadas
- Lenguaje SQL estándar y expresivo para reportes complejos
- Window functions para análisis temporal sin código adicional

**MongoDB:**
- Documentos embebidos eliminan la necesidad de JOINs en lecturas
- Pipeline de agregación para transformaciones complejas
- Operadores $push/$pull para arrays sin reestructurar el esquema

### 4. Transacciones

**PostgreSQL:** Transacciones ACID nativas con BEGIN/COMMIT/ROLLBACK
- Garantía total de atomicidad en operaciones multi-tabla
- Aislamiento configurable (READ COMMITTED, SERIALIZABLE)

**MongoDB:** Transacciones multi-documento disponibles desde v4.0
- Para documentos únicos: operaciones atómicas por defecto
- Transacciones multi-colección tienen overhead de rendimiento

### 5. Escalabilidad

**PostgreSQL:** Escalado vertical principalmente; replicación para lectura
**MongoDB:** Sharding horizontal nativo para grandes volúmenes

### 6. Decisión de Diseño para Este Proyecto

La combinación de ambas tecnologías permite:
1. Mantener integridad transaccional en el core del negocio (PostgreSQL)
2. Escalar el sistema de logs sin impactar las transacciones (MongoDB)
3. Almacenar configuraciones dinámicas sin migraciones de esquema (MongoDB)
4. Aprovechar las fortalezas de cada paradigma según el caso de uso
```

**Verificación:**

- El archivo `analisis_comparativo.md` debe existir en `src/main/resources/`
- La consulta SQL debe retornar las 2 líneas del pedido 1 con todos los detalles
- El documento MongoDB debe mostrar la estructura del log de creación de pedido

## Validación y Pruebas

### Criterios de Éxito

- [ ] Las 5 tablas PostgreSQL fueron creadas con todos sus constraints y las 5 relaciones de clave foránea están correctamente definidas
- [ ] Los datos de prueba fueron insertados: 9 categorías, 10 clientes, 12 productos, 12 pedidos, 21 detalles de pedido
- [ ] Las 10 consultas SQL se ejecutaron sin errores, incluyendo window functions (Consultas 8 y 9)
- [ ] Las 3 transacciones explícitas fueron implementadas: Transacción 1 (crear pedido), Transacción 2 (cancelar pedido), Transacción 3 (demostración ROLLBACK)
- [ ] Las 3 colecciones MongoDB fueron creadas con validación de esquema JSON Schema
- [ ] Los documentos de prueba fueron insertados: 10 logs, 3 configuraciones, 2 sesiones
- [ ] Los 3 pipelines de agregación MongoDB se ejecutaron correctamente
- [ ] El proyecto Spring Boot compila e inicia sin errores con configuración dual JPA + MongoDB
- [ ] El endpoint `GET /api/dashboard` retorna datos combinados de ambas bases de datos
- [ ] El archivo de análisis comparativo fue creado y documenta las diferencias observadas

### Procedimiento de Pruebas

1. Verificar el esquema PostgreSQL:
   ```sql
   SELECT table_name, constraint_name, constraint_type
   FROM information_schema.table_constraints
   WHERE table_schema = 'public'
   ORDER BY table_name, constraint_type;
   ```
   **Resultado Esperado:** Al menos 15 constraints (PKs, FKs, CHECKs, UNIQUEs)

2. Verificar la integridad referencial con una inserción inválida:
   ```sql
   -- Esta operación DEBE fallar
   INSERT INTO pedidos (cliente_id, fecha_pedido, estado, total)
   VALUES (9999, NOW(), 'PENDIENTE', 100.00);
   ```
   **Resultado Esperado:** `ERROR: insert or update on table "pedidos" violates foreign key constraint`

3. Verificar las colecciones MongoDB:
   ```javascript
   db.getCollectionInfos().forEach(c => print(c.name + ": " + db[c.name].countDocuments() + " documentos"))
   ```
   **Resultado Esperado:**
   ```
   configuraciones_dinamicas: 3 documentos
   logs_actividad: 10 documentos
   sesiones_usuario: 2 documentos
   ```

4. Probar el endpoint del dashboard Spring Boot:
   ```bash
   curl -s http://localhost:8080/api/dashboard | python3 -m json.tool
   ```
   **Resultado Esperado:** JSON con campos `total_clientes_postgresql`, `total_logs_mongodb` y `top_usuarios_activos`

5. Verificar que el ROLLBACK funciona correctamente:
   ```sql
   -- Antes del ROLLBACK, verificar stock
   SELECT stock FROM productos WHERE producto_id = 1;
   BEGIN;
   UPDATE productos SET stock = stock - 999 WHERE producto_id = 1;
   SELECT stock FROM productos WHERE producto_id = 1; -- Debe mostrar stock negativo
   ROLLBACK;
   SELECT stock FROM productos WHERE producto_id = 1; -- Debe ser el valor original
   ```
   **Resultado Esperado:** El stock regresa al valor original después del ROLLBACK

## Solución de Problemas

### Problema 1: Error de Conexión a PostgreSQL "Connection refused"

**Síntomas:**
- `psql: error: connection to server on socket failed: Connection refused`
- pgAdmin muestra "Server not listening"
- Spring Boot lanza `org.postgresql.util.PSQLException: Connection refused`

**Causa:**
El servicio de PostgreSQL no está en ejecución o está escuchando en un puerto diferente al 5432.

**Solución:**
```bash
# Verificar si PostgreSQL está en ejecución (Linux/macOS)
sudo systemctl status postgresql
# O para macOS con Homebrew:
brew services list | grep postgresql

# Iniciar PostgreSQL (Linux)
sudo systemctl start postgresql

# Iniciar PostgreSQL (macOS con Homebrew)
brew services start postgresql@16

# Si usas Docker, verificar el contenedor
docker ps -a | grep postgres-integrador
docker start postgres-integrador

# Verificar que el puerto 5432 está disponible
# Linux/macOS:
netstat -tlnp | grep 5432
# Windows PowerShell:
netstat -ano | findstr :5432
```

---

### Problema 2: Error de Autenticación PostgreSQL "password authentication failed"

**Síntomas:**
- `FATAL: password authentication failed for user "admin"`
- pgAdmin no puede conectarse con las credenciales proporcionadas

**Causa:**
Las credenciales configuradas no coinciden con las del servidor PostgreSQL local o del contenedor Docker.

**Solución:**
```bash
# Si usas Docker, recrear el contenedor con credenciales correctas
docker stop postgres-integrador
docker rm postgres-integrador
docker run -d \
  --name postgres-integrador \
  -e POSTGRES_DB=tienda_db \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=admin123 \
  -p 5432:5432 \
  postgres:16-alpine

# Si usas PostgreSQL local, conectarte como superusuario y cambiar contraseña
sudo -u postgres psql
-- Dentro de psql:
ALTER USER admin WITH PASSWORD 'admin123';
\q

# Verificar la conexión
psql -h localhost -U admin -d tienda_db -c "SELECT version();"
```

---

### Problema 3: Error de Validación de Esquema MongoDB "Document failed validation"

**Síntomas:**
- `MongoWriteException: Document failed validation`
- Los insertMany fallan con error de validación

**Causa:**
El documento que se intenta insertar no cumple con el JSON Schema definido en la colección. Usualmente ocurre cuando el campo `accion` tiene un valor no listado en el enum.

**Solución:**
```javascript
// Verificar el esquema de validación actual
db.getCollectionInfos({ name: "logs_actividad" })[0].options.validator

// Opción 1: Modificar el documento para que cumpla el esquema
// Asegurarse de que 'accion' sea uno de los valores permitidos:
// "LOGIN", "LOGOUT", "CREAR_PEDIDO", "CANCELAR_PEDIDO",
// "VER_PRODUCTO", "BUSCAR", "ACTUALIZAR_PERFIL", "PAGO"

// Opción 2: Cambiar validationAction a "warn" (ya configurado así)
// Esto permite insertar documentos inválidos con advertencia
db.runCommand({
  collMod: "logs_actividad",
  validationAction: "warn"
})

// Opción 3: Deshabilitar temporalmente la validación para pruebas
db.runCommand({
  collMod: "logs_actividad",
  validator: {}
})
```

---

### Problema 4: Spring Boot Error "Table 'clientes' doesn't exist" con ddl-auto: validate

**Síntomas:**
- `javax.persistence.PersistenceException: Table 'tienda_db.clientes' doesn't exist`
- La aplicación Spring Boot no inicia

**Causa:**
La configuración `ddl-auto: validate` intenta validar que las tablas JPA existen en la base de datos, pero las tablas no fueron creadas o el esquema no coincide.

**Solución:**
```bash
# Paso 1: Verificar que las tablas existen en PostgreSQL
psql -h localhost -U admin -d tienda_db -c "\dt"

# Si las tablas no existen, ejecutar el script DDL del Paso 1

# Paso 2: Verificar que el nombre de la base de datos en application.yml
# coincide exactamente con la base de datos creada
# En application.yml: url: jdbc:postgresql://localhost:5432/tienda_db
# La base de datos debe llamarse 'tienda_db'

# Paso 3: Si quieres que Spring cree las tablas automáticamente (solo para desarrollo)
# Cambiar en application.yml:
# jpa:
#   hibernate:
#     ddl-auto: create-drop  # Crea al inicio, elimina al cerrar
# ADVERTENCIA: Esto borrará todos los datos al reiniciar
```

```yaml
# Cambio temporal en application.yml para diagnóstico
spring:
  jpa:
    hibernate:
      ddl-auto: update  # Crea/actualiza tablas sin borrar datos existentes
```

---

### Problema 5: Error de Transacción "ERROR: current transaction is aborted"

**Síntomas:**
- `ERROR: current transaction is aborted, commands ignored until end of transaction block`
- Las consultas posteriores a un error fallan aunque parezcan válidas

**Causa:**
En PostgreSQL, cuando una sentencia dentro de un bloque de transacción falla, todas las sentencias posteriores en esa misma transacción son ignoradas hasta que se ejecute ROLLBACK.

**Solución:**
```sql
-- Si ves este error, primero ejecuta ROLLBACK para limpiar el estado
ROLLBACK;

-- Luego verifica cuál fue el error original
-- Revisa el panel de mensajes de pgAdmin o el log de PostgreSQL

-- Para evitar este problema en el futuro, usa bloques DO con manejo de excepciones:
BEGIN;

DO $$
BEGIN
    -- Tu código aquí
    UPDATE productos SET stock = stock - 1 WHERE producto_id = 1;
    
    -- Simulación de error controlado
    IF (SELECT stock FROM productos WHERE producto_id = 1) < 0 THEN
        RAISE EXCEPTION 'Stock no puede ser negativo';
    END IF;
    
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Error capturado: %', SQLERRM;
        RAISE; -- Re-lanzar para que el bloque BEGIN/COMMIT haga ROLLBACK
END $$;

COMMIT;
```

---

### Problema 6: Maven "Could not resolve dependencies" al compilar

**Síntomas:**
- `Could not resolve artifact org.springframework.boot:spring-boot-starter-data-mongodb:jar`
- El proyecto no compila por dependencias no encontradas

**Causa:**
Problemas de conectividad a Maven Central o caché local corrupta.

**Solución:**
```bash
# Limpiar caché local de Maven y forzar descarga
mvn dependency:purge-local-repository -DreResolve=true

# O eliminar manualmente el directorio de caché
# Linux/macOS:
rm -rf ~/.m2/repository/org/springframework/boot/spring-boot-starter-data-mongodb

# Windows PowerShell:
Remove-Item -Recurse -Force "$env:USERPROFILE\.m2\repository\org\springframework\boot\spring-boot-starter-data-mongodb"

# Forzar actualización de todas las dependencias
mvn clean install -U

# Verificar la versión de Spring Boot en pom.xml
# Debe ser 3.2.x para compatibilidad con Java 17
mvn help:evaluate -Dexpression=project.version -q -DforceStdout
```

## Limpieza

```bash
# ============================================================
# LIMPIEZA DE RECURSOS DOCKER
# ============================================================

# Detener los contenedores
docker stop postgres-integrador mongo-integrador

# Eliminar los contenedores (los datos se perderán)
docker rm postgres-integrador mongo-integrador

# Eliminar la red Docker creada
docker network rm proyecto-integrador-net

# Verificar que los contenedores fueron eliminados
docker ps -a | grep -E "postgres-integrador|mongo-integrador"
```

```sql
-- ============================================================
-- LIMPIEZA DE POSTGRESQL (si instalación local, no Docker)
-- Ejecutar en pgAdmin como superusuario
-- ============================================================

-- Eliminar todas las tablas del esquema public
DROP TABLE IF EXISTS detalle_pedidos CASCADE;
DROP TABLE IF EXISTS pedidos CASCADE;
DROP TABLE IF EXISTS productos CASCADE;
DROP TABLE IF EXISTS categorias CASCADE;
DROP TABLE IF EXISTS clientes CASCADE;

-- O eliminar toda la base de datos
-- DROP DATABASE tienda_db;
```

```javascript
// ============================================================
// LIMPIEZA DE MONGODB (si instalación local, no Docker)
// ============================================================
use tienda_logs

// Eliminar todas las colecciones
db.logs_actividad.drop()
db.configuraciones_dinamicas.drop()
db.sesiones_usuario.drop()

// O eliminar toda la base de datos
db.dropDatabase()
```

```bash
# ============================================================
# LIMPIEZA DEL PROYECTO SPRING BOOT
# ============================================================

# Detener la aplicación Spring Boot (Ctrl+C en la terminal donde está corriendo)
# Limpiar los artefactos compilados
mvn clean

# Verificar que el puerto 8080 quedó libre
# Linux/macOS:
lsof -i :8080
# Windows:
netstat -ano | findstr :8080
```

> ⚠️ **Advertencia:** Si eliminas los contenedores Docker, **todos los datos insertados durante el laboratorio se perderán permanentemente**. Si deseas conservar los datos para referencia futura, primero exporta los datos:
> ```bash
> # Exportar datos PostgreSQL
> docker exec postgres-integrador pg_dump -U admin tienda_db > backup_tienda_db.sql
> # Exportar datos MongoDB
> docker exec mongo-integrador mongodump --db tienda_logs --out /tmp/backup
> docker cp mongo-integrador:/tmp/backup ./backup_mongo
> ```

## Resumen

### Lo que Lograste

- **Esquema relacional normalizado:** Diseñaste e implementaste 5 tablas en PostgreSQL aplicando 3FN, con claves primarias, claves foráneas, índices y constraints de integridad que garantizan la consistencia de los datos
- **Consultas SQL avanzadas:** Ejecutaste 10 consultas de complejidad creciente incluyendo JOINs múltiples, subconsultas correlacionadas, funciones de agregación con GROUP BY/HAVING, y window functions (RANK, SUM OVER PARTITION) para análisis de negocio
- **Transacciones ACID:** Implementaste 3 transacciones explícitas con BEGIN/COMMIT/ROLLBACK, demostrando atomicidad en la creación de pedidos, consistencia en la cancelación con restauración de stock, y la garantía de rollback automático ante errores de integridad
- **Modelo documental MongoDB:** Creaste 3 colecciones con validación JSON Schema para logs de actividad, configuraciones dinámicas y sesiones de usuario, aprovechando documentos embebidos y arrays para datos naturalmente jerárquicos
- **Operaciones CRUD avanzadas:** Ejecutaste operaciones con operadores `$set`, `$push`, `$pull`, `$in`, `$regex`, `$gt`, `$lt`, `$exists` y construiste 3 pipelines de agregación con `$match`, `$group`, `$sort` y `$project`
- **Integración Spring Boot dual:** Configuraste Spring Data JPA y Spring Data MongoDB en el mismo proyecto, creando entidades, documentos, repositorios y un servicio que combina datos de ambas fuentes en tiempo real

### Conceptos Clave Aprendidos

- La **normalización** (3FN) elimina redundancias y anomalías de actualización, pero requiere JOINs para reconstituir entidades completas
- Las **claves foráneas** con `ON DELETE RESTRICT` y `ON DELETE CASCADE` implementan automáticamente reglas de negocio críticas a nivel de base de datos
- Las **window functions** como `RANK() OVER (PARTITION BY ...)` permiten análisis comparativos dentro de grupos sin subconsultas complejas
- Los **bloques BEGIN/COMMIT/ROLLBACK** garantizan que operaciones multi-paso sean atómicas: o todo se confirma o nada se confirma
- El **modelo documental** es superior para datos con esquema variable, alta frecuencia de escritura y estructuras naturalmente anidadas como logs y configuraciones
- La **coexistencia** de Spring Data JPA y Spring Data MongoDB en un mismo proyecto permite usar cada tecnología donde mejor se adapta, sin compromisos en ninguna dirección

### Próximos Pasos

- **Laboratorio 6:** Implementar Spring Batch para procesamiento por lotes de los datos del proyecto, incluyendo importación masiva de productos desde CSV y generación de reportes periódicos usando los datos de PostgreSQL y MongoDB configurados en este laboratorio
- **Profundización SQL:** Practicar vistas materializadas, procedimientos almacenados y funciones en PostgreSQL para encapsular la lógica de negocio compleja identificada en las consultas de reporte
- **MongoDB Avanzado:** Explorar el uso de índices compuestos, índices de texto completo y Change Streams para notificaciones en tiempo real cuando se insertan nuevos logs de actividad
- **Optimización de consultas:** Usar `EXPLAIN ANALYZE` en PostgreSQL y `.explain("executionStats")` en MongoDB para identificar y optimizar las consultas más lentas del proyecto

## Recursos Adicionales

- **PostgreSQL Official Documentation - SQL Commands:** Referencia completa de todas las sentencias SQL soportadas por PostgreSQL, incluyendo window functions y CTEs. Disponible en: https://www.postgresql.org/docs/16/sql-commands.html
- **MongoDB Manual - Aggregation Pipeline:** Documentación oficial de todas las etapas del pipeline de agregación con ejemplos detallados. Disponible en: https://www.mongodb.com/docs/manual/core/aggregation-pipeline/
- **Spring Data JPA Reference Documentation:** Guía oficial para configurar repositorios JPA, queries derivados y queries JPQL en Spring Boot 3.x. Disponible en: https://docs.spring.io/spring-data/jpa/reference/
- **Spring Data MongoDB Reference Documentation:** Guía oficial para configurar MongoTemplate, repositorios MongoDB y aggregations en Spring Boot 3.x. Disponible en: https://docs.spring.io/spring-data/mongodb/reference/
- **"Use The Index, Luke" - Markus Winand:** Recurso gratuito en línea sobre optimización de consultas SQL con índices, aplicable a PostgreSQL. Disponible en: https://use-the-index-luke.com/
- **PostgreSQL EXPLAIN Visualizer (explain.dalibo.com):** Herramienta visual gratuita para interpretar los planes de ejecución de EXPLAIN ANALYZE. Disponible en: https://explain.dalibo.com/
- **Elmasri, R., & Navathe, S. B. (2015). Fundamentals of Database Systems (7th ed.):** Referencia académica completa sobre el modelo relacional, normalización y diseño de esquemas. Editorial Pearson.
