# SQL - Comandos Esenciales

Manual completo de SQL con ejemplos prácticos y explicaciones detalladas.

---

## 1. FUNDAMENTOS DE SQL

### 1.1 ¿Qué es SQL?

**SQL (Structured Query Language)** es el lenguaje estándar para gestionar bases de datos relacionales.

**Categorías de comandos SQL:**
- **DDL** (Data Definition Language): Define estructuras (CREATE, ALTER, DROP)
- **DML** (Data Manipulation Language): Manipula datos (SELECT, INSERT, UPDATE, DELETE)
- **DCL** (Data Control Language): Controla acceso (GRANT, REVOKE)
- **TCL** (Transaction Control Language): Gestiona transacciones (COMMIT, ROLLBACK)

---

### 1.2 Tipos de Datos en MySQL

| Tipo | Descripción | Ejemplo |
|------|-------------|---------|
| **Numéricos** |
| `INT` | Entero (-2,147,483,648 a 2,147,483,647) | `id INT` |
| `BIGINT` | Entero grande | `contador BIGINT` |
| `DECIMAL(M,D)` | Decimal preciso | `precio DECIMAL(10,2)` |
| `FLOAT` | Punto flotante | `temperatura FLOAT` |
| **Cadenas** |
| `VARCHAR(n)` | Cadena variable (hasta 65,535) | `nombre VARCHAR(100)` |
| `CHAR(n)` | Cadena fija | `codigo CHAR(5)` |
| `TEXT` | Texto largo (hasta 65,535) | `descripcion TEXT` |
| `LONGTEXT` | Texto muy largo (4GB) | `contenido LONGTEXT` |
| **Fechas** |
| `DATE` | Fecha (YYYY-MM-DD) | `fecha_nac DATE` |
| `DATETIME` | Fecha y hora | `created_at DATETIME` |
| `TIMESTAMP` | Marca de tiempo | `updated_at TIMESTAMP` |
| `TIME` | Hora | `hora_entrada TIME` |
| **Otros** |
| `BOOLEAN` | True/False (0/1) | `activo BOOLEAN` |
| `BLOB` | Datos binarios | `imagen BLOB` |
| `ENUM` | Lista de valores | `estado ENUM('activo','inactivo')` |

---

## 2. DDL (DATA DEFINITION LANGUAGE)

### 2.1 Bases de Datos y Esquemas

**Crear base de datos:**
```sql
CREATE DATABASE empresa;

-- Con opciones de codificación
CREATE DATABASE empresa 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_unicode_ci;

-- Solo si no existe
CREATE DATABASE IF NOT EXISTS empresa;
```

**Usar base de datos:**
```sql
USE empresa;
```

**Listar bases de datos:**
```sql
SHOW DATABASES;
```

**Eliminar base de datos:**
```sql
DROP DATABASE empresa;

-- Con verificación
DROP DATABASE IF EXISTS empresa;
```

**Crear esquema (organización lógica):**
```sql
-- Los esquemas permiten organizar objetos por áreas funcionales
CREATE SCHEMA IF NOT EXISTS gestion_rrhh;

-- Usar el esquema
USE gestion_rrhh;
```

---

### 2.2 Tablas

**Crear tabla básica:**
```sql
CREATE TABLE empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100) NOT NULL,
    apellido VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    fecha_contratacion DATE,
    salario DECIMAL(10,2)
);
```

**Crear tabla con todas las constraints:**
```sql
CREATE TABLE empleados (
    -- AUTO_INCREMENT: genera valores automáticos secuenciales
    id INT PRIMARY KEY AUTO_INCREMENT,
    
    -- NOT NULL: campo obligatorio
    nombre VARCHAR(100) NOT NULL,
    apellido VARCHAR(100) NOT NULL,
    
    -- UNIQUE: no permite duplicados
    email VARCHAR(150) UNIQUE,
    dni CHAR(9) UNIQUE,
    
    -- DEFAULT: valor por defecto si no se especifica
    fecha_contratacion DATE DEFAULT (CURRENT_DATE),
    activo BOOLEAN DEFAULT TRUE,
    
    -- CHECK: valida que cumpla una condición
    salario DECIMAL(10,2) CHECK (salario >= 1000 AND salario <= 100000),
    edad INT CHECK (edad >= 18 AND edad <= 70),
    
    -- FOREIGN KEY: relación con otra tabla
    id_departamento INT,
    FOREIGN KEY (id_departamento) REFERENCES departamentos(id)
        ON DELETE CASCADE      -- Si elimino depto, elimino empleados
        ON UPDATE CASCADE,     -- Si actualizo id depto, actualiza aquí
    
    -- Índice para búsquedas rápidas
    INDEX idx_nombre (nombre),
    INDEX idx_apellido (apellido)
);
```

**Clonar tablas:**
```sql
-- Copiar estructura y datos
CREATE TABLE empleados_backup AS 
SELECT * FROM empleados;

-- Copiar solo estructura (sin datos)
CREATE TABLE empleados_vacia LIKE empleados;

-- Copiar datos filtrados
CREATE TABLE empleados_it AS 
SELECT * FROM empleados 
WHERE id_departamento = 2;

-- Copiar solo algunas columnas
CREATE TABLE empleados_mini AS 
SELECT id, nombre, salario 
FROM empleados;
```

---

### 2.3 Modificar Tablas (ALTER TABLE)

**Añadir columnas:**
```sql
-- Añadir columna al final
ALTER TABLE empleados 
ADD COLUMN telefono VARCHAR(15);

-- Añadir con posición específica
ALTER TABLE empleados 
ADD COLUMN segundo_nombre VARCHAR(50) AFTER nombre;

-- Añadir al inicio
ALTER TABLE empleados 
ADD COLUMN codigo_empleado VARCHAR(10) FIRST;

-- Añadir múltiples columnas
ALTER TABLE empleados 
ADD COLUMN direccion VARCHAR(200),
ADD COLUMN ciudad VARCHAR(100);
```

**Modificar columnas:**
```sql
-- Cambiar tipo de dato
ALTER TABLE empleados 
MODIFY COLUMN telefono VARCHAR(20);

-- Cambiar nombre y tipo
ALTER TABLE empleados 
CHANGE COLUMN telefono telefono_movil VARCHAR(20);

-- Añadir restricción
ALTER TABLE empleados 
MODIFY COLUMN email VARCHAR(150) NOT NULL UNIQUE;
```

**Eliminar columnas:**
```sql
-- Eliminar una columna
ALTER TABLE empleados 
DROP COLUMN segundo_nombre;

-- Eliminar múltiples columnas
ALTER TABLE empleados 
DROP COLUMN direccion,
DROP COLUMN ciudad;
```

**Gestionar claves:**
```sql
-- Añadir clave primaria
ALTER TABLE empleados 
ADD PRIMARY KEY (id);

-- Eliminar clave primaria
ALTER TABLE empleados 
DROP PRIMARY KEY;

-- Añadir clave foránea
ALTER TABLE empleados 
ADD CONSTRAINT fk_departamento 
FOREIGN KEY (id_departamento) 
REFERENCES departamentos(id);

-- Eliminar clave foránea
ALTER TABLE empleados 
DROP FOREIGN KEY fk_departamento;
```

