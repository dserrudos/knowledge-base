# MySQL - Administración en Ubuntu

Manual completo de administración, configuración, optimización y mantenimiento de MySQL en sistemas Linux Ubuntu.

---

## 1. INSTALACIÓN Y CONFIGURACIÓN INICIAL

### 1.1 Instalación de MySQL

**Instalar MySQL Server:**
```bash
# Actualizar repositorios
sudo apt update

# Instalar MySQL
sudo apt install mysql-server

# Verificar instalación
mysql --version
# Salida: mysql  Ver 8.0.xx for Linux on x86_64
```

**Verificar que el servicio esté corriendo:**
```bash
sudo systemctl status mysql
```

---

### 1.2 Configuración Inicial de Seguridad

**Ejecutar script de seguridad:**
```bash
sudo mysql_secure_installation
```

**Este script preguntará:**

1. **¿Configurar plugin de validación de contraseñas?**
   - `Y` → Establece políticas de contraseñas fuertes
   - Niveles: LOW (8 caracteres), MEDIUM (números, mayúsculas, símbolos), STRONG (diccionario)

2. **¿Establecer contraseña para root?**
   - `Y` → Ingresa contraseña segura

3. **¿Eliminar usuarios anónimos?**
   - `Y` → Elimina usuarios sin nombre (más seguro)

4. **¿Deshabilitar login remoto de root?**
   - `Y` → Root solo puede conectarse desde localhost (recomendado)

5. **¿Eliminar base de datos 'test'?**
   - `Y` → Elimina base de datos de prueba

6. **¿Recargar tablas de privilegios?**
   - `Y` → Aplica cambios inmediatamente

---

### 1.3 Acceder a MySQL

**Conectar como root:**
```bash
sudo mysql -u root -p
# Ingresa la contraseña configurada
```

**Conectar desde terminal sin contraseña (solo root del sistema):**
```bash
sudo mysql
```

**Conectar a base de datos específica:**
```bash
mysql -u usuario -p nombre_base_datos
```

**Conectar a servidor remoto:**
```bash
mysql -h 192.168.1.100 -u usuario -p
```

**Salir de MySQL:**
```sql
EXIT;
-- o también:
QUIT;
-- o atajo:
\q
```

---

### 1.4 Ubicaciones Importantes

```bash
# Archivos de configuración
/etc/mysql/                          # Directorio principal
/etc/mysql/my.cnf                    # Archivo de configuración principal
/etc/mysql/mysql.conf.d/mysqld.cnf   # Configuración del servidor
/etc/mysql/conf.d/                   # Configuraciones adicionales

# Datos
/var/lib/mysql/                      # Bases de datos y tablas

# Binarios
/usr/bin/mysql                       # Cliente MySQL
/usr/sbin/mysqld                     # Servidor MySQL
/usr/bin/mysqldump                   # Herramienta de backup

# Logs
/var/log/mysql/error.log             # Log de errores
/var/log/mysql/                      # Directorio de logs

# Socket
/var/run/mysqld/mysqld.sock          # Socket Unix para conexiones locales
```

---

## 2. ARQUITECTURA DE MYSQL

### 2.1 Componentes del Servidor

**Capas de MySQL:**

```
┌─────────────────────────────────────┐
│   Capa de Conexión                  │
│   - Gestión de conexiones           │
│   - Autenticación                   │
│   - Thread pooling                  │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│   Capa de Servicios SQL             │
│   - Parser (análisis SQL)           │
│   - Optimizer (optimizador)         │
│   - Cache                           │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│   Motores de Almacenamiento         │
│   - InnoDB (default)                │
│   - MyISAM                          │
│   - Memory                          │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│   Sistema de Archivos               │
└─────────────────────────────────────┘
```

---

### 2.2 Motores de Almacenamiento

**InnoDB (Recomendado - Motor por defecto):**

```sql
-- Ver motor de una tabla
SHOW CREATE TABLE empleados;

-- Crear tabla con InnoDB
CREATE TABLE productos (
    id INT PRIMARY KEY,
    nombre VARCHAR(100)
) ENGINE=InnoDB;
```

**Características de InnoDB:**
- ✅ **Transacciones ACID** (atomicidad, consistencia, aislamiento, durabilidad)
- ✅ **Claves foráneas** (FOREIGN KEY)
- ✅ **Bloqueo a nivel de fila** (mejor concurrencia)
- ✅ **Recuperación ante fallos** (crash recovery)
- ✅ **Compresión de datos**
- ❌ Usa más recursos (RAM, disco)