**Renombrar tabla:**
```sql
RENAME TABLE empleados TO trabajadores;

-- O con ALTER
ALTER TABLE empleados RENAME TO trabajadores;
```

---

### 2.4 Índices

**¿Para qué sirven los índices?**
Los índices aceleran las búsquedas, similar al índice de un libro. Sin índice, MySQL debe leer toda la tabla.

**Tipos de índices:**

```sql
-- Índice simple (acelera búsquedas por ese campo)
CREATE INDEX idx_nombre ON empleados(nombre);

-- Índice compuesto (para búsquedas por múltiples campos)
CREATE INDEX idx_nombre_apellido ON empleados(nombre, apellido);

-- Índice único (no permite duplicados)
CREATE UNIQUE INDEX idx_email ON empleados(email);

-- Índice de texto completo (para búsquedas FULLTEXT)
CREATE FULLTEXT INDEX idx_descripcion ON productos(descripcion);
```

**Ver índices:**
```sql
SHOW INDEX FROM empleados;
```

**Eliminar índices:**
```sql
DROP INDEX idx_nombre ON empleados;
```

**⚠️ Cuándo usar índices:**
- ✅ Columnas en WHERE frecuentemente
- ✅ Columnas en JOIN
- ✅ Columnas en ORDER BY
- ❌ Tablas pequeñas (< 1000 registros)
- ❌ Columnas que cambian mucho (ralentiza INSERT/UPDATE)

---

### 2.5 Vistas

**¿Qué son las vistas?**
Tablas virtuales basadas en consultas. Los datos no se almacenan, se calculan al consultarlas.

**Crear vista simple:**
```sql
-- Vista que muestra solo empleados activos
CREATE VIEW empleados_activos AS
SELECT id, nombre, apellido, salario
FROM empleados
WHERE activo = TRUE;

-- Usar la vista como si fuera una tabla
SELECT * FROM empleados_activos;
```

**Vista con JOIN:**
```sql
-- Vista que combina empleados con departamentos
CREATE VIEW empleados_completo AS
SELECT 
    e.id,
    e.nombre,
    e.apellido,
    e.salario,
    d.nombre_departamento
FROM empleados e
INNER JOIN departamentos d ON e.id_departamento = d.id;
```

**Vista con agregación:**
```sql
-- Vista que muestra resumen por departamento
CREATE VIEW resumen_departamentos AS
SELECT 
    d.nombre_departamento,
    COUNT(e.id) AS total_empleados,
    AVG(e.salario) AS salario_promedio,
    MAX(e.salario) AS salario_maximo
FROM departamentos d
LEFT JOIN empleados e ON d.id = e.id_departamento
GROUP BY d.id, d.nombre_departamento;
```

**Modificar vista:**
```sql
CREATE OR REPLACE VIEW empleados_activos AS
SELECT id, nombre, apellido, email, salario
FROM empleados
WHERE activo = TRUE AND salario > 2000;
```

**Eliminar vista:**
```sql
DROP VIEW empleados_activos;
```

**✅ Ventajas de las vistas:**
- Simplifica consultas complejas
- Seguridad (oculta columnas sensibles)
- Abstracción (cambiar estructura sin afectar consultas)

---

### 2.6 Triggers (Disparadores)

**¿Qué son los triggers?**
Código que se ejecuta automáticamente cuando ocurre un evento (INSERT, UPDATE, DELETE).

**Sintaxis básica:**
```sql
CREATE TRIGGER nombre_trigger
{BEFORE | AFTER} {INSERT | UPDATE | DELETE}
ON tabla
FOR EACH ROW
BEGIN
    -- código SQL
END;
```

**Trigger BEFORE INSERT (validar datos antes de insertar):**
```sql
DELIMITER //

CREATE TRIGGER validar_salario_antes_insert
BEFORE INSERT ON empleados
FOR EACH ROW
BEGIN
    -- NEW representa el nuevo registro que se va a insertar
    -- Si el salario es menor a 1000, lo ajusta automáticamente
    IF NEW.salario < 1000 THEN
        SET NEW.salario = 1000;
    END IF;
    
    -- Convertir email a minúsculas automáticamente
    SET NEW.email = LOWER(NEW.email);
END//

DELIMITER ;
```

**Trigger AFTER INSERT (auditoría):**
```sql
-- Primero crear tabla de auditoría
CREATE TABLE auditoria_empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    accion VARCHAR(50),
    id_empleado INT,
    nombre_empleado VARCHAR(100),
    usuario VARCHAR(100),
    fecha DATETIME
);

DELIMITER //

CREATE TRIGGER registrar_insert_empleado
AFTER INSERT ON empleados
FOR EACH ROW
BEGIN
    -- NEW contiene los datos del nuevo registro
    INSERT INTO auditoria_empleados (accion, id_empleado, nombre_empleado, usuario, fecha)
    VALUES ('INSERT', NEW.id, NEW.nombre, USER(), NOW());
END//

DELIMITER ;
```

**Trigger BEFORE UPDATE:**
```sql
DELIMITER //

CREATE TRIGGER actualizar_fecha_modificacion
BEFORE UPDATE ON empleados
FOR EACH ROW
BEGIN
    -- OLD = valores antes del UPDATE
    -- NEW = valores después del UPDATE
    SET NEW.fecha_modificacion = NOW();
    
    -- Registrar quién hizo el cambio
    SET NEW.modificado_por = USER();
END//

DELIMITER ;
```

**Trigger AFTER DELETE (auditoría de eliminaciones):**
```sql
DELIMITER //

CREATE TRIGGER registrar_delete_empleado
AFTER DELETE ON empleados
FOR EACH ROW
BEGIN
    -- OLD contiene los datos del registro eliminado
    INSERT INTO auditoria_empleados (accion, id_empleado, nombre_empleado, usuario, fecha)
    VALUES ('DELETE', OLD.id, OLD.nombre, USER(), NOW());
END//

DELIMITER ;
```

**Ver triggers:**
```sql
SHOW TRIGGERS;

-- Ver trigger específico
SHOW CREATE TRIGGER validar_salario_antes_insert;
```

**Eliminar trigger:**
```sql
DROP TRIGGER validar_salario_antes_insert;
```

---

### 2.7 Eventos Programados

**¿Qué son los eventos?**
Tareas programadas que se ejecutan automáticamente en intervalos de tiempo (como cron jobs).

**Habilitar el programador de eventos:**
```sql
SET GLOBAL event_scheduler = ON;

-- Verificar estado
SHOW VARIABLES LIKE 'event_scheduler';
```

**Evento que se ejecuta cada mes:**
```sql
DELIMITER //

CREATE EVENT aumentar_salarios_mensual
ON SCHEDULE EVERY 1 MONTH
STARTS '2024-01-01 00:00:00'
DO
BEGIN
    -- Aumenta 5% el salario de analistas
    UPDATE empleados 
    SET salario = salario * 1.05 
    WHERE puesto = 'Analista';
    
    -- Registra en log
    INSERT INTO log_eventos (evento, fecha)
    VALUES ('Aumento mensual aplicado', NOW());
END//

DELIMITER ;
```

**Evento que se ejecuta una sola vez:**
```sql
CREATE EVENT limpiar_datos_antiguos
ON SCHEDULE AT '2024-12-31 23:59:59'
DO
DELETE FROM logs WHERE fecha < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

**Evento con intervalo personalizado:**
```sql
-- Ejecuta cada 6 horas
CREATE EVENT backup_automatico
ON SCHEDULE EVERY 6 HOUR
DO
CALL procedimiento_backup();

-- Ejecuta cada semana los lunes a las 3 AM
CREATE EVENT resumen_semanal
ON SCHEDULE EVERY 1 WEEK
STARTS '2024-01-08 03:00:00'
DO
CALL generar_reporte_semanal();
```

**Ver eventos:**
```sql
SHOW EVENTS;
```

**Modificar evento:**
```sql
ALTER EVENT aumentar_salarios_mensual
ON SCHEDULE EVERY 2 MONTH;
```

**Desactivar/Activar evento:**
```sql
ALTER EVENT aumentar_salarios_mensual DISABLE;
ALTER EVENT aumentar_salarios_mensual ENABLE;
```

**Eliminar evento:**
```sql
DROP EVENT aumentar_salarios_mensual;
```

---

### 2.8 Stored Procedures (Procedimientos Almacenados)

**¿Qué son?**
Conjuntos de instrucciones SQL guardadas que se pueden reutilizar.

**Procedimiento básico:**
```sql
DELIMITER //

CREATE PROCEDURE obtener_empleados_departamento(IN depto_id INT)
BEGIN
    -- IN = parámetro de entrada
    SELECT * FROM empleados 
    WHERE id_departamento = depto_id;
END//

DELIMITER ;

-- Ejecutar procedimiento
CALL obtener_empleados_departamento(2);
```

**Procedimiento con parámetros de salida:**
```sql
DELIMITER //

CREATE PROCEDURE contar_empleados_salario(
    IN salario_minimo DECIMAL(10,2),
    OUT total INT
)
BEGIN
    -- OUT = parámetro de salida
    SELECT COUNT(*) INTO total
    FROM empleados
    WHERE salario >= salario_minimo;
END//

DELIMITER ;

-- Ejecutar y obtener resultado
CALL contar_empleados_salario(3000, @resultado);
SELECT @resultado;
```

**Procedimiento complejo con lógica:**
```sql
DELIMITER //

CREATE PROCEDURE ajustar_salario_empleado(
    IN emp_id INT,
    IN porcentaje DECIMAL(5,2)
)
BEGIN
    DECLARE salario_actual DECIMAL(10,2);
    DECLARE nuevo_salario DECIMAL(10,2);
    
    -- Obtener salario actual
    SELECT salario INTO salario_actual
    FROM empleados
    WHERE id = emp_id;
    
    -- Calcular nuevo salario
    SET nuevo_salario = salario_actual * (1 + porcentaje/100);
    
    -- Actualizar si el nuevo salario es válido
    IF nuevo_salario >= 1000 AND nuevo_salario <= 100000 THEN
        UPDATE empleados
        SET salario = nuevo_salario
        WHERE id = emp_id;
        
        SELECT CONCAT('Salario actualizado a: ', nuevo_salario) AS mensaje;
    ELSE
        SELECT 'Salario fuera de rango permitido' AS mensaje;
    END IF;
END//

DELIMITER ;

-- Ejecutar: aumentar 10% al empleado 5
CALL ajustar_salario_empleado(5, 10);
```

**Ver procedimientos:**
```sql
SHOW PROCEDURE STATUS WHERE Db = 'empresa';
```

**Eliminar procedimiento:**
```sql
DROP PROCEDURE obtener_empleados_departamento;
```

---

### 2.9 Functions (Funciones)

**¿Diferencia con procedimientos?**
- Functions devuelven UN valor y se usan en consultas SELECT
- Procedures pueden devolver múltiples valores y se ejecutan con CALL

**Función simple:**
```sql
DELIMITER //