**MyISAM (Antiguo, no recomendado):**

```sql
CREATE TABLE logs (
    id INT PRIMARY KEY,
    mensaje TEXT
) ENGINE=MyISAM;
```

**Características de MyISAM:**
- ✅ Rápido para lectura
- ✅ Poco uso de recursos
- ❌ **No soporta transacciones**
- ❌ **No soporta claves foráneas**
- ❌ Bloqueo a nivel de tabla (baja concurrencia)

**Memory (Tablas en RAM):**

```sql
CREATE TABLE cache_temporal (
    id INT PRIMARY KEY,
    datos VARCHAR(100)
) ENGINE=MEMORY;
```

**Características de Memory:**
- ✅ Extremadamente rápido
- ✅ Ideal para datos temporales
- ❌ Se pierden al reiniciar MySQL
- ❌ No soporta tipos TEXT/BLOB

**Ver motores disponibles:**
```sql
SHOW ENGINES;
```

**Cambiar motor de tabla existente:**
```sql
ALTER TABLE empleados ENGINE=InnoDB;
```

---

### 2.3 Estructura de Directorios

```bash
# Ver ubicación de datos
mysql -u root -p -e "SHOW VARIABLES LIKE 'datadir';"

# Contenido de /var/lib/mysql/
ls -lh /var/lib/mysql/

# Estructura típica:
/var/lib/mysql/
├── auto.cnf                 # UUID del servidor
├── binlog.000001            # Binary logs
├── binlog.index             # Índice de binary logs
├── ca-key.pem               # Certificados SSL
├── ca.pem
├── client-cert.pem
├── client-key.pem
├── ib_buffer_pool           # Buffer pool InnoDB
├── ibdata1                  # Tablespace compartido InnoDB
├── ib_logfile0              # Redo logs InnoDB
├── ib_logfile1
├── mysql/                   # Base de datos del sistema
├── performance_schema/      # Esquema de rendimiento
├── sys/                     # Vistas del sistema
├── empresa/                 # Tu base de datos
│   ├── empleados.ibd        # Archivo de tabla InnoDB
│   └── departamentos.ibd
└── mysql.sock               # Socket Unix
```

---

## 3. GESTIÓN DEL SERVICIO

### 3.1 Comandos systemctl

**Iniciar MySQL:**
```bash
sudo systemctl start mysql
```

**Detener MySQL:**
```bash
sudo systemctl stop mysql
```

**Reiniciar MySQL:**
```bash
sudo systemctl restart mysql
```

**Recargar configuración sin reiniciar:**
```bash
sudo systemctl reload mysql
```

**Ver estado del servicio:**
```bash
sudo systemctl status mysql

# Salida típica:
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled)
     Active: active (running) since Mon 2024-12-09 10:30:15 CET
   Main PID: 1234 (mysqld)
     Status: "Server is operational"
      Tasks: 39 (limit: 4915)
```

**Habilitar inicio automático al arrancar el sistema:**
```bash
sudo systemctl enable mysql
```

**Deshabilitar inicio automático:**
```bash
sudo systemctl disable mysql
```

**Ver logs del servicio:**
```bash
sudo journalctl -u mysql
# Ver últimas 50 líneas
sudo journalctl -u mysql -n 50
# Seguir logs en tiempo real
sudo journalctl -u mysql -f
```

---

### 3.2 Verificar Procesos y Puertos

**Ver proceso MySQL:**
```bash
ps aux | grep mysql

# Salida típica:
mysql    1234  0.5 10.2 1234567 123456 ?  Ssl  10:30  0:05 /usr/sbin/mysqld
```

**Ver puerto en uso (default: 3306):**
```bash
sudo netstat -tlnp | grep mysql
# o con ss:
sudo ss -tlnp | grep mysql

# Salida:
tcp  0  0  127.0.0.1:3306  0.0.0.0:*  LISTEN  1234/mysqld
```

**Verificar conexiones activas:**
```bash
sudo netstat -an | grep 3306
```

**Cambiar puerto (si es necesario):**
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Buscar y modificar:
port = 3307