CREATE FUNCTION calcular_bonus(salario DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE bonus DECIMAL(10,2);
    
    IF salario < 2000 THEN
        SET bonus = salario * 0.10;
    ELSEIF salario < 5000 THEN
        SET bonus = salario * 0.15;
    ELSE
        SET bonus = salario * 0.20;
    END IF;
    
    RETURN bonus;
END//

DELIMITER ;

-- Usar en consulta
SELECT 
    nombre, 
    salario, 
    calcular_bonus(salario) AS bonus_anual
FROM empleados;
```

**Eliminar función:**
```sql
DROP FUNCTION calcular_bonus;
```

---

## 3. DML (DATA MANIPULATION LANGUAGE)

### 3.1 INSERT (Insertar Datos)

**Insertar un registro:**
```sql
INSERT INTO empleados (nombre, apellido, email, salario, id_departamento)
VALUES ('Juan', 'Pérez', 'juan.perez@email.com', 3500.00, 1);
```

**Insertar múltiples registros:**
```sql
INSERT INTO empleados (nombre, apellido, email, salario, id_departamento)
VALUES 
    ('María', 'García', 'maria.garcia@email.com', 4000.00, 2),
    ('Pedro', 'López', 'pedro.lopez@email.com', 3800.00, 1),
    ('Ana', 'Martínez', 'ana.martinez@email.com', 4200.00, 3);
```

**Insertar sin especificar columnas (todos los campos):**
```sql
-- Debe seguir el orden exacto de las columnas en la tabla
INSERT INTO empleados 
VALUES (NULL, 'Carlos', 'Ruiz', 'carlos.ruiz@email.com', '2024-01-15', 3600.00, 2, TRUE);
```

**Insertar desde otra tabla (subconsulta):**
```sql
-- Copiar empleados con salario alto a tabla de ejecutivos
INSERT INTO ejecutivos (nombre, apellido, salario)
SELECT nombre, apellido, salario
FROM empleados
WHERE salario > 5000;
```

**INSERT con DEFAULT:**
```sql
INSERT INTO empleados (nombre, apellido, email)
VALUES ('Laura', 'Torres', 'laura.torres@email.com');
-- fecha_contratacion y activo toman sus valores DEFAULT
```

**INSERT IGNORE (ignora duplicados):**
```sql
-- Si existe el email, no inserta y no da error
INSERT IGNORE INTO empleados (nombre, apellido, email, salario)
VALUES ('Juan', 'Pérez', 'juan.perez@email.com', 3500.00);
```

**INSERT ... ON DUPLICATE KEY UPDATE:**
```sql
-- Si existe, actualiza; si no existe, inserta
INSERT INTO empleados (id, nombre, salario)
VALUES (1, 'Juan Pérez', 4000.00)
ON DUPLICATE KEY UPDATE salario = 4000.00;
```

**REPLACE (elimina e inserta):**
```sql
-- Si existe, lo elimina y lo vuelve a insertar
REPLACE INTO empleados (id, nombre, apellido, salario)
VALUES (1, 'Juan', 'Pérez', 4500.00);
```

---

### 3.2 SELECT (Consultar Datos)

**SELECT básico:**
```sql
-- Todas las columnas
SELECT * FROM empleados;

-- Columnas específicas
SELECT nombre, apellido, salario FROM empleados;

-- Con alias
SELECT nombre AS 'Nombre', salario AS 'Salario Mensual' FROM empleados;
```

**WHERE (filtrar registros):**
```sql
-- Condición simple
SELECT * FROM empleados WHERE salario > 4000;

-- Múltiples condiciones con AND
SELECT * FROM empleados 
WHERE salario > 3000 AND id_departamento = 2;

-- Múltiples condiciones con OR
SELECT * FROM empleados 
WHERE id_departamento = 1 OR id_departamento = 3;

-- Combinación de AND y OR (usar paréntesis)
SELECT * FROM empleados 
WHERE (id_departamento = 1 OR id_departamento = 2) 
  AND salario > 3500;
```

**Operadores de comparación:**
```sql
-- Igualdad
SELECT * FROM empleados WHERE id_departamento = 2;

-- Diferente
SELECT * FROM empleados WHERE id_departamento != 2;
SELECT * FROM empleados WHERE id_departamento <> 2;

-- Mayor, menor
SELECT * FROM empleados WHERE salario >= 4000;
SELECT * FROM empleados WHERE edad < 30;

-- BETWEEN (rango inclusivo)
SELECT * FROM empleados WHERE salario BETWEEN 3000 AND 5000;
-- Equivale a: salario >= 3000 AND salario <= 5000

-- IN (lista de valores)
SELECT * FROM empleados WHERE id_departamento IN (1, 2, 3);

-- NOT IN
SELECT * FROM empleados WHERE id_departamento NOT IN (1, 2);
```

**LIKE (búsqueda de patrones):**
```sql
-- % = cualquier cantidad de caracteres
-- _ = un solo carácter

-- Nombres que empiezan con 'J'
SELECT * FROM empleados WHERE nombre LIKE 'J%';

-- Nombres que terminan con 'a'
SELECT * FROM empleados WHERE nombre LIKE '%a';

-- Nombres que contienen 'ar'
SELECT * FROM empleados WHERE nombre LIKE '%ar%';

-- Nombres de 4 letras
SELECT * FROM empleados WHERE nombre LIKE '____';

-- Email de Gmail
SELECT * FROM empleados WHERE email LIKE '%@gmail.com';
```

**IS NULL / IS NOT NULL:**
```sql
-- Empleados sin departamento asignado
SELECT * FROM empleados WHERE id_departamento IS NULL;

-- Empleados con email registrado
SELECT * FROM empleados WHERE email IS NOT NULL;
```

**DISTINCT (eliminar duplicados):**
```sql
-- Departamentos únicos
SELECT DISTINCT id_departamento FROM empleados;

-- Combinaciones únicas
SELECT DISTINCT id_departamento, puesto FROM empleados;
```

**ORDER BY (ordenar resultados):**
```sql
-- Orden ascendente (por defecto)
SELECT * FROM empleados ORDER BY salario;
SELECT * FROM empleados ORDER BY salario ASC;

-- Orden descendente
SELECT * FROM empleados ORDER BY salario DESC;

-- Múltiples columnas
SELECT * FROM empleados 
ORDER BY id_departamento ASC, salario DESC;
```

**LIMIT (limitar resultados):**
```sql
-- Primeros 5 registros
SELECT * FROM empleados LIMIT 5;

-- Del registro 10 al 15 (paginación)
SELECT * FROM empleados LIMIT 10, 5;
-- O con OFFSET
SELECT * FROM empleados LIMIT 5 OFFSET 10;

-- Los 3 salarios más altos
SELECT * FROM empleados 
ORDER BY salario DESC 
LIMIT 3;
```

---

### 3.3 Funciones de Agregación

**COUNT (contar registros):**
```sql
-- Total de empleados
SELECT COUNT(*) AS total_empleados FROM empleados;

-- Empleados con email (no cuenta NULL)
SELECT COUNT(email) FROM empleados;

-- Empleados únicos por departamento
SELECT COUNT(DISTINCT id_departamento) FROM empleados;
```

**SUM (suma):**
```sql
-- Suma total de salarios
SELECT SUM(salario) AS masa_salarial FROM empleados;

-- Suma de salarios por condición
SELECT SUM(salario) FROM empleados WHERE id_departamento = 2;
```

**AVG (promedio):**
```sql
-- Salario promedio
SELECT AVG(salario) AS salario_promedio FROM empleados;

-- Promedio redondeado
SELECT ROUND(AVG(salario), 2) AS salario_promedio FROM empleados;
```

**MIN y MAX:**
```sql
-- Salario mínimo y máximo
SELECT 
    MIN(salario) AS salario_minimo,
    MAX(salario) AS salario_maximo
FROM empleados;
```

**GROUP BY (agrupar):**
```sql
-- Total de empleados por departamento
SELECT 
    id_departamento,
    COUNT(*) AS total_empleados
FROM empleados
GROUP BY id_departamento;

-- Salario promedio por departamento
SELECT 
    id_departamento,
    COUNT(*) AS total,
    AVG(salario) AS salario_promedio,
    MIN(salario) AS salario_minimo,
    MAX(salario) AS salario_maximo
FROM empleados
GROUP BY id_departamento;

-- Agrupar por múltiples columnas
SELECT 
    id_departamento,
    puesto,
    COUNT(*) AS total,
    AVG(salario) AS salario_promedio
FROM empleados
GROUP BY id_departamento, puesto;
```

**HAVING (filtrar grupos):**
```sql
-- HAVING filtra después de GROUP BY
-- WHERE filtra antes de GROUP BY

-- Departamentos con más de 5 empleados
SELECT 
    id_departamento,
    COUNT(*) AS total
FROM empleados
GROUP BY id_departamento
HAVING COUNT(*) > 5;

-- Departamentos con salario promedio mayor a 4000
SELECT 
    id_departamento,
    AVG(salario) AS salario_promedio
FROM empleados
GROUP BY id_departamento
HAVING AVG(salario) > 4000;

-- Combinación WHERE y HAVING
SELECT 
    id_departamento,
    COUNT(*) AS total_activos
FROM empleados
WHERE activo = TRUE  -- Filtra antes de agrupar
GROUP BY id_departamento
HAVING COUNT(*) > 3;  -- Filtra después de agrupar
```

---

### 3.4 UPDATE (Actualizar Datos)

**UPDATE básico:**
```sql
-- Actualizar un campo
UPDATE empleados 
SET salario = 4500.00 
WHERE id = 1;

-- Actualizar múltiples campos
UPDATE empleados 
SET 
    salario = 5000.00,
    puesto = 'Senior Developer',
    fecha_modificacion = NOW()
WHERE id = 5;
```

**UPDATE con condiciones:**
```sql
-- Aumentar 10% a todos los desarrolladores
UPDATE empleados 
SET salario = salario * 1.10 
WHERE puesto LIKE '%Developer%';

-- Actualizar basado en rango
UPDATE empleados 
SET nivel = 'Senior' 
WHERE salario > 5000;
```

**UPDATE con JOIN:**
```sql
-- Actualizar empleados según su departamento
UPDATE empleados e
INNER JOIN departamentos d ON e.id_departamento = d.id
SET e.salario = e.salario * 1.15
WHERE d.nombre_departamento = 'Ventas';
```

**UPDATE con subconsulta:**
```sql
-- Actualizar salario al promedio del departamento
UPDATE empleados e1
SET salario = (
    SELECT AVG(salario)
    FROM empleados e2
    WHERE e2.id_departamento = e1.id_departamento
)
WHERE id_departamento = 2;
```

**⚠️ UPDATE sin WHERE actualiza TODOS los registros:**
```sql
-- PELIGRO: Esto cambiará TODOS los salarios
UPDATE empleados SET salario = 3000;
```

---

### 3.5 DELETE (Eliminar Datos)

**DELETE básico:**
```sql
-- Eliminar un registro específico
DELETE FROM empleados WHERE id = 10;

-- Eliminar múltiples registros
DELETE FROM empleados WHERE id_departamento = 5;

-- Eliminar con condiciones
DELETE FROM empleados 
WHERE activo = FALSE AND fecha_contratacion < '2020-01-01';
```

**DELETE con subconsulta:**
```sql
-- Eliminar empleados con salario menor al promedio
DELETE FROM empleados
WHERE salario < (SELECT AVG(salario) FROM empleados);
```

**⚠️ DELETE sin WHERE elimina TODOS los registros:**
```sql
-- PELIGRO: Esto eliminará TODOS los empleados
DELETE FROM empleados;
```

---

### 3.6 TRUNCATE vs DELETE

**Diferencias clave:**

```sql
-- DELETE: Elimina registros uno por uno, se puede revertir con ROLLBACK
DELETE FROM empleados;

-- TRUNCATE: Elimina toda la tabla y la recrea, más rápido, NO se puede revertir
TRUNCATE TABLE empleados;
```

| Característica | DELETE | TRUNCATE |
|----------------|--------|----------|
| Velocidad | Lento | Rápido |
| WHERE | Sí | No |
| ROLLBACK | Sí | No |
| Triggers | Se ejecutan | No se ejecutan |
| AUTO_INCREMENT | No se reinicia | Se reinicia a 1 |

---

## 4. CONSULTAS AVANZADAS

### 4.1 JOINs (Unir Tablas)

**¿Qué son los JOINs?**
Permiten combinar registros de dos o más tablas basándose en una relación entre ellas.

**Tablas de ejemplo:**
```sql
-- Tabla departamentos
id | nombre_departamento
1  | Ventas
2  | IT
3  | RRHH

-- Tabla empleados
id | nombre  | id_departamento
1  | Juan    | 1
2  | María   | 2
3  | Pedro   | NULL
4  | Ana     | 1
```

**INNER JOIN (solo coincidencias):**
```sql
-- Muestra SOLO empleados que tienen departamento asignado
SELECT 
    e.nombre AS empleado,
    d.nombre_departamento AS departamento
FROM empleados e
INNER JOIN departamentos d ON e.id_departamento = d.id;

/*
Resultado:
Juan  | Ventas
María | IT
Ana   | Ventas

Pedro NO aparece (no tiene departamento)
*/
```

**LEFT JOIN (todos los de la izquierda):**
```sql
-- Muestra TODOS los empleados, tengan o no departamento
SELECT 
    e.nombre AS empleado,
    d.nombre_departamento AS departamento
FROM empleados e
LEFT JOIN departamentos d ON e.id_departamento = d.id;

/*
Resultado:
Juan  | Ventas
María | IT
Pedro | NULL    <- Aparece aunque no tiene departamento
Ana   | Ventas
*/
```

**RIGHT JOIN (todos los de la derecha):**
```sql
-- Muestra TODOS los departamentos, tengan o no empleados
SELECT 
    e.nombre AS empleado,
    d.nombre_departamento AS departamento
FROM empleados e
RIGHT JOIN departamentos d ON e.id_departamento = d.id;

/*
Resultado:
Juan  | Ventas
Ana   | Ventas
María | IT
NULL  | RRHH    <- Aparece aunque no tiene empleados
*/
```

**FULL OUTER JOIN (todos de ambos):**
```sql
-- MySQL no soporta FULL OUTER JOIN directamente
-- Se simula con UNION de LEFT y RIGHT JOIN
SELECT e.nombre, d.nombre_departamento
FROM empleados e
LEFT JOIN departamentos d ON e.id_departamento = d.id
UNION
SELECT e.nombre, d.nombre_departamento
FROM empleados e
RIGHT JOIN departamentos d ON e.id_departamento = d.id;
```

**CROSS JOIN (producto cartesiano):**
```sql
-- Combina cada empleado con cada departamento
SELECT e.nombre, d.nombre_departamento
FROM empleados e
CROSS JOIN departamentos d;

/*
Si hay 4 empleados y 3 departamentos = 12 combinaciones
*/
```

**SELF JOIN (tabla consigo misma):**
```sql
-- Encontrar empleados que ganan lo mismo
SELECT 
    e1.nombre AS empleado1,
    e2.nombre AS empleado2,
    e1.salario
FROM empleados e1
INNER JOIN empleados e2 
    ON e1.salario = e2.salario 
    AND e1.id < e2.id;  -- Evita duplicados
```

**JOIN con múltiples tablas:**
```sql
-- Empleados con su departamento y proyecto asignado
SELECT 
    e.nombre AS empleado,
    d.nombre_departamento AS departamento,
    p.nombre_proyecto AS proyecto
FROM empleados e
INNER JOIN departamentos d ON e.id_departamento = d.id
INNER JOIN proyectos p ON e.id_proyecto = p.id;
```

---

### 4.2 Subconsultas

**¿Qué son?**
Consultas dentro de otras consultas.

**Subconsulta en WHERE:**
```sql
-- Empleados con salario mayor al promedio
SELECT nombre, salario
FROM empleados
WHERE salario > (SELECT AVG(salario) FROM empleados);

-- Empleados del mismo departamento que Juan
SELECT nombre, id_departamento
FROM empleados
WHERE id_departamento = (
    SELECT id_departamento 
    FROM empleados 
    WHERE nombre = 'Juan'
);
```

**Subconsulta con IN:**
```sql
-- Empleados de departamentos con más de 5 personas
SELECT nombre, id_departamento
FROM empleados
WHERE id_departamento IN (
    SELECT id_departamento
    FROM empleados
    GROUP BY id_departamento
    HAVING COUNT(*) > 5
);
```

**Subconsulta con EXISTS:**
```sql
-- Departamentos que tienen empleados
SELECT d.nombre_departamento
FROM departamentos d
WHERE EXISTS (
    SELECT 1 
    FROM empleados e 
    WHERE e.id_departamento = d.id
);
```

**Subconsulta en FROM (tabla derivada):**
```sql
-- Salario promedio por departamento, ordenado
SELECT 
    dept_id,
    salario_promedio
FROM (
    SELECT 
        id_departamento AS dept_id,
        AVG(salario) AS salario_promedio
    FROM empleados
    GROUP BY id_departamento
) AS promedios
WHERE salario_promedio > 4000
ORDER BY salario_promedio DESC;
```

**Subconsulta en SELECT:**
```sql
-- Mostrar empleado con su posición en ranking de salarios
SELECT 
    nombre,
    salario,
    (SELECT COUNT(*) 
     FROM empleados e2 
     WHERE e2.salario > e1.salario) + 1 AS ranking_salario
FROM empleados e1
ORDER BY salario DESC;
```

---

### 4.3 UNION, UNION ALL

**UNION (elimina duplicados):**
```sql
-- Combinar empleados actuales y anteriores
SELECT nombre, 'Actual' AS tipo FROM empleados
UNION
SELECT nombre, 'Antiguo' AS tipo FROM empleados_antiguos;
```

**UNION ALL (mantiene duplicados, más rápido):**
```sql
SELECT nombre FROM empleados_ventas
UNION ALL
SELECT nombre FROM empleados_it;
```

**⚠️ Reglas de UNION:**
- Mismo número de columnas
- Tipos de datos compatibles
- Nombres de columnas del primer SELECT

---

### 4.4 CTEs (Common Table Expressions)

**¿Qué son los CTEs?**
Consultas temporales con nombre que mejoran la legibilidad.

**CTE básico:**
```sql
-- Crear CTE y usarlo
WITH empleados_senior AS (
    SELECT * FROM empleados
    WHERE salario > 5000
)
SELECT nombre, salario 
FROM empleados_senior
WHERE id_departamento = 2;
```

**Múltiples CTEs:**
```sql
WITH 
salarios_altos AS (
    SELECT * FROM empleados WHERE salario > 5000
),
ventas AS (
    SELECT * FROM empleados WHERE id_departamento = 1
)
SELECT s.nombre, s.salario
FROM salarios_altos s
INNER JOIN ventas v ON s.id = v.id;
```

**CTE recursivo (jerarquías):**
```sql
-- Estructura organizacional (jefe -> empleados)
WITH RECURSIVE jerarquia AS (
    -- Caso base: jefes principales
    SELECT id, nombre, id_jefe, 1 AS nivel
    FROM empleados
    WHERE id_jefe IS NULL
    
    UNION ALL
    
    -- Caso recursivo: subordinados
    SELECT e.id, e.nombre, e.id_jefe, j.nivel + 1
    FROM empleados e
    INNER JOIN jerarquia j ON e.id_jefe = j.id
)
SELECT * FROM jerarquia ORDER BY nivel, nombre;
```

---

### 4.5 Window Functions (Funciones de Ventana)

**¿Qué son?**
Realizan cálculos sobre un conjunto de filas relacionadas sin agruparlas.

**ROW_NUMBER (numerar filas):**
```sql
-- Asignar número secuencial a cada empleado
SELECT 
    nombre,
    salario,
    ROW_NUMBER() OVER (ORDER BY salario DESC) AS ranking
FROM empleados;

/*
nombre  | salario | ranking
Juan    | 6000    | 1
María   | 5500    | 2
Pedro   | 5000    | 3
*/
```

**RANK y DENSE_RANK:**
```sql
SELECT 
    nombre,
    salario,
    RANK() OVER (ORDER BY salario DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salario DESC) AS dense_rank
FROM empleados;

/*
Si hay empates:
RANK:       1, 2, 2, 4, 5  (salta números)
DENSE_RANK: 1, 2, 2, 3, 4  (no salta)
*/
```

**PARTITION BY (agrupar ventanas):**
```sql
-- Ranking de salarios DENTRO de cada departamento
SELECT 
    nombre,
    id_departamento,
    salario,
    ROW_NUMBER() OVER (
        PARTITION BY id_departamento 
        ORDER BY salario DESC
    ) AS ranking_dept
FROM empleados;

/*
nombre | dept | salario | ranking_dept
Juan   | 1    | 5000    | 1  (primero en dept 1)
Ana    | 1    | 4500    | 2  (segundo en dept 1)
María  | 2    | 6000    | 1  (primero en dept 2)
Pedro  | 2    | 5500    | 2  (segundo en dept 2)
*/
```

**SUM, AVG con OVER:**
```sql
-- Salario acumulado
SELECT 
    nombre,
    salario,
    SUM(salario) OVER (ORDER BY id) AS salario_acumulado
FROM empleados;

-- Promedio móvil
SELECT 
    nombre,
    salario,
    AVG(salario) OVER (
        ORDER BY id 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS promedio_movil
FROM empleados;
```

**LEAD y LAG (valores anteriores/siguientes):**
```sql
SELECT 
    nombre,
    salario,
    LAG(salario) OVER (ORDER BY id) AS salario_anterior,
    LEAD(salario) OVER (ORDER BY id) AS salario_siguiente
FROM empleados;
```

---

### 4.6 CASE WHEN (Condicionales)

**CASE simple:**
```sql
SELECT 
    nombre,
    salario,
    CASE 
        WHEN salario < 2000 THEN 'Bajo'
        WHEN salario BETWEEN 2000 AND 5000 THEN 'Medio'
        ELSE 'Alto'
    END AS nivel_salarial
FROM empleados;
```

**CASE en UPDATE:**
```sql
UPDATE empleados
SET nivel = CASE
    WHEN salario < 3000 THEN 'Junior'
    WHEN salario < 6000 THEN 'Mid'
    ELSE 'Senior'
END;
```

**CASE con agregación:**
```sql
SELECT 
    id_departamento,
    COUNT(CASE WHEN salario > 5000 THEN 1 END) AS salarios_altos,
    COUNT(CASE WHEN salario <= 5000 THEN 1 END) AS salarios_bajos
FROM empleados
GROUP BY id_departamento;
```

---

## 5. FUNCIONES SQL

### 5.1 Funciones de Cadena

```sql
-- CONCAT: Unir cadenas
SELECT CONCAT(nombre, ' ', apellido) AS nombre_completo FROM empleados;

-- CONCAT_WS: Unir con separador
SELECT CONCAT_WS(' - ', nombre, puesto, email) FROM empleados;

-- UPPER / LOWER: Mayúsculas / minúsculas
SELECT UPPER(nombre), LOWER(email) FROM empleados;

-- LENGTH: Longitud de cadena
SELECT nombre, LENGTH(nombre) AS longitud FROM empleados;

-- SUBSTRING: Extraer parte de cadena
SELECT SUBSTRING(email, 1, 5) AS inicio_email FROM empleados;

-- TRIM, LTRIM, RTRIM: Eliminar espacios
SELECT TRIM('  texto  ') AS limpio;

-- REPLACE: Reemplazar texto
SELECT REPLACE(email, '@gmail.com', '@empresa.com') FROM empleados;

-- LEFT / RIGHT: Primeros/últimos caracteres
SELECT LEFT(nombre, 3), RIGHT(apellido, 4) FROM empleados;
```

---

### 5.2 Funciones de Fecha

```sql
-- NOW: Fecha y hora actual
SELECT NOW();

-- CURDATE: Fecha actual
SELECT CURDATE();

-- CURTIME: Hora actual
SELECT CURTIME();

-- DATE_FORMAT: Formatear fecha
SELECT DATE_FORMAT(NOW(), '%d/%m/%Y %H:%i') AS fecha_formateada;

-- DATEDIFF: Diferencia en días
SELECT nombre, DATEDIFF(NOW(), fecha_contratacion) AS dias_trabajados
FROM empleados;

-- DATE_ADD / DATE_SUB: Sumar/restar tiempo
SELECT DATE_ADD(NOW(), INTERVAL 30 DAY) AS en_30_dias;
SELECT DATE_SUB(NOW(), INTERVAL 1 YEAR) AS hace_un_año;

-- YEAR, MONTH, DAY: Extraer partes
SELECT YEAR(fecha_contratacion), MONTH(fecha_contratacion) FROM empleados;

-- TIMESTAMPDIFF: Diferencia en unidades específicas
SELECT 
    nombre,
    TIMESTAMPDIFF(YEAR, fecha_contratacion, NOW()) AS años_trabajados
FROM empleados;
```

---

### 5.3 Funciones Numéricas

```sql
-- ROUND: Redondear
SELECT ROUND(123.456, 2);  -- 123.46

-- CEIL / CEILING: Redondear hacia arriba
SELECT CEIL(123.1);  -- 124

-- FLOOR: Redondear hacia abajo
SELECT FLOOR(123.9);  -- 123

-- ABS: Valor absoluto
SELECT ABS(-50);  -- 50

-- MOD: Módulo (resto de división)
SELECT MOD(10, 3);  -- 1

-- POWER / POW: Potencia
SELECT POWER(2, 3);  -- 8

-- SQRT: Raíz cuadrada
SELECT SQRT(16);  -- 4

-- RAND: Número aleatorio (0 a 1)
SELECT RAND();
```

---

### 5.4 Funciones de Conversión

```sql
-- CAST: Convertir tipo de dato
SELECT CAST('123' AS UNSIGNED) AS numero;
SELECT CAST(salario AS CHAR) AS salario_texto FROM empleados;

-- CONVERT: Similar a CAST
SELECT CONVERT('2024-01-15', DATE);

-- COALESCE: Primer valor no NULL
SELECT COALESCE(telefono, email, 'Sin contacto') FROM empleados;

-- IFNULL / NULLIF
SELECT IFNULL(telefono, 'No disponible') FROM empleados;
```

---

## 6. DCL (DATA CONTROL LANGUAGE)

### 6.1 Gestión de Usuarios

**Crear usuario:**
```sql
-- Usuario local
CREATE USER 'desarrollador'@'localhost' IDENTIFIED BY 'pass123';

-- Usuario remoto desde cualquier host
CREATE USER 'admin'@'%' IDENTIFIED BY 'secure_pass';

-- Usuario desde IP específica
CREATE USER 'app_user'@'192.168.1.100' IDENTIFIED BY 'app_pass';
```

**Modificar usuario:**
```sql
-- Cambiar contraseña
ALTER USER 'desarrollador'@'localhost' IDENTIFIED BY 'nueva_pass';

-- Expirar contraseña (forzar cambio en siguiente login)
ALTER USER 'desarrollador'@'localhost' PASSWORD EXPIRE;

-- Bloquear usuario
ALTER USER 'desarrollador'@'localhost' ACCOUNT LOCK;

-- Desbloquear usuario
ALTER USER 'desarrollador'@'localhost' ACCOUNT UNLOCK;
```

**Eliminar usuario:**
```sql
DROP USER 'desarrollador'@'localhost';
```

**Ver usuarios:**
```sql
SELECT user, host FROM mysql.user;
```

---

### 6.2 GRANT (Otorgar Permisos)

**Permisos globales (todos los databases):**
```sql
-- Todos los privilegios
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost';

-- Solo lectura global
GRANT SELECT ON *.* TO 'readonly'@'localhost';
```

**Permisos por base de datos:**
```sql
-- Todos los privilegios en una base de datos
GRANT ALL PRIVILEGES ON empresa.* TO 'dev'@'localhost';

-- Permisos específicos
GRANT SELECT, INSERT, UPDATE, DELETE ON empresa.* TO 'app_user'@'localhost';
```

**Permisos por tabla:**
```sql
-- Solo SELECT en tabla específica
GRANT SELECT ON empresa.empleados TO 'analista'@'localhost';

-- Múltiples permisos
GRANT SELECT, INSERT, UPDATE ON empresa.empleados TO 'editor'@'localhost';
```

**Permisos por columnas:**
```sql
-- Solo ver columnas específicas
GRANT SELECT (id, nombre, apellido) ON empresa.empleados TO 'limitado'@'localhost';

-- UPDATE solo en columna específica
GRANT UPDATE (salario) ON empresa.empleados TO 'rrhh'@'localhost';
```

**WITH GRANT OPTION (puede otorgar permisos a otros):**
```sql
GRANT ALL PRIVILEGES ON empresa.* TO 'superadmin'@'localhost' 
WITH GRANT OPTION;
```

**Aplicar cambios:**
```sql
FLUSH PRIVILEGES;
```

---

### 6.3 REVOKE (Revocar Permisos)

**Revocar permisos:**
```sql
-- Revocar permiso específico
REVOKE INSERT ON empresa.empleados FROM 'editor'@'localhost';

-- Revocar todos los permisos
REVOKE ALL PRIVILEGES ON empresa.* FROM 'dev'@'localhost';

-- Revocar GRANT OPTION
REVOKE GRANT OPTION ON empresa.* FROM 'superadmin'@'localhost';
```

**Ver permisos de un usuario:**
```sql
SHOW GRANTS FOR 'desarrollador'@'localhost';
```

---

## 7. TCL (TRANSACTION CONTROL LANGUAGE)

### 7.1 ¿Qué son las transacciones?

**Principios ACID:**
- **Atomicidad**: Todo o nada (si falla una parte, falla todo)
- **Consistencia**: Los datos mantienen integridad
- **Aislamiento**: Transacciones no interfieren entre sí
- **Durabilidad**: Los cambios confirmados son permanentes

---

### 7.2 Comandos de Transacción

**Transacción básica:**
```sql
START TRANSACTION;

-- Operaciones
INSERT INTO cuentas (cliente, saldo) VALUES ('Juan', 1000);
UPDATE cuentas SET saldo = saldo - 100 WHERE cliente = 'María';

-- Si todo está bien, confirmar
COMMIT;

-- Si algo falla, revertir
ROLLBACK;
```

**Ejemplo real (transferencia bancaria):**
```sql
START TRANSACTION;

-- Restar dinero de cuenta origen
UPDATE cuentas 
SET saldo = saldo - 500 
WHERE id_cuenta = 1;

-- Verificar que haya saldo suficiente
SET @saldo_origen = (SELECT saldo FROM cuentas WHERE id_cuenta = 1);

IF @saldo_origen < 0 THEN
    ROLLBACK;  -- Revertir si no hay fondos
ELSE
    -- Sumar dinero a cuenta destino
    UPDATE cuentas 
    SET saldo = saldo + 500 
    WHERE id_cuenta = 2;
    
    COMMIT;  -- Confirmar transacción
END IF;
```

---

### 7.3 SAVEPOINT

**Puntos de guardado dentro de una transacción:**
```sql
START TRANSACTION;

INSERT INTO empleados (nombre, salario) VALUES ('Juan', 3000);
SAVEPOINT punto1;

INSERT INTO empleados (nombre, salario) VALUES ('María', 4000);
SAVEPOINT punto2;

-- Si algo falla, volver a punto2
ROLLBACK TO punto2;

-- O volver a punto1
ROLLBACK TO punto1;

-- Confirmar todo
COMMIT;
```

---

### 7.4 Niveles de Aislamiento

```sql
-- READ UNCOMMITTED: Puede leer cambios no confirmados (dirty read)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- READ COMMITTED: Solo lee cambios confirmados
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- REPEATABLE READ: Lecturas consistentes (default en MySQL)
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- SERIALIZABLE: Máximo aislamiento
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

## 8. BACKUP Y RESTAURACIÓN

### 8.1 mysqldump (Backup Lógico)

**Backup completo de una base de datos:**
```bash
# Desde terminal Linux
mysqldump -u root -p empresa > /home/dario/backups/empresa_backup.sql

# Con fecha en el nombre
mysqldump -u root -p empresa > /home/dario/backups/empresa_$(date +%Y%m%d).sql
```

**Backup de tabla específica:**
```bash
mysqldump -u root -p empresa empleados > backup_empleados.sql
```

**Backup de múltiples bases de datos:**
```bash
mysqldump -u root -p --databases empresa ventas > backup_multiple.sql
```

**Backup de todas las bases de datos:**
```bash
mysqldump -u root -p --all-databases > backup_all.sql
```

**Backup solo estructura (sin datos):**
```bash
mysqldump -u root -p --no-data empresa > estructura_empresa.sql
```

**Backup solo datos (sin estructura):**
```bash
mysqldump -u root -p --no-create-info empresa > datos_empresa.sql
```

**Backup con compresión:**
```bash
mysqldump -u root -p empresa | gzip > empresa_backup.sql.gz
```

---

### 8.2 Restauración

**Restaurar base de datos completa:**
```bash
mysql -u root -p empresa < /home/dario/backups/empresa_backup.sql
```

**Restaurar creando la base de datos primero:**
```bash
mysql -u root -p -e "CREATE DATABASE empresa_nueva"
mysql -u root -p empresa_nueva < backup.sql
```

**Restaurar desde archivo comprimido:**
```bash
gunzip < empresa_backup.sql.gz | mysql -u root -p empresa
```

---

### 8.3 Estrategias de Backup

**Backup automático con cron:**
```bash
# Editar crontab
crontab -e

# Backup diario a las 2 AM
0 2 * * * /usr/bin/mysqldump -u root -pPASSWORD empresa > /backups/empresa_$(date +\%Y\%m\%d).sql

# Backup semanal completo los domingos
0 3 * * 0 /usr/bin/mysqldump -u root -pPASSWORD --all-databases > /backups/full_backup_$(date +\%Y\%m\%d).sql
```

---

## 9. BUENAS PRÁCTICAS

### 9.1 Nomenclatura

```sql
-- ✅ RECOMENDADO:
tabla_empleados
id_departamento
nombre_completo
fecha_creacion

-- ❌ EVITAR:
TablasEmpleados  (CamelCase)
IdDepartamento
Nombre Completo  (espacios)
fechaCreacion
```

---

### 9.2 Normalización (Resumen)

**1NF (Primera Forma Normal):**
- Eliminar grupos repetidos
- Cada celda contiene un solo valor

**2NF (Segunda Forma Normal):**
- Cumple 1NF
- Todos los campos dependen de la clave primaria completa

**3NF (Tercera Forma Normal):**
- Cumple 2NF
- Eliminar dependencias transitivas

---

### 9.3 Optimización de Consultas

```sql
-- ✅ Usar índices en columnas de WHERE, JOIN, ORDER BY
CREATE INDEX idx_salario ON empleados(salario);

-- ✅ Seleccionar solo columnas necesarias
SELECT nombre, salario FROM empleados;  -- Bueno
SELECT * FROM empleados;  -- Evitar si no necesitas todo

-- ✅ Usar LIMIT en consultas grandes
SELECT * FROM logs ORDER BY fecha DESC LIMIT 100;

-- ✅ Evitar SELECT en loops
-- ❌ MAL:
FOR EACH empleado DO
    SELECT * FROM departamentos WHERE id = empleado.id_dept;
END FOR;

-- ✅ BIEN:
SELECT e.*, d.* 
FROM empleados e 
LEFT JOIN departamentos d ON e.id_dept = d.id;

-- ✅ Usar EXPLAIN para analizar consultas
EXPLAIN SELECT * FROM empleados WHERE salario > 5000;
```

---

## TABLA DE REFERENCIA RÁPIDA

| Categoría | Comando | Descripción |
|-----------|---------|-------------|
| **DDL** |
| | `CREATE DATABASE` | Crear base de datos |
| | `CREATE TABLE` | Crear tabla |
| | `ALTER TABLE` | Modificar tabla |
| | `DROP TABLE` | Eliminar tabla |
| | `CREATE INDEX` | Crear índice |
| | `CREATE VIEW` | Crear vista |
| | `CREATE TRIGGER` | Crear trigger |
| **DML** |
| | `SELECT` | Consultar datos |
| | `INSERT` | Insertar datos |
| | `UPDATE` | Actualizar datos |
| | `DELETE` | Eliminar datos |
| | `TRUNCATE` | Vaciar tabla |
| **DCL** |
| | `GRANT` | Otorgar permisos |
| | `REVOKE` | Revocar permisos |
| | `CREATE USER` | Crear usuario |
| **TCL** |
| | `START TRANSACTION` | Iniciar transacción |
| | `COMMIT` | Confirmar cambios |
| | `ROLLBACK` | Revertir cambios |
| | `SAVEPOINT` | Punto de guardado |
| **Funciones** |
| | `COUNT(), SUM(), AVG()` | Agregación |
| | `CONCAT(), UPPER()` | Cadenas |
| | `NOW(), DATE_FORMAT()` | Fechas |
| | `ROUND(), CEIL()` | Numéricas |
| **JOINs** |
| | `INNER JOIN` | Solo coincidencias |
| | `LEFT JOIN` | Todos de izquierda |
| | `RIGHT JOIN` | Todos de derecha |
| **Avanzado** |
| | `UNION` | Combinar resultados |
| | `WITH (CTE)` | Consulta temporal |
| | `OVER (Window)` | Funciones de ventana |

---

**Última actualización:** Diciembre 2024