# Reiniciar MySQL
sudo systemctl restart mysql
```

---

## 4. ARCHIVOS DE CONFIGURACIÓN

### 4.1 my.cnf y mysqld.cnf

**Archivo principal de configuración:**
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

**Estructura básica del archivo:**
```ini
[mysqld]
# === CONFIGURACIÓN BÁSICA ===
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
datadir         = /var/lib/mysql

# === CONFIGURACIÓN DE RED ===
bind-address    = 127.0.0.1    # Solo conexiones locales
# bind-address  = 0.0.0.0      # Permitir conexiones remotas

max_connections = 151           # Máximo de conexiones simultáneas

# === MOTORES DE ALMACENAMIENTO ===
default-storage-engine = InnoDB

# === LOGS ===
log_error = /var/log/mysql/error.log

# Slow query log (consultas lentas)
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2             # Registrar queries > 2 segundos

# General query log (todas las consultas)
general_log = 0                 # Desactivado por defecto (impacta rendimiento)
general_log_file = /var/log/mysql/query.log

# Binary log (replicación y recuperación)
log_bin = /var/log/mysql/mysql-bin.log
expire_logs_days = 10           # Mantener logs por 10 días
max_binlog_size = 100M

# === BUFFER POOL (MEMORIA) ===
innodb_buffer_pool_size = 128M  # Ajustar según RAM disponible
                                # Recomendado: 70-80% de RAM en servidor dedicado

# === OTRAS CONFIGURACIONES ===
key_buffer_size = 16M           # Para MyISAM
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 16K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
```

**Aplicar cambios:**
```bash
# Verificar sintaxis del archivo
sudo mysqld --validate-config

# Reiniciar MySQL
sudo systemctl restart mysql
```

---

### 4.2 Variables del Servidor

**Ver todas las variables:**
```sql
SHOW VARIABLES;
```

**Buscar variable específica:**
```sql
-- Variables con 'buffer' en el nombre
SHOW VARIABLES LIKE '%buffer%';

-- Variables con 'storage'
SHOW VARIABLES LIKE '%storage%';

-- Variable específica
SHOW VARIABLES LIKE 'max_connections';
```

**Ver valor de variable:**
```sql
SELECT @@max_connections;
SELECT @@innodb_buffer_pool_size;
```

**Cambiar variable en tiempo de ejecución (temporal):**
```sql
-- Solo dura hasta reiniciar MySQL
SET GLOBAL max_connections = 200;
```

**Cambiar variable permanentemente:**
```bash
# Editar mysqld.cnf
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Agregar/modificar:
max_connections = 200

# Reiniciar
sudo systemctl restart mysql
```

---

### 4.3 Configuración de Memoria

**Calcular uso óptimo de memoria:**

```bash
# Ver RAM total del sistema
free -h

# Para servidor dedicado a MySQL:
# innodb_buffer_pool_size = 70-80% de RAM total

# Ejemplo: Servidor con 8GB RAM
# innodb_buffer_pool_size = 6GB
```

**Configuración para servidor con 8GB RAM:**
```ini
[mysqld]
innodb_buffer_pool_size = 6G        # 75% de 8GB
innodb_log_file_size = 512M
innodb_log_buffer_size = 16M
key_buffer_size = 256M              # Para MyISAM
tmp_table_size = 64M
max_heap_table_size = 64M
```

**Ver uso actual de memoria:**
```sql
-- Tamaño del buffer pool
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- Estadísticas del buffer pool
SHOW STATUS LIKE 'innodb_buffer_pool%';
```

---

### 4.4 Configuración de Red

**Permitir conexiones remotas:**

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Cambiar de:
bind-address = 127.0.0.1

# A (todas las IPs):
bind-address = 0.0.0.0

# O IP específica:
bind-address = 192.168.1.100

# Reiniciar
sudo systemctl restart mysql
```

**Crear usuario para conexión remota:**
```sql
-- Usuario que puede conectarse desde cualquier IP
CREATE USER 'admin'@'%' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%';
FLUSH PRIVILEGES;

-- Usuario desde IP específica
CREATE USER 'app'@'192.168.1.50' IDENTIFIED BY 'app_password';
GRANT SELECT, INSERT, UPDATE ON empresa.* TO 'app'@'192.168.1.50';
FLUSH PRIVILEGES;
```

**Configurar firewall (UFW):**
```bash
# Permitir MySQL desde cualquier IP
sudo ufw allow 3306/tcp

# Permitir solo desde IP específica
sudo ufw allow from 192.168.1.50 to any port 3306

# Verificar reglas
sudo ufw status
```

---

## 5. SISTEMA DE LOGS

### 5.1 Error Log

**¿Qué contiene?**
- Errores críticos del servidor
- Mensajes de inicio y apagado
- Advertencias (warnings)
- Información de diagnóstico

**Ubicación:**
```bash
/var/log/mysql/error.log
```

**Ver últimas líneas del log:**
```bash
sudo tail -f /var/log/mysql/error.log

# Ver últimas 50 líneas
sudo tail -n 50 /var/log/mysql/error.log

# Buscar errores específicos
sudo grep -i "error" /var/log/mysql/error.log
sudo grep -i "crash" /var/log/mysql/error.log
```

**Configurar en mysqld.cnf:**
```ini
[mysqld]
log_error = /var/log/mysql/error.log
```

---

### 5.2 General Query Log

**¿Qué contiene?**
- **TODAS las consultas** ejecutadas por el servidor
- Conexiones de clientes
- Consultas exitosas y fallidas

**⚠️ Advertencia:** Impacta el rendimiento. Usar solo para depuración.

**Habilitar:**
```sql
-- Temporalmente (hasta reiniciar)
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/query.log';

-- Ver estado
SHOW VARIABLES LIKE 'general_log%';
```

**O en mysqld.cnf:**
```ini
[mysqld]
general_log = 1
general_log_file = /var/log/mysql/query.log
```

**Ver log:**
```bash
sudo tail -f /var/log/mysql/query.log
```

**Deshabilitar:**
```sql
SET GLOBAL general_log = 'OFF';
```

---

### 5.3 Slow Query Log

**¿Qué contiene?**
Consultas que tardan más que un tiempo definido (por defecto 10 segundos).

**Habilitar:**
```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- Registrar queries > 2 segundos
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
```

**O en mysqld.cnf:**
```ini
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
log_queries_not_using_indexes = 1  # Registrar queries sin índices
```

**Ver log:**
```bash
sudo tail -f /var/log/mysql/slow.log
```

**Analizar slow query log:**
```bash
# Resumen de consultas lentas
sudo mysqldumpslow /var/log/mysql/slow.log

# Top 10 consultas más lentas
sudo mysqldumpslow -s t -t 10 /var/log/mysql/slow.log

# Top 10 consultas más frecuentes
sudo mysqldumpslow -s c -t 10 /var/log/mysql/slow.log
```

---

### 5.4 Binary Log

**¿Qué contiene?**
- Eventos de modificación de datos (INSERT, UPDATE, DELETE)
- Cambios en estructura (DDL)
- Usado para replicación y recuperación point-in-time

**Habilitar:**
```ini
[mysqld]
log_bin = /var/log/mysql/mysql-bin.log
expire_logs_days = 10            # Auto-limpiar logs antiguos
max_binlog_size = 100M           # Rotar cuando llegue a 100MB
```

**Ver binary logs:**
```sql
SHOW BINARY LOGS;

-- Ver eventos en un binlog
SHOW BINLOG EVENTS IN 'mysql-bin.000001';
```

**Limpiar binary logs antiguos:**
```sql
-- Eliminar logs antes de una fecha
PURGE BINARY LOGS BEFORE '2024-11-01 00:00:00';

-- Eliminar logs hasta un archivo específico
PURGE BINARY LOGS TO 'mysql-bin.000010';

-- Eliminar todos excepto los últimos 3 días
PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 3 DAY);
```

**Deshabilitar binary log (no recomendado):**
```ini
[mysqld]
skip-log-bin
```

---

### 5.5 Rotación de Logs

**Configurar logrotate para MySQL:**
```bash
sudo nano /etc/logrotate.d/mysql-server

# Contenido:
/var/log/mysql/*.log {
    daily                    # Rotar diariamente
    rotate 7                 # Mantener 7 días
    missingok               # No error si falta el log
    notifempty              # No rotar si está vacío
    compress                # Comprimir logs antiguos
    delaycompress           # No comprimir el último
    sharedscripts
    postrotate
        if test -x /usr/bin/mysqladmin && \
           /usr/bin/mysqladmin ping &>/dev/null
        then
           /usr/bin/mysqladmin --local flush-error-log \
              flush-engine-log flush-general-log flush-slow-log
        fi
    endscript
}
```

**Rotar manualmente:**
```bash
sudo logrotate -f /etc/logrotate.d/mysql-server
```

---

## 6. SEGURIDAD

### 6.1 Usuarios y Autenticación

**Plugins de autenticación en MySQL 8.0:**

```sql
-- Ver plugin de un usuario
SELECT user, host, plugin FROM mysql.user;

/*
Plugins comunes:
- mysql_native_password  (antiguo, compatible)
- caching_sha2_password  (nuevo, más seguro - default en MySQL 8.0)
- auth_socket            (autenticación por socket Unix)
*/
```

**Crear usuario con plugin específico:**
```sql
-- Con caching_sha2_password (default MySQL 8.0)
CREATE USER 'user1'@'localhost' IDENTIFIED BY 'password123';

-- Con mysql_native_password (compatibilidad antigua)
CREATE USER 'user2'@'localhost' 
IDENTIFIED WITH mysql_native_password BY 'password123';

-- Con auth_socket (sin contraseña, usa usuario del sistema)
CREATE USER 'admin'@'localhost' IDENTIFIED WITH auth_socket;
```

**Cambiar plugin de usuario existente:**
```sql
ALTER USER 'root'@'localhost' 
IDENTIFIED WITH mysql_native_password BY 'nueva_password';
```

---

### 6.2 Políticas de Contraseñas

**Ver política actual:**
```sql
SHOW VARIABLES LIKE 'validate_password%';
```

**Configurar política:**
```sql
-- Nivel de validación
SET GLOBAL validate_password.policy = MEDIUM;
-- LOW: solo longitud
-- MEDIUM: longitud + números + mayúsculas/minúsculas + símbolos
-- STRONG: MEDIUM + diccionario

-- Longitud mínima
SET GLOBAL validate_password.length = 12;

-- Caracteres especiales mínimos
SET GLOBAL validate_password.special_char_count = 2;
```

**Expiración de contraseñas:**
```sql
-- Contraseña expira en 90 días
ALTER USER 'usuario'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;

-- Contraseña nunca expira
ALTER USER 'usuario'@'localhost' PASSWORD EXPIRE NEVER;

-- Forzar cambio en próximo login
ALTER USER 'usuario'@'localhost' PASSWORD EXPIRE;
```

---

### 6.3 Conexiones Seguras (SSL/TLS)

**Verificar soporte SSL:**
```sql
SHOW VARIABLES LIKE '%ssl%';

-- Ver estado SSL de conexión actual
\s
-- O:
SHOW STATUS LIKE 'Ssl_cipher';
```

**Ubicación de certificados:**
```bash
ls -l /var/lib/mysql/*.pem

# Archivos:
ca.pem          # Certificado CA
server-cert.pem # Certificado del servidor
server-key.pem  # Llave privada del servidor
client-cert.pem # Certificado del cliente
client-key.pem  # Llave privada del cliente
```

**Requerir SSL para usuario:**
```sql
CREATE USER 'secure_user'@'%' 
IDENTIFIED BY 'password' 
REQUIRE SSL;

-- O modificar usuario existente
ALTER USER 'usuario'@'%' REQUIRE SSL;
```

**Conectar con SSL:**
```bash
mysql -u usuario -p \
  --ssl-ca=/var/lib/mysql/ca.pem \
  --ssl-cert=/var/lib/mysql/client-cert.pem \
  --ssl-key=/var/lib/mysql/client-key.pem
```

---

### 6.4 Firewall y Hardening

**Firewall UFW:**
```bash
# Denegar todo por defecto
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Permitir SSH
sudo ufw allow 22/tcp

# Permitir MySQL solo desde red interna
sudo ufw allow from 192.168.1.0/24 to any port 3306

# Activar firewall
sudo ufw enable

# Ver estado
sudo ufw status verbose
```

**Hardening básico:**

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

```ini
[mysqld]
# Solo conexiones locales
bind-address = 127.0.0.1

# Deshabilitar LOCAL INFILE (seguridad)
local_infile = 0

# Límite de conexiones
max_connections = 100
max_connect_errors = 10

# Deshabilitar LOAD DATA LOCAL
secure_file_priv = /var/lib/mysql-files/
```

**Permisos de archivos:**
```bash
# Verificar permisos del datadir
ls -ld /var/lib/mysql
# Debe ser: drwx------ mysql mysql

# Corregir si es necesario
sudo chown -R mysql:mysql /var/lib/mysql
sudo chmod 700 /var/lib/mysql
```

---

### 6.5 Auditoría

**Habilitar auditoría básica con general log:**
```sql
SET GLOBAL general_log = 'ON';
```

**Tabla de auditoría personalizada:**
```sql
CREATE TABLE auditoria (
    id INT PRIMARY KEY AUTO_INCREMENT,
    usuario VARCHAR(100),
    accion VARCHAR(50),
    tabla_afectada VARCHAR(100),
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    ip_origen VARCHAR(45)
);

-- Trigger de auditoría
DELIMITER //
CREATE TRIGGER audit_empleados_insert
AFTER INSERT ON empleados
FOR EACH ROW
BEGIN
    INSERT INTO auditoria (usuario, accion, tabla_afectada)
    VALUES (USER(), 'INSERT', 'empleados');
END//
DELIMITER ;
```

---

## 7. OPTIMIZACIÓN Y PERFORMANCE

### 7.1 EXPLAIN (Análisis de Consultas)

**¿Qué es EXPLAIN?**
Muestra cómo MySQL ejecuta una consulta y ayuda a identificar problemas de rendimiento.

**Usar EXPLAIN:**
```sql
EXPLAIN SELECT * FROM empleados WHERE salario > 5000;
```

**Salida de EXPLAIN:**
```
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table     | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | empleados | ALL  | NULL          | NULL | NULL    | NULL | 1000 | Using where |
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+
```

**Columnas importantes:**

- **type**: Tipo de acceso (de mejor a peor)
  - `const`: Búsqueda por clave primaria/única (ÓPTIMO)
  - `eq_ref`: Búsqueda por índice único en JOIN
  - `ref`: Búsqueda por índice no único
  - `range`: Rango de índice (BETWEEN, >, <)
  - `index`: Escaneo completo de índice
  - `ALL`: Escaneo completo de tabla (MALO)

- **possible_keys**: Índices que MySQL puede usar
- **key**: Índice que MySQL usa realmente
- **rows**: Estimación de filas a examinar (menor = mejor)
- **Extra**: Información adicional

**Extra importantes:**
- `Using index`: Usa solo el índice (muy bueno)
- `Using where`: Filtro WHERE aplicado
- `Using temporary`: Crea tabla temporal (evitar si es posible)
- `Using filesort`: Ordenamiento costoso (agregar índice)

**EXPLAIN con formato extendido:**
```sql
EXPLAIN FORMAT=JSON SELECT * FROM empleados WHERE salario > 5000\G
```

---

### 7.2 Índices y Optimización

**Crear índices estratégicamente:**

```sql
-- Índice simple para búsquedas frecuentes
CREATE INDEX idx_salario ON empleados(salario);

-- Índice compuesto (orden importa)
CREATE INDEX idx_dept_salario ON empleados(id_departamento, salario);

-- Este índice sirve para:
-- WHERE id_departamento = X
-- WHERE id_departamento = X AND salario > Y
-- Pero NO para: WHERE salario > Y (solo)

-- Índice único
CREATE UNIQUE INDEX idx_email ON empleados(email);

-- Índice en columna calculada (MySQL 8.0+)
CREATE INDEX idx_nombre_lower ON empleados((LOWER(nombre)));
```

**Analizar uso de índices:**
```sql
-- Ver índices de una tabla
SHOW INDEX FROM empleados;

-- Estadísticas de uso de índices
SELECT * FROM sys.schema_unused_indexes;

-- Índices duplicados o redundantes
SELECT * FROM sys.schema_redundant_indexes;
```

**Cuándo NO usar índices:**
- ❌ Tablas pequeñas (< 1000 registros)
- ❌ Columnas con pocos valores distintos (ej: género M/F)
- ❌ Columnas que se actualizan mucho
- ❌ Demasiados índices en una tabla (ralentiza INSERT/UPDATE)

---

### 7.3 Claves Primarias y Foráneas

**Importancia de la clave primaria:**
```sql
-- InnoDB organiza datos por clave primaria (clustered index)
-- Mejor usar INT AUTO_INCREMENT que UUID o VARCHAR

-- ✅ RECOMENDADO:
CREATE TABLE productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100)
);

-- ❌ EVITAR (menos eficiente):
CREATE TABLE productos (
    uuid CHAR(36) PRIMARY KEY,
    nombre VARCHAR(100)
);
```

**Claves foráneas mejoran integridad:**
```sql
CREATE TABLE pedidos (
